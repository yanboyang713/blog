---
title: "RouterOS Port Exposing from LAN to WAN"
date: 2025-08-15
draft: false
---

For example, forward WAN port 8006 to LAN server 192.168.1.11:8006 WAN port name WAN
Same works: [VyOS Publishing and exposing ports (options)]({{< relref "2025-04-10-012017-vyos.md#vyos-publishing-and-exposing-ports--options" >}})


## DNAT WAN:8006 → Server:8006 {#dnat-wan-8006-server-8006}

Example: expose tcp/8006 on WAN to 192.168.1.11:8006 on LAN:

```bash
/ip firewall nat add chain=dstnat \
  in-interface-list=WAN \
  protocol=tcp dst-port=8006 \
  action=dst-nat to-addresses=192.168.1.11 to-ports=8006
  comment="Port forward 8006 from WAN → 192.168.1.11"
```

**NOTE**:
Use your real in-interface or in-interface-list=WAN and server IP.


### [Firewall]({{< relref "20230404162828-firewall.md" >}}) Setting for DNAT {#firewall--20230404162828-firewall-dot-md--setting-for-dnat}

If you are using the default MikroTik firewall, there is usually already a rule:

```bash
/ip firewall filter
add chain=forward connection-nat-state=dstnat action=accept comment="Allow all traffic that was dst-natted (port forwards)"
```

Place it above your generic drop rule in forward.

If you prefer to be strict and only allow a specific server/port:

```bash
/ip firewall filter
add chain=forward \
    in-interface-list=WAN \
    dst-address=192.168.1.11 \
    protocol=tcp dst-port=8006 \
    connection-state=new \
    action=accept \
    comment="Allow new WAN→192.168.1.11:8006"
```

Make sure you still have the usual rules:

```bash
/ip firewall filter
add chain=forward connection-state=established,related action=accept comment="allow established/related"
add chain=forward connection-state=invalid action=drop comment="drop invalid"
# (your allow rules here, e.g. dstnat / LAN→WAN)
add chain=forward action=drop comment="drop everything else"
```


## SNAT: make the server see the router’s LAN IP {#snat-make-the-server-see-the-router-s-lan-ip}

This makes server1 (192.168.1.11) see 192.168.1.1 (L009 LAN IP) as the source for all forwarded :8006 connections.
Translates the source to the gateway IP to keep return traffic symmetric.

```bash
/ip firewall nat add chain=srcnat \
    out-interface-list=LAN \
    dst-address=192.168.1.11 \
    protocol=tcp dst-port=8006 \
    action=src-nat to-addresses=192.168.1.1 \
    comment="SNAT forwarded 8006 so server replies to 192.168.1.1"
```

**Notes**:

-   dst-address=192.168.1.11 → your PVE / server1.
-   dst-port=8006 → the forwarded port.
-   to-addresses=192.168.1.1 → L009’s LAN IP (what the server will see as the client).
-   out-interface-list=LAN assumes you’re using interface lists (LAN / WAN like in your post).
    -   If not, swap it for out-interface=bridge or the actual LAN interface name.
-   You don’t need a special firewall rule just because you added SNAT. SNAT happens in postrouting, after the forward chain filter decision is already made.
