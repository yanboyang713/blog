---
title: "RAN architecture"
tags: ["RAN", "Open-RAN", "O-RAN"]
draft: false
---

## Overview {#overview}

The radio access network (RAN) is the part of a [Cellular]({{< relref "20230614221419-cellular_network.md" >}}) system that connects user equipment to the rest of the network over the air interface. In a [5G]({{< relref "20230616140740-5g.md" >}}) system, the RAN sits between devices and the [5G core network]({{< relref "20230602124354-5g_core_network.md" >}}), handling radio transmission, scheduling, mobility at the network edge, and transport toward the core.


## RAN Evolution {#ran-evolution}

RAN architecture has evolved from [D-RAN (traditional RAN)]({{< relref "20230614154151-d_ran_traditional_ran.md" >}}) to [C-RAN (Cloud RAN)]({{< relref "20230614154328-c_ran_cloud_ran.md" >}}) to [vRAN (Virtual RAN)]({{< relref "20230614154422-vran_virtual_ran.md" >}}) to Open RAN.

The common direction of this evolution is increased disaggregation:

-   D-RAN keeps most processing tightly integrated at the cell site
-   C-RAN centralizes more baseband functionality
-   vRAN virtualizes those functions on general-purpose compute
-   Open RAN adds standardized open interfaces and multivendor interoperability goals

{{< figure src="https://telcocloudbridge.com/wp-content/uploads/2021/04/image-13-768x258.png?ezimgfmt=ng:webp/ngcb1" >}}


## Open-RAN (O-RAN) {#open-ran--o-ran}

Open RAN, often written as O-RAN in the context of the [O-RAN Alliance](https://www.o-ran.org/), extends the idea of [vRAN (Virtual RAN)]({{< relref "20230614154422-vran_virtual_ran.md" >}}) by trying to reduce vendor lock-in through open interfaces and clearer functional splits.

In a traditional closed deployment, major RAN components such as the [Radio Unit (RU)](#radio-unit--ru), [Distributed Unit (DU)](#distributed-unit--du), and [Central Unit (CU)](#central-unit--cu) often come from the same vendor. Open RAN reframes these as O-RU, O-DU, and O-CU and aims to let operators mix components that conform to common specifications.

{{< figure src="https://telcocloudbridge.com/wp-content/uploads/2021/04/image-4.png?ezimgfmt=ng:webp/ngcb1" >}}

This matters because Open RAN is not just a hardware split. It is an architectural and operational model built around:

-   disaggregated RU, DU, and CU functions
-   open interfaces between components
-   software control and automation
-   support for multivendor integration


## Open-Source RAN Projects {#open-source-ran-projects}

In practice, three open-source RAN software stacks often come up in labs and testbeds:

-   [srsRAN]({{< relref "2024-10-28-221714-srsran.md" >}})
-   [OpenAirInterface (OAI)]({{< relref "2024-10-28-224828-openairinterface.md" >}})
-   [OCUDU]({{< relref "2026-04-06-193729-ocudu.md" >}})

These projects matter because they provide concrete software implementations of disaggregated RAN ideas rather than only architecture diagrams.

-   [srsRAN]({{< relref "2024-10-28-221714-srsran.md" >}}) is widely used for practical 5G gNB experimentation, CU/DU deployment, and interoperability work in labs and private-network environments.
-   [OpenAirInterface (OAI)]({{< relref "2024-10-28-224828-openairinterface.md" >}}) is a major research-oriented open RAN platform with strong relevance to CU/DU separation, O-RAN-style deployment, and end-to-end experimentation.
-   [OCUDU]({{< relref "2026-04-06-193729-ocudu.md" >}}) is another open-source disaggregated RAN option centered on split-RAN deployment and O-RAN-style software decomposition.

Taken together, these projects make the abstract ideas in RAN architecture easier to study in real systems:

-   how CU, DU, and RU are separated
-   how split options affect deployment design
-   how open interfaces and multivendor integration behave in practice


## O-RAN Functional Context {#o-ran-functional-context}

Important O-RAN-related building blocks include:

-   [SMO (Service Management and Orchestration)]({{< relref "20230228052825-smo_service_management_and_orchestration.md" >}})
-   [Near real-time RIC]({{< relref "20230228052933-near_real_time_ric.md" >}})
-   [Radio Unit (RU)](#radio-unit--ru)
-   [O-RAN Alliance Split Architecture](#o-ran-alliance-split-architecture)

In practical terms, the RAN handles radio access while the core handles subscriber management, policy, and connectivity to external data networks. Open RAN focuses on how the radio-side system is decomposed, managed, and opened to interoperable implementations.


## O-RAN Alliance Split Architecture {#o-ran-alliance-split-architecture}

The O-RAN split model describes how radio-access functions are partitioned across the radio unit, distributed unit, and central unit. These splits matter because they determine:

-   where real-time processing happens
-   what latency and bandwidth the transport network must support
-   how much functionality can be centralized
-   how easily operators can mix vendors and scale parts of the RAN independently

{{< figure src="https://www.radisys.com/sites/default/files/2021-06/x5G,P20Diagram,P2006022021B.png.pagespeed.ic.fPnAZnu5Jj.webp" >}}


### How to Read the Splits {#how-to-read-the-splits}

At a high level:

-   moving more functions toward the RU reduces transport demands but leaves more processing at the edge
-   moving more functions toward the DU or CU increases centralization but usually tightens timing and fronthaul requirements
-   no split is universally "best"; the choice depends on transport constraints, hardware availability, synchronization, and operational goals

For quick orientation:

| Split | Main boundary                         | Typical takeaway                                                                         |
|-------|---------------------------------------|------------------------------------------------------------------------------------------|
| 7.2   | lower PHY / higher PHY style boundary | common in O-RAN and vRAN because it balances flexibility and transport feasibility       |
| 6     | MAC / PHY style boundary              | pushes more processing toward the edge and can fit smaller or more localized deployments |
| 2     | CU / DU boundary over F1              | popular higher-layer disaggregation point with more relaxed transport constraints        |
| E1    | CU-CP / CU-UP boundary                | separates control and user plane inside the CU for independent scaling                   |


### Disaggregation {#disaggregation}


#### split 7.2 (vRAN model) {#split-7-dot-2--vran-model}

Split 7.2 is one of the most common O-RAN deployment models. In this arrangement, the RU keeps RF and low-PHY work close to the antenna, while the DU takes higher-PHY and other lower-layer processing. The CU remains responsible for upper-layer functions.

It is widely used because it offers a practical balance:

-   more disaggregation than a tightly integrated base station
-   more realistic transport requirements than pushing every function centrally
-   good alignment with current O-RAN and vRAN implementations


#### split 6 (small cell model) {#split-6--small-cell-model}

Split 6 places more functionality on the distributed side than higher-layer splits and is often discussed around small-cell-style deployments. Conceptually, it keeps less centralized coordination than split 2 while relaxing some of the tighter transport demands seen in lower-layer disaggregation.

The tradeoff is straightforward:

-   less dependence on very demanding fronthaul
-   more compute and implementation complexity closer to the cell site


#### split 2 (F1 split) {#split-2--f1-split}

Split 2 is the CU/DU boundary commonly associated with the F1 interface. It separates higher-layer CU functions from DU functions and is widely used in disaggregated RAN systems where CU and DU can be placed independently.

This split is attractive because:

-   transport requirements are generally easier than lower-layer splits
-   it allows CU functions to be centralized across multiple DUs
-   it fits well with practical multi-site deployments


#### E1 split (CU-CP and CU-UP) {#e1-split--cu-cp-and-cu-up}

The E1 split separates the CU control-plane and user-plane functions into CU-CP and CU-UP. This improves flexibility by allowing control and user-plane processing to scale or be placed independently.

This matters when an operator wants:

-   separate scaling of signaling and user traffic handling
-   cleaner control/user-plane decomposition inside the RAN
-   more flexible placement of CU functions


### Components {#components}


#### Radio Unit (RU) {#radio-unit--ru}

The Radio Unit handles low-level radio work such as RF processing, beamforming, filtering, amplification, and conversion between radio-frequency signals and baseband-oriented data streams. Operationally, it sits near the antenna and strongly influences the coverage characteristics of the cell.

[ns-O-RAN]({{< relref "20221220075950-ns_o_ran.md" >}})

RRU: Remote Radio Unit interfaces with an [antenna]({{< relref "20230614155634-antenna.md" >}}) on one end and [BBU]({{< relref "20230614155229-bbu.md" >}})-side processing on the other. It connects toward centralized or distributed processing through fronthaul technologies such as CPRI and handles RF conversion, filtering, and amplification.

<!--list-separator-->

-  [5G NR frequency bands]({{< relref "2024-10-29-233837-5g_nr_frequency_bands.md" >}})

<!--list-separator-->

-  O-RU Products

    -   [Benetel]({{< relref "2024-10-30-130349-benetel.md" >}})


#### Distributed Unit (DU) {#distributed-unit--du}

The Distributed Unit handles time-sensitive lower-layer work, commonly including RLC, MAC, and parts of the PHY layer. It is usually deployed closer to the [Radio Unit (RU)](#radio-unit--ru) to keep latency and transport demands manageable.

In practice, the DU is where operators often balance:

-   edge placement for timing-sensitive functions
-   pooling efficiency across multiple cells
-   interoperability with RU implementations and selected split options


#### Central Unit (CU) {#central-unit--cu}

The Central Unit handles higher-layer protocol functions such as RRC and PDCP, and SDAP in 5G deployments. A single CU can connect to multiple DUs, and it may be colocated with the DU or placed farther away depending on the transport and deployment model.

The CU is the part most naturally centralized when the goal is:

-   coordination across multiple DUs
-   simplified higher-layer management
-   independent scaling of control-plane and user-plane functions when E1 is used


## Reference List {#reference-list}

1.  <https://telcocloudbridge.com/blog/c-ran-vs-cloud-ran-vs-vran-vs-o-ran/>
2.  <https://www.redhat.com/architect/mobile-architecture-cloud-ran>
3.  Polese, M., Bonati, L., D'Oro, S., Basagni, S., &amp; Melodia, T. (2022). Understanding O-RAN: Architecture, interfaces, algorithms, security, and research challenges. arXiv preprint arXiv:2202.01032.
4.  <https://orandownloadsweb.azurewebsites.net/specifications>
5.  <https://www.radisys.com/blog/open-ran-functional-splits-explained>
