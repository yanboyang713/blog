---
title: "Service Mesh in Edge Computing Environments"
draft: false
---

[service mesh]({{< relref "20221124053019-mesh_service.md" >}})
[edge computing]({{< relref "20230504174218-edge_computing.md" >}})

existing service mesh solutions are mainly designed for centralized cloud environments which are typically located within a single data center, and thus have very limited support for distributed cloud scenarios.

This situation is exaggerated for the emerging edge cloud paradigm [44] which advocates to distribute resources along the path between centralized data centers and the logical endpoints of a network (that may have an increasingly large number of devices) in order to improve the performance, operating cost and reliability of applications and services. On the other hand, the rising adoption of container technologies [45], [46] and microservices methodology [47], [48] in edge computing [49], has indicated the need and trend of exploration of service mesh in this area as the merging applications running on edge clouds are expected to be designed cloud-natively.

It would be even more interesting and challenging to explore how service mesh can be applied in [Mobile Edge Computing (MEC)]({{< relref "20230612170915-mobile_edge_computing_mec.md" >}}) [50] environments.

deployment of a mobile edge cloud may consist of multiple sites, which usually are heterogeneous in their HW/SW setup, geographically distributed, and typically structured hierarchically. Besides, multiple VNFs2 (Virtual Network Functions [51], which may belong to different tenants) reside within the same sites where the services are provisioned. In consequence, such complexities will result in extensive effort to identify the problems and issues when migrating the service mesh into the mobile edge cloud, and design the required extensions on top of existing service mesh solutions in order to tackle the identified problems and make them optimized for mobile edge cloud environments. Example of such extensions could be optimized load balancing or service routing mechanism, with support for multi-tenancy, multi-protocol, and flexible networking typologies.


## Reference List {#reference-list}

1.  Li, W., Lemieux, Y., Gao, J., Zhao, Z., &amp; Han, Y. (2019, April). Service mesh: Challenges, state of the art, and future research opportunities. In 2019 IEEE International Conference on Service-Oriented System Engineering (SOSE) (pp. 122-1225). IEEE.
