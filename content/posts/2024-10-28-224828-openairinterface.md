---
title: "OpenAirInterface (OAI)"
tags: ["OAI", "OpenAirInterface", "RAN", "5G"]
draft: false
---

## Overview {#overview}

OpenAirInterface (OAI) is an open-source mobile-network software project used to build and study 4G and 5G systems. In practice, it is especially important as a research and lab platform for open and software-defined radio access networks.

Within this note's context, OAI is most useful as:

-   an open RAN experimentation platform
-   a software implementation for 5G SA lab validation
-   a practical reference for CU/DU and split-RAN deployment work


## Why It Matters {#why-it-matters}

OAI is valuable because it gives researchers and builders direct access to an end-to-end cellular software stack rather than just documentation or black-box vendor gear. That makes it useful for:

-   validating [RAN architecture]({{< relref "20230614153814-ran_architecture.md" >}}) ideas
-   experimenting with disaggregation and open interfaces
-   testing interoperability with open cores and user equipment
-   learning how practical 5G radio software is assembled and deployed


## O-RAN RU Context {#o-ran-ru-context}

When OAI is used in O-RAN-style deployments, radio-unit compatibility and transport details become important. The note below is therefore a practical starting point when working with external radio units:

-   [Lite-On]({{< relref "2024-10-29-184819-lite_on.md" >}})
-   [baicells]({{< relref "2024-10-29-224840-baicells.md" >}})
-   [Benetel]({{< relref "2024-10-30-130349-benetel.md" >}})


## Practical Reading {#practical-reading}

If the goal is to use OAI in a lab or testbed, the most useful way to read the project is:

-   first understand the overall [RAN architecture]({{< relref "20230614153814-ran_architecture.md" >}})
-   then understand the split model, especially [split 7.2 (vRAN model)]({{< relref "20230614153814-ran_architecture.md#split-7-dot-2--vran-model" >}})
-   then focus on OAI-specific CU/DU deployment details
-   finally validate radio-unit, UE, and core-network interoperability


## Resources {#resources}

-   System requirements:
    <https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/system_requirements.md#o-ran-radio-units>
-   O-RAN fronthaul split 7.2 tutorial:
    <https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/ORAN_FHI7.2_Tutorial.md>


## Reference List {#reference-list}

1.  <https://openairinterface.org/>
