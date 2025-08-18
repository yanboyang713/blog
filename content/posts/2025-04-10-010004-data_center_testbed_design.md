---
title: "Data Center Testbed Design"
draft: false
---

## Introduction and Objectives {#introduction-and-objectives}

This testbed is designed to support advanced research in network performance, [Software-Defined Networking (SDN)]({{< relref "2025-04-10-011054-software_defined_networking_sdn.md" >}}), [Network Function Virtualization (NFV)]({{< relref "2025-04-10-011707-network_function_virtualization_nfv.md" >}}),and 5G deployment.

It leverages four [Dell PowerEdge]({{< relref "2024-05-16-232311-data_centre_infrastructure.md#dell-poweredge-r730" >}}) servers (each running [Proxmox VE]({{< relref "20230228043925-proxmox_ve.md" >}}) hypervisor) interconnected by a [P4 Programmable Switche]({{< relref "2024-03-02-023757-p4_programmable_switches.md" >}}), with an [Open vSwitch (OVS)]({{< relref "2023-12-06-170847-open_vswitch_ovs.md" >}}) bridge on each server. The environment will enable experiments with software-defined networking (through OVS and SDN controllers), NFV (via virtual network functions on the VMs), and a 5G standalone network (with disaggregated RAN and core components).

All servers run [Proxmox VE]({{< relref "20230228043925-proxmox_ve.md" >}}) (Debian-based), and each is equipped with a [SmartNIC]({{< relref "2025-04-10-011945-smartnic.md" >}}) (programmable NIC) to offload packet processing and support P4 programs in hardware.

One server also provides WAN connectivity through a [VyOS]({{< relref "2025-04-10-012017-vyos.md" >}}) router VM (using a Wi-Fi uplink to campus network), and another runs a [BIND 9]({{< relref "2024-05-25-015638-bind_9.md" >}}) [(DNS)]({{< relref "20230417083145-dns.md" >}}) service to support the on-premises [OKD]({{< relref "20230518171737-okd.md" >}}) cluster name resolution.


## Physical Topology and Components {#physical-topology-and-components}

{{< figure src="https://res.cloudinary.com/dkvj6mo4c/image/upload/v1744289068/Data_Center_Testbed_uss7bx.png" >}}


### [Proxmox VE]({{< relref "20230228043925-proxmox_ve.md" >}}) {#proxmox-ve--20230228043925-proxmox-ve-dot-md}

All Proxmox servers are part of a single Proxmox cluster (for ease of management, enabling features like VM live migration across servers). They share the management network for cluster coordination. VMs on any server can reach VMs on another server via the switch.


### Management Ethernet Switch (dedicated) {#management-ethernet-switch--dedicated}

A dedicated management Ethernet switch connects the [Integrated Dell Remote Access Controller (iDRAC)]({{< relref "2025-03-19-000630-integrated_dell_remote_access_controller_idrac.md" >}}) out-of-band management ports of all servers on an isolated management network (for remote power/reset and monitoring).


### [MikroTik]({{< relref "20230226101927-mikrotik.md" >}}) cAP ax {#mikrotik--20230226101927-mikrotik-dot-md--cap-ax}

ID: cAPGi-5HaxD2HaxD-US
FCC ID: TV7CPG52X
IC: 7442A-CAPAX
Eth MAC: 78:9A:18:59:78:80
WIFI1(5.8Ghz) MAC: 78:9A:18:59:78:83
WIFI2(2.4Ghz) MAC: 78:9A:18:59:78:82
SN: HF2098EMRR7/343/US

Set as WAN for L009UiGS-2HaxD-IN
[add AP to CAPsMAN]({{< relref "2024-03-11-134846-add_ap_to_capsman.md" >}})


### [MikroTik]({{< relref "20230226101927-mikrotik.md" >}}) L009UiGS-2HaxD-IN {#mikrotik--20230226101927-mikrotik-dot-md--l009uigs-2haxd-in}

FCC ID: TV7L0092AXIN
IC: 7442A-L0092AXIN
SN: HFC092SVAWD/345
Integration WIFI MAC: 78:9A:18:B6:B0:B5
IP: 192.168.88.1/24
MASK: 255.255.255.0
[Port Forward]({{< relref "2025-08-15-212717-routeros_port_forward_from_lan_to_wan.md" >}}) 192.168.88.2/24 port 8006 to WAN


### [MikroTik]({{< relref "20230226101927-mikrotik.md" >}}) L009UiGS-RM {#mikrotik--20230226101927-mikrotik-dot-md--l009uigs-rm}

SN: HFE097YP05K/346

WAN IP: 192.168.88.2/24
MASK: 255.255.255.0
DNS:8.8.8.8;8.8.4.4

LAN: 192.168.1.1/24
MASK: 255.255.255.0

[Port Forward]({{< relref "2025-08-15-212717-routeros_port_forward_from_lan_to_wan.md" >}}) 192.168.1.11/24 port 8006 to WAN


### DIY server {#diy-server}

Hostname: server1.testbed.com
192.168.1.11/24
Gateway: 192.168.1.1
DNS: 192.168.1.1


### T470s {#t470s}

Hostname: server2.testbed.com
192.168.1.12/24
Gateway: 192.168.1.1
DNS: 192.168.1.1


### T420s {#t420s}

Hostname: server3.testbed.com
192.168.1.13/24
Gateway: 192.168.1.1
DNS: 192.168.1.1


### GPU PC {#gpu-pc}

Hostname: server4.testbed.com
192.168.1.14/24
Gateway: 192.168.1.1
DNS: 192.168.1.1


### Dell R730 Server {#dell-r730-server}

Hostname: server5.testbed.com
192.168.1.15/24
Gateway: 192.168.1.1
DNS: 192.168.1.1


### [spine-leaf architecture]({{< relref "2025-04-10-063517-spine_leaf_architecture.md" >}}) {#spine-leaf-architecture--2025-04-10-063517-spine-leaf-architecture-dot-md}

All of servers have their primary Proxmox host NICs (the SmartNICs) connected to ports on individual **leaf switches**. These leaf switches, in turn, are interconnected via a high-speed **spine switch**, forming a classic spine-leaf topology.

Each server connects to its respective leaf switch through the SmartNIC, which supports P4-programmable hardware offloads. The OVS bridge on each Proxmox host bridges the internal VMs to the physical SmartNIC interface, which uplinks to the leaf switch. This architecture allows for traffic from VMs on different servers to be routed through the spine switch, enabling scalable and low-latency east-west communication.

The SmartNICs on each server can filter, route, or encapsulate packets in hardware using their P4-programmable pipeline before sending them out. This effectively distributes switching and network logic between the edge (SmartNICs) and the fabric (leaf and spine switches). The programmable nature of both the NICs and the switches provides flexibility for implementing SDN policies, slicing, and advanced telemetry in the data center fabric.


### [VyOS]({{< relref "2025-04-10-012017-vyos.md" >}}) [WWAN - External Network]({{< relref "2025-04-10-012017-vyos.md#wwan-wireless-wide-area-network" >}}) Gateway {#vyos--2025-04-10-012017-vyos-dot-md--wwan-external-network--2025-04-10-012017-vyos-dot-md--gateway}

One of the servers has an external Wi-Fi card that links to the university’s Wi-Fi network. This interface is passed to a VyOS VM, which acts as the **gateway router** for the testbed.

The VyOS VM has two interfaces: one connects to the Wi-Fi WAN (providing Internet access/DHCP from campus network), and the other connects to the Proxmox OVS bridge (LAN). This VM provides NAT, firewall, and routing between the testbed’s internal LAN and the outside network.


### [Kubernetes]({{< relref "20230105185343-kubernetes.md" >}})/[OpenShift]({{< relref "20230420020557-openshift.md" >}})([OKD]({{< relref "20230518171737-okd.md" >}})) Cluster {#kubernetes--20230105185343-kubernetes-dot-md--openshift--20230420020557-openshift-dot-md----okd-20230518171737-okd-dot-md---cluster}

The research project will deploy [Aether 5G]({{< relref "2024-05-18-220246-aether.md" >}}) components on a Kubernetes cluster. We will create several VMs to act as master and worker nodes for this cluster. For instance, 3 control-plane VMs and 2 worker VMs (depending on resource needs) distributed across the all of servers.

**NOTE:**

1.  The [BIND 9]({{< relref "2024-05-25-015638-bind_9.md" >}}) DNS VM provides the required DNS records for this cluster’s operation
2.  The Kubernetes network (for pod communication) will be handled by an [Calico]({{< relref "20230609130541-calico.md" >}}), but that is separate from our physical topology – so, we require [BGP]({{< relref "20230608230531-bgp.md" >}})
3.  Needs a [Load Balancer]({{< relref "20230601231737-loadbalancer.md" >}}) to distribute traffic across all control plane nodes.


### Storage Network (Optional) {#storage-network--optional}

If shared storage or [Ceph]({{< relref "20230107215132-ceph.md" >}}) is used for the VMs or containers, we might consider a separate VLAN or even direct links for storage traffic. However, for this design, we assume local storage on each server for simplicity.


### [Central Unit (CU)]({{< relref "2024-10-28-235334-radio_access_network_ran_splits.md#central-unit--cu" >}})/[Distributed Unit (DU)]({{< relref "2024-10-28-235334-radio_access_network_ran_splits.md#distributed-unit--du" >}}) {#central-unit--cu----2024-10-28-235334-radio-access-network-ran-splits-dot-md--distributed-unit--du----2024-10-28-235334-radio-access-network-ran-splits-dot-md}

<https://docs.aetherproject.org/master/onramp/gnb.html#gnodeb-setup>


### [User Equipment (UE)]({{< relref "2024-10-28-221714-srsran.md#user-equipment--ue" >}}) {#user-equipment--ue----2024-10-28-221714-srsran-dot-md}


### [Precision Time Protocol (PTP)]({{< relref "2024-05-20-050327-precision_time_protocol_ptp.md" >}}) Synchronization {#precision-time-protocol--ptp----2024-05-20-050327-precision-time-protocol-ptp-dot-md--synchronization}


### [SmartNIC DPU]({{< relref "2025-04-10-011945-smartnic.md" >}}) {#smartnic-dpu--2025-04-10-011945-smartnic-dot-md}


## Reference List {#reference-list}

1.  [data center infrastructure]({{< relref "2024-05-16-232311-data_centre_infrastructure.md" >}})
