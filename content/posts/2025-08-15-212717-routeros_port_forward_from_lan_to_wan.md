---
title: "RouterOS Port Forward from LAN to WAN"
date: 2025-08-15
draft: false
---

For example, forward WAN port 8006 to LAN server 192.168.1.11:8006 WAN port name WAN


## Forward WAN:8006 → Server:8006 {#forward-wan-8006-server-8006}

```bash
/ip firewall nat add chain=dstnat \
  in-interface-list=WAN \
  protocol=tcp dst-port=8006 \
  action=dst-nat to-addresses=192.168.1.11 to-ports=8006
```


## Allow traffic {#allow-traffic}

[firewall]({{< relref "20230404162828-firewall.md" >}})

```bash
/ip firewall filter add chain=forward \
  in-interface-list=WAN \
  dst-address=192.168.1.11 \
  protocol=tcp dst-port=8006 \
  action=accept
```
