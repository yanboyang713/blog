---
title: "Linux Network Namespace and Kubernetes Namespace"
tags: ["namespace"]
draft: false
---

Before we delve into the [Kubernetes Networking Model]({{< relref "20230609135219-kubernetes_networking_model.md" >}}), it is necessary to understand the namespace model and distinguish between the Linux network namespace and Kubernetes namespace and the associated impact on setting up of the CNI.


## Linux network namespace {#linux-network-namespace}

Linux network namespace is designed for network isolation. Each Pod has its own network namespace, which is isolated from the host network namespace and the network namespace of other Pods. This enables the Pod to operate its own network stack and interfaces without interference and collision with other Pods. When a new Pod is created, the container runtime interface (CRI) creates the network namespace for the new Pod. Thereafter it invokes the CNI plugin, which allocates the IP address for the Pod, attaches the virtual Ethernet pair (veth-pair) to link up the Pod’s network namespace to the host network namespace, and add the corresponding routing and network policy rules.


## [Kubernetes]({{< relref "20230105185343-kubernetes.md" >}}) namespace {#kubernetes--20230105185343-kubernetes-dot-md--namespace}

On the other hand, the Kubernetes namespace is used to divide the physical cluster into multiple virtual clusters. This enables sharing the physical cluster resources among different groups of users and makes the management of the cluster more flexible. This also means that the Pods in different Kubernetes namespaces are not strictly isolated. Kubernetes starts with several system namespaces typically prefixed with ‘kube-’ (e.g., kube-system, kube-pubilc, kube-node-lease, and default). Users are also allowed to create their own Kubernetes namespaces to isolate their workloads from other users. When creating the user namespaces, users can specify the resource quota for the created namespace, such as the maximum number of running Pods, CPU, and memory limits to avoid the threat of exorbitant resources requests. The Kubernetes namespace can be used as a selector in the network policy, to apply a specific network policy to a group of Pods in that namespace, which decouples the Pods from their static IP addresses and improves the resource management efficiency in a large scale cluster.


## Reference List {#reference-list}

1.  Qi, S., Kulkarni, S. G., &amp; Ramakrishnan, K. K. (2020). Assessing container network interface plugins: Functionality, performance, and scalability. IEEE Transactions on Network and Service Management, 18(1), 656-671.
