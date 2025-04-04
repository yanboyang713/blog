---
title: "RouterOS Controller Bridge and Port Extender"
draft: false
---

[RouterOS]({{< relref "20230417083431-routeros.md" >}})


## Summary {#summary}

Controller Bridge (CB) and Port Extender (PE) is an IEEE 802.1BR standard implementation in RouterOS. It allows virtually extending the CB ports with a PE device and manage these extended interfaces from a single controlling device. Such configuration provides a simplified network topology, flexibility, increased port density and ease of manageability. An example of Controller Bridge and Port Extender topology can be seen below.

{{< figure src="https://help.mikrotik.com/docs/download/attachments/37224456/CB_summary.png?version=1&modificationDate=1598966520696&api=v2" >}}

The Controller Bridge establishes communication with the Port Extender through a cascade port. Similarly, the Port Extender will communicate with the Controller Bridge only through an upstream port. On a PE device, control ports must be configured and only one port (closest to the CB) will act as an upstream port, other control ports can act as a backup for upstream port or even cascade port for switches connected in series (e.g. Port Extender 2 and 3 in the image above). Cascade and upstream ports are used to transmit and receive control and network traffic. Extended ports are interfaces that are controlled by the CB device and they are typically connected to the end hosts. Extended ports only transmit and receive network traffic.


## Quick setup {#quick-setup}

**NOTE**: In order to exclude some port from being extended (e.g. for out-of-band management purposes), additionally, configure excluded-ports property.

**NOTE**: Make sure not to include the cascade-ports and control-ports in any routing or bridging configurations. These ports are recommended only for a CB and PE usage.

In this example, we will create a Controlling Bridge that will connect to a single Port Extender through an SFP+1 interface.

First, configure a bridge with enabled VLAN filtering on a CB device:

```bash
/interface bridge
add name=bridge1 vlan-filtering=yes
```

On the same device, configure a port that is connected to the PE device and will act as cascade port:


## Reference List {#reference-list}

1.  <https://help.mikrotik.com/docs/display/ROS/Controller+Bridge+and+Port+Extender>
