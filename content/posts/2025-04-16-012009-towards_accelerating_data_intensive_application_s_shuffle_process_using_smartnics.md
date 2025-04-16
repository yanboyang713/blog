---
title: "Towards Accelerating Data‑Intensive Application’s Shuffle Process Using SmartNICs"
date: 2025-04-16T01:20:00-04:00
draft: false
---

[Jiaxin Lin]({{< relref "2025-04-15-031524-university_of_texas_austin.md#jiaxin-lin" >}})

**Problem:** In frameworks such as Spark, the **shuffle** (all‑to‑all data exchange between map and reduce tasks) dominates job time and host‑CPU cycles because it involves
(i) fine‑grained partitioning,
(ii) expensive operators (filter, aggregate, sort), and
(iii) billions of tiny I/O requests.


## **Key idea – SmartShuffle.** {#key-idea-smartshuffle-dot}

Move most of the shuffle pipeline onto programmable SmartNICs while keeping the host free for the rest of the job:

1.  **Co‑ordinated two‑level offload.**
    -   Map‑side NIC: merges mapper output, performs first‑level partitioning and optional filter/aggregate/sort.
    -   Reduce‑side NIC: collects fragments from all mappers, finishes aggregation/sort, materialises large contiguous blocks for the reducer.

2.  **Liquid offloading.**
    -   **Partial offload + spilling:** when on‑NIC DRAM (4‑16 GB) is tight, stateful operators stream intermediate results out instead of blocking.
    -   **Dynamic migration:** if NIC cores become the bottleneck, a rate‑based policy shifts part of the workload back to host threads until the NIC’s buffer fill‑rate stabilises.

3.  **On‑NIC traffic scheduler.**
    Batches DMA/RDMA transfers, segments oversized chunks, and load‑balances data among worker threads, drastically cutting the number of network and disk I/Os.


## Implementation &amp; evaluation {#implementation-and-evaluation}

-   4 × Spark executors, each equipped with an **8‑core ARM A72 Broadcom Stingray PS225 25 GbE SoC SmartNIC**.
-   4 K LoC C++ on the NIC, 2 K LoC Scala plugin for Spark.
-   Benchmarks: Hadoop BigData (TeraSort, TeraCount, PageRank), and TPC‑H queries.


## Results: {#results}

-   Job completion time ↓ 10–30 % (up to 40 % vs Spark RDMA)
-   Host‑CPU time ↓ 30–70 % and shuffle I/O requests reduced by orders of magnitude.

****Which SmartNICs are actually used?****
All prototypes and experiments run ****exclusively on Broadcom’s Stingray SoC SmartNIC (PS225, 25 GbE)****. Other NICs such as NVIDIA BlueField and Intel IPU are discussed only for context; they are **not** used in the evaluation.

****Take‑away:**** By treating the SmartNIC as a programmable “middle stage” between map and reduce tasks and by dynamically balancing work across host and NIC, SmartShuffle turns the shuffle bottleneck into an opportunity, delivering significant CPU and job‑time savings without modifying application code.
