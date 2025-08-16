---
title: "RouterOS port forward from LAN to WAN"
date: 2025-08-15
draft: false
---

For example, forward WAN port 8006 to LAN server 192.168.1.11:8006 WAN port name WAN

1.  Make sure ether1 is in the WAN list:

<!--listend-->

```bash
/interface list add name=WAN comment="WAN uplinks"  ;# (skip if it already exists)
/interface list member add list=WAN interface=ether1
```

1.  Reference the list in your rules:

<!--listend-->

```bash
# DNAT
/ip firewall nat add chain=dstnat in-interface-list=WAN protocol=tcp dst-port=8006 \
  action=dst-nat to-addresses=192.168.1.11 to-ports=8006

# Allow the forwarded traffic (for any dst-nat)
/ip firewall filter add chain=forward connection-nat-state=dstnat action=accept
```

1.  (Optional) General src-NAT / masquerade for outbound

If you don’t already have it:

```bash
/ip firewall nat add chain=srcnat out-interface-list=WAN action=masquerade
```

1.  (Optional) Hairpin NAT (LAN clients hitting your public IP)

Replace &lt;bridge&gt; with your actual bridge name (often bridge):

```bash
/ip firewall nat add chain=srcnat src-address=192.168.1.0/24 dst-address=192.168.1.11 \
  out-interface=<bridge> action=masquerade
```
