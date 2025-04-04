---
title: "service mesh research opportunities"
draft: false
---

Although service mesh development is still in its early stages, it already significantly complements microservices architectures in terms of observability, traceability, and manageability. It also opens several opportunities for future research.


## Unified Architecture {#unified-architecture}

As mentioned before, there are different architectures that might lead to confusion for service mesh adopters. For instance, the service mesh end-user should decide if they want to have sidecar proxies or proxyless architecture. Both strategies have some advantages and disadvantages. However, at the time of this writing, challenges are still open in currently existing solutions - for example, there is no unified architecture for service meshes, and the service mesh adopter community are confused about how to choose the right option.


## Performance {#performance}

There are a number of studies that have tried to improve the performance of service mesh solutions, but the performance analyses of existing solutions are unknown. Additionally, there is no guideline or best practices to optimize the tuning of service mesh features.


## Data Analytics {#data-analytics}

The service mesh layer presents a significant opportunity for data analytics on microservices applications. The service mesh layer collects core monitoring data in software systems, such as metrics, logging, and traces, via proxies that intercept all network communication between microservices. This allows researchers and developers to focus on data analytics rather than building pipelines for data collection. AIOps, workflowa scheduling, and sustainable computing are just a few of the research directions that data analytics can provide.


## Adoption of service mesh in edge environments {#adoption-of-service-mesh-in-edge-environments}

Currently, service mesh technologies have been largely designed for centralized cloud environment; there is little consideration for distributed cloud scenarios, such as Edge and Fog environments [98], etc. This is an important omission, as microservices (and container technology in general) is increasingly being used in distributed cloud systems, with many applications now designed to be cloud native. Mobile edge computing environments [99] are a particularly compelling frontier on which service mesh technology can be applied; the utility and ubiquity of such environments will be further enhanced as modern telecommunications networks (e.g. 5G
and upcoming 6G systems) become more prevalent.


## [eBPF]({{< relref "20230613171810-ebpf.md" >}}) trend {#ebpf--20230613171810-ebpf-dot-md--trend}

The eBPF is an increasingly popular tool used by the cloud native community, and enables management and monitoring from kernel space; some tools and service meshes have already employed eBPF for monitoring or even managing the network layer events. Approaches for employing eBPF in service meshes for root cause analysis, high performance monitoring and management of defined policies have the potential to be very useful. For instance, Cilium mesh uses eBPF to handle network layer events, whilst Bumblebee [100] is a tool to monitor the Istio service mesh and export the extracted metrics to Prometheus [101].


## Reference List {#reference-list}

1.  Sedghpour, M. R. S., &amp; Townend, P. (2022, August). Service Mesh and eBPF-Powered Microservices: A Survey and Future Directions. In 2022 IEEE International Conference on Service-Oriented System Engineering (SOSE) (pp. 176-184). IEEE.
