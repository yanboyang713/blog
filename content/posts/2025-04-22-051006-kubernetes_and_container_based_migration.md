---
title: "Kubernetes and container-based migration"
date: 2025-04-22
draft: false
---

[Kubernetes]({{< relref "20230105185343-kubernetes.md" >}})

Containers encapsulate applications and can be relocated more quickly than VMs, though transferring in-memory state still poses challenges. By default, Kubernetes does not provide live migration of running pods – if a pod is rescheduled to another node, it usually involves shutting it down and restarting on the target node (a “cold” migration). Recent research therefore focuses on extending container orchestrators to support live container migration, often using checkpoint/restore tools such as CRIU (Checkpoint/Restore In Userspace) to snapshot a container’s state and restore it on a new node
link.springer.com
. This approach can preserve in-memory state and open connections, enabling stateful migration of microservices.
