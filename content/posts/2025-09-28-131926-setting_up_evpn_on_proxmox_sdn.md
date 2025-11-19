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

Below, will based on my [Data Center Testbed]({{< relref "2025-04-10-010004-data_center_testbed_design.md" >}}).


### ASNs and roles {#asns-and-roles}

-   [VyOS]({{< relref "2025-04-10-012017-vyos.md" >}}) edge (both routers): ASN 65001
    -   R0 Main Gateway (connected to UVA's CS): 192.168.1.5/24
    -   R1 T470s (backup): 192.168.1.2/24
    -   R2 T420s (backup): 192.168.1.3/24
    -   VRRP LAN GW: 192.168.1.4/24
-   PVE EVPN fabric (all Proxmox nodes): ASN 65010
    -   DIY: 192.168.1.11/24
    -   T470s PVE: 192.168.1.12/24
    -   T420s PVE: 192.168.1.13/24
    -   GPU PC: 192.168.1.14/24
    -   Dell R730: 192.168.1.15/24

Peer to real router IPs (1.2 &amp; 1.3 &amp; 1.5), not the VRRP VIP, for BGP stability.


### VyOS [BGP]({{< relref "20230608230531-bgp.md" >}}) configs (on treble gateways; same neighbors; only router-id differs) {#vyos-bgp--20230608230531-bgp-dot-md--configs--on-treble-gateways-same-neighbors-only-router-id-differs}


#### R0 {#r0}

```bash
configure
set protocols bgp system-as 65001
set protocols bgp parameters router-id '192.168.1.5'

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

Use a single EVPN controller with explicit Peers (no SDN Fabric), and separate BGP controllers on your exit nodes for the VyOS eBGP peering.


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

BGP controller #1 (server5)

-   Node: server5
-   ASN #: 65010
-   Peers: 192.168.1.5, 192.168.1.2, 192.168.1.3 (your VyOS R0/R1/R2)
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
-   VRF-VXLAN: 10000 (any unique number; this is the VRF identifier for the zone)
-   MTU: 1450 (good default for VXLAN)
-   Advertise Subnets: ✓ (so your EVPN subnet routes are exported into BGP type-5)
-   Exit Nodes: select server5, server2 (these are the nodes that peer via BGP controllers to VyOS)
-   Primary Exit Node: server5 (matches your preferred egress; also prefer R0 on VyOS with a lower MED if you want)
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

1.  In SDN → VNets, click your new VNet row testnet1 to select it.
2.  Click Create → Subnet (or Add → Subnet, depending on your build).
3.  Fill the dialog:
4.  Subnet: 10.60.10.0/24
5.  Gateway: 10.60.10.1 -&gt; Proxmox will install 10.60.10.1 as the anycast VRF gateway on the EVPN fabric. Advertise 10.60.10.0/24 to VyOS via BGP (because you enabled “Advertise Subnets” in the EVPN zone).
6.  SNAT: tick this, the PVE exit node will masquerade/NAT this subnet to its own uplink. If you don’t want that because VyOS is your Internet gateway doing NAT. Enabling it would create double-NAT/confusion. You should set [Internet access / NAT for EVPN subnet](#internet-access-nat-for-evpn-subnet) on VyOS.
7.  DNS Zone Prefix: (optional, only if you configured a DNS plugin under SDN → DNS, e.g., PowerDNS)
8.  DHCP Ranges (optional)
    -   If you want Proxmox to hand out DHCP for this VNet, add a range, e.g.: Start: 10.60.10.100 End: 10.60.10.199
    -   If you prefer static addressing or cloud-init, don’t add a range (no DHCP will run for this subnet).

Click OK, then go to Datacenter → SDN → Options → Apply.


### Apply SDN Changes {#apply-sdn-changes}

In the Proxmox Admin web UI, go to Datacenter &gt; SDN.
Click on Apply to propagate the changes across all nodes.


### Quick sanity checks {#quick-sanity-checks}

On an exit node (server5/server2):

```console
root@server5:~# ip link show type vrf
25: vrf_evpntest: <NOARP,MASTER,UP,LOWER_UP> mtu 65575 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether ba:8d:dc:cb:5a:bf brd ff:ff:ff:ff:ff:ff
```

```console
root@server5:~# ip -4 route show vrf vrf_evpntest | grep -E '10\.60\.1
0\.0/24|default'
10.60.10.0/24 dev testnet1 proto kernel scope link src 10.60.10.1
```

```console
root@server5:~# vtysh -c 'show bgp l2vpn evpn summary'
BGP router identifier 192.168.1.15, local AS number 65010 VRF default vrf-id 0
BGP table version 0
RIB entries 19, using 2432 bytes of memory
Peers 4, using 94 KiB of memory
Peer groups 2, using 128 bytes of memory

Neighbor              V         AS   MsgRcvd   MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd   PfxSnt Desc
server1(192.168.1.11) 4      65010       586       584       24    0    0 00:28:52            2        3 FRRouting/10.3.1
server2(192.168.1.12) 4      65010       586       584       24    0    0 00:28:52            1        3 FRRouting/10.3.1
server3(192.168.1.13) 4      65010       584       584       24    0    0 00:28:48            2        3 FRRouting/10.3.1
server4(192.168.1.14) 4      65010       586       584       24    0    0 00:28:52            2        3 FRRouting/10.3.1

Total number of neighbors 4
```

```console
root@server5:~# vtysh -c 'show bgp ipv4 unicast summary'
BGP router identifier 192.168.1.15, local AS number 65010 VRF default vrf-id 0
BGP table version 27
RIB entries 3, using 384 bytes of memory
Peers 3, using 70 KiB of memory
Peer groups 2, using 128 bytes of memory

Neighbor          V         AS   MsgRcvd   MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd   PfxSnt Desc
vyos(192.168.1.2) 4      65001      3324      3324       27    0    0 02:45:58            2        3 N/A
vyos(192.168.1.3) 4      65001      3326      3326       27    0    0 02:45:57            2        3 N/A
vyos(192.168.1.5) 4      65001      3325      3325       27    0    0 02:45:57            2        3 N/A

Total number of neighbors 3
```


### Verify Configuration {#verify-configuration}

```console
root@server5:~# cat /etc/pve/sdn/controllers.cfg
bgp: bgpserver2
        asn 65010
        node server2
        peers 192.168.1.5, 192.168.1.2, 192.168.1.3
        bgp-multipath-as-path-relax 0
        ebgp 1
        ebgp-multihop 1

bgp: bgpserver5
        asn 65010
        node server5
        peers 192.168.1.5, 192.168.1.2, 192.168.1.3
        bgp-multipath-as-path-relax 0
        ebgp 1
        ebgp-multihop 1

evpn: evpn
        asn 65010
        peers 192.168.1.11,192.168.1.12,192.168.1.13,192.168.1.14,192.168.1.15

```

Ensure the EVPN and VXLAN configurations are correctly applied.
Check the status of the BGP sessions and routing tables using FRRouting commands:

```console
root@server5:~# vtysh -c "show bgp summary"

IPv4 Unicast Summary:
BGP router identifier 192.168.1.15, local AS number 65010 VRF default vrf-id 0
BGP table version 27
RIB entries 3, using 384 bytes of memory
Peers 3, using 70 KiB of memory
Peer groups 2, using 128 bytes of memory

Neighbor          V         AS   MsgRcvd   MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd   PfxSnt Desc
vyos(192.168.1.2) 4      65001      3372      3372       27    0    0 02:48:22            2        3 N/A
vyos(192.168.1.3) 4      65001      3374      3374       27    0    0 02:48:21            2        3 N/A
vyos(192.168.1.5) 4      65001      3373      3373       27    0    0 02:48:21            2        3 N/A

Total number of neighbors 3

L2VPN EVPN Summary:
BGP router identifier 192.168.1.15, local AS number 65010 VRF default vrf-id 0
BGP table version 0
RIB entries 19, using 2432 bytes of memory
Peers 4, using 94 KiB of memory
Peer groups 2, using 128 bytes of memory

Neighbor              V         AS   MsgRcvd   MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd   PfxSnt Desc
server1(192.168.1.11) 4      65010       655       655       26    0    0 00:32:19            2        5 FRRouting/10.3.1
server2(192.168.1.12) 4      65010       655       655       26    0    0 00:32:19            1        5 FRRouting/10.3.1
server3(192.168.1.13) 4      65010       653       655       26    0    0 00:32:15            2        5 FRRouting/10.3.1
server4(192.168.1.14) 4      65010       655       655       26    0    0 00:32:19            2        5 FRRouting/10.3.1

Total number of neighbors 4
```

```console
root@server5:~# vtysh -c "show evpn vni"
VNI        Type VxLAN IF              # MACs   # ARPs   # Remote VTEPs  Tenant VRF
100        L2   vxlan_testnet1        1        1        4               vrf_evpntest
10000      L3   vrfvx_evpntest        0        0        n/a             vrf_evpntest
```


### Create a VM on 10.60.10.0/24 {#create-a-vm-on-10-dot-60-dot-10-dot-0-24}

Once the EVPN zone, VNet \`testnet1\`, and subnet \`10.60.10.0/24\` with gateway \`10.60.10.1\` are in place, you can attach VMs to this fabric-backed network.


#### Create the VM and attach to \`testnet1\` {#create-the-vm-and-attach-to-testnet1}

1.  In the Proxmox web UI, click on a node (for example \`server5\`) → Create VM.
2.  Walk through the wizard as usual (OS, disks, CPU/RAM).
3.  On the **Network** step:
    -   Bridge: \`testnet1\` (the SDN VNet bridge created earlier)
    -   VLAN Tag: leave empty (the SDN VNet already handles encapsulation/VXLAN tag 100)
    -   Model: \`virtio\` (recommended)
4.  Finish the wizard.


#### Configure the VM IP inside the guest {#configure-the-vm-ip-inside-the-guest}

If you configured DHCP for \`10.60.10.0/24\` in the SDN subnet:

-   Simply boot the VM; it should get an address in \`10.60.10.0/24\` and default gateway \`10.60.10.1\` via DHCP.

If you did **not** configure DHCP:

-   Set a static IP inside the VM:
    -   IP: \`10.60.10.20\`
    -   Netmask: \`255.255.255.0\`
    -   Gateway: \`10.60.10.1\`
    -   DNS: your preferred resolver (for example \`1.1.1.1\` or your internal DNS)


#### Connectivity tests from the VM {#connectivity-tests-from-the-vm}

From inside the VM, run:

```console
ping -c3 10.60.10.1        # anycast EVPN gateway on the fabric
ping -c3 192.168.1.5       # VyOS R0 (via EVPN → BGP → VyOS)
ping -c3 8.8.8.8           # Internet reachability via VyOS (if default route is advertised)
```

If these pings work and \`vtysh -c 'show bgp ipv4 unicast'\` on \`server5\` shows \`10.60.10.0/24\` learned/advertised as expected, your VM is successfully using the EVPN-backed subnet.


### Internet access / NAT for EVPN subnet {#internet-access-nat-for-evpn-subnet}

Because \`10.60.10.0/24\` is an RFC1918 private subnet, it needs NAT somewhere on the path to reach public Internet. In this design, Proxmox SDN leaves \`SNAT\` unchecked and delegates NAT to VyOS (your existing Internet edge).


#### Verify that VyOS learns 10.60.10.0/24 via BGP {#verify-that-vyos-learns-10-dot-60-dot-10-dot-0-24-via-bgp}

On VyOS, confirm the EVPN subnet is present in the routing table:

```bash
show ip route 10.60.10.0/24
```

The next-hop should be one of your Proxmox exit nodes (for example \`server5\`), learned via BGP from ASN 65010.


#### Add source NAT on VyOS for 10.60.10.0/24 {#add-source-nat-on-vyos-for-10-dot-60-dot-10-dot-0-24}

On the primary VyOS router (R0 at 192.168.1.5), add a source NAT rule that translates \`10.60.10.0/24\` to the router’s WAN address. Adjust \`WAN_IF\` below to your real uplink interface (for example \`eth0\`, \`pppoe0\`, etc.) and pick a free rule number:

```bash
configure

set nat source rule 110 description 'masquerade EVPN 10.60.10.0/24'
set nat source rule 110 outbound-interface 'WAN_IF'
set nat source rule 110 source address '10.60.10.0/24'
set nat source rule 110 translation address 'masquerade'

commit
save
```

If you also want R1/R2 to provide Internet during failover, mirror the same \`nat source\` rule on those VyOS nodes and ensure your upstream routing/VRRP design sends outbound traffic via the currently active gateway.


#### Re-test from the EVPN VM {#re-test-from-the-evpn-vm}

From the VM at \`10.60.10.20\`:

```console
ping -c3 8.8.8.8
```

If this now succeeds while \`ping\` to \`10.60.10.1\` and \`192.168.1.5\` still works, you have full Internet access for the EVPN-backed subnet using NAT at VyOS instead of Proxmox.


## Reference List {#reference-list}

1.  <https://bennetgallein.de/blog/setting-up-evpn-on-proxmox-sdn-a-comprehensive-guide>
