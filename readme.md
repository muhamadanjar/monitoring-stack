
## DIAGRAM ARSITEKTUR MONITORING (Grafana + Loki + Prometheus)
```
                 ┌──────────────────────────┐
                 │        SERVER PUSAT       │
                 │ (Grafana + Prometheus +   │
                 │          Loki)            │
                 └─────────────┬────────────┘
                               │
                     Metrics   │    Logs
                               │
       ┌───────────────────────┼────────────────────────┐
       │                       │                        │
       ▼                       ▼                        ▼
┌────────────┐         ┌────────────┐           ┌────────────┐
│ Server A   │         │ Server B   │           │ Server C   │
│ promtail   │         │ promtail   │           │ promtail   │
│ node-exp   │         │ node-exp   │           │ node-exp   │
│ cadvisor   │         │ cadvisor   │           │ cadvisor   │
└────────────┘         └────────────┘           └────────────┘

```

- promtail  → mengirim log ke Loki pusat
- node-exporter → mengirim metrics CPU/RAM/Disk ke Prometheus pusat
- cadvisor → mengirim metrics container ke Prometheus pusat


## QUERY API

### A) Query ke Loki (get logs)

Ambil log dari job `docker`:

```bash
GET http://MASTER_IP:3100/loki/api/v1/query?query={job="docker"}
```

Ambil log container tertentu:

```bash
GET http://MASTER_IP:3100/loki/api/v1/query?query={container="nginx"}

```

Ambil log real-time (stream):

```
GET http://MASTER_IP:3100/loki/api/v1/tail?query={job="docker"}
```
### B) Query ke Prometheus (metrics)

CPU Usage server:
```bash
http://MASTER_IP:9090/api/v1/query?query=rate(node_cpu_seconds_total{mode!="idle"}[5m])

```
RAM usage:
```bash
http://MASTER_IP:9090/api/v1/query?query=node_memory_MemAvailable_bytes

```
Disk usage:
```bash
http://MASTER_IP:9090/api/v1/query?query=node_filesystem_avail_bytes

```
Usage CPU container:

```bash
http://MASTER_IP:9090/api/v1/query?query=rate(container_cpu_usage_seconds_total[5m])

```
RAM container:

```bash
http://MASTER_IP:9090/api/v1/query?query=container_memory_usage_bytes
```