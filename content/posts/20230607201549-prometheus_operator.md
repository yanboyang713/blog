---
title: "Prometheus Operator"
draft: false
---

## Overview {#overview}

The Prometheus Operator provides [Kubernetes]({{< relref "20230105185343-kubernetes.md" >}}) native deployment and management of [Prometheus]({{< relref "20230602142910-prometheus.md" >}}) and related monitoring components. The purpose of this project is to simplify and automate the configuration of a Prometheus based monitoring stack for Kubernetes clusters.

The [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator.git) facilitates the deployment and management of Prometheus and related monitoring components within Kubernetes using the core Prometheus operator pod. Its primary goal is to streamline and automate the setup of a Prometheus-based monitoring stack for Kubernetes clusters based on the Prometheus operator version.

The Prometheus operator includes, but is not limited to, the following features:

-   Kubernetes Custom Resources: Use Kubernetes custom resources to deploy and manage Prometheus, Alertmanager, and related components.
-   Simplified Deployment Configuration: Configure the fundamentals of Prometheus like versions, persistence, retention policies, and replicas from a native Kubernetes resource.
-   Prometheus Target Configuration: Automatically generate monitoring target configurations based on familiar Kubernetes label queries; no need to learn a Prometheus specific configuration language.

For an introduction to the Prometheus Operator, see the [getting started guide](https://github.com/prometheus-operator/prometheus-operator/blob/main/Documentation/user-guides/getting-started.md).

When one installs Prometheus operator, some of the features of the Prometheus operator include:

-   The ability to use Kubernetes custom resources to deploy and manage Prometheus, Alertmanager, and related components.
-   Simplified configuration for basic Prometheus functionalities such as versions, persistence, retention policies, and replicas using native Kubernetes resources.
-   Automatic generation of monitoring target configurations based on Kubernetes label queries, eliminating the need to learn a Prometheus-specific configuration language.


## Reference List {#reference-list}

1.  <https://github.com/prometheus-operator/prometheus-operator>
2.  <https://mp.weixin.qq.com/s?__biz=MzkwODIxNDI1Mg==&mid=2247484256&idx=1&sn=9ea7d2e0b34a6965a0624e97c58bd415&chksm=c0cc29e3f7bba0f562b4f1b15c208a6105a530c18f7d638188662afdc79ad7381c2e6bc70b27&mpshare=1&scene=1&srcid=0104ZdulFjRKsG96qHYtGPXG&sharer_shareinfo=eaf5acb25358541d501f3fa847c9e12e&sharer_shareinfo_first=eaf5acb25358541d501f3fa847c9e12e&exportkey=n_ChQIAhIQLUmoDwkMC1Ul%2BuNwLymtMxKWAgIE97dBBAEAAAAAALxPEKcLQh4AAAAOpnltbLcz9gKNyK89dVj0pw4CXKj5cgV%2F6J1uYqaWxtP8CJWZJlUYY2h9CwrBoOnLaI99yhhPE99s%2FMsf0%2FbnG8KQmhxTJ7TOvz3Dy2dPG%2BuDc1RfErOIAkkqnp6DKNgWDljZBH809yO6%2BBoljfchZVcPTppKRuwjM22NgJ8iMzPH0Y4NjGLTl%2FhbjIGEscRx3jokiyIDS69162BcgjAk%2Be5WfdT69DYX6q3mSJtAVai89XymJa1DSDwgqG%2BRd7tlTNGiDGnSM0BSSRyZkzvvH2v5RVKojJZqU%2FUbvzblHt3giHsSnmtncVLqGPP7NE49qSAw6PfkkRt9iIfCMNDY&acctmode=0&pass_ticket=KlX9YRhWoRoRgsKCmFs2f%2BzM7upft%2BvaRmy%2F4oiMRgB%2BvKk%2BeXS%2F21JTkm9pdgBPuiZ2NuxmLu8CLXHwbcXthQ%3D%3D&wx_header=0#rd>
