# 🚀 Secure Daily Metrics Collector for System Performance

## 📌 Overview

This project implements a **secure, scalable, and observable system performance metrics pipeline** designed for modern infrastructure. Using **Python**, **Docker**, **Kubernetes**, **Prometheus**, **Grafana**, and **Flask**, I built an end-to-end pipeline that collects real-time system metrics across multiple servers, applies security best practices, and enables alerting and visualization through Grafana dashboards.

<img width="352" alt="dg" src="https://github.com/user-attachments/assets/91443ac8-cdea-418e-b8aa-50366a99ab45" />


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
What it does:
Gathers system performance metrics (CPU, memory, disk, network) using the psutil Python module.
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
📌 Screenshot Placeholder: Metrics being printed/logged

<img width="445" alt="colle" src="https://github.com/user-attachments/assets/59c78ba3-a1be-4b9f-8054-c19c5960ce5c" />


<img width="300" alt="collect" src="https://github.com/user-attachments/assets/6588cf92-28c8-4656-aaeb-47d06c111f78" />


<img width="461" alt="mwtric" src="https://github.com/user-attachments/assets/ef92bca5-8669-4a91-9113-7d7474000670" />

### 2️⃣ Dockerfile
Dockerfile Example:

```Dockerfile
FROM python:3.10
COPY collect_metrics.py /app/
RUN pip install psutil requests
CMD ["python", "/app/collect_metrics.py"]
```

---
📌 Screenshot Placeholder: Docker image built and tagged

<img width="251" alt="DockerF" src="https://github.com/user-attachments/assets/d40531fc-01a7-4cd8-a492-a8d42b8ed680" />


<img width="463" alt="docor" src="https://github.com/user-attachments/assets/b0bee35c-9257-4a72-849b-9950ece2f26e" />



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
📌 Screenshot Placeholder: CronJob showing successful execution in Kubernetes dashboard

<img width="463" alt="kubnow" src="https://github.com/user-attachments/assets/3a6a7409-4778-4096-8742-ca73666b2381" />


<img width="464" alt="kubur" src="https://github.com/user-attachments/assets/8c278f9d-2727-420b-8f47-01a738eed342" />


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


Security Measures:

✅ Mutual TLS for API calls

<img width="459" alt="mlts" src="https://github.com/user-attachments/assets/02014bed-d60d-43e5-84ef-6821308e0c00" />

✅ IP whitelisting in Flask middleware

✅ Basic auth on Prometheus endpoints

✅ Kubernetes Network Policies

<img width="413" alt="netpo" src="https://github.com/user-attachments/assets/4f18b90a-17eb-4ffd-9fb6-53997dba755d" />

<img width="464" alt="netpol" src="https://github.com/user-attachments/assets/af347d84-e392-4f31-8f7c-e1a0db468581" />

🔐 Secure Flask API for Metrics Ingestion
We used mutual TLS to secure communication between the collector and Flask API.

📌 Client Certificate Generation

<img width="332" alt="cert2" src="https://github.com/user-attachments/assets/c07fff59-447c-4b18-9bc2-06913991f2d6" />

<img width="358" alt="certs" src="https://github.com/user-attachments/assets/a5ef94a5-a038-4821-a96d-4d4b2c48d81f" />

### 5️⃣  TimescaleDB Integration

PostgreSQL + TimescaleDB extension for efficient time-series storage.

Dockerized and deployed in-cluster or externally.

📌 Screenshot Placeholder: Sample rows in TimescaleDB from psql

<img width="464" alt="timescale" src="https://github.com/user-attachments/assets/3364bfbd-b406-4ac2-9121-9d9c27534ec7" />

<img width="409" alt="sqldb" src="https://github.com/user-attachments/assets/5cf0c16a-09b2-43ef-a936-385797b70a2b" />


### 6️⃣ Prometheus & Grafana

Prometheus config points to Flask API scrape endpoint

Grafana dashboards for:

📈 CPU usage

💾 Memory consumption

🔌 Network stats

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
📌 Screenshot Placeholder: Grafana dashboard

<img width="462" alt="lgraf" src="https://github.com/user-attachments/assets/9dd82cb6-c5f6-4751-a482-9a47cdec8fa9" />

<img width="440" alt="graf" src="https://github.com/user-attachments/assets/8376c815-72e9-4056-926b-64c7978ca405" />


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
