---
title: "Kubernetes Monitoring"
draft: false
---

## Kubernetes Monitoring Tools {#kubernetes-monitoring-tools}

[Kubernetes]({{< relref "20230105185343-kubernetes.md" >}}) is a complex environment, and containerized applications can be distributed across multiple environments. Monitoring solutions must be able to aggregate metrics from across the distributed environment, and deal with the ephemeral nature of containerized resources. The following are popular monitoring tools designed for a containerized environment.

-   [Kubernetes Dashboard tools]({{< relref "20230516092022-kubernetes_dashboard_tools.md" >}})

The Kubernetes dashboard will give you a bird’s eye view of what’s going on in your clusters, but it’s not enough for production monitoring. This requires a dedicated Kubernetes monitoring tool such as [Prometheus]({{< relref "20230602142910-prometheus.md" >}}).


## Reference List {#reference-list}

1.  <https://www.tigera.io/learn/guides/kubernetes-monitoring/>
