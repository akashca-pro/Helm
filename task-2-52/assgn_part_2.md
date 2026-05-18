## Prometheus Metrics Analysis

### 1. `node_cpu_seconds_total`
- **Metric Type**: **Counter**
- **Explanation**: This metric measures the total cumulative CPU time (in seconds) consumed by the node, broken down by different CPU modes (user, system, idle, iowait, etc.). Use `rate()` function to calculate actual CPU usage percentage.

---

### 2. `node_memory_MemAvailable_bytes`
- **Metric Type**: **Gauge**
- **Explanation**: This metric shows the current amount of memory (in bytes) that is available for starting new applications on the node without requiring swapping.

---

### 3. `kube_pod_info`
- **Metric Type**: **Gauge**
- **Explanation**: This is an informational gauge metric (value is always `1`) that provides metadata about running pods such as pod name, namespace, node name, labels, and container image for discovery and joining with other metrics.

---

### 4. `apiserver_request_duration_seconds_bucket`
- **Metric Type**: **Histogram**
- **Explanation**: This histogram metric records the duration of requests to the Kubernetes API server in seconds, divided into buckets. It is used to calculate latency percentiles (p50, p95, p99, etc.).

---

### 5. `go_gc_duration_seconds`
- **Metric Type**: **Summary**
- **Explanation**: This summary metric measures the time taken by the Go runtime garbage collector (GC) pauses, providing quantile values (e.g., 50th, 95th, 99th percentile) of garbage collection stop-the-world duration.

---