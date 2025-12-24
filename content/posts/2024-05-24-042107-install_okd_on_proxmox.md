---
title: "Deploying OKD Single Node on Proxmox"
draft: false
---

This guide documents the complete setup of [OKD]({{< relref "20230518171737-okd.md" >}}) (The Community Distribution of [Kubernetes]({{< relref "20230105185343-kubernetes.md" >}}) that powers Red Hat OpenShift) as a Single Node Operator (SNO) cluster on a virtual machine hosted by [Proxmox VE]({{< relref "20230228043925-proxmox_ve.md" >}}).

[OKD Bootstrap]({{< relref "2024-05-24-051502-okd_bootstrap.md" >}})


## Prerequisites {#prerequisites}

-   [Proxmox Testbed]({{< relref "2025-04-10-010004-data_center_testbed_design.md" >}}) with [[CPU virtualization (Intel VT-x / AMD-V)]({{< relref "2025-12-19-212625-hardware_virtualization.md#cpu-virtualization--intel-vt-x-amd-v" >}}) enabled in the BIOS/UEFI.
-   Sufficient Resources:
    -   CPU: Minimum 8 vCPUs recommended for SNO.
    -   RAM: Minimum 32GB RAM recommended for SNO.
    -   Storage: Minimum 120GB-150GB fast storage (SSD/NVMe) for the OKD VM.
-   Administrative Machine (Client): A Linux machine (e.g., Ubuntu, Fedora) to run openshift-install, oc, podman, and other client tools. This will be referred to as your “Admin Client Machine”.
-   Red Hat Pull Secret: Obtain a pull secret from [Red Hat OpenShift Cluster Manager](https://console.redhat.com/openshift/install/pull-secret) (a free Red Hat developer account is sufficient). This is needed for some certified operators and images.
-   Admin access to the local [PowerDNS]({{< relref "2024-05-25-025431-powerdns.md" >}}) with reverse DNS
-   An SSH public key


## Phase 1: Preparation on Admin Client Machine {#phase-1-preparation-on-admin-client-machine}

All commands in this phase are executed on your Admin Client Machine VM, not [Linux Containers (LXC)]({{< relref "2023-12-03-143424-lxc.md" >}}) (Disk: 20G; RAM: 8G(8192M); CPU: 2 cores).

-   OS: Ubuntu 24.04
-   IP: 192.168.1.50/24
-   Gateway: 192.168.1.5
-   PowerDNS:
    -   DNS Domain: okd.admin.testbed.com
    -   DNS Servers: 192.168.1.23


### Set Environment Variables {#set-environment-variables}

Define the OKD version and architecture for consistency.

```bash
export OKD_VERSION=4.20.0-okd-scos.13 # Check for the latest stable SCOS release
export ARCH=x86_64
```


## DNS {#dns}


### Forward DNS definitions {#forward-dns-definitions}

```file
; OKD
haproxy                 IN      A       192.168.88.8
helper                  IN      A       192.168.88.8
helper.okd              IN      A       192.168.88.8
api.okd                 IN      A       192.168.88.8
api-int.okd             IN      A       192.168.88.8
*.apps.okd              IN      A       192.168.88.8
bootstrap.okd           IN      A       192.168.88.12
master0.okd             IN      A       192.168.88.9
master1.okd             IN      A       192.168.88.10
master2.okd             IN      A       192.168.88.11
```


### Reverse DNS {#reverse-dns}

```file
; okd
8   IN      PTR     haproxy.yanboyang.com.
8   IN      PTR     helper.yanboyang.com.
8   IN      PTR     helper.okd.yanboyang.com.
8   IN      PTR     api.okd.yanboyang.com.
8   IN      PTR     api-int.okd.yanboyang.com.
12   IN      PTR     bootstrap.okd.yanboyang.com.
9   IN      PTR     master0.okd.yanboyang.com.
10   IN      PTR     master1.okd.yanboyang.com.
11   IN      PTR     master2.okd.yanboyang.com.
```


## Reference List {#reference-list}

1.  <https://github.com/gardart/okd-proxmox-scripts>
2.  <https://github.com/pvelati/okd-proxmox-scripts>
3.  <https://github.com/pvelati/ansible-okd-proxmox>
4.  <https://www.pivert.org/deploy-openshift-okd-on-proxmox-ve-or-bare-metal-tutorial/>
5.  <https://andrearaponi.it/devops/deploy-okd-on-proxmox/>
6.  <https://docs.okd.io/latest/installing/installing_platform_agnostic/installing-platform-agnostic.html>
