
#  Kube_monitoring-Logging_stack

### Observability Stack — Loki, Promtail, Grafana, MinIO

This repository provides a full observability stack for Kubernetes-based CTF (Capture The Flag) environments. It captures, stores, and visualizes logs from all pods across all namespaces using:

- 🔸 **Promtail** — log collector (runs as DaemonSet on each node)
- 🔸 **Loki** — scalable log aggregation and storage
- 🔸 **Grafana** — dashboard for querying and visualizing logs
- 🔸 **MinIO** — object storage backend used by Loki

---

##  Components & Architecture

```
[PODS + CONTAINERS LOGS]
         ↓
   [Promtail on each node]
         ↓
       [Loki Gateway]
         ↓
[MinIO (Loki backend storage)]
         ↓
     [Grafana Dashboards]
```

---

##  Steps to Deploy

### 1. Clone the Repo

```bash
git clone https://github.com/<your-username>/ctf-observability.git
cd ctf-observability
```

---

### 2. Create the Namespace

```bash
kubectl create namespace observer
```

---

### 3. Deploy Loki (using MinIO as backend)

```bash
cd loki
helm repo add grafana https://grafana.github.io/helm-charts
helm upgrade --install loki grafana/loki-distributed -n observer -f values.yaml
```

---

### 4. Deploy Promtail (DaemonSet on all nodes)

```bash
cd ../promtail
helm upgrade --install promtail grafana/promtail -n observer -f values.yaml
```

 **Verify Promtail is running on all nodes:**

```bash
kubectl get pods -n observer -l app.kubernetes.io/name=promtail -o wide
```

 **Test ingestion metrics:**

```bash
kubectl --namespace observer port-forward daemonset/promtail 3101
curl http://127.0.0.1:3101/metrics | grep log_lines_total
```

---

### 5. Deploy Grafana

```bash
cd ../grafana
helm upgrade --install grafana grafana/grafana -n observer -f values.yaml
```

> Default credentials:
> - **Username:** `admin`
> - **Password:** `prom-operator`

 **Expose Grafana (optional if not using Ingress):**

```bash
kubectl port-forward svc/grafana 3000:80 -n observer
```

 Open in browser: [http://localhost:3000](http://localhost:3000)

---

### 6. Connect Loki to Grafana

1. Go to **Grafana → Settings → Data Sources**
2. Click **Add data source**
3. Choose **Loki**
4. Set URL to:
   ```
   http://loki-distributed-gateway/loki
   ```
5. Click **Save & Test**

---

### 7. Explore Logs in Grafana

- Go to **Explore** section in Grafana
- Start querying logs:
  - By namespace
  - By pod name
  - By container
  - By log level (`info`, `warn`, `error`, etc.)

---

##  Folder Structure

```
ctf-observability/
├── grafana/
│   └── values.yaml
├── loki/
│   └── values.yaml
├── promtail/
│   └── values.yaml
└── README.md
```

---

##  Tips

- Promtail is installed as a **DaemonSet**, so it runs on every node.
- Promtail collects logs from `/var/log/pods/*/*/*.log`
- CRI parsing is enabled for containerd logs.
- Tolerations are applied so Promtail runs on control-plane nodes too.
- Loki stores compressed log chunks in **MinIO**, make sure the S3 config is correct.
- Use `logcli` or Grafana Explore UI for advanced queries.
