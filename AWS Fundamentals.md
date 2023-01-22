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

## S3

* Global storage platform - regional based/resilient (buckets).
* Public service (resides in AWS Public Zone), unlimited data storage & multi-users.
* Bucket names are globally unique, 3-63 characters, all lower-case, no underscores.
* Starts with lowercase letter or number, can't be ip formatted (like 1.1.1.1).
* Buckets have a 100 soft limit and 1000 hard limit per account.
* Unlimited objects in bucket. Object size ranges from 0 bytes to 5TB.
* Object key is its name, object value is its data. No "Folder" inside buckets, it has a flat structure. However, object keys can have prefixes, like /my/object.
* S3 is object store, not file or block. You can't mount an S3 bucket.

## CloudFormation

* Enables IaC. Templates are made either in YAML or Json.
* If you have a "Description" section in the template, and a "AwsTemplateFormationVersion" section as well, the description part has to be just betlow the Version one.
* CloudFormation templates have to have AT LEAST a "Resources" section, which are the items actually being deployed.
* Other items include:
    - Metadata: control things like what is being displayed in CFN UI when deploying.
    - Paremeters: optional inputs to request to the user during deployment.
    - Mappings: map parameters to other objects in AWS.
    - Conditions: allows conditional resource deployments.
    - Outputs: control what is being displayed after the deployment.
* CloudFormation controls stacks. In each template, the resource section has a logical resource (Ex. Instance) that corresponds to a Type (ex. "AWS::EC2::Instance"). For each logical resources, CFN will actually create a physical resource.