---
title: "Kubernetes Metrics Server"
draft: false
---

[Kubernetes]({{< relref "20230105185343-kubernetes.md" >}})


## Set-up {#set-up}

[kubeadm]({{< relref "20230420062104-kubeadm.md" >}}) doesn’t install [metrics server](https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/) component during its initialization. We have to install it separately.

To verify this, if you run the top command, you will see the Metrics API not available error.

```console
root@master-node:~# kubectl top nodes
error: Metrics API not available
```

To install the metrics server, execute the following metric server manifest file. It deploys metrics server version v0.6.2

```bash
kubectl apply -f https://raw.githubusercontent.com/yanboyang713/kubeadm-scripts/main/manifests/metrics-server.yaml
```

This manifest is taken from the official [metrics server](https://github.com/kubernetes-sigs/metrics-server) repo. I have added the --kubelet-insecure-tls flag to the container to make it work in the local setup and hosted it separately. Or else, you will get the following error.

```bash
because it doesn't contain any IP SANs" node=""
```

Once the metrics server objects are deployed, it takes a minute for you to see the node and pod metrics using the top command.

```bash
kubectl top nodes
```

You should be able to view the node metrics as shown below.

```console
root@master-node:/home/vagrant# kubectl top nodes
NAME            CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
master-node     111m         5%     1695Mi          44%
worker-node01   28m          2%     1078Mi          57%
worker-node02   219m         21%    980Mi           52%
```

You can also view the pod CPU and memory metrics using the following command.

```bash
kubectl top pod -n kube-system
```
