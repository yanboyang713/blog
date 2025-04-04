---
title: "mount usb with udiskctl"
draft: false
---

The udisks2 package allows mounting without sudo via its CLI tool udiskctl


## Ensure udisks2 is installed {#ensure-udisks2-is-installed}

```bash
paru -S udisks2
```


## List available drives {#list-available-drives}

```bash
lsblk
```


## mount disk {#mount-disk}

```bash
udisksctl mount -b /dev/sdX1
```

Example:

```console
yanboyang713@Meta-Scientific-Linux ~/media % udisksctl mount -b /dev/sda1
Mounted /dev/sda1 at /run/media/yanboyang713/0104-0B69
```


## Unmount the partition {#unmount-the-partition}

```console
  yanboyang713@Meta-Scientific-Linux ~ % udisksctl unmount -b /dev/sda1
Unmounted /dev/sda1.
```


## If you wish to safely remove the device after unmounting, run {#if-you-wish-to-safely-remove-the-device-after-unmounting-run}

```console
udisksctl power-off -b /dev/sda1
```
