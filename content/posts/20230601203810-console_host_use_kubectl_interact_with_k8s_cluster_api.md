---
title: "console host use kubectl interact with k8s cluster API"
draft: false
---

[Arch Linux]({{< relref "20230220222636-arch_linux.md" >}})

```bash
paru -S kubectl
```

Copy $HOME/.kube/config file to console host

```bash
mkdir -p $HOME/.kube/
vim config
# write your config file context from contrl plate DIR $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

You can find server version

```console
(base) [yanboyang713@archlinux .kube]$ kubectl version
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.1", GitCommit:"4c9411232e10168d7b050c49a1b59f6df9d7ea4b", GitTreeState:"archive", BuildDate:"2023-04-15T11:34:19Z", GoVersion:"go1.20.3", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v5.0.1
Server Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.5", GitCommit:"890a139214b4de1f01543d15003b5bda71aae9c7", GitTreeState:"clean", BuildDate:"2023-05-17T14:08:49Z", GoVersion:"go1.19.9", Compiler:"gc", Platform:"linux/amd64"}
```

Test

```console
(base) [yanboyang713@archlinux .kube]$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.88.11:6443
CoreDNS is running at https://192.168.88.11:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

next I recommend you install [Kubernetes Dashboard]({{< relref "20230516092022-kubernetes_dashboard_tools.md" >}})
