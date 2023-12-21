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

## R53 Weighted Routing
* Used for simple load balancing or testing new software versions. 
* Each entry for the same record (ex. www) is assigned a weight. 
* Each record is returned based on its record weight vs the total weight. 
* For ex. 3 www records with weight 40 40 and 20. The first www is returned 40% of the time. 
* If weight = 0, it is not returned. Unless all entries are 0. 
* If a chosen record is unhealthy according to checks, the process selection is repeated until one healthy record is chosen. 

## Latency Based Routing
* Use it for optimizing the performance and user experience.
* You have multiple entries for the same record (ex. 4 A recored wwww with different target IP).
* For each entry, you specify one AWS region. Latency based routing supports one record with the same name in each AWS region.
* The record returned to a client request is the record that has the lowest latency to the client and is healthy.
* AWS maintains a (non-realtime) DB of user latencies based on the client source IP.

## Geolocation Routing
* With Geolocation routing, records are tagged with a location: subdivision (US state), country, continent or default.
* An IP check verifies the location of the client (usually the resolver).
* R53 then checks for records: 1. In the state, 2. In the country, 3. The content and 4. (Optionally) the default one.
* It stops if it has a match, in the above order, and return the result. If it doesn't have any (and no default), it returns NOT_FOUND.
* IMPORTANT: Geolocation doesn't return the closest record. Only the relevant "location" record.
* It can be used to implement regional restrictions, language-specific content, or load-balancing across regional endpoints.

## Route53 Interoperability
* R53 is conceptually divided in two, even if it's not obvious: the domain registrar part and the domain hosting part.
* It can do both or only one or the other, while the other part is provided by a 3rd party.
* If you use as the domain registrar + domain hosting part (the "full package"):
    1. AWS accepts your money after you chose your domain (domain registration fee).
    2. AWS creates a new public DNS zone for your domain and allocates 4 DNS servers for it (domain hosting part)
    3. AWS liases with the registrar of your domain TLD (for ex. .com or .io) and registers your domain by specifying the 4 above DNS servers for this domain.
* But AWS can also only provide one or the other part. For example, you can have a third party (or your own DNS on-pre) that hosts your zone.
* You then register the domain with R53 and specify the NS (3rd party or your own) to AWS who will liaise with domain TLD and register those NS.
* Or you can do it the other way around and use a third party registrar and specify R53 NS (the ones hosting your zone).

## Implementing DNSSEC
* When you enable DNSSEC on a public zone, AWS will create KMS assymetric KP (so a private and public one) in the US-EAST-1 region (cannot change).
* These KMS keypair can be roughly assimiled to the KSK (key signing key) of your zone.
* Internally, R53 will then generate a ZSK KP. It will also handle the rotation internally (you don't manage it.)
* R53 will then update your hosted zone records to include the public KSK and ZSK keys in two different DNSKEY record.
* It will also include the signed public ZSK key by the private KSK in the RRSIG DNSKEY record.
* You need then, via the AWS Console interface for ex., to provide the signing algorithm and the public ZSK Key to the TLD registrar (ex. .org).
* R53 will then contact the TLD and ask the him to include a record containing the hash of your public KSK in his record.
* The chain of trust will then be completed and DNSSEC enabled.
* You can create alarm for DNSSEC in CloudWatch and check for failures such as DNSSECInternalFailure and DNSSECKeySigningKeysNeedingAction.
* You can also enable DNSSEC-Validation for your VPC. If so, invalid results on DNSSEC enabled zones won't be returned (ex. bad signature)