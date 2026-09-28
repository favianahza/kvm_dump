# KVM (Kernel-based Virtual Machine)

KVM (Kernel-based Virtual Machine) is a virtualization solution that transforms the Linux kernel into a hypervisor. It was first introduced in the Linux kernel version 2.6.20.

KVM converts the Linux Kernel into a **Type-1 Hypervisor**. However, because the Linux distribution itself remains a general-purpose operating system with applications that compete for resources with virtual machines, KVM can also be categorized as a **Type-2 Hypervisor**.

***

## Key Terminology

Here are some common terms associated with KVM:

* **Host**: This is the server that runs the KVM hypervisor. It is the physical machine that provides the hardware resources such as CPU, memory, and storage for the virtual machines.
* **Virtual Machine / Guest / VM**: This refers to the guest virtual machine that runs on top of the KVM Host. Each VM operates as an independent server with its own operating system and applications.
* **virt-manager**: This is a graphical tool (VM Manager) used for deploying and managing Virtual Machines. It provides a user-friendly interface for tasks like creating, starting, stopping, and monitoring VMs.
* **virt-install**: This is a command-line tool used to install guests from the command-line interface. It is a powerful utility for automating the provisioning of new virtual machines through scripts.