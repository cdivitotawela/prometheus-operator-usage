# Prometheus Kubernetes Operator Usage

This repository provides information on Prometheus Kubernetes operator. 

## Installation

Set up the Promtheus and demo application. Kind will be use to run kubernetes and Helm as the Kubernetes package manager

```sh
# Create kind cluster 
kind create cluster --name prom

# Install Prometeus
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prom prometheus-community/kube-prometheus-stack -n monitoring --create-namespce

# Install demo deployment and service
kubectl create ns demo
kubectl apply -n demo -f deploy.yml
kubectl apply -n demo -f service.yml

# Verify the application
kubectl port-forward -n demo svc/demo 8080:8080
curl http://127.0.0.1:8080/metrics
```

## Service Monitor

Service monitor is a CRD installed as part of the prometheus operator. Adding service monitor helps
adding a Kubernetes service to Prometheus scrape to collect the metrics of the service. 

Prometheus is configured to look for specific label name to load the service monitors. This is configured
in the CRD Promethues. Prometheus operator only check the serviceMonitors with specific label to add into
to monitoring. 

```sh
# Get the service monitor selector value
kubectl -n monitoring get prometheus prom-kube-prometheus-stack-prometheus -o=jsonpath="{.spec.serviceMonitorSelector}"
```

Default value is `release: "prom"`. This label must be added to the serviceMonitor for it to be picked up and configured
as a service monitor. Operator seems to scan the serviceMonitor kubernetes resources every 3min, update any config changes 
and reload the prometheus update with config changes. Monitor the Prometheus Pod logs.

```sh
# Apply service-monitor.yml 
kubectl apply -n demo service-monitor.yml

# Prometheus port-forward
kubectl port-forward -n monitoring svc/prom-kube-prometheus-stack-prometheus 9090:9090
```

Check the Prometheus Pod logs to see that configuration reload. Access the Prometheus web via https://127.0.0.1:9090/
Navigate to Status->Targets and Demo app has been added as a target.

## Configure Pod Labels to be Prometheus Labels

Any Pod labels aren't included as Prometheus labels. If any of the Pod labels need to be included in Prometheus, 
the ServiceMonitor should be configured with podTargetLabels. Check the [service-monitor.yml](service-monitor.yml)
which has the configuration `podTargetLabels: ["burn"]`. This will add Pod label `burn` as a Prometheus label.



# References
- [Prometheus Operator User Guide Getting Started](https://github.com/prometheus-operator/prometheus-operator/blob/main/Documentation/user-guides/getting-started.md)
- [Service Monitor Spec](https://github.com/prometheus-operator/prometheus-operator/blob/main/Documentation/api.md#monitoring.coreos.com/v1.Prometheus)

