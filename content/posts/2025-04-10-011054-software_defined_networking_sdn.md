---
title: "Software-Defined Networking (SDN)"
date: 2025-04-10
draft: false
---

Software-Defined Networking (SDN) is an approach to networking where the **control plane** (decision-making) is decoupled from the **data plane** (packet forwarding), enabling the network to be configured and automated through software. SDN typically introduces a logically centralized controller (or a cluster of controllers) that programs forwarding devices via well-defined APIs, making it easier to express intent and enforce policy consistently across the network.


## Key Ideas {#key-ideas}

-   **Separation of planes:** devices focus on forwarding; controllers compute policy and paths.
-   **Logical centralization:** a “single” control view can be achieved with distributed controllers.
-   **Programmability and automation:** network behavior is exposed via APIs rather than per-box CLI.
-   **Abstraction:** higher-level policies map to lower-level device configuration and forwarding state.


## Architecture (High Level) {#architecture--high-level}

-   **Data plane:** switches/routers that forward packets based on flow tables or forwarding rules.
-   **Control plane:** SDN controller(s) that compute state, policy, and paths.
-   **Southbound API:** controller → devices (examples: OpenFlow, NETCONF/YANG, gNMI, P4Runtime).
-   **Northbound API:** controller → apps/automation (often REST/gRPC; policy and intent interfaces).
-   **East/West API:** controller ↔ controller (state synchronization, clustering, federation).


## Benefits {#benefits}

-   Faster change and safer automation (versioned configs, CI/CD-style workflows).
-   Consistent policy enforcement (microsegmentation, ACLs, QoS, routing intent).
-   Better observability and control-loop automation (telemetry → analysis → action).
-   Enables network virtualization/overlays (common in modern data centers and clouds).


## Challenges / Trade-offs {#challenges-trade-offs}

-   **Scalability and resiliency:** controller performance, clustering, failure domains.
-   **Security:** controller/API protection, blast radius of misconfiguration, supply-chain risk.
-   **Interoperability:** device feature parity and vendor-specific behaviors.
-   **Troubleshooting complexity:** overlay + underlay interactions; distributed state debugging.


## Common Use Cases {#common-use-cases}

-   Data center fabrics (automation, segmentation, traffic engineering).
-   WAN/SD-WAN (centralized policy, path control, intent-based routing).
-   Campus networks (policy-driven access, segmentation, simplified operations).
-   NFV/service chaining (steering flows through virtualized network functions).


## Related Notes {#related-notes}

-   [Proxmox SDN]({{< relref "2025-12-14-142616-proxmox_sdn.md" >}})
-   [Setting Up EVPN on Proxmox SDN]({{< relref "2025-09-28-131926-setting_up_evpn_on_proxmox_sdn.md" >}})


## Reference List {#reference-list}

1.  <https://en.wikipedia.org/wiki/Software-defined_networking>
2.  <https://www.rfc-editor.org/rfc/rfc7426>
