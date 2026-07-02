---
title: "USB Flash Drive as Installation Medium"
tags: ["DD"]
draft: false
---

## Download the installer ISO image {#download-the-installer-iso-image}


## Prepare a USB Flash Drive as Installation Medium {#prepare-a-usb-flash-drive-as-installation-medium}


### list Disk in Linux <span class="tag"><span class="fdisk">fdisk</span><span class="lsblk">lsblk</span></span> {#list-disk-in-linux}

Find the Correct USB Device Name

There are two ways to find out the name of the USB flash drive.

-   The first one is use lsblk (strong recommend).
    ```console
    boyang:~$ lsblk
    NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sda           8:0    1   3.8G  0 disk
    ├─sda1        8:1    1   242K  0 part
    ├─sda2        8:2    1   2.8M  0 part
    ├─sda3        8:3    1 859.2M  0 part /media/yanboyang713/PVE1
    └─sda4        8:4    1   300K  0 part
    nvme0n1     259:0    0 465.8G  0 disk
    ├─nvme0n1p1 259:1    0   512M  0 part /boot/efi
    └─nvme0n1p2 259:2    0 465.3G  0 part /
    ```

-   The second way is to compare the output of the fdisk -l command. Both example at the below.

<!--listend-->

```console
boyang:~$ sudo fdisk -l
[sudo] password for yanboyang713:
Disk /dev/nvme0n1: 465.8 GiB, 500107862016 bytes, 976773168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: C60CB535-938E-4A9F-9BAC-92FDE1DD99B0

Device           Start       End   Sectors   Size Type
/dev/nvme0n1p1    2048   1050623   1048576   512M EFI System
/dev/nvme0n1p2 1050624 976771071 975720448 465.3G Linux filesystem

Disk /dev/sda: 3.8 GiB, 4081057792 bytes, 7970816 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 50FC8F62-2C89-43B6-8D1F-C8D3CC3F8B72

Device       Start     End Sectors   Size Type
/dev/sda1       64     547     484   242K Microsoft basic data
/dev/sda2      548    6307    5760   2.8M EFI System
/dev/sda3     6308 1765843 1759536 859.2M Apple HFS/HFS+
/dev/sda4  1765844 1766443     600   300K Microsoft basic data
```


## Make a bootable USB using DD <span class="tag"><span class="dd">dd</span><span class="bootable">bootable</span><span class="usb">usb</span></span> {#make-a-bootable-usb-using-dd}

On Unix-like operating system use the dd command to copy the ISO image to the USB flash drive. First find the correct device name of the USB flash drive (see above). Then run the dd command.

```console
[yanboyang713@manjaro] ➜ ~ sudo dd bs=1M conv=fdatasync if=/home/yanboyang713/Downloads/proxmox-ve.iso of=/dev/sda
812+1 records in
812+1 records out
852299776 bytes (852 MB, 813 MiB) copied, 185.781 s, 4.6 MB/s
```
