---
title: "Proxmox VE DHCP Relay for EVPN"
date: 2025-12-17
draft: false
---

To get DHCP working for an EVPN VNet when your DHCP server (Kea at 192.168.1.25) is not inside that L2 segment, you need a DHCP
  relay agent on a Proxmox node that can see the EVPN broadcast domain.

1.  Pick where the relay runs

2.  Run the relay on your Proxmox exit node(s) (e.g. server5/server2), or on every node that hosts VMs on testnet1.
3.  The relay must listen on the EVPN bridge (your VNet bridge, e.g. testnet1) and send unicast DHCP to 192.168.1.25 via your LAN
    bridge/uplink (often vmbr0 / vmbr1).

4.  Install and configure ISC DHCP relay on the chosen PVE node(s)

On each relay node:

-   Install: apt install isc-dhcp-relay
-   Set /etc/default/isc-dhcp-relay roughly like:
    -   SERVERS="192.168.1.25"
    -   INTERFACES="testnet1 vmbr0"  (replace vmbr0 with the interface that reaches 192.168.1.25)
    -   OPTIONS="-4"
-   Start/enable: systemctl enable --now isc-dhcp-relay

(If you’re unsure which uplink to use: ip route get 192.168.1.25 will show the egress interface.)

1.  Configure Kea to accept relayed traffic for the EVPN subnet

In Kea, your subnet4 for 10.60.10.0/24 should include a relay match, because the requests arrive with giaddr = 10.60.10.1 (your EVPN
anycast gateway), e.g.:

-   subnet: 10.60.10.0/24
-   pool: 10.60.10.100-10.60.10.200
-   router option: 10.60.10.1
-   relay: ip-addresses: [ "10.60.10.1" ]

-   Common gotchas

-   Don’t configure a DHCP range in Proxmox SDN for that EVPN subnet (it won’t work with EVPN and can cause confusion).
-   Ensure firewalls allow UDP 67/68 on the PVE host and on the Kea host.
-   If you run relays on multiple nodes, clients may see multiple offers; usually OK. To keep it simple, start with the primary exit
    node only.
    ```console
    root@server5:~# ip a
      1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
          link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
          inet 127.0.0.1/8 scope host lo
             valid_lft forever preferred_lft forever
          inet6 ::1/128 scope host noprefixroute
             valid_lft forever preferred_lft forever
      4: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master vmbr0 state UP group default qlen 1000
          link/ether 24:6e:96:59:5f:d0 brd ff:ff:ff:ff:ff:ff
          altname enp1s0f0
          altname enx246e96595fd0
      5: eno2: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
          link/ether 24:6e:96:59:5f:d2 brd ff:ff:ff:ff:ff:ff
          altname enp1s0f1
          altname enx246e96595fd2
      6: vmbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
          link/ether 24:6e:96:59:5f:d0 brd ff:ff:ff:ff:ff:ff
          inet 192.168.1.15/24 scope global vmbr0
             valid_lft forever preferred_lft forever
          inet6 fe80::266e:96ff:fe59:5fd0/64 scope link proto kernel_ll
             valid_lft forever preferred_lft forever
      7: vxlan_testnet1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master testnet1 state UNKNOWN group default qlen 1000
          link/ether fa:31:02:f6:ad:eb brd ff:ff:ff:ff:ff:ff
      8: testnet1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master vrf_evpntest state UP group default qlen 1000
          link/ether bc:24:11:b6:fe:2c brd ff:ff:ff:ff:ff:ff
          inet 10.60.10.1/24 scope global testnet1
             valid_lft forever preferred_lft forever
          inet6 fe80::be24:11ff:feb6:fe2c/64 scope link proto kernel_ll
             valid_lft forever preferred_lft forever
      9: vrf_evpntest: <NOARP,MASTER,UP,LOWER_UP> mtu 65575 qdisc noqueue state UP group default qlen 1000
          link/ether 72:ba:9a:70:11:81 brd ff:ff:ff:ff:ff:ff
      10: vrfvx_evpntest: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master vrfbr_evpntest state UNKNOWN group default qlen
      1000
          link/ether b2:4b:7f:6f:29:78 brd ff:ff:ff:ff:ff:ff
      11: vrfbr_evpntest: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master vrf_evpntest state UP group default qlen 1000
          link/ether b2:4b:7f:6f:29:78 brd ff:ff:ff:ff:ff:ff
          inet6 fe80::b04b:7fff:fe6f:2978/64 scope link proto kernel_ll
             valid_lft forever preferred_lft forever
      12: tap114i0: <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP> mtu 1500 qdisc fq_codel master vmbr0 state UNKNOWN group default qlen 1000
          link/ether 2a:b8:07:22:c2:5d brd ff:ff:ff:ff:ff:ff
      13: veth104i0@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master vmbr0 state UP group default qlen 1000
          link/ether fe:4d:c5:f3:b5:19 brd ff:ff:ff:ff:ff:ff link-netnsid 0
      14: tap100i0: <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP> mtu 1500 qdisc fq_codel master vmbr0 state UNKNOWN group default qlen 1000
          link/ether f2:95:df:1d:b4:71 brd ff:ff:ff:ff:ff:ff
      15: tap108i0: <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP> mtu 1500 qdisc fq_codel master vmbr0 state UNKNOWN group default qlen 1000
          link/ether 1a:06:80:75:dd:c7 brd ff:ff:ff:ff:ff:ff
      16: veth109i0@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master vmbr0 state UP group default qlen 1000
          link/ether fe:f4:31:d5:c3:d3 brd ff:ff:ff:ff:ff:ff link-netnsid 1
      17: veth111i0@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master vmbr0 state UP group default qlen 1000
          link/ether fe:97:36:67:15:73 brd ff:ff:ff:ff:ff:ff link-netnsid 2
      19: veth106i0@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master vmbr0 state UP group default qlen 1000
          link/ether fe:f8:d1:ed:2f:96 brd ff:ff:ff:ff:ff:ff link-netnsid 3
      root@server5:~#

      root@server5:~# ip route get 192.168.1.25
      192.168.1.25 dev vmbr0 src 192.168.1.15 uid 0
          cache
      root@server5:~#
    ```

DHCP relay on server5, you want to relay from the VNet bridge (testnet1) to your LAN bridge (vmbr0) where Kea
  (192.168.1.25) lives.

What those EVPN/VRF interfaces are

-   vrf_evpntest: a Linux VRF device (a separate routing table/domain). Proxmox puts EVPN zone interfaces under this VRF so tenants
    are isolated at L3.
-   vrfbr_evpntest: an internal bridge used for the EVPN L3 VNI plumbing of the VRF (it ties the VRF to the L3 VXLAN device).
-   vrfvx_evpntest: the VXLAN interface for the L3 VNI (your VRF-VXLAN 10000). It carries L3 EVPN routes between nodes.
-   (Related) testnet1: your tenant L2 bridge for VNI/tag 100. This is where VM DHCP broadcasts appear.
-   (Related) vxlan_testnet1: the VXLAN interface for the L2 VNI (it’s a port on testnet1).

Which interface to use for DHCP relay

-   Client side (where to listen): testnet1 (it has 10.60.10.1/24, so it can be used as giaddr)
-   Server side (how to reach Kea): vmbr0 (your ip route get 192.168.1.25 already shows it)

Proxmox node relay config (ISC dhcrelay)
On server5 (and optionally server2 as well), configure the relay to listen on both interfaces and forward to Kea:

-   SERVERS="192.168.1.25"
-   INTERFACES="testnet1 vmbr0"
-   start isc-dhcp-relay

Conceptually, the relay does:

-   receive DHCPDISCOVER on testnet1
-   forward unicast to 192.168.1.25 via vmbr0
-   receive DHCPOFFER back on vmbr0
-   broadcast/unicast it back to the VM on testnet1

Kea side (must match relayed subnet)
In your Kea subnet4 for 10.60.10.0/24, make sure Kea accepts relayed requests for that subnet by matching the relay’s giaddr:

-   relay ip match: 10.60.10.1 (the address on testnet1)
-   router option: 10.60.10.1

If Kea isn’t matching giaddr=10.60.10.1 to the 10.60.10.0/24 subnet, it will ignore the request or pick the wrong subnet.

```file
• Here’s a solid Kea DHCPv4 subnet4 entry for your EVPN subnet, assuming:

  - Subnet: 10.60.10.0/24
  - Anycast gateway / relay giaddr: 10.60.10.1 (your testnet1 on PVE)
  - Pool: 10.60.10.100-10.60.10.200
  - DNS: set to your preference (example below uses 192.168.1.25)

  {
    "id": 106010,
    "subnet": "10.60.10.0/24",
    "pools": [
      { "pool": "10.60.10.100-10.60.10.200" }
    ],
    "relay": {
      "ip-addresses": [ "10.60.10.1" ]
    },
    "option-data": [
      { "name": "routers", "data": "10.60.10.1" },
      { "name": "domain-name-servers", "data": "192.168.1.25" }
    ],
    "valid-lifetime": 3600,
    "renew-timer": 900,
    "rebind-timer": 1800
  }
```

Direct CLI invocation (what the service ultimately runs): e.g. dhcrelay -4 -i testnet1 -i vmbr0 192.168.1.25 (start/stop via
    systemd or your automation)
