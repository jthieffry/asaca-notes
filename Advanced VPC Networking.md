# Key Notes

## VPC Flow Logs
* Captures metadata (NOT CONTENT)
* Attached to:
    - A VPC: captures all ENIs in that vpc
    - A subnet: captures all ENIs in that subnet
    - ENIs directly
* Flow logs are not RT
* Logs destination can either be:
    - S3
    - Cloudwatch Logs
    - Or athena for querying
* Flow logs can either capture:
    - Accepted connections
    - Rejected connections
    - All connections
* Flow logs doesnt capture:
    - Metadata svc
    - DHCP
    - Amazon DNS server
    - Amazon Windows License server

## Egress only IGW
* Behaves like a normal IGW (managed by AWS, scalable, resilient by design)
* BUT it is used for ipv6 traffic whenever we want to restrict connection to be initiated from outside. 
* Only allows ipv6 traffic OUT. 

## VPCE (GW)
* Provides private access to S3 and dyndb. 
* S3 and dyndb are public svc, which means that resources in vpc should technically need something like a Nat gw with a public Ip to access those. 
* But VPcEgw allows us to bypass all that complicated config. 
* A vpcegw is set per service and per region and is associated with subnets. 
* It works by creating a prefix list of the svc in scope (for ex, a pl with the ip addr of the s3 svc), and configure the route table of the subnets in scope to redirect traffic destinated to this PL to the vpcegw. 
* Vpcegw is HA accross all az in the region by default. 
* Can define endpoint policy to define what the vpce can access (for ex. only a subset of s3 buckets)
* vpcegw are regional, cant access cross-region svc. 
* Can be used to prevent leaky bucket. s3 bucket can be set to only allow access from a vpcegw endpoint. 
* vpcgw are not accessible outside the vpc theyve been created. 

## VPCE (interface)
* Provide private access to aws public svc. 
* Historically, used for anything NOT s3 or dyndb. But now S3 is supported. 
* Difference with vpcegw is that vpcei are directly added to specific subnets via a dedicated ENI. IT IS NOT HA. 
* If you want HA, you need to add one endpoint to one subnet per AZ used in the vpc. 
* Can control vpcei access via secgrp. 
* Endpoint policies can also be used to restrict what the vpcei can access 
* TCP and ipv4 only. 
* Implemented thanks to privatelink which allows to have access to external res and 3rd parties from your vpc. 
* Endpoint provides a new svc endpoint dns (ex vpce-123-xyz.sns.us-east-1.amazonaws.com)
* Two dns types are given:
    - Endpoint regional dns: good for resiliency, resolvable for all az in the region
    - Endpoint zonal dnd, one per az. 
* Application can optionally use these or
* Use privateDNS which overrides the default dns for these svc by creating a record in the vpc private hosted zone (default)
* In this case, apps dont need to change their setting. 

## VPC Peering
* Direct encrypted network link between two and only two vpcs only. 
* Works same/cross-region and same/cross-account
* (optional) Can have public hostname of ec2 res in peered vpc to resolve to private ip instead of public ip
* Peers in the same region can have their SG reference each other logically (way more efficient, but only if same region)
* VPC peering does NOT support transitive peering. 
* Routing configuration is needed and sg/nacl can filter. 