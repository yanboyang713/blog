---
title: "7-Zip compressed file"
draft: false
---

## install {#install}


### [ubuntu]({{< relref "20230220222545-ubuntu.md" >}}) {#ubuntu--20230220222545-ubuntu-dot-md}

```bash
sudo apt-get install p7zip-full
```


### [Arch Linux]({{< relref "20230220222636-arch_linux.md" >}}) {#arch-linux--20230220222636-arch-linux-dot-md}

<https://wiki.archlinux.org/title/p7zip>

```bash
paru -S p7zip
```


## 7-Zip Extract {#7-zip-extract}

extract a .7z file with the following command:

```bash
7z x yourfile.7z
```

If you want to extract the contents to a specific directory, you can use the -o option. For example:

```bash
7z x yourfile.7z -o/path/to/extract/directory/
```
