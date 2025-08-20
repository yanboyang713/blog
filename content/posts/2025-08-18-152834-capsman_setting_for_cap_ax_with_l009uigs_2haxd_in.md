---
title: "CAPsMAN Setting for cAP ax with L009UiGS-2HaxD-IN"
date: 2025-08-18
draft: false
---

One station config for 5 GHz uplink, and one or more AP configs for your LAN SSIDs. Use a datapath without a bridge for the station (WAN), and a datapath with the CAP’s bridge for AP SSIDs. (You can reference the remote CAP’s bridge name via CLI even if it doesn’t show up in the GUI.)


## Two parts {#two-parts}


### cAP ax {#cap-ax}


#### give the CAP a unique identity {#give-the-cap-a-unique-identity}

```bash
/system identity set name=cap-uvapod
```


#### Give both radios to CAPsMAN (and remove any local bridge enslaving) {#give-both-radios-to-capsman--and-remove-any-local-bridge-enslaving}

```bash
  /interface/wifi
set [ find default-name=wifi1 ] configuration.manager=capsman
set [ find default-name=wifi2 ] configuration.manager=capsman
```


#### Make sure radios aren't locally added to your bridge (CAPsMAN will attach VAPs) {#make-sure-radios-aren-t-locally-added-to-your-bridge--capsman-will-attach-vaps}

```bash
 /interface/bridge/port
:do { remove [find interface=wifi1] } on-error={}
:do { remove [find interface=wifi2] } on-error={}
```


#### Point CAP to the controller {#point-cap-to-the-controller}

```bash
/interface/wifi/cap
set enabled=yes caps-man-addresses=YOUR_L009_IP
```


#### Point CAP to the controller {#point-cap-to-the-controller}

```bash
/interface/wifi/cap
set enabled=yes caps-man-addresses=YOUR_L009_IP
```

**NOTE**: my controller IP: 192.168.88.2


#### Register the uplink MAC for [wahoo]({{< relref "2025-08-08-101821-uva_eduroam_wireless_network_under_linux.md#wahoo" >}}) {#register-the-uplink-mac-for-wahoo--2025-08-08-101821-uva-eduroam-wireless-network-under-linux-dot-md}

wahoo won’t hand you DHCP otherwise.


#### Attach DHCP client to the station interface (auto-detects it) {#attach-dhcp-client-to-the-station-interface--auto-detects-it}

```bash
   # 3b) DHCP client on the station (if not already)
 /ip/dhcp-client
 :if ([:len [find interface=$staIf]] = 0) do={
     add interface=$staIf use-peer-dns=yes add-default-route=yes disabled=no
 }

/ip/firewall/nat add chain=srcnat out-interface-list=WAN action=masquerade comment="WAN via wahoo"
 # Run it:
 /system/script/run setup-wahoo-uplink

```

/ip/firewall/nat add chain=srcnat out-interface=wifi1 action=masquerade comment="WAN via wahoo"


### L009 setting {#l009-setting}

run on the CAPsMAN controller – L009): build profiles + provisioning so the cAP’s 5 GHz comes up as station to wahoo (WAN) and the 2.4 GHz broadcasts your AP SSID(s)


#### Set Datapaths {#set-datapaths}

```bash
  # --- Datapaths ---
# WAN datapath: leave *no bridge* so the station interface is NOT bridged
/interface/wifi/datapath
add name=dp-wan comment="no bridge for WAN uplink (station)"

# LAN datapath: send AP VAPs into the CAP's bridge (remote name is fine)
/interface/wifi/datapath
add name=dp-lan bridge=bridgeLocal comment="bridge on the CAP for LAN SSIDs"
```

**NOTE**: bridgeLocal is the name of your local bridge.


#### Security for your AP SSIDs {#security-for-your-ap-ssids}

```bash
  /interface/wifi/security
add name=sec-ap authentication-types=wpa2-psk,wpa3-psk passphrase=YourStrongPass
```


#### Configs {#configs}

```bash
  # 5 GHz uplink to hidden, open 'wahoo'
/interface/wifi/configuration
add name=cfg-uplink-5 mode=station ssid="wahoo" \
    country="United States" datapath=dp-wan \
    security.authentication-types="" comment="hidden+open"

# 2.4 GHz AP (add more as needed)
/interface/wifi/configuration
add name=cfg-ap-24 mode=ap ssid="Lab-boyang" country="United States" \
    security=sec-ap datapath=dp-lan
```


#### Provisioning rules {#provisioning-rules}

```bash
# Tip: match this specific CAP by identity so other CAPs aren't turned into stations
# Replace 'cap-uvapod' with the /system identity on your cAP ax.
/interface/wifi/provisioning/add action=create-enabled identity-regexp="^cap-uvapod\$" supported-bands=5ghz-ax master-configuration=cfg-uplink-5 comment="cAP: 5 GHz becomes station (WAN)"

/interface/wifi/provisioning/add action=create-enabled identity-regexp="^cap-uvapod\$" supported-bands=2ghz-ax master-configuration=cfg-ap-24     comment="cAP: 2.4 GHz serves LAN SSIDs"
```


#### Enable CAPsMAN (wifiwave2) {#enable-capsman--wifiwave2}

```bash
/interface/wifi/capsman
set enabled=yes ca-certificate=auto
```


#### Set WAN as wifi1 {#set-wan-as-wifi1}
