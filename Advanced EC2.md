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

