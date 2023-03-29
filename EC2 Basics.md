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

## EC2 Instance Types

* Includes multiple aspects:
    - Raw CPU, RAM, Local storage capacity and type
    - Resources ratio (for ex. CPU optimized / RAM optimized: best value spent for a particular workload)
    - System architecture / vendor (AMD / Intel)
    - Additional features and capabilities
* EC2 Categories:
    - General Purpose - DEFAULT - diverse workload, equal resources ratio
    - Compute Optimized - Media processing, scientific calculus, Gaming, etc.
    - Memory Optimized - For processing large memory datased and in-memory DB.
    - Accelerated Computing - GPU, FPGA, etc.
    - Storage Optimized - Fast and efficient local storage, random / sequential IO. Ex. elasticsearch, analytics workload, scaling out DB, etc.
* EC2 Types: Example: R5dn.8xlarge
    - R is the instance family.
    - 5 is the generation.
    - After the . (8xlarge) is the instance size.
    - Between the generation and the dot, could be anything that denotes additional capacities (or nothing)

## Storage Refresher

* Direct (local) attached storage: storage directly on the EC2 host. Fast but not resilient.
* Network attached storage: volumes delivered over the network (EBS). Slower but more resilient. Using protocols like FC or ISCSI.
* Ephemeral storage: temporary storage. Not persistent. In EC2, it's the direct attached storage or instance store.
* Persistent storage: lives on past the lifetime of the instance. EBS.
* Block storage: volume presented to the OS as a collection of blocks. Mountable, bootable, no structure. Network ones are EBS, SAN, NAS.
* File storage: presented as a file share. Mountable, not bootable. Can use SMB as network protocol.
* Object storage: flat structure of kv pair, with v being any objects. Not mountable, not bootable. Ex S3.
* Storage performance:
    - IO (block) size: size of a block used by the storage / process. ("size of the wheels") Ex. 4kB
    - IOPS: number of IO operations per second ("revolution per min"). Ex. 100 IOPS. Less informative, usually only shows the potential performance.
    - Throughput: data operated per second ("speed") Ex. 400KB/s. Usually provides concrete information about data bandwidth.
    - "Generally": IO * IOPS = Throughput (If operations act on sequential blocks. If random, it's more difficult)
* Performance is not only limited by the capability of the disk. Everything from the application to the disk, including OS, network, etc. play a role.

## EBS Service Architecture

* Block storage, can be encrypted using KMS. Instances see block device.
* STORAGE IS PROVISIONED IN ONE AZ - AZ-RESILIENT
* Attached to one (or multiple, if application cluster available) to multiple EC2 instances over storage network in the same AZ.
* Can be detached / attached. Independent of the lifecycle of the instance. Persistent.
* Can take snapshot, uploaded to S3. Can then be moved to another AZ via S3 snapshot or even different region.
* Different storage type and perf. profiles. Bill based on GB/month and sometimes performance.

## EBS Volume Type - General Purpose

* GP2 is the actual "default". General purpose SSD ranging from 1gb to 16TB. 
* IO-credit is 16kb. 1 IOPS is 1 IO/s. One volume has a 5.4 million IO-credit. Fills at the rate of baseline performance.
* Baseline performance is 3 IOcredit/s per GB of volume size.
* For volumes <= 1000GB, burst up to 3000IOPS.
* For volume superior to this value, IO credits are not used and you always achieve baseline, up to 16000 IO credits/s.
* GP2 is good for boot volumes, low latency apps and dev/test.
* GP3 is expected to become the new default soon. Its system is easier than Gp2.
* It's always 3000IOPS max or 125MiB/s (standard).
* You can add extra costs to go further than that, up to 16kIOPS or 1000MiB/s.
* Gp3 is usually cheaper than gp2 and max througput is 4x faster than gp2 (1000MiB/s vs. 250MiB/s)
* Good for virtual desktops, medium-sized single db instance (mysql/oracle), low-lat apps, boot volumes, dev/test.