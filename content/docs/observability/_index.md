---
title: Observability
weight: 700
description: Gain insight into the health and performance of your gateways.
---

Gain insight into the health and performance of your gateways.

## About

Metrics are essential to gain insight into the health and performance of your gateway proxies. [OpenTelemetry](https://opentelemetry.io/) is a flexible open source framework that provides a set of APIs, libraries, and instrumentation to help capture and export telemetry data, such as metrics. The framework can also be used to collect traces and logs from your apps. Then, you can use observability tools, such as Grafana or Prometheus, to visualize your metrics so that you can analyze the health of your gateway and troubleshoot issues more easily. 

In this guide, you deploy an OpenTelemetry collector that scapes metrics from the {{< reuse "docs/snippets/product-name.md" >}} proxies in the data plane, the {{< reuse "docs/snippets/product-name.md" >}} pods in the control plane, and the external auth and rate limiting add-ons. The metrics that are collected by the OpenTelemetry collector are exposed in Prometheus format. To visualize these metrics, you also deploy a Grafana instance that scrapes the metrics from the OpenTelemetry collector.

{{% callout type="info" %}}
If you do not want to deploy an OpenTelemetry collector and Grafana, you can quickly see the raw Prometheus metrics that are automatically exposed on the gateway proxy by accessing the Prometheus metrics on your gateway. 
1. Port-forward the gateway deployment on port 19000.
   ```sh
   kubectl -n gloo-system port-forward deployment/gloo-proxy-http 19000
   ```
2. Access the gateway metrics by reviewing the [Prometheus statistics](http://localhost:19000/stats/prometheus). 
{{% /callout %}}

## Before you begin

{{< reuse "docs/snippets/prereq.md" >}}

## Set up an OpenTelemetry collector

1. Add the Helm repository for OpenTelemetry. 
   ```sh
   helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
   helm repo update
   ```

2. Install the OpenTelemetry collector in your cluster. This command sets up pipelines that scrape metrics from the gateway proxies, {{< reuse "docs/snippets/product-name.md" >}} control plane, and external auth and rate limiting add-ons, and exposes them in Prometheus format.
   ```sh
   helm upgrade --install opentelemetry-collector open-telemetry/opentelemetry-collector \
   --version 0.97.1 \
   --set mode=deployment \
   --set image.repository="otel/opentelemetry-collector-contrib" \
   --set command.name="otelcol-contrib" \
   --namespace=otel \
   --create-namespace \
   -f -<<EOF
   clusterRole:
     create: true
     rules:
     - apiGroups:
       - ''
       resources:
       - 'pods'
       - 'nodes'
       verbs:
       - 'get'
       - 'list'
       - 'watch'
   ports:
     promexporter:
       enabled: true
       containerPort: 9099
       servicePort: 9099
       protocol: TCP
   config:
     receivers:
       prometheus/gloo-dataplane:
         config:
           scrape_configs:
           # Scrape the {{< reuse "docs/snippets/product-name.md" >}} proxies
           - job_name: gloo-gateways
             honor_labels: true
             kubernetes_sd_configs:
             - role: pod
             relabel_configs:
               - action: keep
                 regex: kube-gateway
                 source_labels:
                 - __meta_kubernetes_pod_label_gloo
               - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
                 action: keep
                 regex: true
               - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
                 action: replace
                 target_label: __metrics_path__
                 regex: (.+)
               - action: replace
                 source_labels:
                 - __meta_kubernetes_pod_ip
                 - __meta_kubernetes_pod_annotation_prometheus_io_port
                 separator: ':'
                 target_label: __address__
               - action: labelmap
                 regex: __meta_kubernetes_pod_label_(.+)
               - source_labels: [__meta_kubernetes_namespace]
                 action: replace
                 target_label: kube_namespace
               - source_labels: [__meta_kubernetes_pod_name]
                 action: replace
                 target_label: pod
       prometheus/gloo-controlplane:
         config:
           scrape_configs:
           # Scrape the {{< reuse "docs/snippets/product-name.md" >}} control plane
           - job_name: gloo-gateways
             honor_labels: true
             kubernetes_sd_configs:
             - role: pod
             relabel_configs:
               - action: keep
                 regex: gloo
                 source_labels:
                 - __meta_kubernetes_pod_label_gloo
               - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
                 action: keep
                 regex: true
               - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
                 action: replace
                 target_label: __metrics_path__
                 regex: (.+)
               - action: replace
                 source_labels:
                 - __meta_kubernetes_pod_ip
                 - __meta_kubernetes_pod_annotation_prometheus_io_port
                 separator: ':'
                 target_label: __address__
               - action: labelmap
                 regex: __meta_kubernetes_pod_label_(.+)
               - source_labels: [__meta_kubernetes_namespace]
                 action: replace
                 target_label: kube_namespace
               - source_labels: [__meta_kubernetes_pod_name]
                 action: replace
                 target_label: pod
       prometheus/gloo-addons:
         config:
           scrape_configs:
           # Scrape the extauth and ratelimit workloads
           - job_name: gloo-gateways
             honor_labels: true
             kubernetes_sd_configs:
             - role: pod
             relabel_configs:
               - action: keep
                 regex: extauth|rate-limit
                 source_labels:
                 - __meta_kubernetes_pod_label_gloo
               - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
                 action: keep
                 regex: true
               - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
                 action: replace
                 target_label: __metrics_path__
                 regex: (.+)
               - action: replace
                 source_labels:
                 - __meta_kubernetes_pod_ip
                 - __meta_kubernetes_pod_annotation_prometheus_io_port
                 separator: ':'
                 target_label: __address__
               - action: labelmap
                 regex: __meta_kubernetes_pod_label_(.+)
               - source_labels: [__meta_kubernetes_namespace]
                 action: replace
                 target_label: kube_namespace
               - source_labels: [__meta_kubernetes_pod_name]
                 action: replace
                 target_label: pod
     exporters:
       prometheus:
         endpoint: 0.0.0.0:9099
       debug: {}
     service:
       pipelines:
         metrics:
           receivers: [prometheus/gloo-dataplane, prometheus/gloo-controlplane, prometheus/gloo-addons]
           processors: [batch]
           exporters: [prometheus]
   EOF
   ```

3. Verify that the OpenTelemetry collector pod is running. 
   ```sh
   kubectl get pods -n otel
   ```
   
   Example output: 
   ```console
   NAME                                       READY   STATUS    RESTARTS   AGE
   opentelemetry-collector-6d658bf47c-hw6v8   1/1     Running   0          12m
   ```

## Set up Grafana 

1. Deploy Grafana and other Prometheus components in your cluster. The following example uses the [kube-prometheus-stack community Helm chart](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) to install these components. 
   ```yaml
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm repo update

   helm upgrade --install kube-prometheus-stack \
   prometheus-community/kube-prometheus-stack \
   --version 61.2.0 \
   --namespace monitoring \
   --create-namespace \
   --values - <<EOF
   alertmanager:
     enabled: false
   grafana: 
     service: 
       type: LoadBalancer
       port: 3000
   nodeExporter:
     enabled: false   
   prometheus: 
     prometheusSpec: 
       ruleSelectorNilUsesHelmValues: false
       serviceMonitorSelectorNilUsesHelmValues: false
       podMonitorSelectorNilUsesHelmValues: false
   EOF
   ```
   
2. Verify that the Prometheus stack's components are up and running. 
   ```sh
   kubectl get pods -n monitoring
   ```
   
   Example output: 
   ```console
   NAME                                                        READY   STATUS    RESTARTS   AGE
   kube-prometheus-stack-grafana-86844f6b47-frwn9              3/3     Running   0          20s
   kube-prometheus-stack-kube-state-metrics-7c8d64d446-6cs7m   1/1     Running   0          21s
   kube-prometheus-stack-operator-75fc8896c7-r7bgk             1/1     Running   0          20s
   prometheus-kube-prometheus-stack-prometheus-0               2/2     Running   0          17s 
   ```

3. Create a PodMonitor resource to scrape metrics from the OpenTelemetry collector. 
   ```yaml
   kubectl apply -n otel -f- <<EOF
   apiVersion: monitoring.coreos.com/v1
   kind: PodMonitor
   metadata:
     name: otel-monitor
   spec:
     podMetricsEndpoints:
     - interval: 30s
       port: promexporter
       scheme: http
     selector:
       matchLabels:
         app.kubernetes.io/name: opentelemetry-collector
   EOF
   ```
   
4. Save the [sample Grafana dashboard configuration](grafana.json) as `envoy.json`. 

5. Import the Grafana dashboard. 
   ```sh
   kubectl -n monitoring create cm envoy-dashboard \
   --from-file=envoy.json
   kubectl label -n monitoring cm envoy-dashboard grafana_dashboard=1
   ```
   
## Visualize metrics in Grafana
   
1. Generate traffic for the httpbin app. 
   ```sh
   for i in {1..5}; do curl -v http://$INGRESS_GW_ADDRESS:8080/headers -H "host: www.example.com:8080"; done
   ```

2. Open Grafana and log in to Grafana by using the username `admin` and password `prom-operator`. 
   {{< tabs items="Cloud Provider LoadBalancer,Port-forward for local testing" >}}
   {{% tab %}}
   ```sh
   open "http://$(kubectl -n monitoring get svc kube-prometheus-stack-grafana -o jsonpath="{.status.loadBalancer.ingress[0]['hostname','ip']}"):3000"
   ```
   {{% /tab %}}
   {{% tab  %}}
   ```sh
   kubectl port-forward deployment/kube-prometheus-stack-grafana -n monitoring 3000
   ```
   {{% /tab %}}
   {{< /tabs >}}
   
3. Go to **Dashboards** > **Envoy** to open the dashboard that you imported. Verify that you see the traffic that you generated for the httpbin app. 
   
   {{< reuse-image src="img/grafana-dashboard.png" >}}
   
## Cleanup

{{< reuse "docs/snippets/cleanup.md" >}}

1. Remove the configmap for the Envoy dashboard. 
   ```sh
   kubectl delete cm envoy-dashboard -n monitoring
   ```

2. Remove the PodMonitor. 
   ```sh
   kubectl delete podmonitor otel-monitor -n otel
   ```

3. Uninstall Grafana. 
   ```sh
   helm uninstall kube-prometheus-stack -n monitoring  
   ```

4. Uninstall the OpenTelemetry collector. 
   ```sh
   helm uninstall opentelemetry-collector -n otel
   ```

5. Remove the `monitoring` and `otel` namespaces. 
   ```sh
   kubectl delete namespace monitoring otel
   ```
