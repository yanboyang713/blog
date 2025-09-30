---
title: "Setting Up EVPN on Proxmox SDN"
date: 2025-09-28
draft: false
---

In the ever-evolving landscape of network virtualization, Ethernet VPN (EVPN) and Virtual Extensible LAN (VXLAN) have emerged as powerful technologies for creating scalable and efficient network overlays. EVPN, combined with VXLAN, offers robust support for multi-tenancy and simplifies the management of large, complex networks. In this guide, we'll walk through the process of setting up EVPN on [Proxmox VE]({{< relref "20230228043925-proxmox_ve.md" >}}) SDN, leveraging the advanced features of Proxmox VE to create a flexible and resilient network infrastructure.


## Understanding EVPN and VXLAN {#understanding-evpn-and-vxlan}


### What is EVPN? {#what-is-evpn}

Ethernet VPN (EVPN) is a modern control plane technology designed to carry Layer 2 Ethernet traffic over a wide area network (WAN) using protocols such as BGP (Border Gateway Protocol). EVPN provides efficient MAC address learning and distribution, reducing the need for traditional flooding mechanisms and enhancing scalability. It supports advanced features like active-active multihoming, MAC mobility, and ARP suppression, making it ideal for multi-tenant environments.


### What is VXLAN? {#what-is-vxlan}

Virtual Extensible LAN (VXLAN) is a network virtualization technology that encapsulates Layer 2 Ethernet frames within Layer 3 UDP packets. This encapsulation allows for the creation of virtual networks that can span large Layer 3 networks, enabling greater scalability and flexibility. VXLAN uses a 24-bit segment ID, known as the VXLAN Network Identifier (VNI), to uniquely identify each virtual network, supporting up to 16 million unique VNIs.


## Setting Up EVPN on Proxmox SDN {#setting-up-evpn-on-proxmox-sdn}

To harness the power of EVPN and VXLAN in your Proxmox environment, follow these steps to set up EVPN on Proxmox SDN.


### Prerequisites {#prerequisites}

-   Proxmox VE 8.1 or later: Ensure you are running Proxmox VE 8.1 or later, as the core SDN packages are installed by default.
-   FRRouting: Install the frr-pythontools package on all nodes for advanced routing setups.
-   Network Configuration: Ensure your network interfaces are correctly configured and the ifupdown2 package is installed.

<!--listend-->

```bash
apt update
apt install libpve-network-perl frr-pythontools dnsmasq
```


## Reference List {#reference-list}

1.  <https://bennetgallein.de/blog/setting-up-evpn-on-proxmox-sdn-a-comprehensive-guide>
