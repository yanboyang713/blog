---
title: "Mobile Edge Computing (MEC)"
draft: false
---

​Multi-access edge computing (MEC), formerly known as mobile [edge computing]({{< relref "20230504174218-edge_computing.md" >}}), is a network architecture concept defined by the European Telecommunications Standards Institute (ETSI). MEC brings [cloud computing]({{< relref "2023-10-09-180437-cloud_computing.md" >}}) capabilities and an IT service environment to the edge of the network, such as at [Cellular]({{< relref "20230614221419-cellular_network.md" >}}) base stations or other edge nodes. This proximity to end-users reduces network congestion and improves application performance by enabling applications and processing tasks to occur closer to the user.

MEC provides a distributed computing environment for hosting applications and services, with the ability to store and process content near cellular subscribers for faster response times. It also allows access to real-time radio access network (RAN) information. The key component is the MEC application server, integrated at the RAN element, offering computing resources, storage capacity, connectivity, and access to RAN information. This server supports a multitenancy runtime and hosting environment for applications, which can be delivered as virtual machine images or containers.

Deployment of MEC application servers can occur at macro base stations (eNodeB) in LTE networks, at Radio Network Controllers (RNC) in 3G networks, or at multi-technology cell aggregation sites, which may be located indoors or outdoors.

MEC offers several business and technical benefits. It enables cellular operators to efficiently deploy new services tailored to specific customers or customer classes, reduces the signal load on the core network, and allows for cost-effective hosting of applications and services. Additionally, it collects data on storage, network bandwidth, CPU utilization, and more for each application or service deployed by third parties. Application developers and content providers can leverage the close proximity to subscribers and real-time RAN information.

Applications of MEC include computational offloading, content delivery, mobile big data analytics, edge video caching, collaborative computing, connected cars, smart venues, smart enterprises, healthcare, smart grids, service function chaining, and indoor positioning. Some applications incorporating MEC have been available since 2015, such as active device location tracking independent of GPS and distributed content and DNS caching to reduce server load and improve data delivery speeds. A notable commercial product is AWS [Wavelength]({{< relref "2023-10-26-002942-aws_global_infrastructure.md#wavelength-zones" >}}), which allows customers to run applications on AWS services at the edge of a 4G/5G network of a specific telecommunications provider.

Technical standards for MEC are being developed by ETSI, which established an Industry Specification Group in 2014. This group includes various companies such as AT&amp;T, Cisco, Huawei, IBM, Intel, Nokia, Verizon, and Vodafone, among others.
Wikipedia

In summary, MEC enhances network efficiency and application performance by bringing computing capabilities closer to end-users, facilitating rapid deployment of new services, and enabling real-time access to network information.


## Papers {#papers}

-   [5G-MEC testbeds for V2X applications]({{< relref "2025-04-22-040357-5g_mec_testbeds_for_v2x_applications.md" >}})
-   [Container-based Service Relocation for Beyond 5G Networks]({{< relref "2025-04-22-041219-container_based_service_relocation_for_beyond_5g_networks.md" >}})

Benefit from MEC’s ability to reduce round-trip times and offload traffic from the core network.
[Service migration]({{< relref "2025-04-22-041753-service_migration.md" >}}) in MEC is analogous to the follow-me cloud idea from earlier mobile cloud research, aiming to preserve low latency and continuity for moving users by handing off their edge-hosted service to the optimal edge node.


## Reference List {#reference-list}

1.  <https://en.wikipedia.org/wiki/Multi-access_edge_computing>
