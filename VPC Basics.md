# Key Notes

## VPC Sizing and Structure

* VPC design is one of the most critical part when designing a new architecture.
* What size should the VPC be ?
* Are there any networks we can't use ? -> AVOID OVERLAPS WITH EXISTING NETWORKS (on-prems, VPC's, Vendors, etc.)
* Try to predict the future & think about VPC structure (tiers/dev/test/prod, resiliency and AZ)
* VPC Minimum size is /28 (16IP), maximum is /16 (65536 IP).
* Avoid common range in order to avoid future issues. Reserve 2+ networks per region being used, per account.
* Need to ask how many subnets you will need and how many IPs in total with how many per subnets.
* Need to consider first how many AZ you will use in your VPC. 3 is a good default (present almost everywhere) and add +1 for buffer. So 4 AZ.
* Then need to consider how many network "tiers". Let's consider a WEB, DB, APP tiers +1 buffer, so 4 tiers.
* This leads to 4*4 = 16 subnets to use. As a result, if we choose a /16 VPC, using /20 as subnet mask gives us 16 subnets.
* If we consider that we have 4 regions and 10 accounts, we can use VPCs: 10.16.0.0, 10.17.0.0, ... 10.55.0.0 IF we don't have any overlaps with pre-existing networks. We can also reserve multiple VPC per region and per account, for ex 4 VPC per region per account, which would then gives for region A account A: 10.16.0.0, 10.17.0.0, 10.18.0.0, 10.19.0.0. Region A account B will then start for VPC1 from 10.20.0.0.

## Custom VPCs

* VPC is a regional service, all AZs in the region.
* They are isolated network. Nothing goes in or out without explicit configuration.
* Allows hybrid networking with other clouds & on-premises.
* Need to choose the default or dedicated tenancy. With default, you can adjust whether or not resources will run on dedicated hardware or not. With dedicated, you cannot choose, and it's locked in to dedicated hardware. Unless you're sure you want it, you should keep default.
* VPC has ipv4 private cidr blocks and can have a pool of public ips.
* VPC has 1 primary private ipv4 CIDR block (min /28, max /16). It can also have optional secondary ipv4 blocks and an optional single assigned /56 ipv6 CIDR block.
* DNS is provided by R53 in the VPC and is controlled by options "enableDnsHostnames" to give instance DNS names and "enableDNSSUpport" to enable DNS resoltion in VPC. If this last one is choosen, the address of the DNS is the VPC base network + 2 (for ex, with a VPC address of 10.16.0.0, the DNS server will be 10.16.0.2).

## VPC Subnets

* Subnets are AZ resilient. 1 subnet can only be part of one and only one AZ. But an AZ can have multiple subnets or none.
* A subnet cannot overlap with other subnets.
* Subnets can communicate with other subnets in the VPC by default.
* Within each subnets, 5 IPs are reserved: the network and broacast ones, the network + 1 (ex. 10.16.16.1) for the VPC router, the network + 2 (ex 10.16.16.2) for the DNS (reserved on all subnets in a VPC) and network + 3 (ex 10.16.16.3) reserved for future use.
* A VPC has a configuration option applied to it called DHCP options set at creation time, and this setting flows through all subnets in the VPC. On a subnet, you can define auto assign public ivp4 and auto assign ipv6 if you had setup the ipv6 allocation.

## VPC Routing, IGW & Bastion Hosts

* Every VPC has a VPC router, highly available, it just works. It has an interface in each subnet of the VPC (the network + 1 address in the subnet).
* It routes traffic between subnets.
* Each subnet has a route table. A VPC has a main route table which is applied by default to all subnets.
* A route table associated to a subnet controls where the traffic going out of this subnet is redirected. There is always a default route that has the target "local" which is an alias for the VPC CIDR. This is what allows all subnets within a VPC to communicate with each others.
* IGW is a region resilient gateway attached to a VPC. You attach it to a VPC and it's available for ALL AZ within the region of the VPC.
* There is 1-1 mapping between IGW and VPC. A VPC has 1 or 0 IGW, and an IGW can only be attached to one VPC at a time (or none).
* IGW are between the VPC and the AWS public zone. It's the GW traffic between the VPC and the Internet or AWS Public Zone (S3, SQS, etc.) It's fully managed. AWS handles the IGW performances.
* To have a EC2/service communicates to the outside, it works like this:
    1. Create IGW
    2. Attach IGW to VPC
    3. Create custom route table that directs 0.0.0.0/0 (default route) to IGW
    4. Associate IGW with subnet
    5. Allocate public IPV4 to subnet
* A public IPv4 is NOT visible from within the instance. What happens when we associate a public IP to an instance is that we create a record in the IGW to map the private IPv4 to the public IPv4. The IGW acts as a SNAT/DNAT. The instance never sees or is aware that it actually has a IPv4 attached to it !