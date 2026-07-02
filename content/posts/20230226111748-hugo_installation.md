---
title: "hugo installation"
draft: false
---

In order to run the theme locally, you must have the following tools installed.

-   Hugo version v0.109.x (extended) or later.
-   Go language version v1.18.x or later.
-   Node version v18.x and npm version 8.x or later.


## [Arch Linux]({{< relref "20230220222636-arch_linux.md" >}}) {#arch-linux--20230220222636-arch-linux-dot-md}

```bash
paru -S hugo go nodejs npm
```


## Check {#check}

```bash
# Check Hugo version
➜ hugo version
hugo v0.109.0+extended linux/amd64 BuildDate=unknown

# Check Go version
➜ go version
go version go1.19.4 linux/amd64

# Check Node version
➜ node -v
v18.12.1

# Check NPM version
➜ npm -v
8.19.2
```
