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

## EBS Volume Type - Provisioned IOPS

* io1, io2(next-gen) and block express (experimental). Consistent low latency and jitter.
* IOPS can be adjusted independently of size, with a max of 50IOPS/GB (io1), 500IOPS/GB(io2) and 1000IOPS/GB(be)
* Instances performance are also in scope when it comes to how much provioned iops volume we can put. They have a maximum limit.
* Use these volume type for high perf and latency sensitive workloads, io-intensive nosql and relational db.

## EBS Volume Type - HDD

* Only good for sequential reads, two types: st1 (throughput optimized) and sc1 (cold hdd)
* st1 is super cheap and sc1 even cheaper. Range from 125G to 16T
* st1 500IOPS max but bs 1MB so up to 500MB/s in sequential access.
* sc1 250IOPS at 1MB bs so 250MB/s.
* Usage is similar than with ssd type, but it's not io credit but mb credit with baseline and burst.
* Use st1 for big data, warehouse and log processing. Use sc1 for colder data requiring fewer scans per day.

## EC2 Instance Store Volumes

* Local on EC2 Host
* Can only be added AT LAUNCH. Not after.
* LOST when instance moves, resizes or at hardware failure.
* High performance
* ALready paid for with the instance.
* TEMPORARY

## Choosing between Instance Store Volumes and EBS (need to remember the figures)

* If need PERSISTENCE, RESILIENCE, or storage isolated from the INSTANCE LIFECYCLE: EBS
* However, if you can have resilience at the software/app level (clustering software): it depends.
* High performance needs: it depends.
* SUPER HIGH performance needs: instance store.
* Cost: SUPER CHEAP: instance store (included in instance price).
* Cost: CHEAP: st1 or sc1. If for throughput or streaming: st1. Otherwise sc1.
* Boot: NOT st1 or sc1.
* gp2/3: up to 16k iops.
* io1/2: up to 64 iops or 256k with block express.
* However, if using raid0+EBS: max 260k IOPS (the maximum the biggest instance can support in ec2 for ebs)
* If need more than 260k iops: instance store.

## EBS Snapshots

* Snapshots are incremental volume copies to S3
* The first is a full copy of 'data' of the volume (if you have a 10G volume but only 5G of data, only those 5G will be copied and charged for ss)
* Future snaps are incremental (if you modify 2G of the previous 5G, only the 2G will be transffered and billed over)
* You can delete ss in between two other ss without fear that subsequent ss will be corrupted
* Volumes can be restored (created) from ss
* ss can be copied to other regions
* Performances considerations:
    - snaps restore lazily to volumes: fetched gradually
    - requested blocks are fetched immediately, but that will incur initial performance penalty on a running volume
    - can circumvent that by doing some dd to read the entirety of a volume (forcing fetching) before setting the instance to prod
    - or you can use the aws tools that does it for you: FSR(fast snapshot restore): it's immediate restore, but comes at a cost
    - can only use this up to 50 snaps / region. Set on snap and az. So a snap on 4 az counts for 4.