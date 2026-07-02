---
title: "NVIDIA Kubernetes Device Plug-in"
draft: false
---

## [Nvidia]({{< relref "20230405140352-nvidia.md" >}}) [Kubernetes]({{< relref "20230105185343-kubernetes.md" >}}) Device Plug-in Brings Temporal GPU [Concurrency]({{< relref "20230810074202-concurrent_programming.md" >}}) {#nvidia--20230405140352-nvidia-dot-md--kubernetes--20230105185343-kubernetes-dot-md--device-plug-in-brings-temporal-gpu-concurrency--20230810074202-concurrent-programming-dot-md}

provisioning the right-sized GPU acceleration for each workload is key to improving utilization and reducing the operational costs of deployment, whether on-premises or in the cloud.

To address the challenge of GPU utilization in Kubernetes (K8s) clusters, NVIDIA offers multiple GPU concurrency and sharing mechanisms to suit a broad range of use cases. The latest addition is the new GPU time-slicing APIs, now broadly available in Kubernetes with NVIDIA K8s Device Plugin 0.12.0 and the NVIDIA GPU Operator 1.11. Together, they enable multiple GPU-accelerated workloads to time-slice and run on a single NVIDIA GPU.


## When to share NVIDIA GPUs {#when-to-share-nvidia-gpus}

Here are some example workloads that can benefit from sharing GPU resources for better utilization:

-   Low-batch inference serving, which may only process one input sample on the GPU
-   High-performance computing (HPC) applications, such as simulating photon propagation, that balance computation between the CPU (to read and process inputs) and GPU (to perform computation). Some HPC applications may not achieve high throughput on the GPU portion due to bottlenecks on the CPU core performance.
-   Interactive development for ML model exploration using Jupyter notebooks
-   Spark-based data analytics applications, where some tasks, or the smallest units of work, are run concurrently and benefit from better GPU utilization
-   Visualization or offline rendering applications that may be bursty in nature
-   Continuous integration/continuous delivery ([CI/CD]({{< relref "20230812181208-ci_cd.md" >}})) pipelines that want to use any available GPUs for testing


## GPU concurrency mechanisms {#gpu-concurrency-mechanisms}

The NVIDIA GPU hardware, in conjunction with the CUDA programming model, provides a number of different concurrency mechanisms for improving GPU utilization. The mechanisms range from programming model APIs, where the applications need code changes to take advantage of concurrency, to system software and hardware partitioning including virtualization, which are transparent to applications (Figure 1).
![](https://developer-blogs.nvidia.com/wp-content/uploads/2022/06/GPU-Concurrency-Mechanisms-1024x482.png)

-   [CUDA streams]({{< relref "20230813103642-cuda_streams.md" >}})
-   [Time-slicing]({{< relref "20230813103842-nvidia_gpu_time_slicing.md" >}})
-   [CUDA Multi-Process Service]({{< relref "20230813104614-cuda_multi_process_service.md" >}})
-   [Multi-instance GPU (MIG)]({{< relref "20230813104759-multi_instance_gpu_mig.md" >}})

Table 1 summarizes these technologies including when to consider these concurrency mechanisms.

|                                         | Streams                                              | MPS                                                                        | Time-Slicing                                                                    | MIG                                                               | vGPU                                                                                    |
|-----------------------------------------|------------------------------------------------------|----------------------------------------------------------------------------|---------------------------------------------------------------------------------|-------------------------------------------------------------------|-----------------------------------------------------------------------------------------|
| Partition Type                          | Single process                                       | Logical                                                                    | Temporal (Single process)                                                       | Physical                                                          | Temporal &amp; Physical – VMs                                                           |
| Max Partitions                          | Unlimited                                            | 48                                                                         | Unlimited                                                                       | 7                                                                 | Variable                                                                                |
| SM Performance Isolation                | No                                                   | Yes (by percentage, not partitioning)                                      | Yes                                                                             | Yes                                                               | Yes                                                                                     |
| Memory Protection                       | No                                                   | Yes                                                                        | Yes                                                                             | Yes                                                               | Yes                                                                                     |
| Memory Bandwidth QoS                    | No                                                   | No                                                                         | No                                                                              | Yes                                                               | Yes                                                                                     |
| Error Isolation                         | No                                                   | No                                                                         | Yes                                                                             | Yes                                                               | Yes                                                                                     |
| Cross-Partition Interop                 | Always                                               | IPC                                                                        | Limited IPC                                                                     | Limited IPC                                                       | No                                                                                      |
| Reconfigure                             | Dynamic                                              | At process launch                                                          | N/A                                                                             | When idle                                                         | N/A                                                                                     |
| GPU Management (telemetry)              | N/A                                                  | Limited GPU metrics                                                        | N/A                                                                             | Yes – GPU metrics, support for containers                         | Yes – live migration and other industry virtualization tools                            |
| Target use cases (and when to use each) | Optimize for concurrency within a single application | Run multiple applications in parallel but can deal with limited resiliency | Run multiple applications that are not latency-sensitive or can tolerate jitter | Run multiple applications in parallel but need resiliency and QoS | Support multi-tenancy on the GPU through virtualization and need VM management benefits |

With this background, the rest of the post focuses on oversubscribing GPUs using the new time-slicing APIs in Kubernetes.


## Reference List {#reference-list}

1.  <https://www.infoq.com/news/2022/12/k8s-gpu-time-slicing/>
2.  <https://developer.nvidia.com/blog/improving-gpu-utilization-in-kubernetes/>
