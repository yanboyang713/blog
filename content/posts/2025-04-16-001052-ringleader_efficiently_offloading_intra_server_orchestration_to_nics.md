---
title: "RingLeader: Efficiently Off‑loading Intra‑Server Orchestration to NICs (NSDI ’23)"
date: 2025-04-16T00:10:00-04:00
draft: false
---

[Jiaxin Lin]({{< relref "2025-04-15-031524-university_of_texas_austin.md#jiaxin-lin" >}})

[Alveo U280 Data Center Accelerator Card]({{< relref "2024-10-23-163703-amd_pensando.md#alveo-u280-data-center-accelerator-card" >}})

base line:
[Mellanox ConnectX-5 Ex 100Gb NIC]({{< relref "2025-04-15-102932-nvidia_bluefield_dpus.md#mellanox-connectx-5-ex-100gb-nic" >}})


## **Why it matters** {#why-it-matters}

Modern cloud servers juggle thousands of microsecond‑scale RPCs from many services. Ideal orchestration (load‑balancing, request scheduling, and fast CPU core re‑allocation) is centralized, but existing **software‑only** schemes burn cores and stall at high line‑rates. SmartNICs with on‑board ARM cores help little—they are too slow and still add latency. RingLeader asks: **what if we push (almost) the entire orchestration logic into the NIC’s datapath hardware?**

---


## Reference List {#reference-list}

1.  <https://www.usenix.org/conference/nsdi23/presentation/lin>
