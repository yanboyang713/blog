---
title: "Service migration / Application Relocation"
date: 2025-04-22
draft: false
---

Service migration is analogous to the follow-me cloud idea from earlier mobile cloud research, aiming to preserve low latency and continuity for moving users by handing off their edge-hosted service to the optimal edge node.


## Challenge {#challenge}

Service migration can be challenging because it involves transferring not only the application code but often its runtime state or user context with minimal interruption.

The 5G system and MEC standards recognize this challenge: 3GPP’s 5G architecture allows traffic redirection to new MEC data anchors (via UPF re-location), and ETSI’s MEC framework explicitly includes “application mobility” support for relocating MEC application instances and transferring user context between MEC hosts​. In practice, a MEC orchestrator or mobility management function needs to decide when and where to migrate a service as the UE moves, orchestrating the deployment on the target MEC host and ensuring any necessary state transfer. The goal is to achieve seamless service continuity, meaning the user’s session experiences little to no disruption during the migration.

Live migration techniques, well-studied in cloud computing for virtual machines (VMs), are being adapted to the edge context to enable stateful service handoff. Live migration entails moving a running service (VM or container) to a new host with minimal downtime by transferring its memory state incrementally (pre-copy) or by other means while the service keeps running until a brief switchover. In traditional data centers, hypervisor-based VM live migration can achieve downtimes of a few hundred milliseconds. In MEC, live migration must cope with wireless link variability, user mobility, and stricter latency constraints (e.g. for URLLC applications). Moreover, edge servers are often resource-constrained and widely distributed, making the migration decision and execution more complex than in a centralized cloud.

[Kubernetes and container-based migration]({{< relref "2025-04-22-051006-kubernetes_and_container_based_migration.md" >}})
