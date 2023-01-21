# Key Notes

## Public vs Private Services

* Private service in this context is from a network perspective: isolated from the public internet zone.
* There is actually three zones in the context of AWS:
    - The Public internet zone ("the Internet")
    - The AWS Private Zone (VPC, isolated from each others by default)
    - Th AWS Public Zone (sits in the middle of Internet and VPC)
* The AWS Public zone is where public AWS services (endpoints) reside: S3, IAM, etc.
* Services in VPC can access "the Internet" via IGW, but they also access the AWS Public Zone by IGW.

## AWS Global Infrastructure

* At a global level, AWS is divided into Regions and Edge Locations.
* There is way more EL than regions. Regions have the full scope of AWS services (compute, DB, etc.), EL have a limited sets, but are mostly used for content delivery so they can be closer to customers.
* AWS Regions offers Geographic separation (isolated fault domain), Geopolitical separation (different governance: ex. EU/US regulations), and location control (performance).
* Within regions, we have the concept of AZ, which are separate failure domains, that can physically correspond to 1 or multiple DC. If one AZ fails, the other shouldn't be impacted unless it's a region-wide failure. AZ are connected together with high-speed connectivity.
* Globally resilient: only a few services (IAM/S3): it would take the world to fail for this service to be unavailable.
* Region resilient: this service would fail if the region itself fails. If only one or a few AZ fail, it's safe.
* AZ resilient: a global failure in one AZ would cause the service to fail.

## Default VPC

* A VPC is within 1 account & 1 region (region resilient), it's private & isolated by default, and comes in two types: Default and Custom VPC.
* There is only 1 default VPC per region (can be removed and re-created). It's default CIDR is always 172.31.0.0/16.
* This default VPC as /20 subnets in each AZ of the regions, and comes with IGW, Security Group and NACL.
* Those "default subnets" assign public ipv4 addresses (they "project" themselves in AWS Public ZOne).

## EC2

* IAAS, provides instances. Private service by default, uses VPC networking. AZ resilient.
* Local on-host storage or EBS.
* When instance is running, we pay for compute, memory, disk and network. When it's stopped, we will pay for disk. Only when it's terminated, we don't pay.
* AMI ~= instance image. It includes:
    - Permissions (public: everyone can use, owner: only owner can use, explicit: only specific accounts allowed)
    - Root volume (with the OS, etc.)
    - Block device mapping