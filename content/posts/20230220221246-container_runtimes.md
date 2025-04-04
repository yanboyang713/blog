---
title: "Container Runtimes"
tags: ["container", "runtimes"]
draft: false
---

## What is container {#what-is-container}

Containers are a lightweight alternative to fully [Virtual Machine]({{< relref "20230104161110-virtual_machine.md" >}})(VMs). They use the kernel of the host system that they run on, instead of emulating a full operating system (OS). This means that containers can access resources on the host system directly.

The runtime costs for containers is low, usually negligible. However, there are some drawbacks that need be considered:

-   Only Linux distributions can be run in Containers. It is not possible to run other operating systems like, for example, FreeBSD or Microsoft Windows inside a container.
-   For security reasons, access to host resources needs to be restricted. Therefore, containers run in their own separate namespaces. Additionally some syscalls (user space requests to the Linux kernel) are not allowed within containers.


## [Kubernetes]({{< relref "20230105185343-kubernetes.md" >}}) {#kubernetes--20230105185343-kubernetes-dot-md}

Several common container runtimes with [Kubernetes]({{< relref "20230105185343-kubernetes.md" >}})

-   [docker]({{< relref "20230216042646-docker.md" >}})
-   [containerd]({{< relref "20230613173352-containerd.md" >}})
-   [CRI-O]({{< relref "20230613173333-cri_o.md" >}})
-   [Kata Containers]({{< relref "2024-07-10-094412-kata_containers.md" >}})


## System Containers {#system-containers}

-   [LXC]({{< relref "2023-12-03-143424-lxc.md" >}})


## Reference List {#reference-list}

1.  <https://kubernetes.io/docs/setup/production-environment/container-runtimes/>
