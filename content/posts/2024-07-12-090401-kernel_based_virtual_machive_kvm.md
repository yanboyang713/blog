---
title: "Kernel Based Virtual Machive (KVM)"
draft: false
---

The Kernel Based Virtual Machive (KVM) is an open-source hypervisor which converts Linux kernel into a Type1 hypervisor. We typically have two main parts in the KVM hypervisor: KVM kernel module and KVM user-space tool. The KVM kernel module is a loadable Linux kernel module which provides CPU, memory and interrupt virtualization whereas the KVM user-space tool is an application which helps users create and manage Guest instances (or [Virtual Machine (vm)]({{< relref "20230104161110-virtual_machine.md" >}}))

[KVM RISC-V]({{< relref "2024-07-12-090535-kvm_risc_v.md" >}})


## Reference List {#reference-list}

1.  <https://github.com/kvm-riscv/howto/wiki>
