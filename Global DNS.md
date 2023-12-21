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

## R53 Simple Routing
* Simple routing doesn't support health checks. All values are returned when queried. 
* Use simple routing when you want to route requests towards one service such as web server. 

## R53 Health Checks
* Health checks are separate from, but are used by records. 
* Health checkers are located globally. 
* Health checkers check every 30s (or every 10s for extra cost). 
* Can do TCP/HTTP/s and HTTP/s with string matching. 
* Reports as either healthy or unhealthy. 
* Can check either endpoints, CloudWatch Alarms or other checks (calculated). 
* Since checkers are globally distributed, if 18%+ of checkers report as healthy, the health check is healthy. 

## R53 Failover Routing
* A service (ex. www) can have two DNS record (two A) pointing to different targets. 
* One record is the primary (ex ec2) while the other is secondary (s3). 
* If the target of the health check (www) is healthy, the primary is used. 
* If the target is unhealthy, any queries will return thr secondary record. 
* Use this technique when you want to configure active/passive failover. 

## R53 Multi-value routing
* You create multiple records with the same name (ex. 8 www records which points to different ip). 
* When client ask for www, up to 8 records are returned (if more than 8, 8 are selected at random). 
* Client pick one value among those returned. 
* Each record is independent and can have an associated health check. 
* Any record that fails the health check wont be returned when queried. 
* Multivalue improves availability. It is NOT a replacement for load balancing. 

