---
title: "Kubernetes Architecture"
draft: false
---

A general architecture overview of a Kubernetes cluster is shown in Fig. 1, with one control plane host and two worker hosts. The control plane host is in charge of maintaining the cluster state, and the worker host is responsible for running cloud workloads in the execution unit named Pod, while the CNI Plugin facilitates communication among these execution units [3].
![](https://res.cloudinary.com/dkvj6mo4c/image/upload/v1686331425/k8s/Kubernetes-Architecture_yox1xd.png)


## Reference List {#reference-list}

1.  <https://devopscube.com/kubernetes-architecture-explained/>
2.  <https://kubernetes.io/docs/concepts/overview/components/>
3.  Qi, S., Kulkarni, S. G., &amp; Ramakrishnan, K. K. (2020). Assessing container network interface plugins: Functionality, performance, and scalability. IEEE Transactions on Network and Service Management, 18(1), 656-671.
