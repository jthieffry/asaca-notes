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

## Auto-Scaling Groups
* Automatic scaling and self-healing for EC2
* Uses Launch Templates or Launch Configuration
* Has a Minimum, Desired and Maximum size
* Keep running instances at a desired capacity by provisioning or terminating instances
* Scaling policies automate based on metrics
* Scaling Policies:
    - Manual scaling: manually adjust the desired capacity.
    - Scheduled scaling: time-based adjustment (ex. Sales period)
    - Dynamic scaling:
        1. Simple: CPU above 50%:+1, CPU below 50%:-1
        2. Stepped: Bigger instance addition/deletion based on difference: ex CPU above 50%:+1, CPU above 80%:+3
        3. Target tracking: Set a desired metric level and let ASG achieve that. Ex Desirate aggregate CPU: 40%.
    - Don't forget to set cooldown period (ie. grace period to wait after a scale in/out event to avoid rapid fluctuation and associated costs)
* When adding an ASG to the target of a ALB, you can use the ALB checks (HTTP/s) in lieue of ec2 checks to control the behaviour of the ASG.
* ASG are free. Only created resources are billed.
* Prefer more and smaller instance to have better granularity.
* Use ALB for the elasticity: abstraction of the compute instances.
* ASG defines when and where, LT defines what.

## ASG Scaling Policies
* ASG don't need scaling policies, they can have none. In this case, we can also resolve to manual scaling, where operator manually set the desired capacity.
* In addition to simple, step and target tracking scaling policies, we can also have based on SQS approximate number of pending messages.
* Between simple and step scaling, AWS recommends to use step, since the ASG can respond more appropriately to different amount of load.

## ASG Lifecycle Hooks
* Lifecycle hooks allows us to influence scale in/out events.
* If set during a launch or termination instance event, the instance won't terminate / start straight away.
* It will proceed to next step (ie. terminate or create) after a default timeout of 3600s or after we call the ContinueLifecycle action.
* Between that, we can execute custom actions, such as:
    - Sending notification via SNS/eventsbrige
    - Backup/load data to/from the instance
    - Etc.

## ASG Health Checks
* Can be either EC2 (default), ELB (can be enabled) and custom
* EC2: everything that has status: stopping, stopped, terminated, shutting down, impaired (not 2/2 ec2 checks) is considered unhealthy.
* ELB: considered healthy if status RUNNING and passing ELB health checks. It can then be more application aware (L7)
* Custom: instances marked healthy and unhealthy by an external system.
* Health check grace period (default 300s). How long to wait after an instance startup to start health checks on it.
* This allows system bootstrapping and application start. ^

## SSL Offload and Session Stickiness
* SSL Offload, three methods:
    1. Bridging (default):
        - The ELB has the SSL certs and backend ec2 instances too. It receives encrypted traffic, decrypt it to read http content and execute rules based on it. Then encrypts it back and send it to ec2 instances.
        - Pro: safe in transit, can do http rule routing.
        - Cons: EC2 has copy of cert, instances need to manage cert as well and need to dedicate crypto power to it.
    2. Passthrough:
        - The ELB listener is configured for TCP. Doesn't touch the encrypted traffic at all, passes it directly to ec2 instances.
        - Pro: safe in transit, AWS has no visibility on private cert.
        - Cons: ec2 need to manage certs + crypto power, no http routing rule possible at the ELB level.
    3. Offload:
        - SSL is terminated at the ALB.
        - Pro: EC2 doesn't need to manage cert anymore, http rule routing possible
        - Cons: AWS has visibility over the cert, traffic within aws network between alb and ec2 is unencrypted.
* Session stickiness is enabled at the ELB level and is set on a target group.
* If enabled, user connecting to the ELB to this target group will receive a cookie whose expiration date is anything betwen 1s and 7 days.
* Whenever a user connect to the target group, he will always use the same backend server unless:
    - The cookie expires
    - The backend server is unhealthy.
* This method allows stateful backend server to be always used by the same user so user has a better experience.
* The drawback is that it can cause uneven loads, the longer the stickiness and users are present.
* Best way is just to make stateless backend servers and disable session stickiness.

## Gateway Load Balancer
* Resolve the problem of having transparent security analysis appliances at scale. 
* If we need to intercept traffic from/out of an EC2 instance, its fine. But how do we do if instances are elastic and behind an ALB for ex. ?
* We use GWLB : offers security appliance scanning at scale. 
* GWLB:
 - Help run and scale 3rd party appliances. 
 - (FW, IDS, and other sec 3rd party)
 - Inbound and outbound traffic transparent inspection and protection. 
 - Are made of GWLB Endpoints (traffic enters/leaves from these endpoints) and GWLB itself (balance traffic accross multiple backend appliances). 
 - Traffic and metadata is tunnelled using GENEVE protocol to keep src and dst for security appliance scanning. 
* Ingress (gateway) route table directs any traffic destined for the ALB subnet at the GW endpoint in the same AZ. 
* The GW endpoint then directs the traffic at the GWLB which encapsulates traffic and load balance it to sec appliances. 
* Traffic then goes back to the GW endpoint which will redirect the traffic to ALB. 
* Above steps are for inbound. Similar process is done for outbound. 