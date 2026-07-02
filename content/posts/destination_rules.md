---
title: "Destination rules"
tags: ["definition"]
draft: false
---

## Definition {#definition}

Along with [Virtual Service]({{< relref "virtual_service.md" >}}), destination rules are a key part of Istio’s traffic routing functionality. The Virtual services as how to route your traffic to a given destination and then use destination rules to configure what happens to traffic for that destination. Destination rules are applied after virtual service routing rules are evaluated, so they apply to the traffic’s “real” destination.

In particular, destination rules specify named service subsets, such as grouping all a given service’s instances by version. Then use these service subsets in the routing rules of virtual services to control the traffic to different instances of services.

Destination rules also could customize Envoy’s traffic policies when calling the entire destination service or a particular service subset, such as preferred load balancing model, TLS security mode, or circuit breaker settings.


## Examples {#examples}
