---
title: "CUDA Multi-Process Service"
draft: false
---

## CUDA Multi-Process Service {#cuda-multi-process-service}

You can take the oversubscription strategy described earlier a step further with CUDA MPS. MPS enables CUDA kernels from different processes, typically MPI ranks, to be processed concurrently on the GPU when each process is too small to saturate the GPU’s compute resources. Unlike time-slicing, MPS enables the CUDA kernels from different processes to execute in parallel on the GPU.

Newer releases of CUDA (since CUDA 11.4+) have added more fine-grained resource provisioning in terms of being able to specify limits on the amount of memory allocatable (CUDA_MPS_PINNED_DEVICE_MEM_LIMIT) and the available compute to be used by MPS clients (CUDA_MPS_ACTIVE_THREAD_PERCENTAGE). For more information about the usage of these tuning knobs, see the Volta MPS Execution Resource Provisioning.

The tradeoffs with MPS are the limitations with error isolation, memory protection, and quality of service (QoS). The GPU hardware resources are still shared among all MPS clients. You can CUDA MPS with Kubernetes today but NVIDIA plans to improve support for MPS over the coming months.


## Reference List {#reference-list}

1.  <https://developer.nvidia.com/blog/improving-gpu-utilization-in-kubernetes/>
