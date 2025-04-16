---
title: "PANIC: A High-Performance Programmable NIC for Multi-tenant Networks"
date: 2025-04-16T06:59:00-04:00
draft: false
---

[Jiaxin Lin]({{< relref "2025-04-15-031524-university_of_texas_austin.md#jiaxin-lin" >}})

The paper **"PANIC: A High-Performance Programmable NIC for Multi-tenant Networks"** (OSDI 2020) introduces a novel programmable NIC design—PANIC—to address the limitations of current SmartNICs in multi-tenant data center environments.

---


## Summary\* {#summary}

**PANIC** is designed to support:

-   **High throughput and low latency**
-   **Flexible and dynamic offload chaining**
-   **Support for heterogeneous offload engines** (hardware accelerators, embedded CPUs, FPGAs)
-   **Multi-tenant isolation**
-   **Offloads with variable or sub-line-rate performance**

---


## Key Components of PANIC {#key-components-of-panic}

1.  **RMT Pipeline**: Uses match-action tables (like in programmable switches) to determine offload chains per-packet.
2.  **Central Scheduler**: Ensures fair scheduling, load balancing, and tenant isolation.
3.  **Switching Fabric**: High-bandwidth interconnect (crossbar-based) for routing between offloads at line rate.
4.  **Compute Units**: Encapsulated offloads, which can be:
    -   Hardware accelerators (e.g., AES, SHA)
    -   Embedded RISC-V cores for general-purpose tasks

---


## **Evaluation and Results** {#evaluation-and-results}

-   Prototyped on an **FPGA** using the **Alpha Data ADM-PCIE-9V3** SmartNIC
    -   FPGA: **Xilinx Virtex UltraScale+ VU3P-2**
-   Demonstrated:
    -   **100 Gbps throughput**
    -   **Sub-microsecond scheduling latency (&lt;0.8 μs)**
    -   Better isolation and performance over pipeline-based NICs
-   Example Offloads:
    -   AES-256 encryption (HW: 38.4 Gbps vs CPU: 0.154 Gbps)
    -   SHA1 authentication (HW: 113 Gbps vs CPU: 0.192 Gbps)
    -   Neural Network inference (HW: 120 Gbps, 66 ns delay)

---


## Which SmartNICs are used? {#which-smartnics-are-used}

-   The PANIC design was **prototyped on the FPGA-based** [Alpha Data ADM-PCIE-9V3]({{< relref "2025-04-16-075548-alpha_data.md#alpha-data-adm-pcie-9v3" >}}) SmartNIC, equipped with:
    -   **Xilinx UltraScale+ FPGA (VU3P-2)**
-   **No commercial ASIC SmartNIC** (like Mellanox BlueField or Netronome) was used.
-   PANIC is intended for eventual **ASIC implementation**, and their design aligns with ASIC feasibility analysis (e.g., crossbar, PIFO, etc.).

---


## Why use Alpha Data ADM-PCIE-9V3? {#why-use-alpha-data-adm-pcie-9v3}

-   Offers **FPGA reconfigurability** for prototyping
-   Allows integration of:
    -   Custom RMT pipeline
    -   Central scheduler
    -   Compute units (both HW accelerators and [RISC-V]({{< relref "2024-01-26-232655-risc_v.md" >}}) cores)


## Reference List {#reference-list}

1.  <https://www.usenix.org/conference/osdi20/presentation/lin>
