# Setting Up Alerts for NDK

Welcome to the guide for setting up alerts for NDK. This document will walk you through the process of configuring alerts to ensure proactive monitoring and timely notification of any issues or anomalies within your NDK deployment.
</br>
</br>

## Prerequisites

Before proceeding with setting up alerts, ensure that the Prometheus Operator is installed in your Kubernetes cluster. The Prometheus Operator is essential for managing Custom Resource Definitions (CRDs) such as PrometheusRules and ServiceMonitors, which are utilized for generating alerts and scraping metrics from the Kubernetes cluster. In this documentation, we assume that the Prometheus Operator has been installed in the ntnx-system namespace of your Kubernetes cluster. Please ensure to use the appropriate namespace relevant to your cluster when executing the provided commands.
</br>
</br>

## Configuring Prometheus-Operator

To verify the configuration of the event-exporter, it is imperative to access the Prometheus UI. This can be accomplished by exposing the prometheus-operated service located within the ntnx-system namespace of the cluster. The following steps outline the procedure to expose the service and access the Prometheus UI:

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

Now that you have the Node IP and the exposed port, you can proceed to access the Prometheus UI by entering the provided address into your web browser's address bar. This step enables you to verify if the event-exporter is configured correctly and if Prometheus recognizes it as a target.
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
helm -n ntnx-system install deliveryhero/prometheus-k8s-events-exporter --generate-name --set image.tag="v1.0.0"
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

These steps ensure that Prometheus-k8s-event-exporter is properly configured and integrated with Prometheus for event monitoring within your Kubernetes cluster. Once you've completed the setup steps outlined above, you can verify the integration of events from your cluster into Prometheus by querying the `kube_event_unique_events_total` metric in the Prometheus UI.
</br>
</br>

## Configuring Alerts for Your Application

To configure alerts for NDK, you will need to apply the `ndk-alert-config.yaml` file to your Kubernetes cluster. Before proceeding with the application of this YAML configuration for the PrometheusRule object, it's essential to ensure that you provide the necessary label required by your Prometheus configuration. This label enables Prometheus to select the PrometheusRule object present in the cluster if there is a corresponding PrometheusRule selector rule defined in the Prometheus Custom Resource Definition (CRD).
