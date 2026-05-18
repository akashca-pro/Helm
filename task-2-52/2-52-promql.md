# Task 2.52 - PromQL Queries Documentation

## 7. Executed PromQL Queries

### 1. CPU usage rate per node (all cores, excluding idle)
**Query:**
```promql
rate(node_cpu_seconds_total{mode!="idle"}[5m])
```

**What it computes:**  
Calculates the per-second rate of CPU time consumed (excluding idle time), broken down by each CPU core and mode.

**Observed Result:**  
Multiple time series returned (one for each CPU core and non-idle mode such as `user`, `system`, `iowait`, etc.) on instance `192.168.1.28:9100`.

---

### 2. Total CPU usage percentage per node
**Query:**
```promql
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

**What it computes:**  
Computes the overall CPU usage percentage per node by subtracting the average idle CPU time from 100%.

**Observed Result:**  
6.114814814807474 %
### 3. Memory used per node (bytes)
**Query:**
```promql
node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes
```

**What it computes:**  
Shows the absolute amount of memory currently being used on the node in bytes.

**Observed Result:**  
3689582592 bytes

---

### 4. Memory usage percentage per node
**Query:**
```promql
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
```

**What it computes:**  
Calculates the percentage of total memory that is currently in use on the node.

**Observed Result:**  
23.089047643932382 %

---

### 5. Total number of running pods per namespace
**Query:**
```promql
count by (namespace) (kube_pod_info)
```

**What it computes:**  
Counts the total number of pods grouped by each namespace.

**Observed Result:**  
{namespace="devanarayanan-r"}	21
{namespace="akash-c-a-monitoring"}	8
{namespace="akash-c-a"}	9
{namespace="kube-system"}	6
{namespace="default"}	2
{namespace="cert-manager"}	3
{namespace="ingress-nginx"}	1

---

### 6. API server request rate by HTTP verb
**Query:**
```promql
sum(rate(apiserver_request_total[5m])) by (verb)
```

**What it computes:**  
Shows the rate of Kubernetes API server requests per second, grouped by HTTP verb (GET, POST, PUT, DELETE, etc.).

**Observed Result:**  
{verb="CONNECT"}	0
{verb="GET"}	1.5740740740740742
{verb="LIST"}	0.1111111111111111
{verb="WATCH"}	0.7555555555555556
{verb="DELETE"}	0
{verb="PUT"}	0.9592592592592593
{verb="PATCH"}	0.07407407407407407
{verb="APPLY"}	0
{verb="POST"}	0.15555555555555556

---

### 7. P99 latency of API server requests (seconds)
**Query:**
```promql
histogram_quantile(0.99, sum(rate(apiserver_request_duration_seconds_bucket[5m])) by (le, verb))
```

**What it computes:**  
Calculates the 99th percentile (P99) latency of API server requests in seconds, broken down by HTTP verb.

**Observed Result:**  
{verb="GET"}	0.04801562500000002
{verb="LIST"}	4.67
{verb="WATCH"}	60
{verb="APPLY"}	NaN
{verb="DELETE"}	NaN
{verb="PUT"}	0.04845000000000004
{verb="PATCH"}	0.8200000000000006
{verb="CONNECT"}	NaN
{verb="POST"}	0.07800000000000017

---

### 8. API server error rate (4xx and 5xx only)
**Query:**
```promql
sum(rate(apiserver_request_total{code=~"4..|5.."}[5m])) by (code)
```

**What it computes:**  
Shows the rate of error responses (HTTP 4xx and 5xx status codes) from the API server.

**Observed Result:**  
{code="403"}	0
{code="404"}	0
{code="409"}	0
{code="422"}	0
{code="429"}	0
{code="500"}	0
{code="504"}	0

---

## 8. Aggregation Query Modification

**Original Query:**
```promql
count by (namespace) (kube_pod_info)
```

**Modified Query (without `by` clause):**
```promql
count(kube_pod_info)
```
**Observed Result:**  
{}	50

**Explanation:**  
When we use `count by (namespace)`, Prometheus groups the results and returns the pod count **separately for each namespace**.  

After removing the `by (namespace)` clause, Prometheus aggregates **all series** into a single value, giving the **total number of pods across the entire cluster**.  

This demonstrates how the `by` clause controls the grouping / aggregation level in PromQL.

---

## 9. Target Failure Analysis (Status → Targets)

### Observed Target Status
I navigated to **Status → Targets** in Prometheus UI.

**One target is DOWN:**

- **Job**: `serviceMonitor/akash-c-a-monitoring/akash-prometheus-stack-grafana/0`
- **Target**: `http://10.42.0.52:3000/metrics`
- **Pod**: `akash-prometheus-stack-grafana-dd4bcdb96-spv64`
- **State**: **DOWN**
- **Last Error**: `Get "http://10.42.0.52:3000/metrics": dial tcp 10.42.0.52:3000: connect: connection refused`

All other targets (Prometheus, Alertmanager, node-exporter, kubelet, etc.) are **UP**.

---

### What happens to metric values when a target goes down?

When a target goes down:
- Prometheus can no longer scrape metrics from it.
- The special metric **`up`** for that target becomes **`0`**.
- All other metrics from that target **stop updating** and eventually become **stale**.
- In graphs and queries, these metrics will either disappear or return no data after some time.
- This leads to gaps in dashboards and can trigger alerts.

---

### PromQL Alert Expression to detect a target being down

**Basic Expression:**
```promql
up == 0