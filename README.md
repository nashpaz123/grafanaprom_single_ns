# Prometheus and Grafana Setup for Kubernetes

This guide outlines the implementation of Prometheus scraping and exporting metrics from a single namespace on Kubernetes, along with Grafana for visualization. An example dashboard is included to display the metrics.

## Prerequisites

- Kubernetes cluster
- `kubectl` installed and configured

## Setup Instructions

### 1. Install Helm

```bash
wget https://get.helm.sh/helm-v3.14.4-linux-amd64.tar.gz
tar -zxvf helm-v3.14.4-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm
```

### 2. Add Helm Repositories

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

### 3. Set Environment Variables and Aliases

```bash
export NAMESPACE=<your-namespace-name>
alias k=kubectl
alias kk="kubectl -n $NAMESPACE"
```

### 4. Create Namespace

```bash
kubectl create namespace $NAMESPACE || true
```

### 5. Clone and Configure Repository

```bash
git clone https://github.com/nashpaz123/grafanaprom_single_ns.git
cd grafanaprom_single_ns

# Update configuration files
sed -i "s/app-namespace/$NAMESPACE/g" servicemonitor.yaml
sed -i "s/app-namespace/$NAMESPACE/g" prometheus-values.yaml
sed -i "s/app-namespace/$NAMESPACE/g" grafana-values.yaml
```

### 6. Deploy Components

```bash
kubectl apply -f servicemonitor.yaml

helm install prometheus-$NAMESPACE prometheus-community/prometheus \
  --namespace $NAMESPACE \
  --version 25.22.1 \
  --values prometheus-values.yaml

helm install grafana grafana/grafana \
  --namespace $NAMESPACE \
  --version 8.3.2 \
  --values grafana-values.yaml
```

## Notes

- Prometheus takes about 1 minute to load the server and an additional 2 minutes to load pod data.
- To expose Prometheus and Grafana services (optional):

  ```bash
  kubectl expose deployment prometheus-server -n $NAMESPACE --type=NodePort --name promnport
  kubectl get svc -n $NAMESPACE | grep promnport

  kubectl expose deployment grafana -n $NAMESPACE --type=NodePort --name grafananport
  kubectl get svc -n $NAMESPACE | grep grafananport
  ```

## Example Query

A simple query with a specific namespace can be set, e.g.:

```
sum(rate(container_cpu_usage_seconds_total{namespace="kube-system"}[5m])) by (pod)
```

This example shows all pods available from Prometheus.
