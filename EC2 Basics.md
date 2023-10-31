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
    - Migrated application workloads or disaster recovery

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
* For volumes inferior or equal to 1000GB, burst up to 3000IOPS.
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

## EBS Encryption

* Encryption is encryption at rest. It happens between the EC2 Host and the remote storage.
* Since it happens at the host level, it is completely transparent to the instance (OS) -> No performance loss.
* Accounts can be set to encrypt by default. You can use the default KMS key or your custom KMS key.
* KMS Key is used to generate a data encryption key (DEK).
* FOR EACH NEW VOLUME CREATED FROM SCRATCH, A NEW DEK IS GENERATED AND USED.
* Snapshots created from an encrypted EBS will be encrypted using the same DEK. Volumes created from encrypted snapshots will be encrypted.
* They also use the same DEK.
* Otherwise a new volume triggers a new DEK.
* Once a volume has been set as encrypted, you cannot un-encrypt it.
* Decrypted data resides in the EC2 host memory.
* No cost and no performance loss for using EBS encryption.

## EC2 Network and DNS Architecture

* Instance has one primary interface and zero or more secondary interfaces (ENI).
* Secondary interfaces must be chosen in the same AZ as the primary - instances are AZ-resilient.
* Secondary interfaces can be detached and attached to other instances.
* SecGroups are bound to ENI.
* Instance can have multiple private IP (one per ENI) but only one or 0 public IP.
* Instances can have has many elastic IP as they have private IP.
* EIP are static, public IP is dynamic.
* Since some software licences are bound by MAC address, you can attach the software license to a secondary NIC and move it to another instance.
* For different security groups, you can have multiple ENI.
* OS has no notion of public IP, it only sees its privates ones. Public IP is made at the NAT level during translation.
* Once again, public IP are dynamic. A server restart will trigger a new IP allocation.
* The instance public DNS is resolved locally within the VPC (i.e resolves to private IP) but is resolved to the public IP outside VPC.

## Amazon Machine Images (AMI)

* AMI can be used to launch EC2 instances. They can come from AWS or are Community provided, including paid (marketplace).
* They are REGIONAL, and has unique ID per region.
* AMIs have permissions. By default, it is your account only. But can be made public or available to a subset of accounts.
* AMI can be created from an EC2 instance we want to template.
* THe lifecycle of a custom AMI starts with the launch of an EC2 instance. Then we configure it, then we create the image (AMI). Then we launch the AMI.
* During the image creation process, EBS volumes attached to the instance are snapshotted and the device name (dev/xvda) is also saved.
* The AMI is then just a container that has: permissions info, and the block device mapping info with the appropriate EBS snapshot.
* The cost of custom AMI is thus the cost of the associated EBS snapshot.
* An AMI can't be edited. To edit an AMI, we need to create a new one based on the old one.
* AMI can be copied accross regions and as a reminder the default permissions for AMI is your account.

## EC2 Purchase Options (AKA 'Launch Types')

* On demand:
    - Isolated VM but runs on shared hardware with multiple customers.
    - Per-second billing when the instance is running.
    - Default option. No interruptions. No capacity reservation, no upfront costs, no discounts.
    - Good for: short-term workloads, unknown workloads, apps that can't be interrupted.
* Spot:
    - AWS selling unused capacity, up to 90% discount.
    - Spot price based on spare capacity at anytime.
    - Customers bid a 'maximum' price they're willing to pay for instance. If spot price <= bid, instance is scheduled.
    - If spot price > bid, instance gets terminated.
    - Good for: non-time critical workload that can be rerun, bursty capacity needs, stateless & cost-sensitive workloads.
    - Never use spot for workloads that can't tolerate interruptions !
* Reserved:
    - You can buy a reservation, locked to a AZ or a region. With AZ you also lock capacity but with region you're more flexible.
    - A matching instance corresponding to the reservation either has a reduced price/s or even no more price/s.
    - Unused reservation are still billed. If an instance scheduled is larger than the reservation, there is partial coverage (small redux on the price/s).
    - Reservation are either 1y or 3y. 3y offers the greatest discount but also the greatest risks.
    - You can either pay with no-upfront cost, then you pay price/s at a discounted rate.
    - You can pay all upfront: greatest discount and no price/s.
    - Or partial upfront: a mix of both above: smaller upfront but also small price/s.
* Dedicated Hosts:
    - You purchase the actual EC2 host. You manage the host itself, including its capacity.
    - No instance charge obviously, but VM are also linked to the host (cannot be rescheduled elsewhere).
    - The real benefit of that is that some license are bound to a physical machine socket/core usage, which makes this launch type necessary.
    - Drawback is that you manage the EC2 host, including its capacity.
* Dedicated Instance:
    - You don't manage an EC2 physical host anymore but you pay premium fee/s on instance with the guarantee that only your instance runs on the hardware.
    - It is good for some audited workload which require a dedicated physical host but you don't need to actually manage it.
On-demand, spot and reserved used the default/shared tenancy model. Dedicated hosts and instances are other tenancy models.

## Additional Info on Reserved Instances

- In addition to the 'normal' reserved instance, you also have the 'scheduled reserved instance' launch type.
    - It is reserved instance which don't need to run 24/365. Ideal for long-term usage which doesn't run constantly.
    - Schedule can be daily: ex. 5hrs a day starting at 12:00 for a batch processing job.
    - Can be weekly: ex. market analysis that runs every Friday for 24hrs.
    - Can be monthly: ex. 100hrs of EC2 / month for scientific batch job.
- Scheduled reserved instance are not supported in all regions and for all instance type.
- Scheduled reserved instances have a commitment of at least 1y and 1200hrs per year minimum.
- On another topic, capacity reservation:
    - With regional reservation: you have billing discount for valid instance in any AZ of the region, but you don't reserve capacity.
    - With zonal reservation: billing discount for valid instance in that AZ only. But you also reserve capacity (you have priority in terms of supply shortage).
    - You can also have on-demand capacity reservation: no discount but you reserve capacity in one AZ. No need to commit for years but you pay regardless of the usage of the capacity or not.
- EC2 Savings Plans:
    - Hourly commitment for 1 or 3 years. For ex. you commit to pay 20$/hr for 3 years.
    - SP can be a reservation of general compute resources (EC2 / Fargate / Lambda).
    - Or a reservation of EC2 resources only (but you have more flexibility for the OS and the size of resources).
    - All of these products (EC2 / lambda / fargate) have an on-demand and a SP rate price.
    - You pay the reduced SP rate price up until your commitment. Beyond that, the on-demand pricing is applied. 

## Instances Status Check and Recovery

- When launching/starting instances, EC2 has a set of two checks performed:
    - One at the system level (power on the EC2 physical host, network connectivity, software/hw issue, etc)
    - One at the instance level (corrupted file system, kernel issue, bad network settings, etc)
- You can create a status check alarm for an instance where a cloudwatch rule sends you an email if checks fail for a specific duration.
- The status check alarm can also:
    - Reboot the instance
    - Auto-recover the instance (move it to different host in obviously the same AZ)
    - Terminate the instance.
- In case of large scale AZ failure, auto recover won't do much, but except that it is still useful nonetheless.

## Horizontal and Vertical Scaling

- Vertical Scaling:
    - Go from a small instance size to a large instance size.
    - Each resize requires a reboot: disruption & larger instances have a price premium.
    - We can also only grow that big: instance size cap.
    - Good parts though is that no application modification is required, works everywhere, for all apps, even monoliths.
- Horizontal Scaling:
    - You need to ensure state is preserved accross hosts: either via direct application support or off-host sessions (ex: external DB, redis, etc) so hosts are stateless.
    - If you have that though, only benefits: no disruption during scaling, no limits, usually less expensive, and more granular in the scaling.

## Metadata Service

 - EC2 Service provides data to instance. Data is about environment, networking, authentication, user-data, etc.
 - It's accessible inside all instances, and it is not authenticated nor encrypted. It's an exposed service.
 - Available under http://169.254.169.254/latest/meta-data. Remember this URL, important to remember perfectly.