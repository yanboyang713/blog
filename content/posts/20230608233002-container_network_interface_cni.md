---
title: "Container Network Interface (CNI)"
tags: ["CNI"]
draft: false
---

To bootstrap [Kubernetes]({{< relref "20230105185343-kubernetes.md" >}}) networking, the Container Network Interface (CNI) provides a unified interface for the interaction between container runtimes. There are several CNI implementations, available as open-source ‘CNI plugins’. While they differ in functionality and performance.

Overlay tunnel offload support in the network interface card plays a significant role in achieving the good performance of CNIs that use overlay tunnels for inter-host Pod-to-Pod communication.

Kubernetes adopts the Container Network Interface (CNI) specification as its core networking foundation [6].

When a new Pod is added, the CNI plugin coordinates with the container runtime and connects the container network namespace with the host network namespace (e.g., by setting up the virtual ethernet (veth) pair), assigns a unique IP address to the new Pod, applies the desired network policies and distributes routing information to the rest of the cluster.

Several open-source CNI plugins are available for use in a Kubernetes environment. Amongst them, [Flannel]({{< relref "20230609133057-flannel.md" >}}) [8], Weave Net (or [Weave]({{< relref "20230609133122-weave.md" >}})) [9], [Cilium]({{< relref "20230609133144-cilium.md" >}}) [10],[Calico]({{< relref "20230609130541-calico.md" >}}) [11], and [Kuberouter]({{< relref "20230609133233-kuberouter.md" >}}) [12] are popular and have been adopted by many Kubernetes distributions [13].


## Major Components {#major-components}

“CNI plugins” comprise of two major components, namely the CNI daemon and CNI binary files. The CNI daemon is mainly used to do network management jobs, such as updating the routing information of the hosts,3 maintaining network policies,4 renewing the subnet leasing,5 Border Gateway Protocol (BGP) updating,6 and maintaining some self-defined resources (e.g., IP-Pool in Calico).7 The CNI binary files are mainly used to create the network devices (e.g., Linux bridge) and allocate IP address to Pods.


## Intra-Host Performance {#intra-host-performance}


### Overall Performance {#overall-performance}


#### TCP throughput {#tcp-throughput}

For TCP throughput, Cilium with its native solution based on eBPF outperforms the other alternatives. Layer-3 routing based solutions (Calico-wp and Calico-np) perform worse than the Layer-2 based solutions.


#### latency {#latency}

Accordingly, we also observe that Cilium achieves the lowest latency and Calico-wp has the worst round-trip latency as shown in Fig. 8(b). This is primarily due to the overheads involved in processing the Netfilter rules and Layer-3 routing, which is avoided with the eBPF based CNIs.


### Overhead Breakdown for Intra-Host Comm {#overhead-breakdown-for-intra-host-comm}

The total [CPU cycles per packet (CPP)]({{< relref "20230609145751-cpu_cycles_per_packet.md" >}}) with the corresponding break down for intra-host communication is shown in Fig. 9. For the overall overhead of the complete network stack, Calico-wp (with Netfilter being the major contributor) has the highest CPP and Cilium the lowest.

For intra-host communication, a native routing datapath based on eBPF is much cheaper than a bridge-based datapath or native routing datapath based on IP forwarding. eBPF combines packet forwarding and filtering together, which reduces the packet forwarding overhead. Thus, Cilium achieves the highest throughput and lowest latency. Further, a fine-grained iptables chain as in Calico-wp unfortunately hurts packet transmission performance.


## Inter-Host Performance {#inter-host-performance}


### Overall Performance {#overall-performance}

without the tunnel offload in the Mellanox ConnectX-4 25Gb NIC, Flannel-off and Calico in IP-in-IP overlays perform poorly, with much lower throughput than Cilium and Calico with native routing (xsub) options. Moreover, for the same datapath, solutions with network policy disabled (Calico-np-xsub and Calico-np-ipip) perform 0.5 ∼ 1.5 Gbps better than when network policy enabled (Calico-wp-xsub and Calico-wp-ipip).


### TCP round-trip latencies {#tcp-round-trip-latencies}

1.  Tunnel offload is necessary to improve the CNI performance, and where the CNIs can support multiple overlay solutions (UDP, VxLAN, IP-in-IP), it is desirable to choose the overlay mode that can be supported via tunnel offload.
2.  the underlay-based CNIs (Calico xsub and Kube-router) perform better than the overlay options, despite the NIC’s offloading of the tunnel-related operations.


### Overhead Breakdown for Inter-Host Comm {#overhead-breakdown-for-inter-host-comm}

The native routing solutions (Kube-router, Calico-wp-xsub and Calico-np-xsub) have lower CPP compared to the overlay solutions (Flannel, Flannel-off, Weave, Cilium, Calico-wp-ipip and Calico-np-ipip). Also, the solutions with simple iptables have lower CPP than the complex iptables chains.


## Performance for Larger-Scale Configurations {#performance-for-larger-scale-configurations}


### Intra-Host Performance With Background Traffic {#intra-host-performance-with-background-traffic}


#### throughout {#throughout}

Cilium outperforms the other CNI plugins because of its low overhead in the datapath and in Netfilter. Calico-wp has the worst performance throughout, due to its large overhead from Netfilter rules.


#### TCP roundtrip time {#tcp-roundtrip-time}

The TCP roundtrip time is also better for Cilium.


### Inter-Host Performance With Background Traffic {#inter-host-performance-with-background-traffic}

native routing solutions (Kube-router and Calico-xsub) outperform the overlay based solutions, because of less overhead on the datapath and Netfilter. In contrast, Calico in IP-in-IP mode performs worse than the others, because of the additional overhead since the NIC does not offload this function. This degraded performance is also observed with Flannel, when the tunneloffload is turned off (Flannel-off). All of these see a precipitous drop in throughput beyond 400 background connections (each generating 10 Mbps) because the CPU is overloaded and the latency is correspondingly higher


### CPU Utilization and Memory Footprint {#cpu-utilization-and-memory-footprint}

Native routing (‘Calico-\*-xsub’, ’Kuberouter’) incurs relatively low CPU overhead, while the overlay mode CNIs (e.g., Flannel, Weave, etc.) have a much higher CPU load.

We observe that Flannel and Kube-router have a low memory footprint (40 ∼ 50 MB), while Weave, Cilium, and Calico have a very high memory footprint (160 ∼ 200 MB).


## Impact of CNI on Typical HTTP Workload {#impact-of-cni-on-typical-http-workload}


### HTTP Performance Results {#http-performance-results}

Calico (both overlay and underlay modes) outperform the rest of the CNIs in a single connection case.
In general, a ‘Layer-3 + Underlay’ CNI (e.g., Calico, native routing) appears better suited for most HTTP traffic, especially at large scale (higher concurrency) traffic patterns.


### Iptables Evaluation {#iptables-evaluation}


#### intra-host case {#intra-host-case}

In the intra-host case, Cilium use eBPF for packet forwarding, which bypasses the iptables processing. Flannel, Weave, Kube-router, and Calico-np (both ‘Calico-np-ipip’ and ‘Caliconp-xsub’ in the intra-host case, as they are equivalent) have the same number of iptables chains and rules, and they all exhibit similar netfiler overhead (∼ 245 CPP) for the intra-host case. Calico-wp (both ‘Calico-wp-ipip’ and ‘Calico-wp-xsub’) is configured with 17 iptables rules by default. This adds to a higher netfilter overhead (324 CPP) compared to the others.


#### inter-host case {#inter-host-case}

In the inter-host case, Calico with network policy enabled (e.g., ‘Calico-wp-ipip’) has more iptables rules configured resulting in higher overhead (749 CPP). Moreover, with a similar number of rules, a CNI with fewer iptables chains applied (e.g., Cilium) has much less overhead (450 CPP) than the CNIs with more iptables chains applied (e.g., Flannel, Weave).

Based on the iptables evaluation, we conclude that CNIs with fewer iptables chains and rules will have relatively less Netfilter overhead. However, to assure Pod network security, users needs to use the CNI’s Network Policy API to install iptables rules, which could increase Netfilter overhead and lead to performance loss.


## Pod Creation Time Analysis {#pod-creation-time-analysis}

Flannel and Kube-router have a smaller network startup latency (∼ 60 ms) compared to the other alternatives. Weave consumes about 165ms in the Pod-host Link Up step, due to the work of appending multicast rule in iptables. Calico spends ∼ 80 ms in the IP Allocation step, which is primarily due to the interaction with the etcd store. The time spent by Cilium in the Endpoint Creation step accounts for ∼ 90 ms. During this step, Cilium generates the eBPF code and links it into the kernel, which contributes to this high latency.

Flannel and Kube-router have the smallest amount of increase in the creation latency as more Pods deployed together, while the latency with Cilium increases much more rapidly, which shows poor scalability.


## Summary {#summary}

While there is no single universally ‘best’ CNI plugin, there is a clear choice depending on the need for intra-host or inter-host Pod-to-Pod communication. For the intra-host case, Cilium appears best, with eBPF optimized for routing within a host. For the inter-host case, Kube-router and Calico are better due to the lighter-weight IP routing mode compared to their overlay counterparts. Although Netfilter rules incur overhead, their rich, fine-grained network policy and customization can enhance cluster security. Tunnel offload is another aspect to be considered, which can help to achieve the maximum performance when working with a CNI’s overlay mode. This may be very desirable for Cloud Service Providers.


## Reference List {#reference-list}

1.  <https://platform9.com/blog/the-ultimate-guide-to-using-calico-flannel-weave-and-cilium/>
2.  Qi, S., Kulkarni, S. G., &amp; Ramakrishnan, K. K. (2020). Assessing container network interface plugins: Functionality, performance, and scalability. IEEE Transactions on Network and Service Management, 18(1), 656-671.
