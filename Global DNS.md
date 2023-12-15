# Key Notes

## R53 Public Hosted Zones
* A R53 hosted zone is a dns db for a domain. 
* Globally resilient (multiple dns servers). 
* Created during domain registration via R53 - or created separatey if using another registrar. 
* Host DNS records (AAAA, MX, etc.)
* Hosted Zones are what the DNS system references - authoritative for a domain. 
* Zone file hosted by R53 (public NS). 
* Accesible from internet and VPC(if dns resolution enabled). 
* Hosted on 4 R53 NameServers specific for that zone. 

## R53 Private Hosted Zones
* It is like a public hosted zone... which isnt public. 
* Associated with VPCs and only accessible from those vpcs. 
* Using different accounts is supported via cli/api. 
* Split view (overlapping public and private) for public and internal use with the same zone name. 

## CNAME vs. R53 Alias
* CNAME maps a name to another name. Ex. www.cat.io -> cat.io.
* CNAME doesnt work for naked/apex domain. Ex. cat.io -> xxxx doesnt work. 
* Since many AWS svc use a DNS name (ex ELBs), mapping cat.io -> ELB with a CNAME wouldnt work. 
* Solution is to use AWS builtin Alias. 
* Alias has a type. Its type should be the same as the record it is pointing at. 
* Alias can be used for both apex and normal record. For the latter it behaves like CNAMe. 
* No charge for alias requests pointing ag AWS resource. 
* For AWS services, default to picking alias (ex apigw, cloudfront, elastic beanstalk, ELB, S3, global accelerator). 