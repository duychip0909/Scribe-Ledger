# 📊 Grafana + Prometheus Monitoring Setup

## 🎯 Overview

Enterprise-grade monitoring for Hyra Scribe Ledger using Grafana and Prometheus. This setup provides:

- ✅ **Professional Dashboards** - Pre-configured Grafana visualizations
- ✅ **Historical Metrics** - 30 days of data retention
- ✅ **Alerting** - (Optional) Alert on key metrics
- ✅ **Multi-Node Monitoring** - Track all cluster nodes
- ✅ **Auto-Discovery** - Prometheus automatically scrapes metrics

---

## 🚀 Quick Start

### **1. Start Monitoring Stack**

```bash
cd /Users/luuhuy/Workspace/Scribe-Ledger

# Start Prometheus + Grafana
docker-compose -f docker-compose-monitoring.yml up -d
```

### **2. Start Your Cluster**

```bash
# Start all 3 nodes (they expose Prometheus metrics)
./target/release/scribe-node --bootstrap --config config-node1.toml &
./target/release/scribe-node --config config-node2.toml &
./target/release/scribe-node --config config-node3.toml &
```

### **3. Access Dashboards**

```bash
# Open Grafana in browser
open http://localhost:3000
```

**Login:**
- Username: `admin`
- Password: `admin`

**Dashboard:** Navigate to **"Hyra Scribe Ledger - Cluster Monitoring"**

---

## 📸 What You'll See

### **Dashboard Overview**

```
╔══════════════════════════════════════════════════════════════╗
║         Hyra Scribe Ledger - Cluster Monitoring             ║
╚══════════════════════════════════════════════════════════════╝

┌─────────────────┬─────────────────┬─────────────────┐
│ Healthy Nodes   │ Current Term    │ Current Leader  │
│      3          │       1         │       1         │
└─────────────────┴─────────────────┴─────────────────┘

┌────────────────────────────────┬────────────────────────────────┐
│  Request Rate by Operation     │  Request Latency (p50/p95/p99) │
│                                │                                │
│  📈 GET:  100/s                │  📊 p50: 5ms                   │
│  📈 PUT:   50/s                │  📊 p95: 15ms                  │
│  📈 DELETE: 10/s               │  📊 p99: 50ms                  │
└────────────────────────────────┴────────────────────────────────┘

┌────────────────────────────────┬────────────────────────────────┐
│  Raft Log Progress             │  Storage - Total Keys          │
│                                │                                │
│  📊 Commit Index: 1500         │  📊 Node 1: 500 keys           │
│  📊 Last Applied: 1500         │  📊 Node 2: 500 keys           │
│                                │  📊 Node 3: 500 keys           │
└────────────────────────────────┴────────────────────────────────┘

┌────────────────────────────────┬────────────────────────────────┐
│  Storage Size (Bytes)          │  Error Rate                    │
│                                │                                │
│  📊 Node 1: 50MB               │  📊 Errors/sec: 0              │
│  📊 Node 2: 50MB               │                                │
│  📊 Node 3: 50MB               │                                │
└────────────────────────────────┴────────────────────────────────┘
```

---

## 🎨 Dashboard Panels

### **1. Cluster Health Overview**

| Panel | Metric | Description |
|-------|--------|-------------|
| **Healthy Nodes** | `scribe_ledger_node_health` | Count of healthy nodes |
| **Current Raft Term** | `scribe_ledger_raft_current_term` | Current consensus term |
| **Current Leader** | `scribe_ledger_raft_current_leader` | Leader node ID |

### **2. Request Metrics**

| Panel | Metric | Description |
|-------|--------|-------------|
| **Request Rate** | `rate(scribe_ledger_requests_total[5m])` | Operations per second |
| **Request Latency** | `histogram_quantile(...)` | p50, p95, p99 latencies |

### **3. Raft Consensus**

| Panel | Metric | Description |
|-------|--------|-------------|
| **Commit Index** | `scribe_ledger_raft_commit_index` | Latest committed log entry |
| **Last Applied** | `scribe_ledger_raft_last_applied` | Latest applied log entry |

### **4. Storage Metrics**

| Panel | Metric | Description |
|-------|--------|-------------|
| **Total Keys** | `scribe_ledger_storage_total_keys` | Number of keys stored |
| **Storage Size** | `scribe_ledger_storage_size_bytes` | Storage size in bytes |

### **5. Error Tracking**

| Panel | Metric | Description |
|-------|--------|-------------|
| **Error Rate** | `rate(scribe_ledger_errors_total[5m])` | Errors per second |

---

## 🔧 Configuration

### **Prometheus Configuration** (`prometheus.yml`)

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'scribe-ledger-nodes'
    static_configs:
      - targets:
          - 'host.docker.internal:8001'  # Node 1
          - 'host.docker.internal:8002'  # Node 2
          - 'host.docker.internal:8003'  # Node 3
    metrics_path: '/metrics/prometheus'
    scrape_interval: 10s
```

**Key Settings:**
- **scrape_interval**: `10s` - How often to collect metrics
- **metrics_path**: `/metrics/prometheus` - Your Prometheus endpoint
- **targets**: All 3 node ports

### **Grafana Configuration**

**Auto-provisioned:**
- ✅ Prometheus datasource
- ✅ Pre-configured dashboard
- ✅ 10-second auto-refresh

**Credentials:**
- Username: `admin`
- Password: `admin` (change on first login)

---

## 📡 Available Metrics

### **Node Health**
```
scribe_ledger_node_health{status="ok"}
```

### **Request Counters**
```
scribe_ledger_requests_total{operation="get|put|delete"}
```

### **Request Latency (Histogram)**
```
scribe_ledger_request_duration_seconds_bucket
scribe_ledger_request_duration_seconds_sum
scribe_ledger_request_duration_seconds_count
```

### **Raft Consensus**
```
scribe_ledger_raft_current_term
scribe_ledger_raft_current_leader
scribe_ledger_raft_commit_index
scribe_ledger_raft_last_applied
```

### **Storage**
```
scribe_ledger_storage_total_keys
scribe_ledger_storage_size_bytes
```

### **Errors**
```
scribe_ledger_errors_total
```

---

## 🎯 Common Use Cases

### **1. Monitor Cluster Health**

**Query in Grafana:**
```promql
count(scribe_ledger_node_health{status="ok"})
```

**Alert Rule:**
```yaml
- alert: ClusterUnhealthy
  expr: count(scribe_ledger_node_health{status="ok"}) < 2
  for: 5m
  annotations:
    summary: "Less than 2 healthy nodes"
```

### **2. Track Request Rate**

**Query:**
```promql
sum(rate(scribe_ledger_requests_total[5m])) by (operation)
```

### **3. Monitor Latency**

**Query:**
```promql
histogram_quantile(0.95, rate(scribe_ledger_request_duration_seconds_bucket[5m]))
```

### **4. Check Raft Lag**

**Query:**
```promql
scribe_ledger_raft_commit_index - scribe_ledger_raft_last_applied
```

---

## 🚨 Setting Up Alerts (Optional)

### **1. Create Alert Rules File**

Create `alerts.yml`:

```yaml
groups:
  - name: scribe_ledger
    interval: 30s
    rules:
      - alert: NodeDown
        expr: up{job="scribe-ledger-nodes"} == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Node {{ $labels.instance }} is down"
          description: "Node has been down for more than 2 minutes"

      - alert: HighErrorRate
        expr: rate(scribe_ledger_errors_total[5m]) > 10
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High error rate on {{ $labels.instance }}"
          description: "Error rate is {{ $value }} errors/sec"

      - alert: HighLatency
        expr: histogram_quantile(0.95, rate(scribe_ledger_request_duration_seconds_bucket[5m])) > 0.5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency on {{ $labels.instance }}"
          description: "P95 latency is {{ $value }}s"

      - alert: LeaderChanged
        expr: changes(scribe_ledger_raft_current_leader[5m]) > 0
        for: 1m
        labels:
          severity: info
        annotations:
          summary: "Raft leader changed"
          description: "New leader is node {{ $value }}"
```

### **2. Update Prometheus Config**

In `prometheus.yml`:

```yaml
rule_files:
  - "alerts.yml"
```

### **3. Setup Alertmanager (Optional)**

```yaml
# docker-compose-monitoring.yml
alertmanager:
  image: prom/alertmanager:latest
  ports:
    - "9093:9093"
  volumes:
    - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
```

---

## 🔍 Advanced Queries

### **Request Throughput by Node**
```promql
sum(rate(scribe_ledger_requests_total[5m])) by (instance)
```

### **Average Request Duration**
```promql
rate(scribe_ledger_request_duration_seconds_sum[5m]) /
rate(scribe_ledger_request_duration_seconds_count[5m])
```

### **Storage Growth Rate**
```promql
rate(scribe_ledger_storage_total_keys[1h])
```

### **Error Ratio**
```promql
rate(scribe_ledger_errors_total[5m]) /
rate(scribe_ledger_requests_total[5m])
```

---

## 🛠️ Troubleshooting

### **Prometheus Not Scraping?**

**Check targets:**
```bash
# Open Prometheus UI
open http://localhost:9090/targets

# Should show all 3 nodes as UP
```

**Fix:**
```bash
# Verify nodes are running
curl http://localhost:8001/metrics/prometheus
curl http://localhost:8002/metrics/prometheus
curl http://localhost:8003/metrics/prometheus
```

### **Grafana Shows "No Data"?**

**Checks:**
1. Is Prometheus configured in Grafana? (Should be auto-provisioned)
2. Is data source working? (Test in Configuration → Data Sources)
3. Are nodes exposing metrics? (Check Prometheus targets)

**Fix:**
```bash
# Restart monitoring stack
docker-compose -f docker-compose-monitoring.yml restart

# Check logs
docker-compose -f docker-compose-monitoring.yml logs -f grafana
docker-compose -f docker-compose-monitoring.yml logs -f prometheus
```

### **Dashboard Not Auto-Loading?**

**Check provisioning:**
```bash
# Verify files exist
ls -la grafana/provisioning/datasources/
ls -la grafana/provisioning/dashboards/
ls -la grafana/dashboards/
```

**Reload:**
```bash
# Restart Grafana
docker-compose -f docker-compose-monitoring.yml restart grafana
```

---

## 📊 Production Deployment

### **1. Security Hardening**

**Change Default Password:**
```yaml
# docker-compose-monitoring.yml
environment:
  - GF_SECURITY_ADMIN_PASSWORD=YOUR_SECURE_PASSWORD
```

**Enable Authentication:**
```yaml
# prometheus.yml
basic_auth:
  username: prometheus
  password: YOUR_PASSWORD
```

### **2. Data Retention**

**Prometheus:**
```yaml
command:
  - '--storage.tsdb.retention.time=90d'  # 90 days
  - '--storage.tsdb.retention.size=50GB'  # Max size
```

**Grafana:**
- Data stored in PostgreSQL/MySQL for persistence

### **3. High Availability**

**Multiple Prometheus Instances:**
```yaml
# Use Thanos or Cortex for HA Prometheus
```

**Grafana HA:**
```yaml
# Use shared database (PostgreSQL)
# Load balancer in front
```

---

## 🎨 Customizing Dashboards

### **Add New Panel**

1. Open Grafana → Your Dashboard
2. Click "Add panel"
3. Enter PromQL query
4. Configure visualization
5. Save dashboard

### **Example: Add CPU Usage**

**If you add CPU metrics to your app:**
```promql
rate(process_cpu_seconds_total[5m])
```

### **Export/Import Dashboards**

**Export:**
1. Dashboard Settings → JSON Model
2. Copy JSON
3. Save to file

**Import:**
1. Create → Import
2. Paste JSON or Upload file

---

## 📚 Resources

### **Prometheus Documentation**
- Query Language: https://prometheus.io/docs/prometheus/latest/querying/basics/
- Best Practices: https://prometheus.io/docs/practices/naming/

### **Grafana Documentation**
- Dashboards: https://grafana.com/docs/grafana/latest/dashboards/
- Variables: https://grafana.com/docs/grafana/latest/variables/

### **Community Dashboards**
- Explore: https://grafana.com/grafana/dashboards/

---

## 🎉 Quick Command Reference

```bash
# Start monitoring stack
docker-compose -f docker-compose-monitoring.yml up -d

# Stop monitoring stack
docker-compose -f docker-compose-monitoring.yml down

# View logs
docker-compose -f docker-compose-monitoring.yml logs -f

# Access Grafana
open http://localhost:3000

# Access Prometheus
open http://localhost:9090

# Check Prometheus targets
open http://localhost:9090/targets

# Reload Prometheus config
curl -X POST http://localhost:9090/-/reload
```

---

## ✅ Complete Setup Checklist

- [ ] Docker and Docker Compose installed
- [ ] Created `docker-compose-monitoring.yml`
- [ ] Created `prometheus.yml`
- [ ] Created Grafana provisioning files
- [ ] Created dashboard JSON
- [ ] Started monitoring stack
- [ ] Started Scribe Ledger nodes
- [ ] Accessed Grafana at http://localhost:3000
- [ ] Verified dashboard loads
- [ ] Checked Prometheus targets are UP
- [ ] Confirmed metrics are being collected

---

<div align="center">

**Enterprise Monitoring Complete!** 📊

**Grafana:** `http://localhost:3000`  
**Prometheus:** `http://localhost:9090`

🔥 **Hyra Scribe Ledger** 🔥

</div>

