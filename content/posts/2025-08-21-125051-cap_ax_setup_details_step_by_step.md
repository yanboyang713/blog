---
title: "cAP ax setup details (step by step)"
date: 2025-08-21
draft: false
---

WIFIs manage on local, not [CAPsMAN]({{< relref "2024-03-11-134846-add_ap_to_capsman.md" >}}).


## Build the LAN side {#build-the-lan-side}

```bash
  # Create the LAN bridge
/interface/bridge add name=lan protocol-mode=rstp

# Put your Ethernet uplink to the switch into the LAN bridge (adjust ports as needed)
/interface/bridge/port
add bridge=lan interface=ether1
```


## Give the LAN an IP + DHCP server {#give-the-lan-an-ip-plus-dhcp-server}

```bash
# Give the router a LAN IP and a small DHCP server (optional but typical)
# Use the subnet you like; 192.168.88.0/24 shown here
/ip address add address=192.168.88.1/24 interface=lan
/ip pool add name=dhcp_pool_lan ranges=192.168.88.10-192.168.88.254
/ip dhcp-server add name=dhcp_lan interface=lan address-pool=dhcp_pool_lan
/ip dhcp-server network add address=192.168.88.0/24 gateway=192.168.88.1 dns-server=192.168.88.1
/ip dhcp-server enable dhcp_lan
```


## Configure Wi-Fi station to wahoo (open, hidden) {#configure-wi-fi-station-to-wahoo--open-hidden}

```bash
/interface/wifi set wifi1 ssid="wahoo" mode=station disabled=no
```


## Get WAN IP via DHCP on wifi1 {#get-wan-ip-via-dhcp-on-wifi1}

```bash
/ip/dhcp-client add interface=wifi1 use-peer-dns=yes add-default-route=yes
```


## NAT + basic firewall {#nat-plus-basic-firewall}

```bash
  /interface/list add name=WAN
/interface/list/member add list=WAN interface=wifi1

/ip/firewall/nat
add chain=srcnat out-interface-list=WAN action=masquerade comment="WAN via wahoo"

/ip/firewall/filter
add chain=input action=accept connection-state=established,related
add chain=input action=accept in-interface=lan comment="manage from LAN"
add chain=input action=drop in-interface-list=WAN comment="drop unsolicited from WAN"

add chain=forward action=accept connection-state=established,related
add chain=forward action=accept in-interface=lan out-interface-list=WAN
add chain=forward action=drop

```
