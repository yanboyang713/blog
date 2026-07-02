---
title: "Proxmox User Management"
draft: false
---

[Proxmox VE]({{< relref "20230228043925-proxmox_ve.md" >}})


## GUI add new User {#gui-add-new-user}

Datacenter -&gt; Permission -&gt; User -&gt; Add
linux user: boyang


## Add to system {#add-to-system}

adduser boyang


## Group {#group}

Datacenter -&gt; Permissions -&gt; Groups

Datacenter -&gt; Permissions -&gt; Add -&gt; Group Permission


## Role {#role}

VM.Allocate VM.Audit VM.Backup VM.Clone VM.Config.CDROM VM.Config.CPU VM.Config.Cloudinit VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Console VM.Migrate VM.Monitor VM.PowerMgmt VM.Snapshot VM.Snapshot.Rollback Sys.Audit Sys.Syslog Datastore.Allocate Datastore.AllocateSpace Datastore.AllocateTemplate Datastore.Audit

[Create Users in Linux]({{< relref "20230529202337-create_users_in_linux.md" >}})


## Reference List {#reference-list}

1.  <https://pve.proxmox.com/wiki/User_Management>
2.  <https://techviewleo.com/create-users-groups-permissions-proxmox/>
