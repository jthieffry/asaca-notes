# Key Notes

## Virtualization 101

* Applications runs in userspace mode. If they want to access HW, they need to go through the Kernel via syscalls. Only the kernel runs in privileged mode.
* Different type of virtualization existed, from the least to the most performant ones:
    - "Emulated Virtualization": HW is not aware of virt. Virt is done at the software level only in the Hypervisor, which performs binary translation on the fly from requests coming from guests OS. Extremely slow, low adoption. Guest OS is NOT modified. QEMU/KVM.
    - Para-virtualization. Guest OS is modified and made aware somehow that it is virtual. Guest OS do hypercalls to hypervisor, which improves performance. QEMU/KVM with VirtIO
    - Hardware Assisted Virtualization. The HW (CPU) itself is aware of virt and directly traps guest OS syscalls as CPU interrupts and redirects them to the Hypervisor for processing. Great performances but still strong bottle for IO-heavy operations such as disk or network, since those are not virtualized. Intel VT-d, IOMMU.
    - SR-IOV: The HW devices themselves are virtualized, like NIC being themselves multiple NIC (SR-IOV). Greatly improves IO performances. In EC2, this is called "Enhanced Networking".

## EC2 Architecture and Resilience

* EC2 instaces are VM (OS+Resources). They run on EC2 hosts which can be shared (among different tenants) or dedicated (we pay for the host itself, not the VM).
* The host is in 1 AZ. If the AZ fails, all VMs in this AZ fail.
* ONe instance is comprised of:
    - Virtual hardware (CPU, Mem.)
    - Ephemeral storage (depending on the app, storage may stay but if the VM changes host, the storage disappear)
    - Data network
    - Remote persistent storage (EBS)
* Network (via subnets), storage (EBS), hardware... Everything is AZ RESILIENT. One VM in a AZ CANNOT ACCESS network or EBS from another AZ.
* EC2 is good for:
    - Traditional OS + App compute, long-running compute or monolithic applications.
    - Server-style application, with either burst or steady-state load.
    - Migrated application workloads or disaster recovery`