---
title: "Permanently Disable Swap Partition"
draft: false
---

[what is Swap]({{< relref "20230531222120-swap.md" >}})


## How to Check Swap Space in Linux {#how-to-check-swap-space-in-linux}

Before actually disabling swap space, first, you need to visualize your memory load degree and then identify the partition that holds the swap area, by issuing the below free command.

```console
boyang@k8s-worker2:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       193Mi       3.3Gi       0.0Ki       340Mi       3.4Gi
Swap:          3.0Gi          0B       3.0Gi
```

Look for the Swap space used size. If the used size is 0B or close to 0 bytes, it can be assumed that swap space is not used intensively and can be safely disabled.


## How to Disable Swap in Linux {#how-to-disable-swap-in-linux}

After you’ve identified the swap partition or file, execute the below command to deactivate the swap area.

```bash
sudo swapoff -a
```

Run free command in order to check if the swap area has been disabled.

```console
boyang@k8s-worker2:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       190Mi       3.3Gi       0.0Ki       342Mi       3.4Gi
Swap:             0B          0B          0B
```


## How to Disable Swap Permanently in Linux {#how-to-disable-swap-permanently-in-linux}

In order to permanently disable swap space in Linux, open /etc/fstab file, search for the swap line and comment on the entire line by adding a # (hashtag) sign in front of the line, as shown in the below.

```console
vi /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to nam
e devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <
pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-oqXcZ9SBcGRIYBMm921eX1ifQSqU4LnDm5YhM
NjdAmLFAYKSU9XfhHl96Q2P2ywd / ext4 defaults 0 1
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/72af1baa-fd38-42a3-a6ea-5b894da6d38b /boot ext4
 defaults 0 1
/swap.img       none    swap    sw      0       0
```

Afterward, reboot the system in order to apply the new swap setting or issue mount -a command in some cases might do the trick.

```bash
mount -a
```


## Auto Script {#auto-script}

```bash
#!/bin/bash

# Function to comment out lines containing the keyword
comment_lines_with_keyword() {
    local keyword=$1
    local file=$2

    # Create a temporary file
    local temp_file=$(mktemp)

    while IFS= read -r line; do
        if [[ $line == *"$keyword"* ]]; then
            echo "# $line" >> "$temp_file"
        else
            echo "$line" >> "$temp_file"
        fi
    done < "$file"
    # Make file backup
    sudo mv "$file" "$file.backup"
    # Overwrite the original file with the modified content
    sudo mv "$temp_file" "$file"
    echo "Modified content written back to $file"
}

# check swap size, if swap great than 1 bytes, disable Swap permanently
disable_swap_permanently() {
    # Get swap size in bytes
    swap_size=$(grep 'SwapTotal' /proc/meminfo | awk '{print $2}')

    # Check if the variable is less than 1
    if (( $(echo "$swap_size < 1" | bc -l) )); then
        echo "The Swap size is less than 1 bytes."
    else
        echo "The variable is not less than 1."
        # Print the swap size
        echo "Swap Size: $swap_size"
        # disable swap permanently
        comment_lines_with_keyword "swap" "/etc/fstab"
    fi

}

echo "K8S required Disable Swap"
read -p "Do you want to Disable Swap Permanently? Otherwise Disable Temporarily (yes/no): " answer

case "$answer" in
    [Yy][Ee][Ss]|[Yy])
        echo "Disable Swap Permanently"
        disable_swap_permanently
        # apply the new swap setting
        mount -a
        ;;
    [Nn][Oo]|[Nn])
        echo "Disable Swap Temporarily"
        sudo swapoff -a
        ;;
    *)
        echo "Invalid input. Please enter 'yes' or 'no'."
        ;;
esac
```
