# Key Notes

## Bootstrapping EC2 with User Data

* EC2 bootstrapping allows EC2 Build Automation via user-data.
* User data are accessed via the user-data IP: 169.254.169.254/latest/user-data.
* Anything under that user-data is executed at launch by guest OS. So it's usually a script.
* ONLY EXECUTED ONCE, AT LAUNCH (not restart).
* EC2 doesn't interpret, OS needs to understand the user-data passed.
* Similar to meta-data, anyone connected to the instance can access user-data. Not secure.
* User data is limited to 16KB. For bigger script, user-data should download the bigger script.
* User-data can be modified when the instance is stopped, but regardless, it only ever executes at the launch time of the instance.
* Bootstrapping and AMI bake can reduce the boot-time-to-service-time. If an app is 90% installation, 10% configuration, 90% can be done via AMI bake and 10% via bootstrapping (user-data).

## Bootstrapping with CFN-INIT

* cfn-init is a helper script installed on EC2 OS. It is like a simple configuration management tool (Puppet).
* It receives directives via Metadata and the AWS::CloudFormation::Init on a CFN resource.
* In CloudFormation template, you specify a section with the above identifier and with Userdata that specifies the cfn-init command on run.
* You can also use the cfn-signal command that will report back to Cloudformation to confirm that the cfn-init command was successful (via CloudFormation CreationPolicy directive.)
* CFN-INIT works with stack updates, which was not the case with bare bone userdata (oneshot).

## EC2 Instance Role & Profiles

* Role attached to an instance are actually attached to that instance via an instance profile which shares the same name as the role.
* Credentials can be fetched from inside the metadata service, under iam/security-credentials/role-name
* Those credentials are automatically rotated - always valid.
* Roles should be the preferred way to provide creds. Long-term store of other type of creds is highly discouraged.
* CLI tools use role credentials automatically.

## SSM Parameter Store

* Storage for configuration & secrets (string, stringlist and securestring)
* Supports hierarchies and versioning, plaintext and ciphertext.
* Has public parameters (managed by AWS), such as the Latest AMI per Region.
* Access controlled by IAM.

## System & Application Logging on EC2

* CloudWatch is the service used for Metrics.
* Cloudwatch Logs is a subset of the above and is used specifically for logs.
* However, neither capture natively data inside an instance. Cloudwatch agent + its perms / config are required.

## EC2 Placement Groups

* Three types: cluster, spread and partition. 
* Cluster:
 - All EC2 instances are in the same rack. Sometimes same host. 
 - ONE AZ ONLY
 - Can span VPC peers but impact performance. 
 - Requires supported instance type. 
 - Recommended: use the same instance type. 
 - Recommended: launch all instances in the cluster at the same time. 
 - 10Gps single stream perf between instances in the cluster (vs 5 by default). 
 - Use cases: perf, fast speed, low latency. 
* Spread:
 - Provide infra isolation. 
 - Each instance runs from a different rack. 
 - Each rack has its own network and power source. 
 - 7 instances per AZ (hard limit). 
 - Not supported for dedicated instances or hosts. 
 - Use case: small number of critical instances that need to be kept separated from each others. 
* Partition Placement Groups
 - 7 partitions per AZ.
 - Instances can be placed in a specific partition. 
 - ...or auto placed. 
 - Great for topology aware apps. Ex: HDFS, HBase, Cassandra. 
 - Contain the impact of failure to part of an application. 

## EC2 Dedicated Hosts

* It's an host dedicated to you. 
* Must choose a specific family. ex: a1, c5, m5
* No instance charge as you pay for the host. 
* Ondemand and reserved optiond available. 
* Host hardwarr has specific physical socket and core. Important for capacity and software licensing. 
* "Legacy" family type host only allows you to schedule one instance size only. 
* "Modern" family type allows you to mix and match instance size for greater flexibility. 
* AMI limits: rhel, suse, windows ami arent supported. 
* Amazon RDS instances arent supported. 
* Placement groups are not supported. 
* Hosts can be shared with other accounts via the RAM service. 

## Enhanced Networking and EBS Optimized

* Enhanced networking use SRIOV: physical NIC is virtualization-aware. 
* Comes at no charge. Available on most EC2 types. 
* Higher IO and lower host cpu usage. More bandwidth. Hogher packet per second. Consistent lower latency. 
* EBS Optimized:
* Historically, data and backbone (EBS) network as shared. 
* EBS optimized means that we now have dedicated capacity (network) for EBS. 
* Most instances support it and have it enablrd by default. 
* Older instance might support it but comes at cost extra. 