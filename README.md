# Prometheus-Grafana-Minikube-Demo

A minimal end-to-end demo showing how to:

- Expose custom metrics from a Flask application
- Deploy the app on a local Kubernetes cluster (Minikube)
- Scrape metrics using Prometheus
- Visualize metrics using Grafana

This repository serves as a learning and reference project for setting up
local observability using Kubernetes.

---

## Project Structure

```bash
ls -R
.
├── app.py                # Flask app exposing Prometheus metrics
├── Dockerfile            # Container definition for the Flask app
├── requirements.txt      # Python dependencies
├── flask-app.yaml        # Kubernetes manifest for Flask app
├── project_flow.txt      # Step-by-step setup notes
├── LICENSE
└── README.md
```

---

## High-Level Flow

### Flask Application with Metrics

1. Build and test the Flask app locally.
2. Expose application metrics using Prometheus client libraries.

### ### Deploy Flask Application on Minikube

```bash
docker build -t flask-metrics-app:latest .
minikube start
minikube image load flask-metrics-app:latest
kubectl apply -f flask-app.yaml
minikube service flask-metrics-app --url
```

### Install Prometheus Using Helm

Verify Helm installation:

```bash
helm version
```

Add Prometheus Helm repository:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

Install Prometheus in the monitoring namespace:

```bash
helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace
```

### Access Prometheus Dashboard

List services:

```bash
kubectl get svc -n monitoring
```

Port-forward Prometheus:

```bash
kubectl port-forward -n monitoring svc/prometheus-server 9090:80
```

Open in browser: [http://localhost:9090](http://localhost:9090)

### Configure Prometheus to Scrape Flask App

Get Flask service ClusterIP:

```bash
kubectl get svc
```

Edit Prometheus ConfigMap:

```bash
kubectl edit configmap prometheus-server -n monitoring
```

Add scrape configuration:

```yaml
scrape_configs:
  - job_name: "flask-app"
    static_configs:
      - targets: ["<FLASK_SERVICE_CLUSTER_IP>:5000"]
```

Restart Prometheus (one of the following):

```bash
kubectl rollout restart deployment prometheus-server -n monitoring
```

or

```bash
kubectl delete pod -l app.kubernetes.io/name=prometheus -n monitoring
```

### Install Grafana

Add Grafana Helm repository:

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

Install Grafana:

```bash
helm install grafana grafana/grafana -n monitoring
```

Port-forward Grafana:

```bash
kubectl port-forward svc/grafana -n monitoring 3000:80
```

### Login to Grafana

Default credentials:

```text
username: admin
```

Get admin password:

```bash
kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

Access Grafana UI: [http://localhost:3000](http://localhost:3000)

### Add Prometheus as a Data Source

Use the in-cluster Prometheus service URL:

```text
http://prometheus-server.monitoring.svc.cluster.local:80
```

Navigate in Grafana:

```text
Connections → Data Sources → Add Data Source → Prometheus → Save & Test
```

---

## Current Status (Important)

⚠️ Project temporarily on hold

During setup, Minikube encountered persistent image pull failures (`ImagePullBackOff` / `ErrImagePull`) caused by DNS resolution issues when pulling images from external registries (e.g., `quay.io`).

This is an environment-specific Minikube networking issue and not a problem with the application, manifests, or Helm charts.

The repository is being uploaded in its current state for:

- Reference
- Documentation
- Future continuation using an alternative local Kubernetes setup (e.g., KIND or MicroK8s)

---

## Notes

- Intended for local development and learning purposes only
- Port-forwarding is used instead of ingress for simplicity
- Assumes Linux, Docker, Minikube, Helm, and kubectl
- `project_flow.txt` contains the original linear command log, while this README presents a cleaned, structured guide.

---

## License

MIT License
