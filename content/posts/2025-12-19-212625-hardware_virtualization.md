---
title: "Hardware Virtualization"
date: 2025-12-19
draft: false
---

## What is hardware virtualization {#what-is-hardware-virtualization}

Hardware virtualization is CPU/chipset support that helps a hypervisor run multiple isolated operating systems (virtual machines) efficiently on the same physical machine.

Typically it provides:

-   [CPU virtualization (Intel VT-x / AMD-V)](#cpu-virtualization--intel-vt-x-amd-v): lets a guest OS run in a controlled mode while the hypervisor retains control.
-   Memory virtualization (e.g., EPT/NPT): accelerates address translation between guest and host memory.
-   I/O and device virtualization (e.g., VT-d/IOMMU, SR-IOV): isolates and optionally passes through devices safely.


## CPU virtualization (Intel VT-x / AMD-V) {#cpu-virtualization--intel-vt-x-amd-v}


### Quick CPU capability check {#quick-cpu-capability-check}

```bash
egrep -wo 'vmx|svm' /proc/cpuinfo | head
```

-   vmx = Intel VT-x
-   svm = AMD-V

If this prints nothing, the CPU either doesn’t support it or it’s disabled in BIOS/UEFI.


## [GPU]({{< relref "2023-10-30-193539-gpu.md" >}}) {#gpu--2023-10-30-193539-gpu-dot-md}

-   [Virtualization with Nvidia vGPU]({{< relref "20230518175221-vgpu.md" >}})
