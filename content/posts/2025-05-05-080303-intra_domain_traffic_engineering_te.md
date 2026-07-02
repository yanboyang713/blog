---
title: "intra-domain traffic engineering (TE)"
date: 2025-05-05
draft: false
---

Intra-domain Traffic Engineering (TE) refers to the methods and practices used within a single administrative network domain (like a single ISP or enterprise network) to optimize the utilization of network resources and enhance overall network performance.

**Key points about Intra-domain Traffic Engineering:**


## **Scope:** {#scope}

-   Operates within a single Autonomous System (AS) or administrative network.
-   Does not involve cooperation or communication between different ISPs or domains.


## **Goals:** {#goals}

-   Maximize efficient use of network resources (e.g., bandwidth, links, routers).
-   Balance network load to avoid congestion.
-   Minimize latency, jitter, and packet loss.
-   Ensure reliability and robustness by adapting to link failures or network issues.


## **Methods and Technologies:** {#methods-and-technologies}

-   Routing protocols adjustments (e.g., OSPF, IS-IS).
-   Multiprotocol Label Switching (MPLS) for explicit route selection.
-   Software-Defined Networking (SDN) for dynamic and programmable path management.
-   Network monitoring and measurement tools for traffic load assessment and route optimization.


## **Limitations:** {#limitations}

-   Typically performed periodically (e.g., every few minutes or hours).
-   Primarily optimized around aggregate traffic demands and utilization metrics rather than per-flow QoS demands.
-   Does not handle end-to-end QoS across multiple domains or WANs (inter-domain).


## **Examples of Systems/Implementations:** {#examples-of-systems-implementations}

-   Google's B4 WAN.
-   Microsoft SWAN (Software-driven WAN).
-   Facebook's Express Backbone (Espresso).

**Contrast with Inter-domain TE:**

-   **Intra-domain TE**: Within a single administrative domain, optimizing internal resource usage.
-   **Inter-domain TE**: Across multiple domains (e.g., across different ISPs), optimizing path selection based on inter-domain routing protocols like BGP.
