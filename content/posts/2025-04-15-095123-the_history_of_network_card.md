---
title: "The history of Network Card"
date: 2025-04-15T09:51:00-04:00
draft: false
---

This article will introduce the three stages of network card development, and compare traditional SmartNICs and DPU SmartNICs to explore the future of network performance.


## Three Stages of Network Card Development {#three-stages-of-network-card-development}

Traditional data centers use von Neumann architecture, with data sent to the CPU for processing. With the rapid development of data centers, the processing rate of CPU can no longer meet the requirements of data processing. The computing architecture has changed from the CPU-centered Onload mode to the data-centered Offload mode, and the important task of reducing the load on the CPU falls on the network card, which has also promoted the rapid development of the network card. Its development can be divided into three stages.


### Stage 1: Basic Function Network Card {#stage-1-basic-function-network-card}

Basic function network cards originally provide 2x10G or 2x25G bandwidth throughput and have less hardware offloading capabilities. In the cloud platform virtualization network, there are three main ways for the basic function network card to provide network access to the virtual machine (VM): the operating system kernel driver takes over the network card and distributes network traffic to the VM; the OVS-DPDK Take over the network card and distribute network traffic to the VM; and provide network access capabilities to the VM through SR-IOV in high-performance scenarios.


### Stage 2: Hardware Offloading Network Card {#stage-2-hardware-offloading-network-card}

Hardware offload network cards can be considered the first generation of SmartNICs, possessing rich hardware offloading capabilities. Typical examples include OVS Fastpath hardware offload, RDMA network hardware offload based on RoCEv1 and RoCEv2, and data plane offload for secure transmission. During this period, SmartNICs primarily focused on offloading the data plane.
Taking NVIDIA's products as an example, the ConnectX-6DX network card serves as a SmartNIC. This card is primarily designed to accelerate critical workload applications in data centers, such as security, virtualization, SDN/NFV, big data, machine learning, and storage. It boasts powerful RDMA capabilities, allowing network traffic to bypass the CPU, avoiding the use of TCP, and eliminating additional data copies to improve performance and free up CPU cores.


### Stage 3: DPU SmartNIC {#stage-3-dpu-smartnic}

DPU is a newly emerged specialized processor, positioned as the "third main chip" in data centers, following CPU and GPU. It can construct its bus system to effectively control and manage other devices, making it a truly centralized chip. Its emergence aims to address three aspects, encompassing five major challenges in data centers:
Inter-node issues such as low server data exchange efficiency and unreliable data transmission.
Intra-node concerns include low execution efficiency of data center models, inefficient I/O switching, and inflexible server architecture.
Network system insecurities.
For more information about DPU, please refer to DPU: One of the Three Pillars of Computing Going Forward.
<https://www.fs.com/blog/dpu-one-of-the-three-pillars-of-computing-going-forward-3761.html>


### Traditional SmartNIC vs [DPU SmartNIC]({{< relref "2025-04-10-011945-smartnic.md" >}}) {#traditional-smartnic-vs-dpu-smartnic--2025-04-10-011945-smartnic-dot-md}

Traditional SmartNIC and DPU SmartNIC are products of different stages of the evolution path of network cards. The two differences in architecture, functions, and implementation methods have various applicable scenarios.
Implementation Differences: SmartNIC achieves partial offloading, only offloading the datapath while the control plane remains on the host CPU. Overall, SmartNIC offloading is a collaborative process within the system. DPU achieves complete offloading, with the server's datapath and control plane running on the embedded CPU inside the DPU. DPU implementation includes software offloading and hardware acceleration, utilizing various hardware engines. In short, DPU involves collaboration between two systems, offloading one system to another running entity and interacting through specific interfaces.
Functional Differences: Traditional SmartNIC supports offloading, while DPU is a programmable intelligent NIC. The evolution from offloading NIC to programmable intelligent NIC has led to increasingly powerful functionality and higher flexibility. Programmable NIC supports eBPF offloading and mixed programming with P4/C language, enabling offloading acceleration for all underlying I/O, including networking, storage, security, virtualization, and more.
Different Application Scenarios: SmartNIC is suitable for mainstream OVS/VROUTER offloading, host overlay scenarios, gateway security, and similar use cases. DPU is suitable for offloading and acceleration of various general tasks and elastic acceleration scenarios for specific business needs, such as container environments, load balancing, network security, and advanced customized networking.

| Type         | Implementation      | Function            | Application                                                                                        |
|--------------|---------------------|---------------------|----------------------------------------------------------------------------------------------------|
| SmartNIC     | Partial offloading  | Supports offloading | Mainstream OVS/VROUTER offloading, host overlay scenarios, gateway security, and similar use cases |
| DPU SmartNIC | Complete offloading | Programmable        | Container environments, load balancing, network security, and advanced customized networking       |


## Conclusion {#conclusion}

While traditional SmartNICs play an important role in traditional network environments, they have limitations when it comes to handling large-scale data processing. DPU SmartNICs, as an innovative solution, bring new possibilities for enhancing network performance, they are expected to become the mainstream choice for network performance optimization and find widespread applications across various industries.


## Reference List {#reference-list}

1.  <https://www.fs.com/blog/traditional-smartnic-vs-dpu-smartnic-5408.html>
