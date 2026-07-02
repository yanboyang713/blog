---
title: "Enabling Portable and High-Performance SmartNIC Programs with Alkali"
date: 2025-04-15T22:47:00-04:00
draft: false
---

[Jiaxin Lin]({{< relref "2025-04-15-031524-university_of_texas_austin.md#jiaxin-lin" >}})

**Alkali: Enabling Portable &amp; High‑Performance SmartNIC Programs (NSDI ’25) — Executive Summary**

---


## Why Alkali? {#why-alkali}

SmartNICs from Netronome, NVIDIA, AMD/Xilinx, Pensando and others look alike on a datasheet but differ wildly in:

-   **Parallel compute units** (48 micro‑engines vs. 8 ARM cores vs. hundreds of FPGA modules)
-   **Memory hierarchies** (on‑chip CAM/SRAM/DRAM mixes)
-   **Vendor‑specific SDKs / languages**

Porting an application from one NIC to another therefore means **rewriting low‑level code and re‑tuning parallelism and memory placement by hand**. Existing frameworks (e.g., P4 back‑ends, Floem, ClickNP) target a single architecture and still leave parallelism decisions to the programmer.

---


## Core Idea {#core-idea}

Alkali is a **cross‑target compilation framework** that lets developers write a **single‑threaded, run‑to‑completion** program (today: a safe subset of C) and automatically produces an ****optimized, pipelined &amp; replicated implementation**** for four very different NIC families (Agilio, BlueField‑2, AMD Alveo FPGA, PANIC ASIC prototype).

---


## Prototype &amp; Back‑ends {#prototype-and-back-ends}

-   **20 KLoC** MLIR‑based compiler + **4 KLoC** runtimes
-   Back‑ends: Micro‑C (Agilio), LLVM/DPDK (BlueField‑2), Verilog (FPGA), bare‑metal RISC‑V (PANIC). Mapping table in Appendix B shows how handlers, tables, etc., map to each architecture.

---


## Evaluation Highlights {#evaluation-highlights}

Five representative NIC workloads were re‑implemented in Alkali‑C (28–289 LoC vs. 128–1312 LoC in vendor SDKs, a 5‑10× reduction)

---


## Limitations &amp; Future Work {#limitations-and-future-work}

-   No automatic handling of **run‑time workload shifts** (requires recompilation)
-   **Lock‑based sharing** not yet modeled; current algorithms rely on partitionable state
-   Performance model is intentionally simple; authors plan ML‑based refinements.

---


## Take‑away {#take-away}

Alkali shows that **a single, NIC‑aware IR plus an iterative optimization loop can deliver portable, near‑expert performance** across SoC, FPGA and ASIC SmartNICs, slashing development effort and vendor lock‑in. The work opens the door to higher‑level languages (Python, P4) and more dynamic optimisation in future.
