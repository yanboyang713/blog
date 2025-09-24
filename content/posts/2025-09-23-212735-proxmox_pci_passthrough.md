---
title: "proxmox PCI passthrough"
date: 2025-09-23
draft: false
---

[Proxmox VE]({{< relref "20230228043925-proxmox_ve.md" >}})


## Requirements {#requirements}

-   CPU requirements: Your CPU has to support hardware virtualization and IOMMU. Most new CPUs support this.
-   Motherboard requirements: Your motherboard needs to support IOMMU. Lists can be found on the Xen wiki and Wikipedia. Note that, as of writing, both these lists are incomplete and very out-of-date and most newer motherboards support IOMMU.


## Reference List {#reference-list}

1.  <https://pve.proxmox.com/wiki/PCI_Passthrough>
