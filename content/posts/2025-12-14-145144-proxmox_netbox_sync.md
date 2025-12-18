---
title: "Proxmox NetBox Sync"
date: 2025-12-14
draft: false
---

[Proxmox VE (PVE)]({{< relref "20230228043925-proxmox_ve.md" >}})
[NetBox: The Network Source of Truth]({{< relref "2025-11-30-174834-netbox.md" >}})


## net-pve-sync {#net-pve-sync}

`net-pve-sync` (based on `netbox-pve-sync`) is a pull-based synchronization job that reads VM/inventory data from the
Proxmox VE API and reconciles it into NetBox (create/update/delete as needed).

-   Sync direction: Proxmox VE -&gt; NetBox (VMs, disks, interfaces, IPs/prefixes, MACs, VLANs, and a few metadata fields).
-   Auth: NetBox API token + Proxmox API token (read-only perms like `VM.Monitor`, `VM.Audit`, `Sys.Audit` are typically sufficient).
-   NetBox prep: create the cluster and physical hosts, and add the custom fields expected by the sync (ex: `autostart`, `ha`, `backup`).

<!--listend-->

```sh
PVE_API_HOST=xx PVE_API_USER=xx PVE_API_TOKEN=xx PVE_API_SECRET=xx \
NB_API_URL=xx NB_API_TOKEN=xx NB_CLUSTER_ID=xx \
nbpxsync
```


## Reference List {#reference-list}

1.  <https://github.com/netdevopsbr/netbox-proxbox>
2.  <https://github.com/creekorful/netbox-pve-sync>
3.  <https://netboxlabs.com/blog/automate-proxmox-virtual-machine-configuration-netbox-flask-application/>
4.  <https://github.com/netboxlabs/netbox-proxmox-automation/>
5.  <https://netboxlabs.com/blog/automate-proxmox-virtual-machine-configuration-netbox-ansible-awx-tower-automation-platform-aap/>
