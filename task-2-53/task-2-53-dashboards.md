# Task 2.53 — Exploring Pre-built Grafana Dashboards

## Part 1 — Verify Dashboards Are Loaded

- Successfully logged into Grafana at `https://grafana.codexakash.space` using `admin` / `prom-operator`.
- Multiple pre-built dashboards are available, organized into folders such as **Kubernetes**, **Node Exporter**, **Alertmanager**, etc.

**Command executed:**

```bash
kubectl get configmap -n monitoring -l grafana_dashboard=1
```

**Observation:** Several ConfigMaps containing dashboard definitions are present.

```text
akash-prometheus-stack-kub-alertmanager-overview               1      7h7m
akash-prometheus-stack-kub-apiserver                           1      7h25m
akash-prometheus-stack-kub-cluster-total                       1      7h25m
akash-prometheus-stack-kub-controller-manager                  1      7h25m
akash-prometheus-stack-kub-etcd                                1      7h25m
akash-prometheus-stack-kub-grafana-overview                    1      7h25m
akash-prometheus-stack-kub-k8s-coredns                         1      7h25m
akash-prometheus-stack-kub-k8s-resources-cluster               1      7h25m
akash-prometheus-stack-kub-k8s-resources-multicluster          1      7h25m
akash-prometheus-stack-kub-k8s-resources-namespace             1      7h25m
akash-prometheus-stack-kub-k8s-resources-node                  1      7h25m
akash-prometheus-stack-kub-k8s-resources-pod                   1      7h25m
akash-prometheus-stack-kub-k8s-resources-workload              1      7h25m
akash-prometheus-stack-kub-k8s-resources-workloads-namespace   1      7h25m
akash-prometheus-stack-kub-kubelet                             1      7h25m
akash-prometheus-stack-kub-namespace-by-pod                    1      7h25m
akash-prometheus-stack-kub-namespace-by-workload               1      7h25m
akash-prometheus-stack-kub-node-cluster-rsrc-use               1      76m
akash-prometheus-stack-kub-node-rsrc-use                       1      76m
akash-prometheus-stack-kub-nodes                               1      76m
akash-prometheus-stack-kub-nodes-aix                           1      76m
akash-prometheus-stack-kub-nodes-darwin                        1      76m
akash-prometheus-stack-kub-persistentvolumesusage              1      7h25m
akash-prometheus-stack-kub-pod-total                           1      7h25m
akash-prometheus-stack-kub-prometheus                          1      7h25m
akash-prometheus-stack-kub-proxy                               1      7h25m
akash-prometheus-stack-kub-scheduler                           1      7h25m
akash-prometheus-stack-kub-workload-total                      1      7h25m
```

**How dashboards get into Grafana without manual import:**

The kube-prometheus-stack uses the **Grafana Sidecar** pattern. ConfigMaps with the label `grafana_dashboard=1` are automatically discovered and provisioned into Grafana by the sidecar container.

---

## Part 2 — Node Exporter / USE Method / Node

**Dashboard:** Node Exporter / USE Method / Node

**Instance Selected:** `192.168.1.28:9100`

**Key Panels & Queries:**

**CPU Utilisation:** `instance:node_cpu_utilisation:rate5m{job="node-exporter",instance="$instance",cluster="$cluster"} != 0` (Counter)

**Memory Utilisation:** Computed from total memory minus available memory.

**Disk IO:** Powered by `node_disk_read_bytes_total` and `node_disk_written_bytes_total`.

**Network Traffic:** Filtered using device label.

**System Load:** Exposed via `node_load1`, `node_load5`, `node_load15`.

**Time Series Question:**

If the cluster had 3 nodes, `node_cpu_seconds_total` would produce approximately **3 × (number of CPU cores) × (number of CPU modes)** time series (e.g., **~240 series for 8-core nodes with 10 modes**).

---

## Part 3 — Kubernetes / Compute Resources

### 3.1 Kubernetes / Compute Resources / Cluster

**Observations:**

CPU Utilisation: **2.89%**

CPU Requests Commitment: **149%**

CPU Limits Commitment: **177%**

Memory Utilisation: **22.3%**

Memory Requests Commitment: **99.6%**

Memory Limits Commitment: **126%**

**High usage namespaces:**

akash-c-a-monitoring: **0.279 CPU** and **1.62 GiB Memory** (dominant consumer)

---

### 3.2 Kubernetes / Compute Resources / Namespace (Pods)

**Namespace Selected:** `akash-c-a`

**Observations:**

CPU Utilisation (from requests): **1.46%**

CPU Utilisation (from limits): **0.292%**

Memory Utilisation (from requests): **12.4%**

Memory Utilisation (from limits): **6.18%**

**Pod using significantly less CPU than requested:**

task-2-36-deployment-6b8ccc6695-42n8q (**CPU request = 0.100, usage ≈ 0**)

**CPU Throttling:**

CPU throttling occurs when a container exceeds its CPU limit. Kubernetes (via Linux CFS scheduler) throttles the container by temporarily pausing it. This is measured using `container_cpu_cfs_throttled_seconds_total`.

**Container Identification:**

Grafana identifies individual containers inside a pod using the **container** label.

---

### 3.3 Kubernetes / Compute Resources / Node (Pods)

**Node Selected:** `k3s-kubenine-intern-3048-ed140f-node-pool-0576-hksa6`

Observed many pods with their CPU and Memory usage (with/without cache). Prometheus and Grafana are among the highest consumers.

---

## Part 4 — Kubernetes / Networking

### 4.1 Kubernetes / Networking / Cluster

**Current Rate:**

akash-c-a-monitoring shows the highest network traffic:

**8.71 Mb/s Received** and **1.52 Mb/s Transmitted**

---

### 4.2 Kubernetes / Networking / Namespace (Pods) & Pod

**Namespace:** `kube-system`

**Pod:** `metrics-server-7b9c9c4b9c-fqmsz`

Current Receive Bandwidth: **96.7 kB/s**

Current Transmit Bandwidth: **115 kB/s**

**Main Metric:** `container_network_receive_bytes_total` and `container_network_transmit_bytes_total` (filtered by namespace and pod).

**Why useful in production?**

This dashboard helps quickly identify noisy pods or namespaces during high network traffic incidents (e.g., log flooding, scraping storms, or external attacks).

---

## Part 5 — Alertmanager Dashboard

**Dashboard:** Alertmanager / Overview

**Namespace:** `akash-c-a-monitoring`

**Top Stat Panels:**

**Alerts Received** — Total alerts ingested by Alertmanager.

**Notifications Send Rate** — Shows send rate for various receivers (Slack, Discord, Email, Webhook, PagerDuty, etc.).

**Notification Latency** — Time taken to deliver notifications.

**Current Status:**

10.42.0.59:9093 → **13 alerts**

10.42.0.63:9093 → **13 alerts**

**When an alert is active:**

These panels would show increased **Alerts Received**, notification send rates, and alert activity across replicas.

---

## Part 6 — Identify Gaps

After exploring all pre-built dashboards, the following gaps were identified:

- No application/team-specific scoped dashboard (e.g., only akash-c-a namespace with business metrics).

- No combined view of resource metrics + Loki log volume/rate.

- No single pane showing Resource Usage + Pod Restarts + OOMKilled events together.

- Missing golden signals (Latency, Error Rate, QPS/Throughput) for custom applications.