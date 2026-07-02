---
title: "Multi-instance GPU (MIG)"
draft: false
---

## Multi-instance GPU (MIG) {#multi-instance-gpu--mig}

The mechanisms discussed so far rely either on changes to the application using the CUDA programming model APIs, such as CUDA streams, or CUDA system software, such as time-slicing or MPS.

With MIG, GPUs based on the NVIDIA Ampere Architecture, such as NVIDIA A100, can be securely partitioned up to seven separate GPU Instances for CUDA applications, providing multiple applications with dedicated GPU resources. These include streaming multiprocessors (SMs) and GPU engines, such as copy engines or decoders, to provide a defined QoS with fault isolation for different clients such as processes, containers or virtual machines (VMs).

When the GPU is partitioned, you can use the prior mechanisms of CUDA streams, CUDA MPS, and time-slicing within a single MIG instance.

For more information, see the MIG user guide and MIG Support in Kubernetes.


## Reference List {#reference-list}

1.  <https://developer.nvidia.com/blog/improving-gpu-utilization-in-kubernetes/>
