# Key Notes

## ELB Evolution
* 3 types of load balancers (ELB) available within AWS.
* Split between v1 (avoid / migrate) and v2 (prefer).
* Classic Load Balancer (CLB) - v1 - introduced in 2009.
* Not really layer 7, lacking features, 1 SSL certificate per CLB.
* Application Load Balancer (ALB) - v2 - HTTP/S/WebSocket LB.
* Network Load Balancer (NLB) - v2 - TCP, TLS, and UDP.
* v2 = faster, cheaper, support target groups and rules.

## ELB Architecture
* One ELB is actually made of multiple nodes. It is configured to run at least 2 AZ (potentially more).
* For EACH AZ, at least ONE node is placed into a subnet in this AZ. More nodes can be added as load scales.
* Each ELB is configured with a A record which resolves to ALL ELB nodes.
* ELB can be internet facing (nodes have a public and private IP) or internal (only has private IP).
* Subnets on which nodes are deployed need to have at minimum 8 free IPs (/28) but usually more is better so /27 is a good default for scaling.
* EC2 doesn't need to have a public IP to work with a LB as a backend.
* Historically, LB in one AZ could only communicate to backends (ex. Ec2) in the same AZ. But now, it is possible (enabled by default) to allow cross-zone LB to allow LB nodes to distributes traffic to backends which are not in the same AZ, avoiding potential imbalance in load.

## ALB vs NLB
* ELB v2 supports rules (how to route) and target groups (backends). 1 SSL certificate available per rule, so you can have ALB consolidation.
* ALB are L7 load balancers, they listen on http/s. They don't understand other L7 (smtp, ssh, gaming,...)
* ALB also dont have tcp/udp/tls listeners.
* However, they understand every L7 content type: cookies, custom headers, user location, app behaviour, etc.
* SSL TERMINATION ON THE ALB. If HTTPS, SSL/TLS is ALWAYS terminaned on the ALB.
* A new connection is actually made from the ALB to the backend. So client <> ALB is one connection and ALB <> backend is another.
* ALBs must have SSL certs if https is enabled.
* ALB are slower than NLB since they have more levels of the stack to cover.
* Health checks available at the ALB to evaluate application health at L7.
* ALB Rules:
    - Direct (route) connections which arrive at a listener.
    - Processed in priority order.
    - Default rule = catchall.
    - Some example of rule condition: host-header, http-header, http-req-method, path-pattern (regex), query-string, source-ip.
    - Actions possible: forward, redirect, fixed-response, authenticate-oidc, authenticate-cognito.
* NLB are L4 LB: TCP, TLS, UDP, TCP_UDP.
* No visibility or understanding of HTTP/s: no headers/cookies/session stickiness.
* Really fast (millions of rps, 25% of ALB latency)
* Good for smtp, ssh, game servers, non http/s financial apps.
* Health chekcs only check ICMP/TCP handshake. Not APP aware.
* NLB can have static IPs - useful for whitelisting. Can also forward TCP to instance, so can forward SSL/TLS without terminating at the ELB.
* Can be used with private link to provide services to other VPCs.

## When to use NLB vs ALB
* If needing unbroken encryption: NLB
* if need static ip for whitelisting: NLB
* Fastest performances: NLB
* Not an http/s protocol: NLB
* Privatelink: NLB
* Everything else: ALB

## Launch Configuration & Templates
* Both allow to define the configuration of an EC2 instance in advance.
* Things like AMI, KP, Storage, Network, Userdata, etc.
* Both are NOT editable. LC is locked forever, while LT allows versioning.
* LT provides newer features (placement groups, elastic graphics, etc). At this point of time, LT are recommended over LC by AWS.
* LC are used to set-up ASG but can't be used to schedule individual adhoc EC2 instances. LT allows both.