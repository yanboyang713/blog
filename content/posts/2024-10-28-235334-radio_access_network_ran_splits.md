---
title: "O-RAN Alliance Split Architecture"
draft: false
---

{{< figure src="https://www.radisys.com/sites/default/files/2021-06/x5G,P20Diagram,P2006022021B.png.pagespeed.ic.fPnAZnu5Jj.webp" >}}


## Disaggergation {#disaggergation}


### split 7.2 (vRAN model) {#split-7-dot-2--vran-model}

Most commonly used. DU manages Physical Layer functions, while CU handles the rest.


### split 6 (small cell model) {#split-6--small-cell-model}


### split 2 (F1 split) {#split-2--f1-split}


### E1 split (CU-CP and CU-UP) {#e1-split--cu-cp-and-cu-up}


## Components {#components}


### Radio Unit (RU) {#radio-unit--ru}

Handles low-level radio functions like beamforming and RF (Radio Frequency) signals.

[ns-O-RAN]({{< relref "20221220075950-ns_o_ran.md" >}})

RRU: Remote Radio unit interfaces with an [antenna]({{< relref "20230614155634-antenna.md" >}}) on one end and [BBU]({{< relref "20230614155229-bbu.md" >}}) on the other. It connects to BBU through CPRI interface and converts RF signal into data signal and vice versa. Further, it does filtering and amplification of RF signal. In fact, it decides the “COVERAGE” of the system”

<https://telcocloudbridge.com/blog/c-ran-vs-cloud-ran-vs-vran-vs-o-ran/>


#### [5G NR frequency bands]({{< relref "2024-10-29-233837-5g_nr_frequency_bands.md" >}}) {#5g-nr-frequency-bands--2024-10-29-233837-5g-nr-frequency-bands-dot-md}


#### O-RU Products {#o-ru-products}

-   [Benetel]({{< relref "2024-10-30-130349-benetel.md" >}})


### Distributed Unit (DU) {#distributed-unit--du}

Manages Layer 2 (MAC/RLC) operations and can be deployed closer to users.

Distributed runs the RLC, MAC, and parts of the PHY layer. We normally place DU closer to [Radio Unit (RU)](#radio-unit--ru).

<https://telcocloudbridge.com/blog/c-ran-vs-cloud-ran-vs-vran-vs-o-ran/>


### Central Unit (CU) {#central-unit--cu}

Handles non-real-time functions (e.g., mobility management, session control).

Centralized Unit handles the RRC and PDCP layers (and SDAP in case of 5G) . One CU can connect to multiple DUs, CU can be co-located with [Distributed Unit (DU)](#distributed-unit--du) or far from [Distributed Unit (DU)](#distributed-unit--du).

<https://telcocloudbridge.com/blog/c-ran-vs-cloud-ran-vs-vran-vs-o-ran/>
