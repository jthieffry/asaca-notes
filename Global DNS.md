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