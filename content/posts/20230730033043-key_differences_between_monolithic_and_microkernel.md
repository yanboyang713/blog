---
title: "key differences between monolithic and microkernel"
draft: false
---

## Here are some key differences between [monolithic kernel]({{< relref "20230730024759-monolithic_kernel.md" >}}) and [microkernel]({{< relref "20230730023554-microkernel.md" >}}) architectures {#here-are-some-key-differences-between-monolithic-kernel--20230730024759-monolithic-kernel-dot-md--and-microkernel--20230730023554-microkernel-dot-md--architectures}

-   System services: In a monolithic kernel, all system services run in kernel space, whereas in a microkernel, only the most basic services (such as memory management and process scheduling) run in kernel space, with other services running in user space.
-   Performance: Monolithic kernels are generally faster and more efficient than microkernels, because there is no overhead associated with moving data between kernel space and user space.
-   Modularity: Microkernels are more modular than monolithic kernels, because services are separated into different processes running in user space. This makes it easier to add or remove services without affecting other parts of the system.
-   Security: Microkernels are generally considered more secure than monolithic kernels, because a bug or vulnerability in a service running in user space is less likely to affect the entire system.
-   Development: Developing a monolithic kernel is generally simpler and faster than developing a microkernel, because all system services are integrated and share the same memory space.

In summary, monolithic kernels are characterized by their tight integration of system services and high performance, while microkernels are characterized by their modularity, simplicity, and security. The choice between a monolithic and microkernel architecture depends on the specific needs and requirements of the operating system being developed.


## Key differences between Monolithic Kernel and Microkernel are as follows {#key-differences-between-monolithic-kernel-and-microkernel-are-as-follows}

| Basics        | Micro Kernel                                                                       | Monolithic Kernel                                                                                            |
|---------------|------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------|
| Size          | Smaller in                                                                         | Larger as OS and user both lie in the same address space                                                     |
| Execution     | Slower                                                                             | Faster                                                                                                       |
| Extendible    | Easily extendible                                                                  | Complex to extend                                                                                            |
| Security      | If the service crashes then there is no effect on working on the microkernel.      | If the process/service crashes, the whole system crashes as both user and OS were in the same address space. |
| Code          | More code is required to write a microkernel.                                      | Less code is required to write a monolithic kernel.                                                          |
| Examples      | [L4Linux]({{< relref "20230730033922-l4linux.md" >}}), macOS                       | Windows, Linux BSD                                                                                           |
| Security      | More secure because only essential services run in kernel mode                     | Susceptible to security vulnerabilities due to the amount of code running in kernel mode                     |
| Platform      | independence and More portable because most drivers and services run in user space | Less portable due to direct hardware access                                                                  |
| Communication | Message passing between user-space servers                                         | Direct function calls within kernel                                                                          |
| Performance   | Lower due to message passing and more overhead                                     | High due to direct function calls and less overhead                                                          |


## Reference List {#reference-list}

1.  <https://www.geeksforgeeks.org/monolithic-kernel-and-key-differences-from-microkernel/>
