---
title: "Deploying OKD Single Node on Proxmox"
draft: false
---

This guide documents the complete setup of [OKD]({{< relref "20230518171737-okd.md" >}}) (The Community Distribution of [Kubernetes]({{< relref "20230105185343-kubernetes.md" >}}) that powers Red Hat OpenShift) as a Single Node Operator (SNO) cluster on a virtual machine hosted by [Proxmox VE]({{< relref "20230228043925-proxmox_ve.md" >}}).

[OKD Bootstrap]({{< relref "2024-05-24-051502-okd_bootstrap.md" >}})


## Prerequisites {#prerequisites}

-   An SSH public key
-   Admin access to the local DNS with reverse DNS (examples for [bind9]({{< relref "2024-05-25-015638-bind_9.md#bind9-settings" >}}))
-   RedHat account to get the Pull Secret, plase the key in a pullSecret.txt file.


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
