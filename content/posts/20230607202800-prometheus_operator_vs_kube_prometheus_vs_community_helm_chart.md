---
title: "Prometheus Operator vs. kube-prometheus vs. community helm chart"
draft: false
---

## [Prometheus Operator]({{< relref "20230607201549-prometheus_operator.md" >}}) {#prometheus-operator--20230607201549-prometheus-operator-dot-md}

The Prometheus Operator uses Kubernetes custom resources to simplify the deployment and configuration of Prometheus, Alertmanager, and related monitoring components.


## [kube-prometheus]({{< relref "20230607200853-kube_prometheus.md" >}}) {#kube-prometheus--20230607200853-kube-prometheus-dot-md}

kube-prometheus provides example configurations for a complete cluster monitoring stack based on Prometheus and the Prometheus Operator. This includes deployment of multiple Prometheus and Alertmanager instances, metrics exporters such as the node_exporter for gathering node metrics, scrape target configuration linking Prometheus to various metrics endpoints, and example alerting rules for notification of potential issues in the cluster.


## [helm chart]({{< relref "20230604152823-kube_prometheus_stack.md" >}}) {#helm-chart--20230604152823-kube-prometheus-stack-dot-md}

The prometheus-community/kube-prometheus-stack helm chart provides a similar feature set to kube-prometheus. This chart is maintained by the Prometheus community.
