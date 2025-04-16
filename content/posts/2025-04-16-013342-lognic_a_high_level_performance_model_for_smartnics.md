---
title: "LogNIC: A High-Level Performance Model for SmartNICs"
date: 2025-04-16T01:33:00-04:00
draft: false
---

[Jiaxin Lin]({{< relref "2025-04-15-031524-university_of_texas_austin.md#jiaxin-lin" >}})

The paper **"LogNIC: A High-Level Performance Model for SmartNICs"** proposes a packet-centric analytical model to help developers reason about the performance of SmartNIC-offloaded applications **without deep hardware knowledge**. Here's a summary and information about the SmartNICs it uses:

---


## \*Problem {#problem}

SmartNICs (Smart Network Interface Cards) are widely used in modern data centers for offloading packet processing tasks from CPUs. However, **developing optimized SmartNIC applications is complex**, due to hardware heterogeneity, overlapping computation/I/O, and traffic-dependent performance.


## \*Solution {#solution}

The authors introduce **LogNIC**, a **high-level performance model** for SmartNICs that abstracts hardware details and models performance based on **packet traversal** through execution graphs comprising compute engines, memory hierarchies, and interconnects.

**Key Features of LogNIC:**

-   Models **latency and throughput** using a **directed acyclic graph (DAG)** abstraction of SmartNIC execution.
-   Considers interleaved traffic, multiple tenants, and accelerator variability.
-   Offers both **estimation mode** (predicts performance) and **optimizer mode** (suggests configurations to meet performance goals).
-   Inspired by and extends architectural models like Roofline, LogCA, and Accelerometer.

**Evaluation Results:**

-   Validated using real SmartNICs, LogNIC accurately estimates performance with **&lt;1% error** in some scenarios.
-   It helps optimize parallelism and placement decisions, improving **throughput by up to 36.4%** and **latency by up to 22.8%**.
-   Can also guide early-stage **hardware design decisions**.


## Which SmartNICs Are Used? {#which-smartnics-are-used}

The authors evaluated LogNIC on the following hardware:

1.  [Marvell LiquidIO-II CN2360]({{< relref "2025-04-16-064746-marvell.md#marvell-liquidio-ii-cn2360" >}})
    -   25GbE
    -   16×1.5GHz cnMIPS cores
    -   Accelerator-rich (e.g., MD5, CRC, ZIP, HFA)
    -   Bump-in-the-wire SmartNIC
2.  [NVIDIA/Mellanox BlueField-2 DPU]({{< relref "2025-04-15-102932-nvidia_bluefield_dpus.md#nvidia-mellanox-bluefield-2-dpu" >}})
    -   100GbE
    -   8×2.5GHz ARM A72 cores
    -   General-purpose SmartNIC with programmable cores
    -   Off-path SmartNIC
3.  [Broadcom Stingray PS1100R]({{< relref "2025-04-16-012816-broadcom.md#broadcom-stingray-ps1100r" >}})
    -   100GbE
    -   8×3.0GHz ARM A72 cores, 8GB DDR4
    -   Supports NVMe-oF workloads
    -   Off-path SmartNIC
4.  **[PANIC]({{< relref "2025-04-16-065925-panic_a_high_performance_programmable_nic_for_multi_tenant_networks.md" >}}) (Academic Prototype)**
    -   Used for design space exploration
    -   Not a commercial product, but demonstrates configurability in research settings
