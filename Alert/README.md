# Setting Up Alerts for NDK

Welcome to the guide for setting up alerts for NDK. This document will walk you through the process of configuring alerts to ensure proactive monitoring and timely notification of any issues or anomalies within your NDK deployment.
</br>
</br>

## Prerequisites

Before proceeding with setting up alerts, ensure that the Prometheus Operator is installed in your Kubernetes cluster. The Prometheus Operator is essential for managing Custom Resource Definitions (CRDs) such as PrometheusRules and ServiceMonitors, which are utilized for generating alerts and scraping metrics from the Kubernetes cluster. In this documentation, we assume that the Prometheus Operator has been installed in the ntnx-system namespace of your Kubernetes cluster. Please ensure to use the appropriate namespace relevant to your cluster when executing the provided commands.
</br>
</br>

## Configuring Prometheus-Operator

In order to view the alerts raised, it's necessary to access the Prometheus UI. This can be achieved by exposing the prometheus-operated service present in the ntnx-system namespace of the cluster. Below are the steps to expose the service and access the Prometheus UI:

</br>

To expose the prometheus-operated service, execute the following command:
```kubectl
kubectl -n ntnx-system expose svc prometheus-operated --type=NodePort --target-port=9090 --name=prometheus-operated-ext
```
</br>

You can access the Prometheus-UI by using the following format: node-IP:Port.

* To obtain the Node-IP, execute the following command:
```kubectl
kubectl -n ntnx-system get nodes -o wide
```
* You can find the exposed port in the port section of the prometheus-operated-ext service.
</br>

With the Node IP and the exposed port, you can now access the Prometheus UI by entering the provided address in your web browser. This grants you access to monitor and review the alerts raised within your Kubernetes environment.
</br>
</br>

## Configuring Prometheus-k8s-event-exporter
The following steps guide you through the installation and configuration of Prometheus-k8s-event-exporter in your cluster to forward cluster events to Prometheus. This process involves using Helm charts for installation.
</br>

1. Add Delivery Hero Public Chart Repo. Before installing Prometheus-k8s-event-exporter, add the Delivery Hero public chart repository to Helm:
```helm
helm -n ntnx-system repo add deliveryhero https://charts.deliveryhero.io/
```
</br>

2. Install Prometheus-k8s-event-exporter with default values:
```helm
helm -n ntnx-system install deliveryhero/prometheus-k8s-events-exporter --generate-name
```
</br>

3. To ensure that the pod prometheus-k8s-event-exporter is targeted by the Prometheus Operator, edit the service by adding a name to the port associated with the prometheus-k8s-event-exporter service. Execute the following command to add the name prom-event to the spec.ports section for port 9102:
```kubectl
kubectl patch svc prometheus-k8s-events-exporter -n ntnx-system --type merge -p '{"spec":{"ports": [{"name":"prom-event","port": 9102, "protocol":"TCP", "targetPort": 9102}]}}'
```
</br>

4. Ensure that Prometheus scrapes metrics from the Prometheus-k8s-event-exporter by applying the ServiceMonitor configuration given below. Before applying the provided YAML configuration for the ServiceMonitor, ensure that you provide the necessary label required by your Prometheus configuration. This label allows Prometheus to select the ServiceMonitor present in the cluster if there is a ServiceMonitor selector rule defined in the Prometheus CRD.
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    ntnx-p8s: prometheus-event-exporter
  name: prometheus-event-exporter
  namespace: ntnx-system
spec:
  endpoints:
  - path: /metrics
    port: prom-event
    scheme: http
  jobLabel: prometheus-event-exporter
  selector:
    matchLabels:
      app.kubernetes.io/name: prometheus-k8s-events-exporter
```
</br>

These steps ensure that Prometheus-k8s-event-exporter is properly configured and integrated with Prometheus for event monitoring within your Kubernetes cluster. Once you've completed the setup steps outlined above, you can verify the integration of events from your cluster into Prometheus by querying the kubernetes_events metric in the Prometheus UI.

##
