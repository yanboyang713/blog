---
title: "Data Center Testbed Design"
draft: false
---

## Introduction and Objectives {#introduction-and-objectives}

This Proxmox VE–based, multi-node testbed supports research and prototyping across [Cellular Networking]({{< relref "20230614221419-cellular_network.md" >}}), [network measurement]({{< relref "2025-11-05-041036-network_measurement.md" >}}), [Software-Defined Networking (SDN)]({{< relref "2025-04-10-011054-software_defined_networking_sdn.md" >}}), and [Network Function Virtualization (NFV)]({{< relref "2025-04-10-011707-network_function_virtualization_nfv.md" >}}). It offers realistic north–south connectivity via dual WWAN gateways, flexible east–west segmentation using EVPN overlays, internal DNS, and a Kubernetes/OKD substrate for deploying [Aether 5G]({{< relref "2024-05-18-220246-aether.md" >}}) components. The environment emphasizes repeatability, observability, and safe remote management.


### Objectives {#objectives}

-   Enable reproducible, isolated experiments for Cellular Networking, SDN, and NFV research.
-   Support precise timing via [PTP]({{< relref "2024-05-20-050327-precision_time_protocol_ptp.md" >}}).
-   Provide resilient egress using dual [VyOS]({{< relref "2025-04-10-012017-vyos.md" >}}) gateways with [VRRP]({{< relref "2024-05-15-200707-vrrp.md" >}}).
-   Offer flexible L2 overlays with Proxmox SDN + [EVPN]({{< relref "2025-09-28-131926-setting_up_evpn_on_proxmox_sdn.md" >}}).
-   Host Aether 5G workloads on a Kubernetes/[OKD]({{< relref "20230518171737-okd.md" >}}) cluster.
-   Centralize service discovery with [BIND 9]({{< relref "2024-05-25-015638-bind_9.md" >}}) DNS.
-   Ensure safe access via a dedicated management network using [iDRAC]({{< relref "2025-03-19-000630-integrated_dell_remote_access_controller_idrac.md" >}}) and [MikroTik]({{< relref "20230226101927-mikrotik.md" >}}) gear.


### Access {#access}

-   Platform: All servers run [Proxmox VE]({{< relref "20230228043925-proxmox_ve.md" >}}) (Debian-based).
-   PVE web UI access from management Gateway: <https://172.27.135.44:8006/>


## Physical Topology and Components {#physical-topology-and-components}

{{< figure src="https://res.cloudinary.com/dkvj6mo4c/image/upload/v1764729697/testbed_2_wlunjv.png" >}}

Topology overview: a dedicated management network for iDRAC access and PVE web UI access; a Proxmox VE LAN with dual VyOS WWAN gateways; EVPN overlays for tenant/lab networks; and a Kubernetes/OKD cluster for Aether 5G.


### Management Network {#management-network}


#### [MikroTik]({{< relref "20230226101927-mikrotik.md" >}}) cAP ax {#mikrotik--20230226101927-mikrotik-dot-md--cap-ax}

-   Model: cAPGi-5HaxD2HaxD-US
-   FCC ID: TV7CPG52X; IC: 7442A-CAPAX
-   Ethernet MAC: 78:9A:18:59:78:80
-   Wi‑Fi1 (5.8 GHz) MAC: 78:9A:18:59:78:83
-   Wi‑Fi2 (2.4 GHz) MAC: 78:9A:18:59:78:82
-   Serial: HF2098EMRR7/343/US
-   Wi‑Fi1 joins hidden SSID “[wahoo]({{< relref "2025-08-08-101821-uva_eduroam_wireless_network_under_linux.md#wahoo" >}})” as a station and obtains an IP from the UVA Wi‑Fi DHCP (current: 172.27.135.44).
-   Bridge groups ether1, ether2, and Wi‑Fi2 as LAN; bridge IP: 192.168.88.1/24; NAT to Wi‑Fi1.
-   [Port‑forward]({{< relref "2025-08-15-212717-routeros_port_forward_from_lan_to_wan.md" >}}) TCP 8006 from WAN (Wi‑Fi1) to 192.168.88.2.
-   Wi‑Fi2 provides LAN SSID \`myLAN\`.
-   Details: [cAP ax setup details (step by step)]({{< relref "2025-08-21-125051-cap_ax_setup_details_step_by_step.md" >}})


#### [MikroTik]({{< relref "20230226101927-mikrotik.md" >}}) L009UiGS-2HaxD-IN {#mikrotik--20230226101927-mikrotik-dot-md--l009uigs-2haxd-in}

-   Role: Management Ethernet LAN switch for a dedicated management network.
-   Connects [iDRAC]({{< relref "2025-03-19-000630-integrated_dell_remote_access_controller_idrac.md" >}}) ports of all servers for remote power/reset and monitoring.
-   FCC ID: TV7L0092AXIN; IC: 7442A-L0092AXIN
-   Serial: HFC092SVAWD/345
-   Built-in Wi‑Fi MAC: 78:9A:18:B6:B0:B5
-   Obtains its IP from the cAP ax (DHCP)


### [Proxmox VE]({{< relref "20230228043925-proxmox_ve.md" >}}) Cluster Network {#proxmox-ve--20230228043925-proxmox-ve-dot-md--cluster-network}


#### [MikroTik]({{< relref "20230226101927-mikrotik.md" >}}) L009UiGS-RM {#mikrotik--20230226101927-mikrotik-dot-md--l009uigs-rm}

-   Serial: HFE097YP05K/346

<!--list-separator-->

-  WAN

    -   IP: 192.168.88.2/24
    -   Netmask: 255.255.255.0
    -   DNS: 8.8.8.8, 8.8.4.4

<!--list-separator-->

-  LAN

    -   Gateway/LAN IP: 192.168.1.1/24
    -   Netmask: 255.255.255.0

    [Port‑forward]({{< relref "2025-08-15-212717-routeros_port_forward_from_lan_to_wan.md" >}}) TCP 8006 from WAN to 192.168.1.11 (PVE on server1).


#### DIY server {#diy-server}

-   Hostname: server1.testbed.com
-   IP: 192.168.1.11/24
-   Gateway: 192.168.1.4
-   DNS: 192.168.1.1
-   Hard Disks:
    -   SanDisk (PVE: local and local-lvm): SIZE - 238.5G; MODEL - CWDISK 256G; SERIAL - J150404B07502
    -   Western Digital (Not in used): SIZE - 1.8T; MODEL - WDC WD20EFAX-68FB5N0; SERIAL - WD-WX91A19E6J3N


#### T470s {#t470s}

-   Hostname: server2.testbed.com
-   IP: 192.168.1.12/24
-   Gateway: 192.168.1.4
-   DNS: 192.168.1.1
-   Hard Disks:
    -   SAMSUNG (PVE: local and local-lvm): SIZE - 238.5G; MODEL - MZVLW256HEHP-000L7; SERIAL - S35ENA1K581888

Wi‑Fi card ([proxmox PCI passthrough]({{< relref "2025-09-23-212735-proxmox_pci_passthrough.md" >}}) for [VyOS]({{< relref "2025-04-10-012017-vyos.md" >}}) Gateway):

-   Description: Wireless interface
-   Product: Intel Wireless 8260
-   Vendor: Intel Corporation
-   Bus info: pci@0000:3a:00.0
-   Logical name: wlp58s0
-   MAC: 14:ab:c5:8f:ab:f6


#### T420s {#t420s}

-   Hostname: server3.testbed.com
-   IP: 192.168.1.13/24
-   Gateway: 192.168.1.4
-   DNS: 192.168.1.1
-   Hard Disks:
    -   INTEL (PVE: local and local-lvm): SIZE - 223.6G; MODEL - SSDSC2KW240H6; SERIAL - CVLT627101DQ240CGN

3G card:

-   Logical name: wwp0s29u1u4
-   MAC: ba:49:c2:37:84:ee

Wi‑Fi card ([proxmox PCI passthrough]({{< relref "2025-09-23-212735-proxmox_pci_passthrough.md" >}}) for [VyOS]({{< relref "2025-04-10-012017-vyos.md" >}}) Gateway):

-   Model: Centrino Advanced‑N 6205 [Taylor Peak]
-   Vendor: Intel Corporation
-   Logical name: wlp3s0
-   MAC: a0:88:b4:75:2a:50


#### GPU PC {#gpu-pc}

-   Hostname: server4.testbed.com
-   IP: 192.168.1.14/24
-   Gateway: 192.168.1.4
-   DNS: 192.168.1.1
-   GPU: Nvidia GeForce RTX 2070
-   Hard Disks:
    -   HGST Ultrastar (PVE: local and local-lvm): SIZE - 465.8G; MODEL - HUSMR1650ASS201; SERIAL - 0QY10YMA


#### Dell R730 Server {#dell-r730-server}

-   Hostname: server5.testbed.com
-   IP: 192.168.1.15/24
-   Gateway: 192.168.1.4
-   DNS: 192.168.1.1
-   GPU: NVIDIA Corporation GK104GL [GRID K2]
-   Hard Disks:
    -   Crucial (PVE: local and local-lvm): SIZE - 465.8G; MODEL - CT500P1SSD8; SERIAL - 1942E223FD79
        -   [ZFS pool]({{< relref "20230517124043-zfs.md" >}}):
            -   sdb    1.1T **X425_STBTE1T2A10** Z4005ZAH sas
            -   sdc    1.1T **X425_STBTE1T2A10** S400N4TC sas
            -   sdd    1.1T **X425_STBTE1T2A10** Z40053QL sas


## Services {#services}


### [Domain Name Service (DNS)]({{< relref "20230417083145-dns.md" >}}) {#domain-name-service--dns----20230417083145-dns-dot-md}

DNS based on [BIND 9]({{< relref "2024-05-25-015638-bind_9.md" >}}).

-   Primary: 192.168.1.21/24
-   Secondary: 192.168.1.22/24
-   Scope: DNS for the Proxmox VE cluster only. The dedicated management network has its own DNS. The Kubernetes/[OKD]({{< relref "20230518171737-okd.md" >}}) cluster uses internal DNS.

[EVPN]({{< relref "2025-09-28-131926-setting_up_evpn_on_proxmox_sdn.md" >}}) DNS [PowerDNS]({{< relref "2024-05-25-025431-powerdns.md" >}})
192.168.1.23/24


### [DHCP]({{< relref "20230531193526-dhcp.md" >}}) {#dhcp--20230531193526-dhcp-dot-md}

[NetBox]({{< relref "2025-11-30-174834-netbox.md" >}}) IP: 192.168.1.24/24
[Kea DHCP]({{< relref "2025-12-11-210940-kea_dhcp.md" >}}) IP: 192.168.1.25/24


### [VyOS]({{< relref "2025-04-10-012017-vyos.md" >}}) [WWAN - External Network]({{< relref "2025-04-10-012017-vyos.md#wwan-wireless-wide-area-network" >}}) Gateway (UVA's [wahoo]({{< relref "2025-08-08-101821-uva_eduroam_wireless_network_under_linux.md#wahoo" >}})) {#vyos--2025-04-10-012017-vyos-dot-md--wwan-external-network--2025-04-10-012017-vyos-dot-md--gateway--uva-s-wahoo-2025-08-08-101821-uva-eduroam-wireless-network-under-linux-dot-md}

-   Primary: 192.168.1.5/24 (To CS Network)
-   Backup: 192.168.1.2/24
-   Backup: 192.168.1.3/24
-   [VRRP]({{< relref "2024-05-15-200707-vrrp.md" >}}) virtual/default gateway (LAN): 192.168.1.4/24

The T470 and T420 laptops host VyOS VMs with Wi‑Fi interfaces passed through. The T470’s VyOS VM serves as primary; the T420’s VyOS VM is secondary. Each VyOS VM has:

-   One interface to the campus Wi‑Fi WAN (Internet via campus DHCP).
-   One interface to the Proxmox OVS bridge (internal LAN).


### [VyOS]({{< relref "2025-04-10-012017-vyos.md" >}}) CS network Gateway {#vyos--2025-04-10-012017-vyos-dot-md--cs-network-gateway}

LAN: 192.168.1.5/24


## [Setting Up EVPN on Proxmox SDN]({{< relref "2025-09-28-131926-setting_up_evpn_on_proxmox_sdn.md" >}}) {#setting-up-evpn-on-proxmox-sdn--2025-09-28-131926-setting-up-evpn-on-proxmox-sdn-dot-md}

EVPN provides L2 overlays across Proxmox nodes for tenant and lab networks while keeping the underlay simple. Proxmox SDN’s EVPN uses VXLAN for data plane and BGP for control plane.

-   Underlay: simple IP reachability between nodes (no L2 stretch).
-   Control plane: BGP EVPN peering between PVE nodes.
-   Data plane: VXLAN VNIs mapped to VLAN-aware bridges for VMs/containers.
-   Segmentation: per‑VNI L2 domains; import/export via route targets.
-   Management: configure declaratively in Proxmox SDN; see [detailed setup]({{< relref "2025-09-28-131926-setting_up_evpn_on_proxmox_sdn.md" >}}).


## [Kubernetes]({{< relref "20230105185343-kubernetes.md" >}})/[OpenShift]({{< relref "20230420020557-openshift.md" >}})([OKD]({{< relref "20230518171737-okd.md" >}})) Cluster {#kubernetes--20230105185343-kubernetes-dot-md--openshift--20230420020557-openshift-dot-md----okd-20230518171737-okd-dot-md---cluster}

The project deploys [Aether 5G]({{< relref "2024-05-18-220246-aether.md" >}}) components on a Kubernetes/OKD cluster. Several VMs act as control-plane and worker nodes (e.g., 3 control-plane + 2 worker, adjusted as resources permit) distributed across the servers.

Notes:

1.  DNS: [BIND 9]({{< relref "2024-05-25-015638-bind_9.md" >}}) provides external DNS records for Kubernetes control-plane nodes. The cluster uses internal DNS (e.g., CoreDNS) for pods and services.
2.  Networking: CNI is Cilium (eBPF). The cluster network is separate from the physical topology; use [BGP]({{< relref "20230608230531-bgp.md" >}}) if advertising pod/service CIDRs to the underlay is required.
3.  Load Balancer: HAProxy fronts the Kubernetes API and balances across control-plane nodes.


## [Bastion Host]({{< relref "2025-12-04-183725-bastion_host.md" >}}) {#bastion-host--2025-12-04-183725-bastion-host-dot-md}

[Teleport]({{< relref "2025-11-19-164307-teleport.md" >}}) is the [bastion host]({{< relref "2025-12-04-183725-bastion_host.md" >}}).


## [Precision Time Protocol (PTP)]({{< relref "2024-05-20-050327-precision_time_protocol_ptp.md" >}}) Synchronization {#precision-time-protocol--ptp----2024-05-20-050327-precision-time-protocol-ptp-dot-md--synchronization}

PTP ensures consistent, sub‑millisecond time synchronization across hosts to support accurate measurements and time‑sensitive 5G components.


## Reference List {#reference-list}

1.  [Data Center Infrastructure]({{< relref "2024-05-16-232311-data_centre_infrastructure.md" >}})
2.  [Proxmox VE]({{< relref "20230228043925-proxmox_ve.md" >}})
3.  [Proxmox SDN EVPN setup]({{< relref "2025-09-28-131926-setting_up_evpn_on_proxmox_sdn.md" >}})
4.  [VyOS]({{< relref "2025-04-10-012017-vyos.md" >}})
5.  [BIND 9]({{< relref "2024-05-25-015638-bind_9.md" >}})
6.  [Cilium]({{< relref "20230609133144-cilium.md" >}})
7.  [HAProxy]({{< relref "2024-05-24-043228-haproxy.md" >}})
8.  [MikroTik]({{< relref "20230226101927-mikrotik.md" >}})
9.  [iDRAC]({{< relref "2025-03-19-000630-integrated_dell_remote_access_controller_idrac.md" >}})
