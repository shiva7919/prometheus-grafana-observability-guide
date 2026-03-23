# prometheus-grafana-observability-guide
# 📊 Prometheus + Grafana — Complete Study Notes
### Your End-to-End Observability Reference for Learning, Interviews & Hands-On Practice

---

## 📌 Table of Contents

1. [Module 1 — Introduction & Fundamentals](#module-1)
2. [Module 2 — Prometheus Metrics & Exporters](#module-2)
3. [Module 3 — Prometheus Architecture & PromQL](#module-3)
4. [Module 4 — Alerting with Alertmanager](#module-4)
5. [Module 5 — Grafana Dashboards](#module-5)
6. [Module 6 — Advanced Prometheus Concepts](#module-6)
7. [Module 7 — Observability & Troubleshooting](#module-7)
8. [Module 8 — Optional Integrations (Loki, Tempo, Cloud)](#module-8)
9. [Module 9 — Projects & Hands-On Labs](#module-9)
10. [Quick Reference Cheat Sheet](#cheat-sheet)

---

# Module 1 — Introduction & Fundamentals {#module-1}

## 1.1 What is Monitoring vs Observability?

| Concept | Definition |
|---|---|
| **Monitoring** | Watching pre-defined metrics and alerting when thresholds breach |
| **Observability** | Understanding the *internal state* of a system from its *external outputs* (metrics, logs, traces) |
| **Telemetry** | Automated collection and transmission of data about a system's behavior |

> **Key Insight:** Monitoring answers *"Is something wrong?"* Observability answers *"Why is something wrong?"*

### The Three Pillars of Observability

```
┌─────────────────────────────────────────────────────────┐
│                  OBSERVABILITY STACK                     │
│                                                         │
│  ┌───────────┐   ┌───────────┐   ┌───────────────────┐  │
│  │  METRICS  │   │   LOGS    │   │      TRACES       │  │
│  │           │   │           │   │                   │  │
│  │ Numerical │   │ Timestamped│  │ Request journey   │  │
│  │ time-     │   │ text      │   │ across services   │  │
│  │ series    │   │ events    │   │ (spans)           │  │
│  │           │   │           │   │                   │  │
│  │Prometheus │   │  Loki     │   │    Tempo          │  │
│  └───────────┘   └───────────┘   └───────────────────┘  │
│                       ↓                                  │
│                 ┌───────────┐                            │
│                 │  GRAFANA  │  ← Unified Visualization   │
│                 └───────────┘                            │
└─────────────────────────────────────────────────────────┘
```

---

## 1.2 What is Prometheus?

**Prometheus** is an open-source systems monitoring and alerting toolkit originally built at SoundCloud (2012), donated to CNCF in 2016.

### Core Characteristics
- **Pull-based model**: Prometheus *scrapes* metrics from targets (vs push-based like StatsD)
- **Time-series database (TSDB)**: Stores all data as timestamped numeric values
- **Dimensional data model**: Uses key-value *labels* to identify metrics
- **PromQL**: Built-in query language for metrics analysis
- **Self-contained**: No external dependencies for single-node operation

### Key Components
| Component | Description |
|---|---|
| **Prometheus Server** | Core service: scrapes, stores, and evaluates rules |
| **Exporters** | Translate system/app metrics into Prometheus format |
| **Alertmanager** | Handles alerts: routing, deduplication, notifications |
| **Pushgateway** | Accepts pushed metrics from short-lived jobs |
| **Client Libraries** | SDKs to instrument your app (Go, Python, Java, etc.) |

---

## 1.3 What is Grafana?

**Grafana** is an open-source analytics and visualization platform. It connects to multiple data sources and renders interactive dashboards.

### Core Characteristics
- **Data source agnostic**: Works with Prometheus, Loki, Elasticsearch, InfluxDB, CloudWatch, MySQL, etc.
- **Rich visualizations**: Time-series graphs, bar charts, pie charts, heatmaps, tables, stat panels
- **Alerting**: Built-in alert rules and notification channels
- **Templating**: Dynamic dashboards with variables and filters
- **Unified UI**: Single pane of glass for metrics, logs, and traces

---

## 1.4 Prometheus vs Other Monitoring Tools

| Feature | Prometheus | Datadog | Nagios | InfluxDB + Telegraf |
|---|---|---|---|---|
| **Model** | Pull | Push/Pull | Active checks | Push |
| **Storage** | Local TSDB | Cloud SaaS | None | Local/Cloud |
| **Query Lang** | PromQL | Custom | None | Flux/InfluxQL |
| **Cost** | Free/OSS | Paid SaaS | Free/OSS | Free/Paid |
| **Scalability** | Federation | Built-in | Plugins | Clusters |
| **Best For** | K8s, cloud-native | Enterprise SaaS | Legacy infra | IoT, high write |
| **Visualization** | Basic (Grafana needed) | Built-in | Basic | Grafana/Chronograf |
| **Tracing** | No (use Tempo) | Yes | No | No |

> **Interview Tip:** Prometheus is the de-facto standard for Kubernetes monitoring. Every CNCF project exposes Prometheus metrics.

---

## 1.5 Use Cases & Benefits

**Prometheus + Grafana excels at:**
- Container and Kubernetes monitoring
- Microservices performance tracking
- SRE/SLO monitoring (error budgets, latency percentiles)
- Infrastructure capacity planning
- Custom application instrumentation

---

## 1.6 Hands-On: Install Prometheus & Grafana Locally

### Prerequisites
- Linux/macOS or Docker
- Ports 9090 (Prometheus), 3000 (Grafana), 9100 (Node Exporter)

### Option A: Docker Compose (Recommended for Quick Start)

Create `docker-compose.yml`:

```yaml
version: "3.8"

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.enable-lifecycle'
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    restart: unless-stopped

volumes:
  prometheus_data:
  grafana_data:
```

Create `prometheus.yml`:

```yaml
global:
  scrape_interval: 15s      # How often to scrape targets
  evaluation_interval: 15s  # How often to evaluate rules

scrape_configs:
  - job_name: 'prometheus'   # Scrape itself
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
```

Start everything:

```bash
docker-compose up -d

# Verify
docker-compose ps
curl http://localhost:9090/metrics   # Prometheus self-metrics
curl http://localhost:9100/metrics   # Node Exporter metrics
```

### Option B: Binary Install (Linux)

```bash
# Download Prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.51.0/prometheus-2.51.0.linux-amd64.tar.gz
tar xvf prometheus-*.tar.gz
cd prometheus-*

# Run
./prometheus --config.file=prometheus.yml

# Download and run Grafana
sudo apt-get install -y adduser libfontconfig1 musl
wget https://dl.grafana.com/oss/release/grafana_10.4.0_amd64.deb
sudo dpkg -i grafana_10.4.0_amd64.deb
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

### Exploring the UIs

**Prometheus UI (http://localhost:9090)**
- `/graph` — PromQL query interface
- `/targets` — See all scrape targets and their status
- `/config` — View loaded configuration
- `/alerts` — See firing alerts
- `/rules` — View loaded recording/alert rules

**Grafana UI (http://localhost:3000)**
- Default login: `admin / admin`
- Add Prometheus data source: Configuration → Data Sources → Add → Prometheus → URL: `http://prometheus:9090`

---

## 1.7 Quick Q&A — Module 1

**Q: What is the difference between monitoring and observability?**
A: Monitoring tracks known failure modes via predefined metrics and alerts. Observability allows you to investigate *unknown unknowns* using the full telemetry data (metrics, logs, traces).

**Q: Why does Prometheus use a pull model?**
A: Pull gives the monitoring system control — it decides when/what to scrape, simplifies firewall rules (scraper opens connections outbound), and makes it easy to detect dead targets (no scrape = target down).

**Q: What port does Prometheus run on by default?**
A: **9090**. Grafana runs on **3000**. Node Exporter on **9100**.

**Q: What is the Prometheus data format?**
A: Each metric line looks like: `metric_name{label1="value1", label2="value2"} numeric_value [timestamp]`

---

# Module 2 — Prometheus Metrics & Exporters {#module-2}

## 2.1 Prometheus Data Model

Every metric in Prometheus is a **time-series**, identified by:
1. A **metric name** (e.g., `http_requests_total`)
2. A set of **labels** (key-value pairs) (e.g., `{method="GET", status="200"}`)
3. A **float64 value** with a **Unix timestamp**

```
http_requests_total{job="api", method="GET", status="200"} 12345 1706000000000
│                   │                                      │     │
│                   └── Labels (dimensions)                │     └── Timestamp (ms)
└── Metric name                                            └── Value
```

### Label Best Practices
- Labels create **cardinality** — every unique label combination = a new time-series
- High cardinality labels (user IDs, UUIDs) cause **cardinality explosion** — avoid them
- Good labels: `method`, `status`, `region`, `service`, `env`
- Bad labels: `user_id`, `session_id`, `request_id`

---

## 2.2 Metric Types

### Counter

```
┌─────────────────────────────────────────────────────────┐
│  COUNTER — Monotonically increasing value                │
│                                                         │
│  Value  ↑                                               │
│         │                  ████                         │
│         │           ███████                             │
│         │     ██████                                    │
│         │ ████                                          │
│         └──────────────────────────────→ Time           │
│                                                         │
│  Examples:                                              │
│  - http_requests_total                                  │
│  - errors_total                                         │
│  - bytes_sent_total                                     │
│                                                         │
│  NEVER decreases (except reset to 0 on restart)         │
└─────────────────────────────────────────────────────────┘
```

```
# HELP http_requests_total Total HTTP requests
# TYPE http_requests_total counter
http_requests_total{method="GET",status="200"} 1234
http_requests_total{method="POST",status="500"} 56
```

**Use with `rate()` to get per-second rate:**
```promql
rate(http_requests_total[5m])   # avg rate over last 5 min
```

---

### Gauge

```
┌─────────────────────────────────────────────────────────┐
│  GAUGE — Can go up or down                               │
│                                                         │
│  Value  ↑                                               │
│         │    ▲     ▲                                    │
│         │   / \   / \   ▲                               │
│         │  /   \ /   \ / \                              │
│         │ /     V     V   \                             │
│         └──────────────────────────────→ Time           │
│                                                         │
│  Examples:                                              │
│  - node_memory_MemAvailable_bytes                       │
│  - container_cpu_usage_seconds_total                    │
│  - active_connections                                   │
│  - temperature_celsius                                  │
└─────────────────────────────────────────────────────────┘
```

```
# HELP node_memory_MemAvailable_bytes Available memory
# TYPE node_memory_MemAvailable_bytes gauge
node_memory_MemAvailable_bytes 2.5e+09
```

---

### Histogram

```
┌─────────────────────────────────────────────────────────┐
│  HISTOGRAM — Samples observations into configurable      │
│              buckets; measures distributions             │
│                                                         │
│  Produces 3 time series:                                │
│  _bucket{le="0.1"}  (count of obs ≤ 0.1s)              │
│  _bucket{le="0.5"}  (count of obs ≤ 0.5s)              │
│  _bucket{le="1.0"}  (count of obs ≤ 1.0s)              │
│  _bucket{le="+Inf"} (total count)                        │
│  _sum               (sum of all values)                  │
│  _count             (total number of obs)                │
│                                                         │
│  Examples:                                              │
│  - http_request_duration_seconds                        │
│  - request_size_bytes                                   │
│                                                         │
│  USE: percentiles, averages, distributions              │
└─────────────────────────────────────────────────────────┘
```

```
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{le="0.05"} 24054
http_request_duration_seconds_bucket{le="0.1"}  33444
http_request_duration_seconds_bucket{le="0.5"}  100392
http_request_duration_seconds_bucket{le="+Inf"} 144320
http_request_duration_seconds_sum   53423
http_request_duration_seconds_count 144320
```

**Compute 95th percentile latency:**
```promql
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

---

### Summary

```
┌─────────────────────────────────────────────────────────┐
│  SUMMARY — Pre-calculated client-side quantiles          │
│                                                         │
│  Similar to Histogram but:                              │
│  - Quantiles computed on CLIENT (not Prometheus server) │
│  - Cannot aggregate across instances (use Histogram!)   │
│  - Lower server load, higher client CPU                 │
│                                                         │
│  Produces:                                              │
│  metric{quantile="0.5"}   ← median                      │
│  metric{quantile="0.9"}   ← 90th percentile             │
│  metric{quantile="0.99"}  ← 99th percentile             │
│  metric_sum                                             │
│  metric_count                                           │
│                                                         │
│  USE: When you need accurate per-instance quantiles     │
│       and don't need cross-instance aggregation         │
└─────────────────────────────────────────────────────────┘
```

### Histogram vs Summary Quick Comparison

| | Histogram | Summary |
|---|---|---|
| Quantile calc | Server-side (PromQL) | Client-side |
| Aggregatable | ✅ Yes | ❌ No |
| Accuracy | Approximate | Exact |
| Configuration | Buckets defined at collection | Quantiles defined at collection |
| **Preferred?** | ✅ Usually preferred | Only for single-instance accuracy |

---

## 2.3 Prometheus Exporters

An **exporter** is a program that fetches metrics from a system and exposes them at `/metrics` in Prometheus format.

```
┌─────────────────────────────────────────────────────────┐
│                    EXPORTER PATTERN                      │
│                                                         │
│  ┌──────────────┐    ┌────────────────┐   ┌──────────┐  │
│  │  Target      │    │    Exporter    │   │Prometheus│  │
│  │  System      │───▶│                │◀──│  Server  │  │
│  │  (e.g. MySQL)│    │ /metrics       │   │(scrapes) │  │
│  └──────────────┘    └────────────────┘   └──────────┘  │
│   Native metrics      Translated to        Stores &      │
│   (not Prometheus)    Prometheus format     Queries       │
└─────────────────────────────────────────────────────────┘
```

### Node Exporter — System Metrics

**Install and Run:**

```bash
# Download
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xvf node_exporter-*.tar.gz
cd node_exporter-*/

# Run
./node_exporter

# As a systemd service
sudo useradd -rs /bin/false node_exporter
sudo cp node_exporter /usr/local/bin/
```

Create `/etc/systemd/system/node_exporter.service`:
```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
ExecStart=/usr/local/bin/node_exporter
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

**Key Node Exporter Metrics:**
```
node_cpu_seconds_total{cpu="0",mode="idle"}         # CPU usage by mode
node_memory_MemAvailable_bytes                       # Free RAM
node_disk_io_time_seconds_total                      # Disk I/O
node_network_receive_bytes_total                     # Network RX
node_filesystem_avail_bytes                         # Disk space
node_load1 / node_load5 / node_load15               # Load averages
node_uname_info                                     # OS info
```

---

### cAdvisor — Container Metrics

```bash
docker run -d \
  --name=cadvisor \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  -p 8080:8080 \
  gcr.io/cadvisor/cadvisor:latest
```

**Key cAdvisor Metrics:**
```
container_cpu_usage_seconds_total{name="my_container"}
container_memory_usage_bytes{name="my_container"}
container_network_receive_bytes_total
container_fs_reads_bytes_total
```

Add to `prometheus.yml`:
```yaml
- job_name: 'cadvisor'
  static_configs:
    - targets: ['cadvisor:8080']
```

---

### Database Exporters

**MySQL Exporter:**
```bash
docker run -d \
  -p 9104:9104 \
  -e DATA_SOURCE_NAME="user:password@(mysql:3306)/" \
  prom/mysqld-exporter
```

```yaml
# prometheus.yml
- job_name: 'mysql'
  static_configs:
    - targets: ['mysqld-exporter:9104']
```

**Key MySQL Metrics:**
```
mysql_global_status_queries              # Total queries
mysql_global_status_connections         # Active connections
mysql_global_status_slow_queries        # Slow query count
mysql_global_status_uptime              # Uptime seconds
```

**PostgreSQL Exporter:**
```bash
docker run -d -p 9187:9187 \
  -e DATA_SOURCE_NAME="postgresql://user:pass@postgres:5432/dbname?sslmode=disable" \
  wrouesnel/postgres_exporter
```

---

### Custom Application Metrics (Python Example)

```bash
pip install prometheus-client
```

```python
# app.py
from prometheus_client import Counter, Gauge, Histogram, start_http_server
import time
import random

# Define metrics
REQUEST_COUNT = Counter(
    'app_requests_total',
    'Total number of requests',
    ['method', 'endpoint', 'status']
)

ACTIVE_USERS = Gauge(
    'app_active_users',
    'Number of currently active users'
)

REQUEST_LATENCY = Histogram(
    'app_request_duration_seconds',
    'Request latency in seconds',
    ['endpoint'],
    buckets=[0.01, 0.05, 0.1, 0.5, 1.0, 5.0]
)

def process_request(endpoint):
    """Simulate processing a request"""
    start = time.time()
    
    # Simulate work
    time.sleep(random.uniform(0.01, 0.5))
    status = random.choice(["200", "200", "200", "500"])
    
    # Record metrics
    REQUEST_COUNT.labels(method='GET', endpoint=endpoint, status=status).inc()
    REQUEST_LATENCY.labels(endpoint=endpoint).observe(time.time() - start)

if __name__ == '__main__':
    # Start metrics server on port 8000
    start_http_server(8000)
    print("Metrics server started on :8000")
    
    ACTIVE_USERS.set(100)
    
    while True:
        process_request('/api/users')
        process_request('/api/orders')
        time.sleep(0.1)
```

Visit `http://localhost:8000/metrics` to see your custom metrics!

Add to `prometheus.yml`:
```yaml
- job_name: 'my-app'
  static_configs:
    - targets: ['host.docker.internal:8000']
```

---

## 2.4 The /metrics Endpoint Format

```
# HELP metric_name Human-readable description of the metric
# TYPE metric_name <counter|gauge|histogram|summary>
metric_name{label1="val1"} value [optional_timestamp]
```

Real example:
```
# HELP go_goroutines Number of goroutines that currently exist
# TYPE go_goroutines gauge
go_goroutines 7

# HELP http_requests_total Total HTTP requests received
# TYPE http_requests_total counter
http_requests_total{code="200",method="get"} 1027
http_requests_total{code="400",method="post"} 3
```

---

## 2.5 Quick Q&A — Module 2

**Q: When should you use a Counter vs Gauge?**
A: Use Counter for things that only ever increase (requests, errors, bytes sent). Use Gauge for values that can go up or down (memory, temperature, queue depth).

**Q: Why should you avoid high cardinality labels?**
A: Each unique label combination creates a new time-series stored in memory and disk. Millions of series cause "cardinality explosion" — Prometheus runs out of memory.

**Q: What is the difference between Histogram and Summary?**
A: Histograms allow server-side aggregation across multiple instances (use `histogram_quantile()` in PromQL). Summaries compute quantiles on the client — accurate per-instance but cannot be meaningfully aggregated.

**Q: What is an exporter?**
A: A separate process that fetches metrics from a system that doesn't natively expose them (e.g., Linux OS, MySQL), translates them to Prometheus format, and exposes a `/metrics` endpoint for Prometheus to scrape.

**Q: On what port does Node Exporter expose metrics?**
A: **9100**

---

# Module 3 — Prometheus Architecture & PromQL {#module-3}

## 3.1 Prometheus Architecture Deep Dive

```
┌─────────────────────────────────────────────────────────────────┐
│                    PROMETHEUS ARCHITECTURE                       │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    SERVICE DISCOVERY                      │   │
│  │    Kubernetes │ Consul │ EC2 │ DNS │ Static Files         │   │
│  └──────────────────────┬───────────────────────────────────┘   │
│                         │ Discovers targets                      │
│                         ▼                                        │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                  PROMETHEUS SERVER                        │   │
│  │                                                          │   │
│  │  ┌──────────────┐  ┌─────────────┐  ┌────────────────┐  │   │
│  │  │  Retrieval   │  │    TSDB     │  │  HTTP Server   │  │   │
│  │  │  (Scraper)   │─▶│  (Storage) │  │  (PromQL API)  │  │   │
│  │  │              │  │             │  │                │  │   │
│  │  │  Pulls /     │  │  Local      │  │  /query        │  │   │
│  │  │  metrics     │  │  on-disk    │  │  /query_range  │  │   │
│  │  │  every 15s   │  │  time-      │  │  /targets      │  │   │
│  │  └──────────────┘  │  series DB  │  └────────────────┘  │   │
│  │                    │             │                        │   │
│  │  ┌──────────────┐  └─────────────┘  ┌────────────────┐  │   │
│  │  │  Rules       │                   │  Alertmanager  │  │   │
│  │  │  Engine      │──────────────────▶│  Integration   │  │   │
│  │  │  (Eval rules)│    fire alerts    └────────────────┘  │   │
│  │  └──────────────┘                                        │   │
│  └──────────────────────────────────────────────────────────┘   │
│                         ▲           ▲                            │
│              ┌──────────┘           └──────────┐                │
│              │                                  │                │
│  ┌───────────────────┐          ┌───────────────────────────┐   │
│  │    EXPORTERS      │          │      PUSHGATEWAY          │   │
│  │  Node Exporter    │          │  Short-lived jobs push    │   │
│  │  cAdvisor         │          │  metrics here; Prometheus │   │
│  │  MySQL Exporter   │          │  scrapes Pushgateway      │   │
│  │  Custom apps      │          └───────────────────────────┘   │
│  └───────────────────┘                                          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3.2 The Scrape Mechanism (Pull Model)

**How scraping works step-by-step:**

```
Step 1: Prometheus reads prometheus.yml (scrape targets)
         ↓
Step 2: At each scrape_interval (e.g. 15s), Prometheus connects to target
         ↓
Step 3: Sends HTTP GET to http://<target>:<port>/metrics
         ↓
Step 4: Target returns text exposition format
         ↓
Step 5: Prometheus parses and stores time-series with current timestamp
         ↓
Step 6: Evaluates alert/recording rules against stored data
```

**Scrape configuration in detail:**

```yaml
global:
  scrape_interval: 15s          # Default scrape interval
  scrape_timeout: 10s           # Timeout per scrape
  evaluation_interval: 15s      # Rule evaluation interval

scrape_configs:
  # Job = logical group of targets
  - job_name: 'web-servers'
    scrape_interval: 30s        # Override for this job
    scrape_timeout: 5s
    metrics_path: /metrics      # Default path
    scheme: http                # http or https
    
    static_configs:
      - targets:
          - 'web1.example.com:9100'
          - 'web2.example.com:9100'
        labels:
          environment: 'production'
          team: 'platform'

  # Kubernetes service discovery
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
```

---

## 3.3 Prometheus Storage (TSDB)

### Data Retention
```yaml
# Set via command-line flags
--storage.tsdb.path=/prometheus          # Data directory
--storage.tsdb.retention.time=15d        # Keep data for 15 days (default: 15d)
--storage.tsdb.retention.size=10GB       # Or by size
--storage.tsdb.wal-compression           # Enable WAL compression (saves ~50%)
```

### Storage Format
```
/prometheus/
├── 01BKGV7JC0RY0KBXKN1/     ← Completed block (2h of data)
│   ├── chunks/               ← Compressed metric data
│   ├── index                 ← Fast lookup index
│   ├── meta.json             ← Block metadata
│   └── tombstones            ← Deleted series markers
├── 01BKGV7JC0RY0KBXKN2/
├── wal/                      ← Write-Ahead Log (recent data)
│   ├── 000001
│   └── 000002
└── lock
```

**Key facts:**
- Data is written to WAL first (in memory), then compacted into 2-hour blocks
- Blocks are periodically merged (compaction) for query efficiency
- Prometheus uses ~1-2 bytes per sample on average

---

## 3.4 Recording Rules

Recording rules pre-compute expensive PromQL expressions and store results as new metrics, greatly speeding up dashboards.

```yaml
# rules/recording_rules.yml

groups:
  - name: http_metrics
    interval: 30s      # Optional: override evaluation interval
    rules:
      # Pre-compute per-job request rate
      - record: job:http_requests:rate5m
        expr: sum by (job) (rate(http_requests_total[5m]))

      # Pre-compute CPU usage percentage
      - record: instance:node_cpu_utilisation:rate5m
        expr: |
          100 - (
            avg by (instance) (
              rate(node_cpu_seconds_total{mode="idle"}[5m])
            ) * 100
          )

      # Pre-compute memory usage ratio
      - record: instance:node_memory_utilisation
        expr: |
          1 - (
            node_memory_MemAvailable_bytes /
            node_memory_MemTotal_bytes
          )
```

Load rules in `prometheus.yml`:
```yaml
rule_files:
  - "rules/*.yml"
```

---

## 3.5 PromQL — Prometheus Query Language

PromQL is a functional query language for querying Prometheus time-series data.

### Data Types in PromQL

| Type | Description | Example |
|---|---|---|
| **Instant vector** | Set of time-series at a single point | `http_requests_total` |
| **Range vector** | Set of time-series over a time range | `http_requests_total[5m]` |
| **Scalar** | A single numeric value | `3.14` |
| **String** | A string value (rarely used) | `"hello"` |

### Selectors & Matchers

```promql
# Exact match
http_requests_total{job="api"}

# Regex match
http_requests_total{status=~"5.."}    # All 5xx

# Negation
http_requests_total{status!="200"}

# Regex negation
http_requests_total{method!~"GET|POST"}

# Range vector (last 5 minutes)
http_requests_total{job="api"}[5m]
```

### Essential Functions

#### rate() and irate()
```promql
# rate() — average per-second rate over window (smoothed, good for graphs)
rate(http_requests_total[5m])

# irate() — instantaneous rate (last 2 samples, good for spikes)
irate(http_requests_total[5m])

# increase() — total increase over window
increase(http_requests_total[1h])
```

#### Aggregation Operators
```promql
# sum() — aggregate all time-series into one
sum(rate(http_requests_total[5m]))

# sum by() — keep specified labels
sum by (job, status) (rate(http_requests_total[5m]))

# sum without() — drop specified labels
sum without (instance) (rate(http_requests_total[5m]))

# avg, max, min, count, stddev
avg by (job) (node_cpu_seconds_total)
max by (instance) (node_memory_usage_bytes)
count by (job) (up)
```

#### Practical Queries

```promql
# ─────────────────────────────────────────────
# CPU USAGE %
100 - (
  avg by (instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  ) * 100
)

# ─────────────────────────────────────────────
# MEMORY USAGE %
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# ─────────────────────────────────────────────
# DISK USAGE %
100 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"} * 100)

# ─────────────────────────────────────────────
# HTTP REQUEST RATE (per second)
sum by (job) (rate(http_requests_total[5m]))

# ─────────────────────────────────────────────
# HTTP ERROR RATE (5xx errors)
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))

# ─────────────────────────────────────────────
# P95 LATENCY
histogram_quantile(0.95, sum by (le) (rate(http_request_duration_seconds_bucket[5m])))

# ─────────────────────────────────────────────
# NUMBER OF TARGETS UP
count(up == 1) by (job)

# ─────────────────────────────────────────────
# TARGETS THAT ARE DOWN
up == 0

# ─────────────────────────────────────────────
# FREE DISK SPACE IN GB
node_filesystem_avail_bytes{mountpoint="/"} / 1024 / 1024 / 1024

# ─────────────────────────────────────────────
# NETWORK RECEIVE RATE (Bytes/s)
rate(node_network_receive_bytes_total{device!="lo"}[5m])
```

#### Time Offset & Subquery
```promql
# Compare current value to value 1 hour ago
http_requests_total offset 1h

# Subquery: downsample an expression
rate(http_requests_total[5m])[30m:1m]
```

---

## 3.6 Lab: Configure Scrape Targets & Write PromQL Queries

### Step 1: Configure multiple scrape targets

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance
        regex: '([^:]+).*'
        replacement: '$1'

  - job_name: 'app'
    scrape_interval: 10s
    static_configs:
      - targets: ['app:8000']
        labels:
          team: 'backend'
          env: 'staging'
```

### Step 2: Verify in Prometheus UI

Navigate to http://localhost:9090/targets → All targets should show `State: UP`

### Step 3: Practice Queries

```promql
# Basic — show all CPU metrics
node_cpu_seconds_total

# Rate — CPU usage rate
rate(node_cpu_seconds_total[5m])

# Filter — only idle mode
rate(node_cpu_seconds_total{mode="idle"}[5m])

# Aggregate — avg CPU across all cores
avg by (instance) (rate(node_cpu_seconds_total{mode!="idle"}[5m]))

# Percentage — CPU utilization %
100 * (1 - avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])))
```

---

## 3.7 Quick Q&A — Module 3

**Q: What is a scrape target?**
A: An HTTP endpoint (`host:port/metrics`) that Prometheus periodically polls to collect metrics.

**Q: What is the difference between `rate()` and `irate()`?**
A: `rate()` averages over the entire range window — smoother, good for trend graphs. `irate()` uses only the last two data points — sensitive to spikes, good for alerting.

**Q: What are recording rules and why use them?**
A: Pre-computed PromQL expressions stored as new metrics. Use them to speed up dashboard queries that run repeatedly and are expensive to compute.

**Q: How does Prometheus store data?**
A: In a local time-series database (TSDB). Recent data goes to a Write-Ahead Log (WAL), then gets compacted into 2-hour blocks. Default retention is 15 days.

**Q: What does `sum by (job)` do?**
A: Aggregates all time-series matching the query into one series per unique `job` label value, discarding all other labels.

---

# Module 4 — Alerting with Alertmanager {#module-4}

## 4.1 Alerting Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     ALERTING FLOW                                │
│                                                                 │
│  Prometheus Server                                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Evaluates alert rules every evaluation_interval        │   │
│  │                                                         │   │
│  │  Rule: IF cpu > 90% FOR 5m → ALERT                      │   │
│  │                                                         │   │
│  │  States: INACTIVE → PENDING → FIRING                    │   │
│  └───────────────────────────┬─────────────────────────────┘   │
│                              │  Sends firing alerts via HTTP     │
│                              ▼                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    ALERTMANAGER                          │   │
│  │                                                          │   │
│  │  ┌──────────┐   ┌──────────┐   ┌──────────────────────┐ │   │
│  │  │  Router  │   │ Grouping │   │   Deduplication      │ │   │
│  │  │          │   │          │   │                      │ │   │
│  │  │ Match    │   │ Bundle   │   │ Same alert from      │ │   │
│  │  │ alerts   │   │ related  │   │ multiple sources     │ │   │
│  │  │ to       │   │ alerts   │   │ → send once          │ │   │
│  │  │ receivers│   │ together │   └──────────────────────┘ │   │
│  │  └──────────┘   └──────────┘                             │   │
│  │                                                          │   │
│  │  ┌──────────┐   ┌──────────┐   ┌──────────────────────┐ │   │
│  │  │ Silences │   │Inhibition│   │    Receivers         │ │   │
│  │  │          │   │          │   │                      │ │   │
│  │  │ Mute     │   │ Suppress │   │ Email, Slack,        │ │   │
│  │  │ specific │   │ child    │   │ PagerDuty, OpsGenie  │ │   │
│  │  │ alerts   │   │ alerts   │   │ Webhook              │ │   │
│  │  └──────────┘   └──────────┘   └──────────────────────┘ │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4.2 Alert States

```
  INACTIVE ──────────────────────────────────────────────────┐
     │                                                        │
     │ threshold breached                                     │
     ▼                                                        │
  PENDING ─────── for duration ──────▶ FIRING ───────────────┘
     │                                    │         resolved
     │ resolved before duration           │
     └────────────────────────────────────┘
```

- **INACTIVE**: Condition not met
- **PENDING**: Condition met but hasn't lasted `for` duration yet
- **FIRING**: Condition met for the full `for` duration — alert sent to Alertmanager

The `for` clause prevents false positives from transient spikes.

---

## 4.3 Defining Alert Rules

```yaml
# rules/alert_rules.yml

groups:
  - name: system_alerts
    rules:

      # ─────── CPU Alert ───────
      - alert: HighCPUUsage
        expr: |
          100 - (avg by (instance) (
            rate(node_cpu_seconds_total{mode="idle"}[5m])
          ) * 100) > 90
        for: 5m
        labels:
          severity: warning
          team: platform
        annotations:
          summary: "High CPU on {{ $labels.instance }}"
          description: "CPU usage is {{ $value | printf \"%.1f\" }}% on {{ $labels.instance }}"
          runbook_url: "https://wiki.example.com/runbooks/high-cpu"

      # ─────── Critical CPU Alert ───────
      - alert: CriticalCPUUsage
        expr: |
          100 - (avg by (instance) (
            rate(node_cpu_seconds_total{mode="idle"}[5m])
          ) * 100) > 98
        for: 2m
        labels:
          severity: critical
          team: platform
        annotations:
          summary: "CRITICAL CPU on {{ $labels.instance }}"
          description: "CPU usage is {{ $value | printf \"%.1f\" }}%"

      # ─────── Memory Alert ───────
      - alert: HighMemoryUsage
        expr: |
          (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          description: "Memory usage is {{ $value | printf \"%.1f\" }}%"

      # ─────── Disk Space Alert ───────
      - alert: DiskSpaceLow
        expr: |
          (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100 < 15
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
          description: "Only {{ $value | printf \"%.1f\" }}% disk space remaining"

      # ─────── Service Down Alert ───────
      - alert: ServiceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Service {{ $labels.job }} is DOWN"
          description: "Target {{ $labels.instance }} has been down for more than 1 minute"

      # ─────── High Error Rate ───────
      - alert: HighErrorRate
        expr: |
          sum by (job) (rate(http_requests_total{status=~"5.."}[5m])) /
          sum by (job) (rate(http_requests_total[5m])) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate for {{ $labels.job }}"
          description: "Error rate is {{ $value | humanizePercentage }}"
```

---

## 4.4 Alertmanager Configuration

```yaml
# alertmanager.yml

global:
  resolve_timeout: 5m
  smtp_smarthost: 'smtp.example.com:587'
  smtp_from: 'alertmanager@example.com'
  slack_api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'

# ─── Routing Tree ───
route:
  # Default receiver for unmatched alerts
  receiver: 'slack-general'
  group_by: ['alertname', 'cluster']
  group_wait: 30s           # Wait 30s to group alerts together
  group_interval: 5m        # Wait 5m between group notifications
  repeat_interval: 4h       # Re-send if still firing after 4h

  routes:
    # Critical alerts → PagerDuty
    - match:
        severity: critical
      receiver: 'pagerduty-critical'
      continue: false        # Stop routing once matched

    # Platform team alerts → Slack #platform-alerts
    - match:
        team: platform
      receiver: 'slack-platform'

    # Database alerts → Email DBA team
    - match_re:
        alertname: '^DB.*'
      receiver: 'email-dba'

# ─── Receivers ───
receivers:
  - name: 'slack-general'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/TOKEN'
        channel: '#monitoring'
        title: '{{ template "slack.title" . }}'
        text: '{{ template "slack.text" . }}'
        send_resolved: true

  - name: 'slack-platform'
    slack_configs:
      - channel: '#platform-alerts'
        color: '{{ if eq .Status "firing" }}danger{{ else }}good{{ end }}'
        title: '🚨 {{ .CommonAnnotations.summary }}'
        text: |
          *Description:* {{ .CommonAnnotations.description }}
          *Severity:* {{ .CommonLabels.severity }}
          *Instance:* {{ .CommonLabels.instance }}

  - name: 'pagerduty-critical'
    pagerduty_configs:
      - routing_key: 'YOUR_PAGERDUTY_ROUTING_KEY'
        description: '{{ .CommonAnnotations.summary }}'

  - name: 'email-dba'
    email_configs:
      - to: 'dba-team@example.com'
        subject: '[ALERT] {{ .CommonAnnotations.summary }}'
        body: |
          Alert: {{ .CommonAnnotations.summary }}
          Description: {{ .CommonAnnotations.description }}

# ─── Inhibition Rules ───
inhibit_rules:
  # If critical alert fires, suppress warning for same instance
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'instance']
```

### Point Prometheus at Alertmanager

```yaml
# prometheus.yml
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

rule_files:
  - "rules/alert_rules.yml"
```

### Run Alertmanager

```bash
docker run -d \
  -p 9093:9093 \
  -v $(pwd)/alertmanager.yml:/etc/alertmanager/alertmanager.yml \
  --name alertmanager \
  prom/alertmanager
```

---

## 4.5 Silences

**Creating a silence via API:**
```bash
# Silence all alerts for instance=web1 for 2 hours
curl -X POST http://localhost:9093/api/v1/silences \
  -H 'Content-Type: application/json' \
  -d '{
    "matchers": [
      {"name": "instance", "value": "web1", "isRegex": false}
    ],
    "startsAt": "2024-01-01T10:00:00Z",
    "endsAt":   "2024-01-01T12:00:00Z",
    "createdBy": "john",
    "comment": "Maintenance window"
  }'
```

Or use the Alertmanager UI at http://localhost:9093 → New Silence.

---

## 4.6 Quick Q&A — Module 4

**Q: What is the difference between PENDING and FIRING?**
A: PENDING means the alert condition is currently true but hasn't lasted the `for` duration yet. FIRING means it has — Alertmanager receives the alert.

**Q: What does grouping do in Alertmanager?**
A: Bundles multiple related alerts into a single notification. E.g., if 20 nodes go down at once, you get 1 Slack message, not 20.

**Q: What is inhibition?**
A: Suppresses lower-priority alerts when a higher-priority alert is already firing for the same resource. E.g., don't send "disk space warning" if "node is completely down" is firing.

**Q: How do silences work?**
A: Silences temporarily mute alerts matching specified label matchers — useful during planned maintenance windows.

**Q: On what port does Alertmanager run?**
A: **9093**

---

# Module 5 — Grafana Dashboards {#module-5}

## 5.1 Connecting Grafana to Prometheus

```
Grafana UI → Configuration → Data Sources → Add data source → Prometheus
  URL: http://prometheus:9090
  Access: Server (default)
  Scrape interval: 15s
  → Save & Test
```

---

## 5.2 Grafana Dashboard Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    GRAFANA DASHBOARD                         │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                      ROWS                           │   │
│  │  ┌─────────────────────────────────────────────┐   │   │
│  │  │                   PANELS                    │   │   │
│  │  │                                             │   │   │
│  │  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  │   │   │
│  │  │  │  Graph   │  │  Stat    │  │  Table   │  │   │   │
│  │  │  │  Panel   │  │  Panel   │  │  Panel   │  │   │   │
│  │  │  │          │  │          │  │          │  │   │   │
│  │  │  │ PromQL   │  │ PromQL   │  │ PromQL   │  │   │   │
│  │  │  │ query    │  │ query    │  │ query    │  │   │   │
│  │  │  └──────────┘  └──────────┘  └──────────┘  │   │   │
│  │  └─────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  Variables: $job = [api, worker, db]                        │
│  Time Range: Last 1 hour ────────────────────────────────   │
└─────────────────────────────────────────────────────────────┘
```

---

## 5.3 Panel Types

| Panel Type | Best For | Example |
|---|---|---|
| **Time series** | Trends over time | CPU usage, request rate |
| **Stat** | Single current value | Uptime, active connections |
| **Gauge** | Value relative to range | Memory % (0-100) |
| **Bar chart** | Category comparison | Requests by service |
| **Table** | Detailed rows | Top slow queries |
| **Heatmap** | Distribution over time | Request latency histogram |
| **Pie chart** | Proportions | HTTP status code breakdown |
| **Logs** | Log streams (Loki) | Application log viewer |
| **Traces** | Distributed tracing (Tempo) | Service dependency map |
| **Alert list** | Active alerts | Current firing alerts |
| **Text** | Markdown documentation | Dashboard notes |

---

## 5.4 Building a System Overview Dashboard

### Panel 1: CPU Usage (Time Series)

```
Panel type: Time series
Title: CPU Usage (%)
Query:
  100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

Legend: {{instance}}
Unit: Percent (0-100)
Thresholds:
  - 70% → yellow (warning)
  - 90% → red (critical)
```

### Panel 2: Memory Usage (Gauge)

```
Panel type: Gauge
Title: Memory Usage (%)
Query:
  (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

Unit: Percent (0-100)
Min: 0, Max: 100
Thresholds: 80% → yellow, 90% → red
```

### Panel 3: Network I/O (Time Series)

```
Panel type: Time series
Title: Network Traffic
Queries:
  A: rate(node_network_receive_bytes_total{device!="lo"}[5m])
     Legend: RX {{instance}} {{device}}
  B: -rate(node_network_transmit_bytes_total{device!="lo"}[5m])
     Legend: TX {{instance}} {{device}}

Unit: bytes/sec
Tip: Negate TX to show it below zero (butterfly chart)
```

### Panel 4: HTTP Request Rate (Time Series)

```
Panel type: Time series
Title: HTTP Requests/sec
Query:
  sum by (job, status) (rate(http_requests_total[5m]))

Legend: {{job}} - {{status}}
Unit: requests/sec
```

### Panel 5: P95 Latency (Stat)

```
Panel type: Stat
Title: P95 Request Latency
Query:
  histogram_quantile(0.95, sum by (le) (rate(http_request_duration_seconds_bucket[5m])))

Unit: seconds
Color mode: Background
Thresholds: 0.5s → yellow, 1s → red
```

---

## 5.5 Variables & Templating

Variables make dashboards dynamic and reusable.

**Creating a `$job` variable:**
```
Dashboard Settings → Variables → Add variable

Name: job
Type: Query
Data source: Prometheus
Query: label_values(up, job)
Multi-value: true
Include All: true
Refresh: On Dashboard Load
```

**Using variables in queries:**
```promql
# Filter by selected job
rate(http_requests_total{job=~"$job"}[5m])

# Filter by instance and environment
node_memory_MemAvailable_bytes{instance=~"$instance", env="$env"}
```

**Chained variables (instance depends on job):**
```promql
# $instance variable query:
label_values(up{job=~"$job"}, instance)
```

---

## 5.6 Grafana Alerting

Grafana has its own alerting engine separate from Prometheus Alertmanager.

```
Alert Rule Setup:
  Alerting → Alert Rules → New Alert Rule

  1. Define query:
     A: avg(rate(node_cpu_seconds_total{mode!="idle"}[5m])) * 100

  2. Define condition:
     WHEN: avg() OF A IS ABOVE 90
     FOR: 5m

  3. Add labels:
     severity = warning
     team = platform

  4. Add annotations:
     summary = High CPU usage detected
     description = CPU usage exceeds 90%

  5. Assign notification policy:
     Contact point: slack-channel
```

**Contact Points:**
```
Alerting → Contact Points → New Contact Point
Type: Slack
Webhook URL: https://hooks.slack.com/services/...
```

---

## 5.7 Combining Multiple Data Sources

Grafana can mix data sources in the same dashboard.

```
┌────────────────────────────────────────────────────────────┐
│              UNIFIED OBSERVABILITY DASHBOARD                │
│                                                            │
│  ┌──────────────────┐  ┌──────────────────────────────┐   │
│  │  Prometheus      │  │         Loki                 │   │
│  │  (Metrics)       │  │         (Logs)               │   │
│  │                  │  │                              │   │
│  │  CPU: 87% ████   │  │  ERROR: DB conn timeout      │   │
│  │  MEM: 62% ████   │  │  WARN: High latency          │   │
│  │  ERR: 2.3%       │  │  INFO: Request processed     │   │
│  └──────────────────┘  └──────────────────────────────┘   │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                    Tempo (Traces)                    │  │
│  │  TraceID: abc123 → service-a → service-b → db        │  │
│  │  Total: 450ms   [100ms]    [200ms]     [150ms]        │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘
```

**Correlating metrics → logs using Explore:**
```
In a metrics panel → right-click → "Explore" → switch to Logs view

Or set up "Data Links" on a metric panel:
  Panel → Field → Data links → Add link
  URL: /explore?...&query={job="${__field.labels.job}"}
```

---

## 5.8 Importing Pre-built Dashboards

Grafana has a community dashboard library at grafana.com/grafana/dashboards

```
Dashboard → Import → Enter Dashboard ID:

Popular dashboards:
  1860  — Node Exporter Full
  893   — Docker monitoring with cAdvisor
  3662  — Prometheus 2.0 Overview
  7249  — Kubernetes Cluster (Prometheus)
  11159 — Node Exporter for Prometheus by rfmoz
```

---

## 5.9 Quick Q&A — Module 5

**Q: How do you connect Grafana to Prometheus?**
A: Configuration → Data Sources → Add → Prometheus → Set URL to `http://prometheus:9090` → Save & Test.

**Q: What is a Grafana variable?**
A: A dynamic value (like `$job` or `$instance`) that users can select from a dropdown, making dashboards reusable across different targets without editing queries.

**Q: What is the difference between Grafana alerting and Prometheus Alertmanager alerting?**
A: Prometheus alerting evaluates PromQL against time-series data and sends to Alertmanager. Grafana alerting evaluates queries across *any* data source and sends directly to notification channels. For Prometheus, Alertmanager is more powerful for routing/grouping; Grafana alerting is easier for mixed data sources.

**Q: What panel type should you use for showing a single current value?**
A: Use **Stat** panel for a single large number, or **Gauge** if you want to show progress relative to min/max.

---

# Module 6 — Advanced Prometheus Concepts {#module-6}

## 6.1 High Availability (HA) with Prometheus Federation

### The Scalability Problem

A single Prometheus instance is limited by:
- Single-machine memory and disk
- Cannot aggregate metrics from multiple DCs
- Single point of failure

### Federation

**Federation** lets a higher-level Prometheus scrape aggregated metrics from multiple lower-level Prometheus servers.

```
┌──────────────────────────────────────────────────────────────┐
│                    FEDERATION HIERARCHY                       │
│                                                              │
│          ┌─────────────────────────────┐                     │
│          │   Global Prometheus (Top)   │                     │
│          │   - Scrapes aggregated      │                     │
│          │     metrics from below      │                     │
│          └────────────┬────────────────┘                     │
│                       │                                      │
│         ┌─────────────┼────────────────┐                     │
│         ▼             ▼                ▼                     │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐               │
│  │ Prometheus │ │ Prometheus │ │ Prometheus │               │
│  │  DC-East   │ │  DC-West   │ │  DC-EU     │               │
│  └────────────┘ └────────────┘ └────────────┘               │
│         │             │                │                     │
│    [app1..n]      [app1..n]        [app1..n]                 │
└──────────────────────────────────────────────────────────────┘
```

**Federation scrape config on Global Prometheus:**

```yaml
scrape_configs:
  - job_name: 'federate'
    scrape_interval: 15s
    honor_labels: true         # Keep labels from source Prometheus
    metrics_path: '/federate'
    params:
      match[]:
        - '{job="kubernetes-pods"}'           # Only scrape these metrics
        - '{__name__=~"job:.*"}'              # Or recording rules
    static_configs:
      - targets:
          - 'prometheus-dc-east:9090'
          - 'prometheus-dc-west:9090'
          - 'prometheus-dc-eu:9090'
```

---

## 6.2 High Availability Setup

Run two Prometheus instances with identical configs:

```yaml
# docker-compose-ha.yml
services:
  prometheus-1:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--web.external-url=http://prometheus-1:9090'

  prometheus-2:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--web.external-url=http://prometheus-2:9090'

  # Load balancer
  nginx:
    image: nginx
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    ports:
      - "9090:9090"
```

```nginx
# nginx.conf
upstream prometheus {
    server prometheus-1:9090;
    server prometheus-2:9090;
}

server {
    location / {
        proxy_pass http://prometheus;
    }
}
```

> **Note:** For true HA with consistent storage, use **Thanos** or **Cortex** which add long-term storage and global query capabilities on top of Prometheus.

---

## 6.3 Pushgateway for Short-Lived Jobs

Prometheus scrapes at intervals — short-lived jobs (batch jobs, cron jobs) may finish before they're scraped.

```
┌──────────────────────────────────────────────────────────┐
│                   PUSHGATEWAY PATTERN                     │
│                                                          │
│  ┌──────────┐   push metrics   ┌─────────────┐          │
│  │  Batch   │─────────────────▶│ Pushgateway │          │
│  │  Job     │                  │             │          │
│  │(finishes │                  │ Stores last  │          │
│  │ quickly) │                  │  known value │          │
│  └──────────┘                  └──────┬──────┘          │
│                                       │ scrape           │
│                                       ▼                  │
│                               ┌──────────────┐          │
│                               │  Prometheus  │          │
│                               └──────────────┘          │
└──────────────────────────────────────────────────────────┘
```

**Running Pushgateway:**
```bash
docker run -d -p 9091:9091 --name pushgateway prom/pushgateway
```

**Pushing metrics (shell/curl):**
```bash
# Push a batch job completion metric
cat << EOF | curl --data-binary @- http://localhost:9091/metrics/job/backup_job/instance/server1
# TYPE backup_duration_seconds gauge
backup_duration_seconds 42.5
# TYPE backup_last_success_timestamp_seconds gauge
backup_last_success_timestamp_seconds $(date +%s)
EOF
```

**Pushing with Python:**
```python
from prometheus_client import CollectorRegistry, Gauge, push_to_gateway

registry = CollectorRegistry()
g = Gauge('batch_job_duration_seconds', 'Duration of batch job', registry=registry)
g.set(42.5)

push_to_gateway('localhost:9091', job='my_batch_job', registry=registry)
```

**Scrape Pushgateway:**
```yaml
- job_name: 'pushgateway'
  honor_labels: true   # Use labels from pushed metrics, not Prometheus
  static_configs:
    - targets: ['pushgateway:9091']
```

> **Caution:** Pushgateway is not a general event pipeline. It's for batch jobs. It can persist stale metrics if a job fails silently.

---

## 6.4 Advanced Recording Rules

```yaml
groups:
  - name: slo_rules
    interval: 1m
    rules:
      # Error budget: 99.9% SLO = 0.1% error budget
      - record: job:slo_error_budget_remaining
        expr: |
          1 - (
            sum by (job) (
              increase(http_requests_total{status=~"5.."}[1h])
            ) /
            sum by (job) (
              increase(http_requests_total[1h])
            )
          ) - 0.999

      # Latency SLO: 95% of requests < 500ms
      - record: job:request_latency_slo
        expr: |
          sum by (job) (
            rate(http_request_duration_seconds_bucket{le="0.5"}[5m])
          ) /
          sum by (job) (
            rate(http_request_duration_seconds_count[5m])
          )
```

---

## 6.5 Best Practices for Prometheus Performance

### Configuration
- Set `scrape_interval` no lower than 10s (15s is fine for most)
- Use recording rules for any query used in dashboards
- Limit label cardinality — avoid user IDs, UUIDs in labels
- Set appropriate retention (`--storage.tsdb.retention.time`)

### Capacity Planning
```
Prometheus memory ≈ (ingested_samples_per_second × 2 bytes × retention_in_seconds) / compression_ratio

Example:
  100,000 samples/sec × 2 bytes × 86400 sec × 15 days / 10 compression
  = ~25 GB RAM needed
```

### Storage Optimization
```bash
# Enable WAL compression (50% storage savings)
--storage.tsdb.wal-compression

# Chunk encoding (automatically uses best encoding)
# Prometheus automatically uses delta/XOR encoding on chunks

# For long-term storage, use remote_write to object storage
remote_write:
  - url: "http://thanos-receiver:19291/api/v1/receive"
```

---

## 6.6 Quick Q&A — Module 6

**Q: What is Prometheus Federation?**
A: A mechanism where a higher-level Prometheus scrapes aggregated metrics from lower-level Prometheus instances — useful for cross-datacenter monitoring.

**Q: When should you use Pushgateway?**
A: For short-lived jobs (cron jobs, batch scripts) that complete before Prometheus would scrape them. Don't use it as a general push endpoint.

**Q: What is Thanos?**
A: An open-source project that adds global query view, unlimited retention (via object storage like S3), and HA deduplication on top of Prometheus.

**Q: What causes high cardinality and why is it bad?**
A: Using labels with many unique values (user IDs, request IDs). Each unique combination = a new time-series, consuming memory and slowing queries.

---

# Module 7 — Observability & Troubleshooting {#module-7}

## 7.1 End-to-End Monitoring Workflow

```
┌─────────────────────────────────────────────────────────────────────┐
│               END-TO-END OBSERVABILITY WORKFLOW                      │
│                                                                     │
│  INFRASTRUCTURE                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  Servers │ Containers │ Kubernetes pods │ Cloud services       │  │
│  └──────────────────────────┬────────────────────────────────────┘  │
│                             │ emit                                   │
│                             ▼                                        │
│  COLLECTION                                                          │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │   Exporters scrape →  Prometheus TSDB   ←  App instrumentation│  │
│  │   Agents push → Loki (logs) │ Tempo (traces)                  │  │
│  └──────────────────────────┬────────────────────────────────────┘  │
│                             │                                        │
│                             ▼                                        │
│  VISUALIZATION & ALERTING                                            │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │   Grafana Dashboards ← Prometheus │ Loki │ Tempo               │  │
│  │   Alertmanager → Slack │ PagerDuty │ Email                     │  │
│  └──────────────────────────┬────────────────────────────────────┘  │
│                             │                                        │
│                             ▼                                        │
│  INSIGHTS & ACTION                                                   │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  RCA → Capacity Planning → SLO Tracking → Auto-remediation    │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 7.2 Root Cause Analysis Workflow

**Scenario: Users report slow API responses**

### Step 1: Check High-Level Metrics
```promql
# Is error rate elevated?
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))

# Is latency elevated?
histogram_quantile(0.95, sum by (le) (rate(http_request_duration_seconds_bucket[5m])))
```

### Step 2: Drill Down by Service
```promql
# Which services are slow?
histogram_quantile(0.95, sum by (le, job) (rate(http_request_duration_seconds_bucket[5m])))

# Which endpoints?
histogram_quantile(0.95, sum by (le, endpoint) (rate(http_request_duration_seconds_bucket[5m])))
```

### Step 3: Check Resources
```promql
# CPU saturated?
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory pressure?
(1 - node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes) * 100

# Disk I/O saturation?
rate(node_disk_io_time_seconds_total[5m]) * 100

# Network saturation?
rate(node_network_receive_bytes_total[5m])
```

### Step 4: Correlate with Logs (Loki)
```logql
# Find errors around the same time
{job="api"} |= "ERROR" | json | line_format "{{.message}}"

# Find slow database queries
{job="db"} |~ "Query_time: [0-9]{2,}" 
```

### Step 5: Trace the Slow Request (Tempo)
- Look for traces with high duration
- Find which span (service/operation) is the bottleneck
- Check database query time vs network time vs application time

---

## 7.3 Lab: Simulate & Diagnose a Service Slowdown

### Step 1: Deploy Sample App with a Simulated Issue

```python
# slow_app.py — intentionally slow endpoint
from flask import Flask
from prometheus_client import Histogram, Counter, generate_latest
import time, random

app = Flask(__name__)

LATENCY = Histogram('http_request_duration_seconds', 'Latency', ['path'])
REQUESTS = Counter('http_requests_total', 'Requests', ['path', 'status'])

@app.route('/api/fast')
def fast():
    start = time.time()
    time.sleep(random.uniform(0.01, 0.05))  # Normal: 10-50ms
    LATENCY.labels(path='/api/fast').observe(time.time() - start)
    REQUESTS.labels(path='/api/fast', status='200').inc()
    return 'OK'

@app.route('/api/slow')
def slow():
    start = time.time()
    time.sleep(random.uniform(2, 5))  # SLOW: 2-5 seconds!
    LATENCY.labels(path='/api/slow').observe(time.time() - start)
    REQUESTS.labels(path='/api/slow', status='200').inc()
    return 'OK (slow)'

@app.route('/metrics')
def metrics():
    return generate_latest()

if __name__ == '__main__':
    app.run(port=5000)
```

### Step 2: Generate Load

```bash
# Install hey or ab for load testing
# hey -n 1000 -c 10 http://localhost:5000/api/fast
# hey -n 100 -c 5 http://localhost:5000/api/slow

# Simple loop
while true; do
  curl -s http://localhost:5000/api/fast > /dev/null
  curl -s http://localhost:5000/api/slow > /dev/null
  sleep 0.1
done
```

### Step 3: Create Diagnostic Dashboard in Grafana

```
Row 1: Service Health
  - P50/P95/P99 Latency by endpoint (time series)
  - Request rate by endpoint (time series)
  - Error rate (stat)

Row 2: Infrastructure
  - CPU % (gauge)
  - Memory % (gauge)
  - Network I/O (time series)

Row 3: Alerts
  - Alert list panel
```

### Step 4: Create an Alert for Slow Endpoint

```yaml
- alert: SlowEndpointDetected
  expr: |
    histogram_quantile(0.95,
      sum by (le, path) (
        rate(http_request_duration_seconds_bucket[5m])
      )
    ) > 1.0
  for: 2m
  labels:
    severity: warning
  annotations:
    summary: "Slow endpoint: {{ $labels.path }}"
    description: "P95 latency is {{ $value | printf \"%.2f\" }}s"
```

---

## 7.4 Performance Optimization Tips

### Prometheus Query Optimization
1. **Use recording rules** for repeated or complex queries in dashboards
2. **Limit range vectors** — `[5m]` not `[1d]` for live panels
3. **Use `without` instead of `by`** when keeping most labels
4. **Avoid regex on metric names** — use label matchers instead
5. **Reduce scrape frequency** for stable metrics (e.g., disk capacity can be 5m)

### Alert Tuning
1. Set appropriate `for` durations to avoid flapping
2. Use `group_wait` in Alertmanager to batch burst alerts
3. Use inhibition rules to reduce noise
4. Write actionable runbook URLs in annotations

### Dashboard Performance
1. Import dashboards from grafana.com as a starting point
2. Set `min_step` on panels to match scrape interval
3. Avoid too many panels on a single dashboard
4. Use dashboard links to drill down rather than one mega-dashboard

---

## 7.5 Quick Q&A — Module 7

**Q: Walk me through diagnosing high API latency using Prometheus and Grafana.**
A: 1) Check P95 latency metric — is it elevated? 2) Break down by service/endpoint to isolate which is slow. 3) Check resource metrics (CPU, memory, disk I/O) on the serving host. 4) Check error rates simultaneously. 5) Correlate with logs in Loki for error messages. 6) Use traces in Tempo to find the slow span.

**Q: What is the USE method?**
A: A performance methodology for resources: **Utilization** (how busy is it?), **Saturation** (how overloaded?), **Errors** (are there errors?). Great for resource-level diagnosis.

**Q: What is the RED method?**
A: A methodology for services: **Rate** (requests per second), **Errors** (failed requests), **Duration** (latency distribution). Great for service-level health.

---

# Module 8 — Optional Integrations {#module-8}

## 8.1 Grafana Loki — Log Aggregation

Loki is "like Prometheus, but for logs" — it indexes only labels (not log content), making it efficient.

### Loki Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    LOKI ARCHITECTURE                          │
│                                                              │
│  Applications                                                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                   │
│  │  App 1   │  │  App 2   │  │  App 3   │                   │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘                   │
│       │              │              │                         │
│       └──────────────┼──────────────┘                        │
│                      │ push logs                             │
│                      ▼                                        │
│            ┌──────────────────┐                              │
│            │   Promtail /     │                              │
│            │   Alloy /        │  ← Log collection agent      │
│            │   Fluent Bit     │                              │
│            └────────┬─────────┘                              │
│                     │                                        │
│                     ▼                                        │
│            ┌──────────────────┐                              │
│            │     LOKI         │                              │
│            │                  │                              │
│            │  Labels index    │  ← Only indexes labels!       │
│            │  Log chunks      │                              │
│            └────────┬─────────┘                              │
│                     │                                        │
│                     ▼                                        │
│            ┌──────────────────┐                              │
│            │    GRAFANA       │  ← Query with LogQL           │
│            └──────────────────┘                              │
└──────────────────────────────────────────────────────────────┘
```

### Running Loki + Promtail

```yaml
# docker-compose-loki.yml
services:
  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml

  promtail:
    image: grafana/promtail:latest
    volumes:
      - /var/log:/var/log:ro
      - ./promtail-config.yml:/etc/promtail/config.yml
    command: -config.file=/etc/promtail/config.yml
```

```yaml
# promtail-config.yml
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: system
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*.log

  - job_name: app
    static_configs:
      - targets:
          - localhost
        labels:
          job: myapp
          env: production
          __path__: /var/log/app/*.log
```

### LogQL — Loki's Query Language

```logql
# Filter by label
{job="myapp"}

# Filter by content
{job="myapp"} |= "ERROR"

# Regex filter
{job="myapp"} |~ "error|exception|timeout"

# Parse JSON logs
{job="myapp"} | json | level="error"

# Count errors per minute
sum(rate({job="myapp"} |= "ERROR" [1m])) by (job)

# Line format — customize log output
{job="myapp"} | json | line_format "{{.level}}: {{.message}}"

# Metric query — request rate from logs
sum(rate({job="nginx"} | logfmt | status >= 500 [5m])) by (path)
```

### Adding Loki to Grafana

```
Configuration → Data Sources → Add → Loki
URL: http://loki:3100
→ Save & Test

Explore → Switch source to Loki → Write LogQL queries
```

---

## 8.2 Grafana Tempo — Distributed Tracing

Tempo stores and queries distributed traces, integrating with OpenTelemetry, Jaeger, and Zipkin.

### Tracing Concepts

```
┌─────────────────────────────────────────────────────────────┐
│                     DISTRIBUTED TRACE                        │
│                                                             │
│  Trace (entire request journey):                            │
│  ├── Span: Frontend (50ms)                                  │
│  │     ├── Span: API Gateway (200ms)                        │
│  │     │     ├── Span: Auth Service (30ms)                  │
│  │     │     ├── Span: User Service (80ms)                  │
│  │     │     │     └── Span: PostgreSQL query (60ms)        │
│  │     │     └── Span: Cache (10ms)                         │
│  │     └── Span: CDN (5ms)                                  │
│                                                             │
│  Total trace duration: ~375ms                               │
│  Bottleneck: PostgreSQL query took 60ms                     │
└─────────────────────────────────────────────────────────────┘
```

### Running Tempo

```yaml
# docker-compose-tempo.yml
services:
  tempo:
    image: grafana/tempo:latest
    command: [ "-config.file=/etc/tempo.yaml" ]
    volumes:
      - ./tempo.yaml:/etc/tempo.yaml
    ports:
      - "3200:3200"   # Tempo HTTP
      - "4317:4317"   # OTLP gRPC
      - "4318:4318"   # OTLP HTTP
```

```yaml
# tempo.yaml
server:
  http_listen_port: 3200

distributor:
  receivers:
    otlp:
      protocols:
        grpc:
        http:

storage:
  trace:
    backend: local
    local:
      path: /tmp/tempo/blocks

querier:
  frontend_worker:
    frontend_address: tempo:9095
```

### Adding Tempo to Grafana

```
Configuration → Data Sources → Add → Tempo
URL: http://tempo:3200
→ Save & Test

Link Traces to Logs:
  In Tempo data source settings:
  Trace to logs: Data source = Loki, Tags = job, pod
```

---

## 8.3 Kubernetes Metrics Server + kube-state-metrics

For Kubernetes monitoring:

```yaml
# prometheus.yml — Kubernetes scrape configs
scrape_configs:
  # Kubernetes API server
  - job_name: 'kubernetes-apiservers'
    kubernetes_sd_configs:
      - role: endpoints
    scheme: https
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    relabel_configs:
      - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name]
        action: keep
        regex: default;kubernetes

  # All pods with prometheus.io/scrape: "true" annotation
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: 'true'
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: (.+)
        replacement: ${1}
```

**Key Kubernetes Metrics:**
```promql
# Pod CPU usage
sum by (pod, namespace) (
  rate(container_cpu_usage_seconds_total{container!=""}[5m])
)

# Pod memory
sum by (pod, namespace) (
  container_memory_working_set_bytes{container!=""}
)

# Pod restart count
kube_pod_container_status_restarts_total

# Node resource requests vs limits
kube_pod_container_resource_requests{resource="cpu"}
kube_pod_container_resource_limits{resource="memory"}

# Deployment availability
kube_deployment_status_replicas_available / kube_deployment_spec_replicas
```

---

## 8.4 AWS CloudWatch Integration

```yaml
# CloudWatch Exporter config
region: us-east-1
metrics:
  - aws_namespace: AWS/EC2
    aws_metric_name: CPUUtilization
    aws_dimensions: [InstanceId]
    aws_statistics: [Average]
    period_seconds: 300

  - aws_namespace: AWS/RDS
    aws_metric_name: DatabaseConnections
    aws_dimensions: [DBInstanceIdentifier]
    aws_statistics: [Average]

  - aws_namespace: AWS/ApplicationELB
    aws_metric_name: TargetResponseTime
    aws_dimensions: [LoadBalancer]
    aws_statistics: [p95]
```

Or use the Grafana AWS CloudWatch data source directly:
```
Configuration → Data Sources → Add → CloudWatch
Auth Provider: AWS SDK Default / Access & Secret Key
Default Region: us-east-1
```

---

## 8.5 Quick Q&A — Module 8

**Q: What is the difference between Loki and Elasticsearch for logs?**
A: Loki only indexes *labels*, not log content — this makes it much cheaper to operate. Elasticsearch indexes everything — more powerful full-text search but expensive. Loki pairs naturally with Prometheus since they share label conventions.

**Q: What is a trace span?**
A: A single unit of work in a distributed trace — has a name, start time, duration, and metadata. Multiple spans form a trace representing a complete request journey.

**Q: What protocol does Tempo use to receive traces?**
A: **OTLP** (OpenTelemetry Protocol) — gRPC on port 4317, HTTP on port 4318. Also supports Jaeger and Zipkin formats.

---

# Module 9 — Projects & Hands-On Labs {#module-9}

## 9.1 Project 1: Complete Monitoring Stack

### Overview

Deploy a sample microservices app and build a full monitoring stack around it.

### Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                  PROJECT 1: MONITORING STACK                      │
│                                                                  │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐                     │
│  │  Frontend│   │  API     │   │   DB     │                     │
│  │  :3000   │──▶│  :5000   │──▶│  :5432   │                     │
│  └────┬─────┘   └────┬─────┘   └────┬─────┘                     │
│       │              │              │                            │
│  /metrics       /metrics       pg_exporter                      │
│       │              │              │                            │
│       └──────────────┼──────────────┘                           │
│                      │  scrape                                   │
│                      ▼                                           │
│              ┌──────────────┐   ┌──────────────┐                │
│              │  Prometheus  │──▶│ Alertmanager │──▶ Slack       │
│              │  :9090       │   │  :9093       │                │
│              └──────┬───────┘   └──────────────┘                │
│                     │                                            │
│              ┌──────▼───────┐                                    │
│              │   Grafana    │                                    │
│              │   :3001      │                                    │
│              └──────────────┘                                    │
└──────────────────────────────────────────────────────────────────┘
```

### Complete docker-compose.yml

```yaml
version: "3.8"

services:
  # ─── Sample App ───
  app:
    image: python:3.11-slim
    working_dir: /app
    volumes:
      - ./app:/app
    command: python app.py
    ports:
      - "5000:5000"
    depends_on:
      - db

  # ─── PostgreSQL ───
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: appdb
    ports:
      - "5432:5432"

  # ─── PostgreSQL Exporter ───
  postgres-exporter:
    image: prometheuscommunity/postgres-exporter
    environment:
      DATA_SOURCE_NAME: "postgresql://app:secret@db:5432/appdb?sslmode=disable"
    ports:
      - "9187:9187"

  # ─── Node Exporter ───
  node-exporter:
    image: prom/node-exporter:latest
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'

  # ─── cAdvisor ───
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro

  # ─── Prometheus ───
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus/rules/:/etc/prometheus/rules/
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.retention.time=30d'
      - '--web.enable-lifecycle'
      - '--web.enable-admin-api'

  # ─── Alertmanager ───
  alertmanager:
    image: prom/alertmanager:latest
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml

  # ─── Grafana ───
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3001:3000"
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning/:/etc/grafana/provisioning/
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin123
      - GF_USERS_ALLOW_SIGN_UP=false

volumes:
  prometheus_data:
  grafana_data:
```

### App Instrumentation (app.py)

```python
from flask import Flask, jsonify
from prometheus_client import Counter, Histogram, Gauge, generate_latest, CONTENT_TYPE_LATEST
import psycopg2, time, random, os

app = Flask(__name__)

# ─── Metrics ───
REQUEST_COUNT = Counter('http_requests_total', 'Total HTTP requests',
                        ['method', 'endpoint', 'status'])
REQUEST_LATENCY = Histogram('http_request_duration_seconds', 'HTTP request duration',
                             ['endpoint'],
                             buckets=[0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5])
ACTIVE_REQUESTS = Gauge('http_active_requests', 'Active HTTP requests')
DB_QUERY_DURATION = Histogram('db_query_duration_seconds', 'DB query duration',
                               ['query_type'])

def get_db():
    return psycopg2.connect(
        host=os.getenv('DB_HOST', 'db'),
        database=os.getenv('DB_NAME', 'appdb'),
        user=os.getenv('DB_USER', 'app'),
        password=os.getenv('DB_PASSWORD', 'secret')
    )

@app.route('/api/users')
def get_users():
    start = time.time()
    ACTIVE_REQUESTS.inc()
    try:
        with get_db() as conn:
            with conn.cursor() as cur:
                db_start = time.time()
                cur.execute("SELECT count(*) FROM information_schema.tables")
                count = cur.fetchone()[0]
                DB_QUERY_DURATION.labels(query_type='select').observe(time.time() - db_start)

        REQUEST_COUNT.labels(method='GET', endpoint='/api/users', status='200').inc()
        REQUEST_LATENCY.labels(endpoint='/api/users').observe(time.time() - start)
        return jsonify({"user_count": count})
    except Exception as e:
        REQUEST_COUNT.labels(method='GET', endpoint='/api/users', status='500').inc()
        return jsonify({"error": str(e)}), 500
    finally:
        ACTIVE_REQUESTS.dec()

@app.route('/api/slow')
def slow_endpoint():
    start = time.time()
    time.sleep(random.uniform(1, 3))  # Simulate slow operation
    REQUEST_COUNT.labels(method='GET', endpoint='/api/slow', status='200').inc()
    REQUEST_LATENCY.labels(endpoint='/api/slow').observe(time.time() - start)
    return jsonify({"status": "slow response"})

@app.route('/health')
def health():
    return jsonify({"status": "ok"})

@app.route('/metrics')
def metrics():
    return generate_latest(), 200, {'Content-Type': CONTENT_TYPE_LATEST}

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### Grafana Provisioning (auto-configure datasources)

```yaml
# grafana/provisioning/datasources/prometheus.yml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    url: http://prometheus:9090
    isDefault: true
    editable: true

  - name: Alertmanager
    type: alertmanager
    url: http://alertmanager:9093
    editable: true
```

### Alert Rules for Project 1

```yaml
# prometheus/rules/app_alerts.yml
groups:
  - name: app_alerts
    rules:
      - alert: AppHighLatency
        expr: |
          histogram_quantile(0.95,
            sum by (le, endpoint) (
              rate(http_request_duration_seconds_bucket[5m])
            )
          ) > 0.5
        for: 3m
        labels:
          severity: warning
        annotations:
          summary: "High latency on {{ $labels.endpoint }}"
          description: "P95 latency is {{ $value | printf \"%.2f\" }}s"

      - alert: AppHighErrorRate
        expr: |
          sum by (endpoint) (rate(http_requests_total{status="500"}[5m])) /
          sum by (endpoint) (rate(http_requests_total[5m])) > 0.01
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High error rate on {{ $labels.endpoint }}"

      - alert: AppDown
        expr: up{job="app"} == 0
        for: 30s
        labels:
          severity: critical
        annotations:
          summary: "Application is DOWN"
```

### Simulating Failures & Troubleshooting

```bash
# 1. Simulate high CPU
stress --cpu 4 --timeout 120s

# 2. Simulate memory pressure
stress --vm 2 --vm-bytes 512M --timeout 60s

# 3. Generate 500 errors
for i in {1..100}; do
  curl -s http://localhost:5000/api/bad-endpoint
done

# 4. Generate high latency
for i in {1..50}; do
  curl -s http://localhost:5000/api/slow &
done

# 5. Kill the app (simulate service down)
docker-compose stop app

# 6. Check Alertmanager for firing alerts
curl http://localhost:9093/api/v1/alerts | jq .
```

---

## 9.2 Project 2: Unified Observability Dashboard

### Connecting All Three Pillars

```yaml
# docker-compose-full.yml — add Loki + Tempo to Project 1

  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml

  promtail:
    image: grafana/promtail:latest
    volumes:
      - /var/log:/var/log:ro
      - ./promtail.yml:/etc/promtail/config.yml
    command: -config.file=/etc/promtail/config.yml

  tempo:
    image: grafana/tempo:latest
    command: -config.file=/etc/tempo/tempo.yaml
    volumes:
      - ./tempo.yaml:/etc/tempo/tempo.yaml
    ports:
      - "3200:3200"
      - "4317:4317"
```

### Grafana Datasource Provisioning (All 3)

```yaml
# grafana/provisioning/datasources/all.yml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    url: http://prometheus:9090
    isDefault: true

  - name: Loki
    type: loki
    url: http://loki:3100
    jsonData:
      derivedFields:
        - name: TraceID
          matcherRegex: "traceID=(\\w+)"
          url: "${__value.raw}"
          datasourceUid: tempo

  - name: Tempo
    type: tempo
    url: http://tempo:3200
    uid: tempo
    jsonData:
      tracesToLogs:
        datasourceUid: loki
        tags: ['job', 'pod']
```

### Building the Unified Dashboard

```
Dashboard structure:

Row 1: SERVICE HEALTH (Prometheus)
  - Request Rate        [time series]
  - Error Rate          [stat + threshold]
  - P50/P95/P99 Latency [time series]
  - Active Connections  [gauge]

Row 2: INFRASTRUCTURE (Prometheus + Node Exporter)
  - CPU Usage %         [time series]
  - Memory Usage %      [gauge]
  - Disk I/O            [time series]
  - Network In/Out      [time series]

Row 3: LOGS (Loki)
  - Error logs          [logs panel]
  - Log volume by level [bar chart]

Row 4: TRACES (Tempo)
  - Trace search        [search panel]
  - Service map         [node graph]

Row 5: ALERTS
  - Alert list          [alert list panel]
```

---

# Quick Reference Cheat Sheet {#cheat-sheet}

## 🔑 Key Ports

| Service | Default Port |
|---|---|
| Prometheus | 9090 |
| Alertmanager | 9093 |
| Grafana | 3000 |
| Node Exporter | 9100 |
| cAdvisor | 8080 |
| Pushgateway | 9091 |
| Loki | 3100 |
| Tempo | 3200 |
| MySQL Exporter | 9104 |
| PostgreSQL Exporter | 9187 |

## 🔑 Essential PromQL Quick Reference

```promql
# CPU Usage %
100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m]))*100)

# Memory Usage %
(1-(node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes))*100

# Disk Usage %
100-(node_filesystem_avail_bytes{mountpoint="/"}/node_filesystem_size_bytes{mountpoint="/"}*100)

# HTTP Request Rate
rate(http_requests_total[5m])

# P95 Latency
histogram_quantile(0.95, sum by(le)(rate(http_request_duration_seconds_bucket[5m])))

# Error Rate %
sum(rate(http_requests_total{status=~"5.."}[5m]))/sum(rate(http_requests_total[5m]))*100

# Targets Down
up == 0

# Network RX Rate
rate(node_network_receive_bytes_total{device!="lo"}[5m])
```

## 🔑 Metric Type Decision Tree

```
What are you measuring?
│
├─ Does it only go up (count of events)?
│     └─ COUNTER  (use with rate())
│
├─ Can it go up and down (current state)?
│     └─ GAUGE
│
├─ Distribution of values (latency, size)?
│   ├─ Need cross-instance aggregation? → HISTOGRAM
│   └─ Per-instance accuracy only?      → SUMMARY
│
└─ Unsure?  → Use HISTOGRAM (more flexible)
```

## 🔑 Alert Rule Template

```yaml
- alert: AlertName
  expr: <PromQL expression returning non-empty result when condition met>
  for: <duration to confirm before firing>
  labels:
    severity: <warning|critical>
    team: <team-name>
  annotations:
    summary: "Short description {{ $labels.instance }}"
    description: "Detailed description. Value: {{ $value }}"
    runbook_url: "https://wiki.example.com/runbooks/alert-name"
```

## 🔑 Grafana Variable Template

```
Name:    job
Type:    Query
Query:   label_values(up, job)
Refresh: On Dashboard Load
```

Use in query: `{job=~"$job"}`

## 🔑 Common Pitfalls & Solutions

| Pitfall | Solution |
|---|---|
| High cardinality crash | Remove high-cardinality labels (user_id, request_id) |
| Scrape targets show DOWN | Check exporter is running; check network/firewall |
| Alerts flapping | Add/increase `for` duration in alert rule |
| Dashboard slow to load | Add recording rules for complex queries |
| Counter resets causing spikes | Always use `rate()` or `increase()` with counters |
| Missing metrics after restart | Check data retention settings; check WAL |
| Pushgateway showing stale data | Delete old metrics: `curl -X DELETE http://pushgateway:9091/metrics/job/name` |
| Alertmanager not receiving | Check `alerting:` section in prometheus.yml |

## 🔑 Interview Questions Summary

**Architecture:**
1. Explain the Prometheus pull model and its advantages.
2. What are the components of the Prometheus ecosystem?
3. How does Prometheus handle high availability?
4. What is federation in Prometheus?

**Metrics:**
5. What are the four Prometheus metric types? When do you use each?
6. What is label cardinality? Why does it matter?
7. When would you use a Histogram vs a Summary?
8. What is the difference between `rate()` and `irate()`?

**PromQL:**
9. Write a query for CPU usage percentage.
10. Write a query for P99 HTTP latency.
11. What does `sum by (job)` do?
12. What are recording rules and when do you use them?

**Alerting:**
13. Explain PENDING vs FIRING alert states.
14. What is grouping in Alertmanager?
15. What is inhibition?
16. How do you create a silence?

**Grafana:**
17. How do you connect Grafana to Prometheus?
18. What are Grafana variables?
19. What is the difference between Grafana alerting and Alertmanager?
20. What is the Grafana Explore view used for?

**Observability:**
21. What are the three pillars of observability?
22. What is the RED method?
23. What is the USE method?
24. How do you correlate metrics, logs, and traces in Grafana?

---

*End of Prometheus + Grafana Study Notes*
*Generated for: Learning, Interviews & Hands-On Practice*
