---
title: "Kubernetes Publishing Services (management external traffic)"
draft: false
---

## Publishing Services (ServiceTypes) {#publishing-services--servicetypes}

For some parts of your application (for example, frontends) you may want to expose a Service onto an external IP address, that's outside of your cluster.

Kubernetes ServiceTypes allow you to specify what kind of Service you want.

Type values and their behaviors are:

-   [ClusterIP]({{< relref "20221124055917-clusterip.md" >}})
-   [NodePort]({{< relref "nodeport.md" >}})
-   [LoadBalancer]({{< relref "20230601231737-loadbalancer.md" >}})


## Routing {#routing}

-   [Ingress]({{< relref "20221124054447-ingress.md" >}})


## Reference List {#reference-list}

1.  <https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0>
