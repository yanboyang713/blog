---
title: "arch linux mirrors"
date: 2025-08-11
draft: false
---

selecting and configuring your [Arch Linux]({{< relref "20230220222636-arch_linux.md" >}}) mirrors, and a listing of current available mirrors.


## Sorting mirrors {#sorting-mirrors}


### Fetching and ranking a live mirror list {#fetching-and-ranking-a-live-mirror-list}

In order to start with a shortlist of up-to-date mirrors based in some countries and feed it to rankmirrors one can fetch the list from the Pacman Mirrorlist Generator. The command below pulls the up-to-date mirrors, it uncomments the servers in the list and then ranks them and outputs the 25 fastest.

```bash
curl -s "https://archlinux.org/mirrorlist/?country=US&protocol=https&use_mirror_status=on"| sed -e 's/^#Server/Server/' -e '/^#/d' | rankmirrors -n 25 -
```


## Reference List {#reference-list}

1.  <https://wiki.archlinux.org/title/Mirrors>
