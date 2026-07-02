---
title: "Balancing SDN Control Plane Availability and Traffic Engineering Efficiency in Data Centers"
date: 2025-04-16T00:02:00-04:00
draft: false
---

[Jiaxin Lin]({{< relref "2025-04-15-031524-university_of_texas_austin.md#jiaxin-lin" >}})

**Modern spine‑free [data center networks]({{< relref "2025-04-16-000645-data_center_networks.md" >}}) need both \*high availability** and **tight traffic‑engineering (TE) control**. The paper shows that today’s “physical sharding” approach to SDN control‑plane partitioning hurts TE badly in those topologies—and proposes ****virtual slicing**** as a clean fix.\*

---


## Why the paper matters {#why-the-paper-matters}

[spine-free]({{< relref "2025-04-16-000741-spine_free_architecture.md" >}}) fabrics (e.g., Google’s Gemini/Jupiter‑evolving) replace the Clos spine with a flat optical mesh to cut cost and power. Operators still partition the SDN control plane (so one bad controller can’t sink the whole fabric) and still rely on centralized TE to keep link utilization low. The authors reveal that the two goals **conflict**: the way links are carved into disjoint shards creates asymmetric, capacity‑fragmented sub‑topologies that TE can’t optimize well.

---


## Key idea — **Virtual slicing** {#key-idea-virtual-slicing}

1.  **Slice switch resources, not links**
    -   Each switch’s flow‑table &amp; group‑table space is divided into **M** logical “virtual slices.”
    -   Every slice’s controller sees the **entire** physical topology, but only programs 1/M of the forwarding entries.
2.  **Label‑based routing**
    -   Source ToR hashes each flow, pushes a slice‑specific label, and uses WCMP weights to split traffic across slices.
    -   Interior switches forward on the label; the egress block pops it.
    -   If a controller dies, ToRs simply stop emitting that slice’s label—traffic migrates instantly to healthy slices without losing physical capacity.

---


## Reference List {#reference-list}

1.  Chang, B., He, K., Chen, S. S., Lin, J., Zhang, M., Wu, W., &amp; Akella, A. (2024, October). Balancing Sdn Control Plane Availability and Traffic Engineering Efficiency in Data Centers. In 2024 IEEE 32nd International Conference on Network Protocols (ICNP) (pp. 1-12). IEEE.
