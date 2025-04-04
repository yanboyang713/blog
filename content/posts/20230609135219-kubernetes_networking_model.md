---
title: "Kubernetes Networking Model"
tags: ["Networking", "model"]
draft: false
---

The Kubernetes networking model is proposed for dealing with four different kinds of communication:

1.  intraPod or Container-to-Container communication within a Pod
2.  inter-Pod or Pod-to-Pod communication
3.  Service-toPod communication
4.  External-to-Service communication [30]

To achieve these four communication services, [Kubernetes]({{< relref "20230105185343-kubernetes.md" >}}) only provides the specification of the network model, while the actual implementation is handed over to the [Container Network Interface (CNI) plugins]({{< relref "20230608233002-container_network_interface_cni.md" >}}) .

The key requirements of the Kubernetes network model include

1.  Pods are IP addressable and must be able to communicate with all other Pods (on the same or different host) without the need for network address translation (NAT)
2.  all the agents on a host (e.g., Kubelet) are able to communicate with all the Pods on that host. CNI plugins may differ in their architecture but meet the above network rules. Thus, there is a range of CNI plugins that adopt different approaches.


## Reference List {#reference-list}

1.  Qi, S., Kulkarni, S. G., &amp; Ramakrishnan, K. K. (2020). Assessing container network interface plugins: Functionality, performance, and scalability. IEEE Transactions on Network and Service Management, 18(1), 656-671.
