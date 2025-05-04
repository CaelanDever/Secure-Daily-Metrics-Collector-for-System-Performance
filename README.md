# 🚀 Secure Daily Metrics Collector for System Performance

## 📌 Overview

This project implements a **secure, scalable, and observable system performance metrics pipeline** designed for modern infrastructure. Using **Python**, **Docker**, **Kubernetes**, **Prometheus**, **Grafana**, and **Flask**, I built an end-to-end pipeline that collects real-time system metrics across multiple servers, applies security best practices, and enables alerting and visualization through Grafana dashboards.

This is ideal for **DevOps engineers, site reliability engineers (SREs),** and **platform engineers** who want to monitor infrastructure health in real-time with a secure and modular stack.

---

## 🏗️ Project Structure

```
secure-metrics-collector/
├── collector/
│   ├── collect_metrics.py
│   └── Dockerfile
├── flask_api/
│   ├── app.py
│   ├── requirements.txt
│   └── certs/
├── k8s/
│   ├── cronjob.yaml
│   └── network-policy.yaml
├── grafana/
│   └── dashboards/
├── prometheus/
│   └── prometheus.yml
├── README.md
└── .env (gitignored)
```

---

## 💡 30-Second Summary

System monitoring is vital for ensuring uptime, performance, and reliability in modern infrastructure. In this hands-on project, I built a secure metrics collection pipeline that runs on Kubernetes, pushing data from multiple servers to a centralized API and visualizing it with Grafana.

**Used by**: DevOps Engineers, SREs, Infrastructure Engineers

### 🔧 Skills and Tools You’ll Learn:
- 🐍 Python with `psutil` for low-level system metrics collection
- 🐳 Dockerizing scripts for portability and repeatability
- ☸️ Kubernetes CronJobs for scheduled jobs
- 🔐 Securing APIs with mutual TLS and network policies
- 📊 Building Grafana dashboards and alerts

---

## 🔍 Detailed Project Sections

### 1️⃣ Metrics Collector (`collect_metrics.py`)

```python
import psutil, json, requests

metrics = {
    "cpu_percent": psutil.cpu_percent(),
    "memory": psutil.virtual_memory()._asdict(),
    "disk": psutil.disk_usage('/')._asdict(),
    "net": psutil.net_io_counters()._asdict()
}
response = requests.post("https://internal-api/metrics", json=metrics, cert=("client.crt", "client.key"))
```

---

### 2️⃣ Dockerfile

```Dockerfile
FROM python:3.10
COPY collect_metrics.py /app/
RUN pip install psutil requests
CMD ["python", "/app/collect_metrics.py"]
```

---

### 3️⃣ Kubernetes CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: system-metrics-collector
spec:
  schedule: "*/15 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: metrics
            image: YOUR_DOCKERHUB_USERNAME/collector:latest
            volumeMounts:
              - name: certs
                mountPath: /certs
            envFrom:
              - secretRef:
                  name: collector-secrets
          volumes:
            - name: certs
              secret:
                secretName: tls-certs
          restartPolicy: OnFailure
```

---

### 4️⃣ Flask API (`app.py`)

```python
from flask import Flask, request
from your_tls_utils import require_mtls

app = Flask(__name__)

@app.route('/metrics', methods=['POST'])
@require_mtls
def receive_metrics():
    data = request.get_json()
    insert_to_timescaledb(data)
    return {"status": "success"}, 200
```

---

### 5️⃣ Prometheus Configuration

```yaml
scrape_configs:
  - job_name: 'flask-api'
    static_configs:
      - targets: ['flask-api.default.svc.cluster.local:443']
    scheme: https
    tls_config:
      ca_file: /certs/ca.crt
      cert_file: /certs/client.crt
      key_file: /certs/client.key
    basic_auth:
      username: admin
      password: yourpassword
```

---

### 🧱 Challenges & Solutions

| Challenge | Solution |
|----------|----------|
| Secure API communication | mTLS + IP filtering |
| CronJob image failed | Pushed image to Docker Hub |
| Metrics format mismatch | JSON schema validation |

---

## 🧠 Summary

### ✅ Key Learnings

- Real-world observability stack setup
- TLS-authenticated APIs
- Kubernetes workload scheduling
- Prometheus + Grafana monitoring pipeline

---

## 🔗 References

- https://psutil.readthedocs.io/
- https://prometheus.io/docs/
- https://grafana.com/docs/
- https://docs.timescale.com/

---

## 🧹 Cleanup Instructions

```bash
kubectl delete cronjob system-metrics-collector
kubectl delete deployment flask-api
kubectl delete svc flask-api
docker rmi YOUR_DOCKERHUB_USERNAME/collector:latest
psql -U postgres -d metricsdb -c "DROP TABLE system_metrics;"
```

---

## 🎉 Congratulations!

You built a:
- ✅ Secure metrics collector using Python
- ✅ Dockerized script running on Kubernetes
- ✅ TLS-protected Flask API
- ✅ Time-series backend with TimescaleDB
- ✅ Monitoring and alerting with Prometheus and Grafana
