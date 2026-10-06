# Monitoring stack: OpenTelemetry, Alloy, Grafana

Alloy menerima telemetri OpenTelemetry dari aplikasi melalui OTLP gRPC atau HTTP. Pipeline mengirim logs ke Loki, metrics ke Prometheus Remote Write, dan traces ke Tempo.

```text
Application --OTLP--> Alloy --logs----> Loki
                            --metrics-> Prometheus
                            --traces--> Tempo
                                      Grafana
```

## Jalankan stack pusat

```bash
docker compose -f infrastructure.yml up -d
```

Endpoint OTLP yang dapat dipakai aplikasi pada mesin stack pusat:

- gRPC: `http://localhost:4317`
- HTTP: `http://localhost:4318`

Untuk aplikasi di container Docker yang terhubung ke network `infra_network`, gunakan `http://alloy:4317` (gRPC) atau `http://alloy:4318` (HTTP). HTTP SDK biasanya memakai base endpoint `http://alloy:4318` dan mengirim ke path `/v1/logs`, `/v1/metrics`, atau `/v1/traces`.

Contoh environment variable aplikasi:

```env
OTEL_SERVICE_NAME=my-app
OTEL_EXPORTER_OTLP_ENDPOINT=http://alloy:4317
OTEL_EXPORTER_OTLP_PROTOCOL=grpc
```

Pilih `http/protobuf` sebagai protocol jika aplikasi memakai port 4318.

Grafana: `http://localhost:3000` (login awal `admin` / `admin123`). Tambahkan data sources Loki `http://loki:3100`, Prometheus `http://prometheus:9090`, dan Tempo `http://tempo:3200`.

## Jalankan Alloy di server aplikasi terpisah

Salin `docker-compose.agent.yml` dan folder `alloy/` ke tiap server aplikasi. Set endpoint Alloy pusat di environment, lalu jalankan agent:

```env
OTEL_GATEWAY_ENDPOINT=MASTER_IP:4317
```

```bash
docker compose -f docker-compose.agent.yml up -d
```

Pastikan firewall server pusat mengizinkan koneksi dari agent ke port 4317 pada Alloy. Untuk aplikasi di server agent yang sama, kirim OTLP ke `http://localhost:4317` (gRPC) atau `http://localhost:4318` (HTTP). Ganti `MASTER_IP` dengan alamat/DNS server pusat yang dapat dijangkau.

## Query data

Loki:

```text
http://MASTER_IP:3100/loki/api/v1/query?query={service_name="my-app"}
```

Prometheus: buka `http://MASTER_IP:9090` dan cari metric yang dikirim aplikasi.

Tempo: buka Grafana Explore, pilih Tempo, lalu cari trace berdasarkan `service.name`.

## Retensi traces

Tempo memakai local storage dengan retensi 24 jam untuk instalasi awal. Sesuaikan `tempo/tempo.yaml` untuk kebutuhan kapasitas dan retensi server.
