---
title: "Efficient Multi-WAN Transport for 5G with OTTER"
date: 2025-04-16
draft: false
---

[Cellular Network]({{< relref "20230614221419-cellular_network.md" >}})


## Overview {#overview}

The paper addresses the challenge of efficiently routing cloudified 5G network traffic across multiple wide-area networks (WANs). As 5G networks increasingly deploy software-based network functions (NFs) and applications in cloud infrastructures, traffic traverses both operator WANs and cloud WANs, complicating the assurance of end-to-end Quality-of-Service (QoS).


## Problem Statement {#problem-statement}

Cloudification introduces complexity in guaranteeing QoS for performance-sensitive 5G traffic due to:

1.  **Dynamic Placement**:
    5G network functions (NFs) and applications dynamically placed in cloud data centers (DCs) and edge sites based on resource availability.

2.  **Inter-domain Traffic Engineering Limitations**:
    Current [intra-domain traffic engineering (TE)]({{< relref "2025-05-05-080303-intra_domain_traffic_engineering_te.md" >}}) approaches do not efficiently manage the complex requirements of inter-domain 5G traffic.

3.  **On-demand Routing Needs**:
    Traditional TE mechanisms, operating periodically, fail to handle real-time 5G flow demands.


## Solution: OTTER (Overlay Traffic Transport and Efficient Resource Allocation) {#solution-otter--overlay-traffic-transport-and-efficient-resource-allocation}

OTTER orchestrates 5G flow placement across multi-WAN overlays combining operator and cloud WANs. It has two main components:

-   **OTTER Controller**:
    Dynamically allocates network and compute resources, optimizing flow placement using real-time performance metrics (throughput, RTT, jitter, packet loss) and available compute resources.

-   **OTTER Orchestrator**:
    Implements scalable forwarding mechanisms to steer traffic along optimal multi-WAN paths, managing cloud resources and compute deployments.


## Contributions {#contributions}

-   **Optimization Algorithm**:
    OTTER employs a [linear-programming]({{< relref "2025-05-05-081102-linear_programming.md" >}})-based algorithm to allocate compute and network resources efficiently to satisfy diverse 5G flow requirements.
-   **Dynamic, Fine-Grained Routing**:
    Unlike traditional periodic TE solutions, OTTER dynamically adjusts flow placements, reacting to real-time network conditions and fine-grained QoS needs of different flows.
-   **Scalable Multi-WAN Overlay**:
    The overlay leverages cloud-native functionalities (VMs, VPN gateways, user-defined routing) without needing access to proprietary or private network data, facilitating large-scale deployments.


## Results and Evaluation {#results-and-evaluation}

Evaluations across two commercial cloud WANs (Azure and Google Cloud) demonstrate OTTER's effectiveness:

-   **Throughput**:
    Achieved a 13% higher average throughput, with maximum improvements up to 136% (adding 6–10 Gbps).
-   **Latency**:
    Reduced average round-trip time (RTT) by 15%, with maximum reductions up to 42 ms.
-   **Jitter and Packet Loss**:
    Reduced jitter by 45% on average; reduced packet loss from an average of 0.06% to below 0.001%.

Additionally, OTTER’s optimization method allocated 26–45% more bytes compared to greedy baseline methods, closely approximating the performance of an ideal but practically infeasible infinitely-fast optimizer.


## Conclusion {#conclusion}

OTTER demonstrates the significant potential of multi-WAN coordination and overlay-based orchestration in next-generation cloudified 5G networks, notably surpassing traditional WAN traffic management methods.

-   Capturing Complex Network Behaviors, real-world constraints and objectives that are often nonlinear in nature.
-   adapting to rapidly changing network conditions, such variable link capacities, energy consumption, and intricate QoS requirements
-   Improved Generalization and Adaptability, generalize from historical data to make informed decisions in previously unseen scenarios.

-   large-scale NLP problems in real-time (additional computational complexity)
-   require more sophisticated implementation strategies


## Reference List {#reference-list}

1.  <https://www.usenix.org/conference/nsdi25/presentation/hogan>
2.  <https://www.microsoft.com/en-us/research/publication/efficient-multi-wan-transport-for-5g-with-otter/?locale=fr-ca>
