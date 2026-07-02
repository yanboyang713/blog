---
title: "Kubernetes Network Policy"
draft: false
---

Kubernetes Network Policy is the means to enforce rules indicating which network traffic is allowed and which Pods can communicate with each other. The policies applied to Pod network traffic can be based on their applicability to ingress traffic (entering the Pod) and egress traffic (outgoing traffic). The control strategies include “allow” and “deny”.

By default, a Pod is in a non-isolated state. Once a network policy is applied to a Pod, all traffic that is not explicitly allowed will be rejected by the network policy. However, other Pods that do not have network policies applied to them are not affected. [Container Network Interface (CNI) plugins]({{< relref "20230608233002-container_network_interface_cni.md" >}}) in Kubernetes can implement elaborate traffic control and isolation mechanisms.


## Reference List {#reference-list}

1.  Qi, S., Kulkarni, S. G., &amp; Ramakrishnan, K. K. (2020). Assessing container network interface plugins: Functionality, performance, and scalability. IEEE Transactions on Network and Service Management, 18(1), 656-671.
