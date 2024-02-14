
# LOGGING & MONITORING INFRASTRUCTURES

This README outlines the setup process for both monitoring and logging infrastructures using various tools like Prometheus, Grafana, and Loki.

## Monitoring Setup

The monitoring setup utilizes the kube-prometheus-stack. More details can be found on their [GitHub page](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack).

### Kube-Prometheus-Stack Installation

To install the kube-prometheus-stack, follow these steps:

```bash
# Create a namespace for monitoring
kubectl create ns monitoring

# Customize values.yaml as per your requirements

# Install Prometheus operator, Prometheus, Grafana, Alertmanager, NodeExporter
helm upgrade --install promstack-dev --version 55.5.0 prometheus-community/kube-prometheus-stack -f monitoring/monitoring.values.yaml -n monitoring

# Access Grafana for testing the installation (use this if the Grafana DNS record does not exist)
kubectl -n monitoring port-forward svc/promstack-dev-grafana 8080:80

# Access localhost:8080 to manage Grafana
# Default user/password for Grafana: admin/prom-operator
```

### Custom Scrape Configuration

For setting up a custom scrape configuration, follow these instructions:

```bash
# Create a ServiceMonitor for all customer-developed services
kubectl apply -f monitoring/servicemonitor.yaml

# Check the ServiceMonitor from the Prometheus UI (use this if the Prometheus DNS record does not exist)
kubectl -n monitoring port-forward svc/prom-stack-dev-prometheus 9090:9090

# Access localhost:9090 to manage Prometheus
# After deploying ServiceMonitor, verify the targets from the Prometheus UI
```

## Logging Infrastructure

The logging infrastructure comprises two main components:

1. **Banzai Cloud Logging Operator**: Used for collecting log records and sending them to log storage. [Reference Architecture](https://kube-logging.dev/docs/)

2. **Grafana Loki**: Chosen for storing and querying logs. [Example Architecture](https://grafana.com/docs/loki/latest/get-started/deployment-modes/#simple-scalable). A simple/scalable architecture is used for the setup.

### Logging Operator Installation

To install the logging operator, use the following commands:

```bash
# Create a namespace for logging
kubectl create ns logging

# Install the logging operator using Helm chart
helm upgrade --install --wait --create-namespace --namespace logging logging-operator oci://ghcr.io/kube-logging/helm-charts/logging-operator
```

### Grafana Loki Installation

For installing Grafana Loki, follow these steps:

```bash
# Create an Azure storage account as a blob storage

# Create admin, chunks, and ruler containers in this storage account

# Create PrivateLink for the Azure storage account and create a private DNS record in the hub network

# Customize Loki's values.yaml to use the Azure storage account as a persistent volume

# Customize Loki's values.yaml to update the Loki-gateway Nginx reverse proxy client max body size

# Install Grafana Loki
helm upgrade --install --create-namespace --namespace logging loki grafana/loki -f logging/loki.values.yaml
```

### Customize Banzai Logging Operator with CRDs

Customize the Banzai Logging Operator with CRDs using the following commands:

```bash
# Create a logging CRD for Fluent Bit and Fluentd configuration
kubectl apply -f logging/logging.yaml

# Create ClusterFlow and ClusterOutput CRDs to collect logs from Kubernetes and push logs to Loki
kubectl apply -f logging/clusterflow.yaml
kubectl apply -f logging/clusteroutput.yaml

# Verify the ClusterFlow and ClusterOutput
kubectl get clusterflow,clusteroutput

# Check Fluentd container logs
kubectl logs -f default-logging-simple-fluentd-0

# Add Loki datasource to Grafana
```

This documentation provides a comprehensive guide for setting up monitoring and logging infrastructures using various Kubernetes tools.
