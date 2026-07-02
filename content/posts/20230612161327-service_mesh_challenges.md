---
title: "service mesh challenges"
tags: ["mesh"]
draft: false
---

To achieve the vision of a [service mesh]({{< relref "20221124053019-mesh_service.md" >}}), three main challenges can be identified as the following:

1.  The first is the design supporting high performance. At the data plane level, the proxy component co-located with the service instances is the heart of a service mesh. It is responsible for intercepting and mediating the traffic among microservices and needs to be light-weight and designed for high performance. The same requirement applies to the components in the control plane, especially for those responsible for data aggregation.
2.  The second is adaptability. To support and adapt to a wide range of cloud-native orchestration platforms, a service mesh needs to provide a certain level of configurability, extensibility, and pluggability.
3.  Third, high availability is also a very critical concern. A highly available service mesh can lower the risk of decision-making with bias due to data/component unavailability. As discussed in Section IV-A, it is also crucial to isolate the impact of unavailability on the application side from service mesh.


## Reference List {#reference-list}

1.  Li, W., Lemieux, Y., Gao, J., Zhao, Z., &amp; Han, Y. (2019, April). Service mesh: Challenges, state of the art, and future research opportunities. In 2019 IEEE International Conference on Service-Oriented System Engineering (SOSE) (pp. 122-1225). IEEE.
