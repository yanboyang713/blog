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


## Step-by-Step Guide {#step-by-step-guide}


### ASNs and roles {#asns-and-roles}

-   VyOS edge (both routers): ASN 65001
    -   R1 T470s (primary): 192.168.1.2/24
    -   R2 T420s (secondary): 192.168.1.3/24
    -   VRRP LAN GW: 192.168.1.4/24
-   PVE EVPN fabric (all Proxmox nodes): ASN 65010
    -   DIY: 192.168.1.11/24
    -   T470s PVE: 192.168.1.12/24
    -   T420s PVE: 192.168.1.13/24
    -   GPU PC: 192.168.1.14/24
    -   Dell R730: 192.168.1.15/24

Peer to real router IPs (1.2 &amp; 1.3), not the VRRP VIP, for BGP stability.


### VyOS [BGP]({{< relref "20230608230531-bgp.md" >}}) configs (on both routers; same neighbors; only router-id differs) {#vyos-bgp--20230608230531-bgp-dot-md--configs--on-both-routers-same-neighbors-only-router-id-differs}


#### R1 {#r1}

```bash
configure
set protocols bgp system-as 65001
set protocols bgp parameters router-id '192.168.1.2'

# eBGP neighbors (your PVE nodes in ASN 65010)
set protocols bgp neighbor 192.168.1.11 remote-as '65010'
set protocols bgp neighbor 192.168.1.12 remote-as '65010'
set protocols bgp neighbor 192.168.1.13 remote-as '65010'
set protocols bgp neighbor 192.168.1.14 remote-as '65010'
set protocols bgp neighbor 192.168.1.15 remote-as '65010'

# BGP Graceful Restart
set protocols bgp parameters graceful-restart stalepath-time '360'
# Fast failure detection
set protocols bgp neighbor 192.168.1.11 bfd
set protocols bgp neighbor 192.168.1.12 bfd
set protocols bgp neighbor 192.168.1.13 bfd
set protocols bgp neighbor 192.168.1.14 bfd
set protocols bgp neighbor 192.168.1.15 bfd
# 4) Activate address-families
# EVPN control-plane (required for EVPN)
set protocols bgp neighbor 192.168.1.11 address-family l2vpn-evpn
set protocols bgp neighbor 192.168.1.12 address-family l2vpn-evpn
set protocols bgp neighbor 192.168.1.13 address-family l2vpn-evpn
set protocols bgp neighbor 192.168.1.14 address-family l2vpn-evpn
set protocols bgp neighbor 192.168.1.15 address-family l2vpn-evpn

# (optional) Unicast AF if you also want to exchange classic IPv4 routes
# e.g., advertise/learn a default 0/0 via unicast:
set protocols bgp neighbor 192.168.1.11 address-family ipv4-unicast
set protocols bgp neighbor 192.168.1.12 address-family ipv4-unicast
set protocols bgp neighbor 192.168.1.13 address-family ipv4-unicast
set protocols bgp neighbor 192.168.1.14 address-family ipv4-unicast
set protocols bgp neighbor 192.168.1.15 address-family ipv4-unicast

# If you DO want VyOS to be the Internet egress for the fabric:
# (a) make sure VyOS has a default route (dhcp/pppoe/static)
# (b) advertise default and mgmt LAN to PVE
set protocols bgp address-family ipv4-unicast network '0.0.0.0/0'
set protocols bgp address-family ipv4-unicast network '192.168.1.0/24'
commit; save
```


#### R2 {#r2}

```bash
set protocols bgp system-as 65001
set protocols bgp parameters router-id '192.168.1.3'
# eBGP neighbors (your PVE nodes in ASN 65010)
set protocols bgp neighbor 192.168.1.11 remote-as '65010'
set protocols bgp neighbor 192.168.1.12 remote-as '65010'
set protocols bgp neighbor 192.168.1.13 remote-as '65010'
set protocols bgp neighbor 192.168.1.14 remote-as '65010'
set protocols bgp neighbor 192.168.1.15 remote-as '65010'

# BGP Graceful Restart
set protocols bgp parameters graceful-restart stalepath-time '360'
# Fast failure detection
set protocols bgp neighbor 192.168.1.11 bfd
set protocols bgp neighbor 192.168.1.12 bfd
set protocols bgp neighbor 192.168.1.13 bfd
set protocols bgp neighbor 192.168.1.14 bfd
set protocols bgp neighbor 192.168.1.15 bfd
# 4) Activate address-families
# EVPN control-plane (required for EVPN)
set protocols bgp neighbor 192.168.1.11 address-family l2vpn-evpn
set protocols bgp neighbor 192.168.1.12 address-family l2vpn-evpn
set protocols bgp neighbor 192.168.1.13 address-family l2vpn-evpn
set protocols bgp neighbor 192.168.1.14 address-family l2vpn-evpn
set protocols bgp neighbor 192.168.1.15 address-family l2vpn-evpn

# (optional) Unicast AF if you also want to exchange classic IPv4 routes
# e.g., advertise/learn a default 0/0 via unicast:
set protocols bgp neighbor 192.168.1.11 address-family ipv4-unicast
set protocols bgp neighbor 192.168.1.12 address-family ipv4-unicast
set protocols bgp neighbor 192.168.1.13 address-family ipv4-unicast
set protocols bgp neighbor 192.168.1.14 address-family ipv4-unicast
set protocols bgp neighbor 192.168.1.15 address-family ipv4-unicast

# If you DO want VyOS to be the Internet egress for the fabric:
# (a) make sure VyOS has a default route (dhcp/pppoe/static)
# (b) advertise default and mgmt LAN to PVE
set protocols bgp address-family ipv4-unicast network '0.0.0.0/0'
set protocols bgp address-family ipv4-unicast network '192.168.1.0/24'
commit; save

```


#### Verify {#verify}

```bash
show ip bgp summary
```


### Prereqs on each PVE node (one-time) {#prereqs-on-each-pve-node--one-time}

```bash
apt update && apt install -y frr frr-pythontools
systemctl restart frr
systemctl status frr
```


### PVE side configure {#pve-side-configure}

Use an SDN Fabric for the EVPN controller (and leave Peers empty). Then use BGP controllers for the VyOS eBGP peering.


#### Create a EVPN Controller {#create-a-evpn-controller}

Open the Proxmox Admin web UI.
Navigate to Datacenter &gt; SDN &gt; Options → Controllers → Add → EVPN

-   ID: evpn
-   ASN #: 65010
-   Peers: 192.168.1.11,192.168.1.12,192.168.1.13,192.168.1.14,192.168.1.15
-   SDN Fabric: leave empty
-   OK


#### Add BGP controllers for the VyOS edge (eBGP) on your exit node(s) {#add-bgp-controllers-for-the-vyos-edge--ebgp--on-your-exit-node--s}

node(s) that will be your “exit nodes.”

BGP controller #1 (server1)

-   Node: server1
-   ASN #: 65010
-   Peers: 192.168.1.2, 192.168.1.3 (your VyOS R1/R2)
-   EBGP: ✅ enabled
-   Loopback Interface: (leave empty) — you’re peering to VyOS on the same LAN, not via loopbacks
-   ebgp-multihop: (off) — directly connected, TTL=1 is fine
-   bgp-multipaths-as-path-relax:
-   leave off if you want a single preferred egress (use MED on VyOS + “Primary Exit Node” in the zone); turn on only if you want ECMP (active/active) egress
-   Click Add.

BGP controller #2 (server2)

-   Repeat the same values with Node = server2. Click Add.


### Create an EVPN Zone {#create-an-evpn-zone}

1.  Datacenter → SDN → Zones → Add → EVPN

2, Fill the dialog:

-   ID: evpntest (≤ 8 chars, lowercase, no spaces/dashes)
-   Controller: evpn (the EVPN controller you already created)
-   Nodes: select server1–server5 (all nodes that should carry this zone)
-   VRF-VXLAN: 10000 (any unique number; this is the VRF identifier for the zone)
-   MTU: 1450 (good default for VXLAN)
-   Advertise Subnets: ✓ (so your EVPN subnet routes are exported into BGP type-5)
-   Exit Nodes: select server1, server2 (these are the nodes that peer via BGP controllers to VyOS)
-   Primary Exit Node: server1 (matches your preferred egress; also prefer R1 on VyOS with a lower MED if you want)
-   IPAM: pve
-   (Optional) Route-Target Import: leave empty for now (set later if you want inter-VRF route leaking)
-   Click Add.
-   Apply the SDN config:

Datacenter → SDN → Options → Apply
(or run pvesh set /cluster/sdn --apply 1 on a node)


### Create a VXLAN VNet {#create-a-vxlan-vnet}

In the Proxmox Admin web UI, navigate to Datacenter &gt; SDN &gt; VNets.

-   Click Add.
-   Fill the dialog:
    -   ID: testnet1 (IDs: ≤8 chars, lowercase letters/digits only; no dashes)
    -   Zone: select your zone (e.g. evpntest)
    -   Tag: 100

(Leave other fields at defaults)


### Add Subnets within Your VNet {#add-subnets-within-your-vnet}

Ensure you do not configure any related VNet's subnet gateway if you don't want Proxmox to handle outgoing traffic directly.

1.  In SDN → VNets, click your new VNet row testnet1 to select it.
2.  Click Create → Subnet (or Add → Subnet, depending on your build).
3.  Fill the dialog:
4.  CIDR: 10.60.10.0/24
5.  Gateway: 10.60.10.1 → This becomes the anycast VRF gateway IP for that VNet. Every node will route for 10.60.10.1 inside the zone’s VRF, so VMs can move between nodes without changing gateway.
6.  SNAT: leave UNchecked → If you tick this, the PVE exit node will masquerade/NAT this subnet to its own uplink. You don’t want that because VyOS is your Internet gateway doing NAT. Enabling it would create double-NAT/confusion.
7.  DNS Zone Prefix: (optional, only if you configured a DNS plugin under SDN → DNS, e.g., PowerDNS)
8.  DHCP Ranges (optional)
    -   If you want Proxmox to hand out DHCP for this VNet, add a range, e.g.: Start: 10.60.10.100 End: 10.60.10.199
    -   If you prefer static addressing or cloud-init, don’t add a range (no DHCP will run for this subnet).

Click OK, then go to Datacenter → SDN → Options → Apply.


### Apply SDN Changes {#apply-sdn-changes}

In the Proxmox Admin web UI, go to Datacenter &gt; SDN.
Click on Apply to propagate the changes across all nodes.


### Quick sanity checks {#quick-sanity-checks}

On an exit node (server1/server2):

```console
root@server1:~# ip link show type vrf
11: vrf_evpnlab: <NOARP,MASTER,UP,LOWER_UP> mtu 65575 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 2e:a9:68:55:f3:40 brd ff:ff:ff:ff:ff:ff
root@server1:~# ip -4 route show vrf vrf_evpnlab | grep -E '10\.60\.10\.0/24|default'
10.60.10.0/24 dev testnet1 proto kernel scope link src 10.60.10.1
root@server1:~# vtysh -c 'show bgp l2vpn evpn summary'
BGP router identifier 192.168.1.11, local AS number 65010 VRF default vrf-id 0
BGP table version 0
RIB entries 19, using 2432 bytes of memory
Peers 4, using 94 KiB of memory
Peer groups 2, using 128 bytes of memory

Neighbor              V         AS   MsgRcvd   MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd   PfxSnt Desc
server2(192.168.1.12) 4      65010       784       783        8    0    0 00:38:54            1        3 FRRouting/10.3.1
server3(192.168.1.13) 4      65010       784       783        8    0    0 00:38:54            2        3 FRRouting/10.3.1
server4(192.168.1.14) 4      65010       784       783        8    0    0 00:38:54            2        3 FRRouting/10.3.1
server5(192.168.1.15) 4      65010       784       783        8    0    0 00:38:54            2        3 FRRouting/10.3.1

Total number of neighbors 4
root@server1:~# vtysh -c 'show bgp ipv4 unicast summary'
BGP router identifier 192.168.1.11, local AS number 65010 VRF default vrf-id 0
BGP table version 10
RIB entries 3, using 384 bytes of memory
Peers 2, using 47 KiB of memory
Peer groups 2, using 128 bytes of memory

Neighbor          V         AS   MsgRcvd   MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd   PfxSnt Desc
vyos(192.168.1.2) 4      65001       786       786       10    0    0 00:39:02            2        3 N/A
vyos(192.168.1.3) 4      65001       786       786       10    0    0 00:39:02            2        3 N/A

Total number of neighbors 2
```


### Verify Configuration {#verify-configuration}

```console
root@server3:~# cat /etc/pve/sdn/controllers.cfg
evpn: evpn
        asn 65010
        peers 192.168.1.11,192.168.1.12,192.168.1.13,192.168.1.14,192.168.1.15

bgp: bgpserver1
        asn 65010
        node server1
        peers 192.168.1.2, 192.168.1.3
        bgp-multipath-as-path-relax 0
        ebgp 1

bgp: bgpserver2
        asn 65010
        node server2
        peers 192.168.1.2, 192.168.1.3
        bgp-multipath-as-path-relax 0
        ebgp 1

```

Ensure the EVPN and VXLAN configurations are correctly applied.
Check the status of the BGP sessions and routing tables using FRRouting commands:

```bash
vtysh -c "show bgp summary"
vtysh -c "show evpn vni"
```


### Install Required Packages on all PVE nodes {#install-required-packages-on-all-pve-nodes}

```bash
apt update && apt install -y libpve-network-perl frr frr-pythontools dnsmasq
```


### Configure FRRouting {#configure-frrouting}


#### Enable in /etc/frr/daemons {#enable-in-etc-frr-daemons}

```ini
zebra=yes
bgpd=yes
```


#### Restart FRR {#restart-frr}

```bash
systemctl restart frr
```


#### Configure FRRouting by editing /etc/frr/frr.conf on each node {#configure-frrouting-by-editing-etc-frr-frr-dot-conf-on-each-node}

```bash
router bgp 65010
 bgp router-id <THIS_NODE_IP>
 bgp log-neighbor-changes

 ! Peers to VyOS edge
 neighbor 192.168.1.2 remote-as 65001
 neighbor 192.168.1.3 remote-as 65001

 ! EVPN full-mesh within fabric (list other PVE nodes)
 neighbor 192.168.1.11 remote-as 65010
 neighbor 192.168.1.12 remote-as 65010
 neighbor 192.168.1.13 remote-as 65010
 neighbor 192.168.1.14 remote-as 65010
 neighbor 192.168.1.15 remote-as 65010

 address-family l2vpn evpn
  neighbor 192.168.1.2 activate
  neighbor 192.168.1.3 activate
  neighbor 192.168.1.11 activate
  neighbor 192.168.1.12 activate
  neighbor 192.168.1.13 activate
  neighbor 192.168.1.14 activate
  neighbor 192.168.1.15 activate
 exit-address-family

 address-family ipv4 unicast
  neighbor 192.168.1.2 activate
  neighbor 192.168.1.3 activate
  neighbor 192.168.1.11 activate
  neighbor 192.168.1.12 activate
  neighbor 192.168.1.13 activate
  neighbor 192.168.1.14 activate
  neighbor 192.168.1.15 activate
 exit-address-family
```

Per-node router-ids:

-   .11 node: bgp router-id 192.168.1.11
-   .12 node: bgp router-id 192.168.1.12
-   .13 node: bgp router-id 192.168.1.13
-   .14 node: bgp router-id 192.168.1.14
-   .15 node: bgp router-id 192.168.1.15


#### Restart FRR {#restart-frr}

```bash
systemctl restart frr
```


## Reference List {#reference-list}

1.  <https://bennetgallein.de/blog/setting-up-evpn-on-proxmox-sdn-a-comprehensive-guide>
