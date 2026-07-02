---
title: "monolithic kernel"
draft: false
---

## Introduction {#introduction}

Monolithic Kernel is another classification of [Kernel]({{< relref "20230620162822-kernel.md" >}}). Like [microkernel]({{< relref "20230730023554-microkernel.md" >}}), this one also manages system resources between application and hardware, but user services and kernel services are implemented under the same address space. It increases the size of the kernel, thus increasing the size of the operating system as well.  This kernel provides CPU scheduling, memory management, file management, and other operating system functions through system calls. As both services are implemented under the same address space, this makes operating system execution faster.

Below is the diagrammatic representation of the Monolithic Kernel:

{{< figure src="https://media.geeksforgeeks.org/wp-content/uploads/20230111170517/monolithic_kernel.jpeg" >}}

If any service fails the entire system crashes, and it is one of the drawbacks of this kernel. The entire operating system needs modification if the user adds a new service.


## Advantages of Monolithic Kernel {#advantages-of-monolithic-kernel}

-   One of the major advantages of having a monolithic kernel is that it provides CPU scheduling, memory management, file management, and other operating system functions through system calls.
-   The other one is that it is a single large process running entirely in a single address space.
-   It is a single static binary file. Examples of some Monolithic Kernel-based OSs are Unix, Linux, Open VMS, XTS-400, z/TPF, [Meta Scientific Linux]({{< relref "20230105201643-meta_scientific_linux.md" >}}), [Arch Linux]({{< relref "20230220222636-arch_linux.md" >}}).


## Disadvantages of Monolithic Kernel {#disadvantages-of-monolithic-kernel}

-   One of the major disadvantages of a monolithic kernel is that if anyone service fails it leads to an entire system failure.
-   If the user has to add any new service. The user needs to modify the entire operating system.


## Reference List {#reference-list}

1.  <https://www.geeksforgeeks.org/monolithic-kernel-and-key-differences-from-microkernel/>
