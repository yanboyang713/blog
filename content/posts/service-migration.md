---
title: "Service Migration in Cellular Networks — To-do List"
draft: false
---

## PROJ Service Migration in Cellular Networks {#service-migration-in-cellular-networks}

This project has three focus parts: Cellular Networks Testbed, LLMs, and optimization algorithms.


### Cellular Networks Testbed {#cellular-networks-testbed}

{{< figure src="https://www.mdpi.com/jsan/jsan-14-00079/article_deploy/html/images/jsan-14-00079-g002-550.jpg" >}}

Image from: Real-Time Service Migration in Edge Networks: A Survey

The testbed architecture spans four tiers: Central Cloud, Regional MEC, Aggregation MEC, and Local MEC.
**Primary focus: Regional MEC and Aggregation MEC.**
Regional MEC, Aggregation MEC, and Local MEC should be deployed to my [PVE Testbed.]({{< relref "2025-04-10-010004-data_center_testbed_design.md" >}})

Each MEC site (or per Metro (metropolitan area) / PoP (Point of Presence)) needs its own control plane because:

-   Survivability: If WAN/backhaul drops, the site keeps running. A single, stretched cluster loses control-plane access and flakes.
-   Latency/etcd constraints: Kubernetes control-plane (etcd) hates WAN latency/packet-loss; cross-site RTTs &gt;~5–10 ms and jitter cause elections and outages.
-   Blast radius &amp; upgrades: Failures and rollouts stay local, enabling per-site upgrades.
-   Regulatory / tenancy: Site-level isolation simplifies policy and compliance.


#### TODO Central Cloud {#central-cloud}

&gt;200 km coverage | ≥50 ms RTT
Global control

Cloud (Azure)
High-level design: Azure Virtual WAN (Standard) with four hubs in a full inter-hub mesh. Regional spokes (AKS VNets) attach to their nearest hub; inter-hub routing provides global any-to-any.

Regions (paired for HA/DR):

-   East US 2 (VA) — primary; paired with Central US (closest to UVA)
-   Central US (IA) — DR for East US 2
-   West US 3 (AZ) — west capacity/DR; paired with East US
-   East US (VA) — additional east capacity and the formal pair for West US/West US 3

(All selected regions provide Availability Zones.)


#### TODO Regional MEC (e.g., Richmond PoP) {#regional-mec--e-dot-g-dot-richmond-pop}

50–200 km coverage | RTT to Aggregation 15–30 ms
Use cases: smart city, cloud gaming, content delivery
Components: SMF/AMF/PCF (control plane) + Regional UPF


#### TODO Aggregation MEC (Hub PoP) {#aggregation-mec--hub-pop}

10–50 km coverage | RTT to Local: 10–20 ms
Use cases: campus control, local CDN

[OKD]({{< relref "20230518171737-okd.md" >}}) (3 master nodes)
Components: SMF/AMF/PCF (control plane) + optional Aggregation UPF


#### TODO Local MEC {#local-mec}

1–5 km coverage | UE↔UPF target: ≤5–10 ms
Use cases: autonomous driving, industrial robots, real-time AR/XR

Components: Local UPF
Deploy two [OKD]({{< relref "20230518171737-okd.md" >}}) SNOs or [MicroShift]({{< relref "2025-10-22-202314-microshift.md" >}}) clusters (MEC-1: Campus South; MEC-2: Campus North)

N3 (gNB ⇄ UPF @ MEC): VLAN/VRF local to the site, low jitter
N6 (UPF ⇄ campus/ISP): routed toward the PoP


#### TODO gNodeB (UEs) {#gnodeb--ues}

<!--list-separator-->

-  Digital Twin

    Must implement N2, N3, and optionally Xn. Focus on mmWave.

    -   [NVIDIA AI Aerial]({{< relref "2024-11-20-021257-nvidia_ai_aerial.md" >}})
    -   [ns-O-RAN]({{< relref "20221220075950-ns_o_ran.md" >}})

<!--list-separator-->

-  Physical RAN

    CBRS band only

    -   [mosoLab]({{< relref "2024-10-29-004514-mosolab.md" >}}) (all-in-one)
    -   [baicells]({{< relref "2024-10-29-224840-baicells.md" >}}) (7.2 split)
    -   [USRP]({{< relref "20230608150156-usrp.md" >}}) (8 split)
    -   [Lite-On]({{< relref "2024-10-29-184819-lite_on.md" >}}) (7.2 split)


### LLMs {#llms}

Focus on multi-agent workflow design and LLM fine-tuning.


#### TODO LLM corpus {#llm-corpus}

<!--list-separator-->

-  3GPP Standards

    <https://www.3gpp.org/specifications-technologies>

<!--list-separator-->

-  ETSI = European Telecommunications Standards Institute

    Famous work includes ETSI MEC (edge computing) and the original ETSI NFV effort.

<!--list-separator-->

-  Kubernetes documentation

    <https://kubernetes.io/docs/home/>

<!--list-separator-->

-  OKD documentation

    <https://docs.okd.io/>

<!--list-separator-->

-  Cilium

    <https://docs.cilium.io/en/stable/network/kubernetes/index.html>


#### TODO LLM Serving {#llm-serving}

Deploy Three LLMs to Regional MEC or Aggregation MEC:

-   Custom LLM (based on [Google Gemma]({{< relref "2025-04-19-013506-google_gemma.md" >}})) fine-tuned on [UVA CS Slurm cluster]({{< relref "2025-08-28-210144-uva_cs_computing_environment.md#uva-cs-id-822ba079-5358-4814-94f5-66a7f741b41a-slurm-cluster" >}}).
-   [Codex CLI]({{< relref "2025-10-09-180435-coding_agent.md#codex-cli" >}}) / [Gemini CLI]({{< relref "2025-10-09-180435-coding_agent.md#gemini-cli" >}}) / [Droid CLI]({{< relref "2025-10-09-180435-coding_agent.md#droid-cli" >}})
-   Time-Series LLM (TBD)

<https://ai-on-openshift.io/generative-ai/llm-serving/>

<!--list-separator-->

- TODO  [Retrieval Augmented Generation (RAG)]({{< relref "2023-12-06-164642-retrieval_augmented_generation_rag.md" >}})

    [Org-roam MCP Server]({{< relref "2025-09-19-190143-org_roam_mcp_server.md" >}})

<!--list-separator-->

- TODO  The Multi-Agent Roster &amp; Roles

    | Agent                              | Location in Testbed                                   | Primary Role                                                                                                                                | Key Inputs                                                                                                      | Research Motivation                                                                                                                                                           |
    |------------------------------------|-------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | Mobility Predictor Agent (MPA)     | Aggregation MEC / Local MEC (Near-RT RIC/O-RAN Layer) | Context Generation: Provides real-time prediction of UE handover and mobility patterns to anticipate service relocation.                    | Real-time Radio KPIs (RSRP, RSRQ), Handover/Xn/N2 events, UE location/velocity.                                 | Proactive Migration: Essential for timely initiation of migration at the lowest latency tiers, ensuring QoE under high mobility.                                              |
    | MEC Resource Agent (RCA)           | All Managed MEC Sites (Local, Aggregation, Regional)  | Local State Reporting: Monitors the instantaneous resource utilization and available capacity of its local compute cluster (OKD/MicroShift) | CPU/Memory/GPU load, Available network bandwidth, K8s/OKD/MicroShift node metrics.                              | Survivability and Autonomy: Guarantees that every control-plane instance has local resource awareness, upholding isolation and independence                                   |
    | Migration Planner Agent (PLA)      | Regional MEC and Aggregation MEC (Control Plane)      | Decision-Making: Determines the optimal migration target, timing, and method based on its scope (Local → Local vs. Regional → Regional).    | Aggregated Predictions (MPA data), Resource Availability (RCA reports), Service SLOs, Migration Cost Model.     | Hierarchical/Decentralized Decision: Enables ultra-low-latency decision-making for local PoP movements and wide-area optimization, avoiding high Central Cloud RTT            |
    | State/Traffic Steering Agent (TSA) | Co-located with SMF/UPF                               | Execution &amp; Cutover: Executes the migration by coordinating state transfer and updating the 5G Core traffic rules                       | PLA's Decision (Target MEC ID), State Transfer Status, 5G Core N4/N11 APIs (for UPF/SMF control plane updates)  | Critical Service Continuity: Directly implements the necessary 5G Core control procedures (PSA Relocation/UL-CL) at all anchor points to shift traffic seamlessly             |
    | Policy Enforcement Agent (PEA)     | Central Cloud (Azure)                                 | Global Policy Management: Distributes high-level, long-term policies, cost objectives, and optimization models across all PLA instances     | Long-term historical data, Global business objectives, Failure tolerance settings, Regulatory/Tenancy policies. | Global Governance: Provides the top-level goals and learning feedback to the decentralized PLA instances, ensuring consistency and alignment with global business objectives. |


### TODO Optimization Algorithms {#optimization-algorithms}

Prioritize [time series forecasting]({{< relref "2024-02-27-111600-time_series_forecasting.md" >}}) and [decision making]({{< relref "2025-10-22-204121-decision_making.md" >}}) (TBD).
