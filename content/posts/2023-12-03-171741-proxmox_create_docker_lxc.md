---
title: "proxmox create docker lxc"
draft: false
---

## Pre-requires {#pre-requires}

-   [Proxmox VE]({{< relref "20230228043925-proxmox_ve.md" >}})
-   [Linux Containers (LXC)]({{< relref "2023-12-03-143424-lxc.md" >}})
-   [docker]({{< relref "20230216042646-docker.md" >}})


## Helper {#helper}

If the LXC is created Privileged, the script will automatically set up USB passthrough.

To create a new Proxmox VE Docker LXC, run the command below in the Proxmox VE Shell.

```bash
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/docker.sh)"
```

⚡ Default Settings: 2GB RAM - 4GB Storage - 2vCPU ⚡

As an alternative option, you can use Alpine Linux and the Docker package to create a Docker LXC container with faster creation time and minimal system resource usage.

To create a new Proxmox VE Alpine-Docker LXC, run the command below in the Proxmox VE Shell.

```bash
bash -c "$(wget -qO - https://github.com/tteck/Proxmox/raw/main/ct/alpine-docker.sh)"
```

⚡ Default Settings: 1GB RAM - 2GB Storage - 1vCPU ⚡

⚠ Run Compose V2 by replacing the hyphen (-) with a space, using docker compose, instead of docker-compose.

Portainer Interface: IP:9443


## Reference list {#reference-list}

1.  <https://tteck.github.io/Proxmox/>
