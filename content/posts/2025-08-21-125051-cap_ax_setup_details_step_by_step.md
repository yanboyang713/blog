---
title: "cAP ax setup details (step by step)"
date: 2025-08-21
draft: false
---

ALL of configation start from Scratch.

WIFIs manage on local, not [CAPsMAN]({{< relref "2024-03-11-134846-add_ap_to_capsman.md" >}}).


## Build the LAN side {#build-the-lan-side}

```bash
  # Create the LAN bridge
/interface/bridge add name=LAN protocol-mode=rstp

# Put your Ethernet uplink to the switch into the LAN bridge (adjust ports as needed)
/interface/bridge/port
add bridge=LAN interface=ether1
add bridge=LAN interface=ether2
add bridge=LAN interface=wifi2
```


## Set DNS {#set-dns}

```bash
/ip dns set servers=8.8.8.8,8.8.4.4
/ip dns set allow-remote-requests=yes
```


## Give the LAN an IP + DHCP server {#give-the-lan-an-ip-plus-dhcp-server}

```bash
# Give the router a LAN IP and a small DHCP server (optional but typical)
# Use the subnet you like; 192.168.88.0/24 shown here
/ip address add address=192.168.88.1/24 interface=LAN
/ip pool add name=dhcp_pool_lan ranges=192.168.88.10-192.168.88.254
/ip dhcp-server add name=dhcp_lan interface=LAN address-pool=dhcp_pool_lan
/ip dhcp-server network add address=192.168.88.0/24 gateway=192.168.88.1 dns-server=192.168.88.1
/ip dhcp-server enable dhcp_lan
```


## Configure Wi-Fi station to wahoo (open, hidden) {#configure-wi-fi-station-to-wahoo--open-hidden}

```bash
/interface/wifi/configuration
add name=cfg-wahoo mode=station ssid="wahoo" country="United States" security.authentication-types=""

/interface/wifi
set [find name="wifi1"] configuration=cfg-wahoo disabled=no
```


## Get WAN IP via DHCP on wifi1 {#get-wan-ip-via-dhcp-on-wifi1}

```bash
  /ip/dhcp-client add interface=wifi1 use-peer-dns=yes add-default-route=yes
  /ip/dhcp-client
enable [find interface="wifi1"]
```


## NAT + basic firewall {#nat-plus-basic-firewall}

```bash
/interface/list add name=WAN
/interface/list/member add list=WAN interface=wifi1

/ip/firewall/nat
add chain=srcnat out-interface-list=WAN action=masquerade comment="WAN via wahoo"

/ip/firewall/filter
add chain=input action=accept connection-state=established,related
add chain=input action=accept in-interface=LAN comment="manage from LAN"
add chain=input action=drop in-interface-list=WAN comment="drop unsolicited from WAN"

add chain=forward action=accept connection-state=established,related
add chain=forward action=accept in-interface=LAN out-interface-list=WAN
add chain=forward action=drop
```


## Create a LAN WIFI {#create-a-lan-wifi}


### Create a security profile (password) {#create-a-security-profile--password}

```bash
/interface/wifi/security
add name=sec-lan authentication-types=wpa2-psk,wpa3-psk passphrase="xxx"
```


### Create an AP configuration profile for LAN Wi-Fi {#create-an-ap-configuration-profile-for-lan-wi-fi}

```bash
/interface/wifi/configuration
add name=cfg-lan mode=ap ssid="LAN-WiFi" security=sec-lan country="United States" installation=indoor
```


### Apply it to wifi2 and enable {#apply-it-to-wifi2-and-enable}

```bash
/interface/wifi
set [find name="wifi2"] configuration=cfg-lan disabled=no
```


### Verify {#verify}

```bash
/interface/wifi print detail where name="wifi2"
/interface/wifi monitor wifi2 once
/interface/wifi registration-table print
```


## Access 192.168.1.x network through [Proxmox VE (PVE) main switch MikroTik L009UiGS-RM]({{< relref "2025-04-10-010004-data_center_testbed_design.md#id-77bd7428-f1ee-4306-8d5a-62f38134dfc5-proxmox-ve--pve--main-switch-id-7b3d4c7a-30a8-4f0f-a587-fdbb39109e57-mikrotik-id-9e995513-6ee6-45d8-b224-080c85d13264-l009uigs-rm" >}}) {#access-192-dot-168-dot-1-dot-x-network-through-proxmox-ve--pve--main-switch-mikrotik-l009uigs-rm--2025-04-10-010004-data-center-testbed-design-dot-md}

```bash
/ip route add dst-address=192.168.1.0/24 gateway=192.168.88.2 comment="to 192.168.1 via L009"
```

Verify

```bash
/ip route print where dst-address=192.168.1.0/24
/ping 192.168.88.2
/ping 192.168.1.1
```

MikroTik cAP (WinBox)

-   IP → Routes → +
-   Dst. Address: 192.168.1.0/24
-   Gateway: 192.168.88.2

Apply/OK


### Allow Winbox (TCP 8291) in the MikroTik firewall(s) {#allow-winbox--tcp-8291--in-the-mikrotik-firewall--s}

On the MikroTik you are trying to manage (the cAP), allow Winbox from 192.168.1.0/24 (adjust if you already have a structured ruleset):

```bash
/ip firewall filter add chain=input src-address=192.168.1.0/24 protocol=tcp dst-port=8291 action=accept comment="Allow Winbox from 192.168.1"
```


### Also confirm Winbox service is enabled and (optionally) restricted to your management subnets {#also-confirm-winbox-service-is-enabled-and--optionally--restricted-to-your-management-subnets}

```bash
/ip service set winbox disabled=no address=192.168.1.0/24,192.168.88.0/24
```

**NOTE**:

-   Connect in Winbox by IP, not by MAC/Neighbors
-   Winbox “Neighbors” discovery is L2/broadcast-based and will not traverse routed subnets. That’s normal.
-   Just type the target device IP (e.g., 192.168.88.1) into “Connect To”.


## [Port Exposing from LAN to WAN]({{< relref "2025-08-15-212717-routeros_port_forward_from_lan_to_wan.md" >}}) {#port-exposing-from-lan-to-wan--2025-08-15-212717-routeros-port-forward-from-lan-to-wan-dot-md}

expose tcp/8006 on WAN to 192.168.1.11:8006 ([Data Center Testbed Design]({{< relref "2025-04-10-010004-data_center_testbed_design.md" >}})'s server 1) on LAN
expose tcp/3128 for [SPICE (Simple Protocol for Independent Computing Environments)]({{< relref "2025-05-04-035528-spice.md" >}}) on WAN to 192.168.1.11:8006 (pve's server 1) on LAN.
