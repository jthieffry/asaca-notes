# Key Notes

## Virtualization 101

* Applications runs in userspace mode. If they want to access HW, they need to go through the Kernel via syscalls. Only the kernel runs in privileged mode.
* Different type of virtualization existed, from the least to the most performant ones:
    - "Emulated Virtualization": HW is not aware of virt. Virt is done at the software level only in the Hypervisor, which performs binary translation on the fly from requests coming from guests OS. Extremely slow, low adoption. Guest OS is NOT modified. QEMU/KVM.
    - Para-virtualization. Guest OS is modified and made aware somehow that it is virtual. Guest OS do hypercalls to hypervisor, which improves performance. QEMU/KVM with VirtIO
    - Hardware Assisted Virtualization. The HW (CPU) itself is aware of virt and directly traps guest OS syscalls as CPU interrupts and redirects them to the Hypervisor for processing. Great performances but still strong bottle for IO-heavy operations such as disk or network, since those are not virtualized. Intel VT-d, IOMMU.
    - SR-IOV: The HW devices themselves are virtualized, like NIC being themselves multiple NIC (SR-IOV). Greatly improves IO performances. In EC2, this is called "Enhanced Networking".