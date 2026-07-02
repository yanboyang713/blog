---
title: "main switch details"
date: 2026-01-03
draft: false
---

```bash
/ip firewall filter add chain=forward src-address=192.168.88.0/24 dst-address=192.168.1.0/24 action=accept comment="ALLOW 88->1"
/ip firewall filter add chain=forward src-address=192.168.1.0/24 dst-address=192.168.88.0/24 action=accept comment="ALLOW 1->88"

```

move both to the top:

```bash
/ip firewall filter move [find comment="ALLOW 88->1"] 0
/ip firewall filter move [find comment="ALLOW 1->88"] 0

```


## Add return routes on three [VyOS]({{< relref "2025-04-10-012017-vyos.md" >}}) to 192.168.88.0/24 network {#add-return-routes-on-three-vyos--2025-04-10-012017-vyos-dot-md--to-192-dot-168-dot-88-dot-0-24-network}

On your 192.168.1.x default gateway (three [VyOS]({{< relref "2025-04-10-012017-vyos.md" >}}))
Route 192.168.88.0/24 via 192.168.1.1

```bash
configure
set protocols static route 192.168.88.0/24 next-hop 192.168.1.1
commit
save
exit
```

Verification:

```bash
show ip route 192.168.88.0/24
ping 192.168.88.2
traceroute 192.168.88.1
```

**NOTE**:

1.  Check default route

<!--listend-->

```bash
ip -4 route show default
```

If default route is empty
find gateway

```bash
show dhcp client leases
```

add Add the missing default route and retest:

```bash
sudo ip route add default via 172.27.160.1 dev wlan0
ip route get 1.1.1.1
```

Persistent fix (if you just want it working)

```bash
configure
set protocols static route 0.0.0.0/0 next-hop 172.27.160.1
commit; save
```


## Add route from MikroTik cAP ax IP: 192.168.88.1 to 192.168.1.0/24 network {#add-route-from-mikrotik-cap-ax-ip-192-dot-168-dot-88-dot-1-to-192-dot-168-dot-1-dot-0-24-network}

On the 192.168.88.x gateway ([MikroTik cAP ax]({{< relref "2025-04-10-010004-data_center_testbed_design.md#id-7b3d4c7a-30a8-4f0f-a587-fdbb39109e57-mikrotik-cap-ax--gateway-between-management-network-and-wahoo" >}}) IP: 192.168.88.1), add:
Route 192.168.1.0/24 via 192.168.88.2 (the MikroTik)
MikroTik cAP (CLI)

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
