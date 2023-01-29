# Key Notes

## IAM Identity Policies

* IAM policies apply on IAM Identities (so either IAM User, IAM Group or IAM Role).
* IAM Policy Document (=Identity Policies): Document that that has one or multiple statements that are made of: SID (Statement ID, optional, describe the statement), Effect (Allow or Deny), Action (service:action, for ex: s3:createBucket. Can use *), Resource (the arn on which this takes effect.)
* Whenever an identity tries to perform an action on a resource, all policies in scope (from potentially different documents or different statements) are collected and the following rule applies: DENY, ALLOW, DENY.
    - First DENY is that an explicit DENY takes the priority (a DENY in a statement has the final word and overrides everything).
    - If there is no explicit DENY on a resource, then if there is an explicit ALLOW on this resource, this is ALLOWED.
    - If there is no explicit ALLOW or DENY for a resource, then it is DENY by default (except if this is the account root user.)
* There is two type of policy documents:
    - Inline policies are directly attached to identities: if you have the same policy for different identities, you have to copy-paste that policy to all the different identities.
    - Managed policies are objects on their own and can be attached to identities as needed. They are more easily to maintain (only need to modify the policy once for every identities attached), and more scalable. There is AWS Managed Policies, provided by AWS and that we cannot change, and custom Managed Policies, created by us.
    - Inline policies should thus only used for exceptions (exceptional deny or allow to one or more entity for example)

## IAM Users and ARN

* IAM Users are an identity used for anything requiring long-term AWS access (humans, applications, service accounts) -> Something that we can identity clearly.
* IAM starts with a principal. A principal is an identity that is trying to access an AWS account.
* Before doing anything, the principal needs to authenticate himself i.e taking one IAM identity. To do so in the case of IAM users, he needs to provide either access keys or username/password.
* Once a principal is identified, IAM knows which policies apply to him thanks to its identity. IAM can now allow or deny the actions on the resources intended by this identity thanks to the policy documents and statement. This is the authorization.
* Authentication and authorization are two different steps made by IAM.
* *exam* There is a hard limit of 5000 IAM users per account and one IAM user can be member of 10 groups max. Which means that for an org. or app that has thousand users, IAM users are usually not the solution.
* Whenever you want to interact with resources in AWS you need to have an identity.

## IAM Groups

* They are fundamentally only a containers for users. They can have inline or managed policies attached to them. No nesting, and soft limit of 300 groups per account.
* Groups are not a true identity so as such, they cannot be referenced as a principal in a policy. They are literally just a dumb container with users on which you can attach policies for convenience and easier management of users.
* AWS doesn't provide a "native" group. It has to be explicitely created and administred by the admin.

## IAM Roles

* Roles can be used when you don't exactly the number of principals who will assume that role. Identities/principals become the role if they are authorized. IAM roles are assumed. Principals / Identities could be internal IAM users, but also internal/external services, users, roles, Facebook, twitter, etc.
* Roles have two types of permissions attached to them: permissions policy (deny, allow, deny) and trust policy (which principals / identity can assume that role). Trust policy can trust things in internal/external identities, etc.
* If an identity is allowed to assume a role, AWS will generate temporary security credentials via sts:AssumeRole (STS service). Those tokens are only valid for a certain amount of time. Those tokens are linked to what the permission policy of the role allows. Whenever we use those tokens, permissions are checked. Once tokens expire, need to renew them.

## When to use IAM Roles 

* For example, when granting to a service permissions (ex. AWS Lambda).
* As an emergency temporary permission granter (ex. Support IAM user who needs exceptional permissions for emergency task). It's only temporary and can be easily logged.
* ID Federation for corporate users who are already part of an IPA/AD system. They can use roles to act on account resources with SSO. We also safely avoid the 5000 IAM users limit.
* Web Identity Federation, for example for a mobile apps that has millions of users who need to access AWS resources. They can use providers like FB, Google, Twitter to assume a role in a target account. It's also SSO for them. No need for AWS creds on the apps, SSO and scales to millions of users.
* Can also be used for cross account access when IAM users in account B can assume a role in account A to access resources in account A. Since they assume an Identity in account A, every resources created with this roles are owned by account A.

## Service-Linked Roles and Passroles

* Service-linked roles are just a more specific type of role. They are IAM roles linked to a specific AWS service, providing permiessions that a service needs to interact with other AWS services on your behalf (for ex. for CFN to be able to create other AWS resources).
* Service might create/delete the role, or we can do it ourselves via setup or within IAM. But we can only delete the role when it's no longer required.
* PassRole is a action (iam:PassRole) that allows an identity the ability to pass an existing role into an AWS service (but to actually create or edit that role). It is just a "user" of that role in a sense that it can pass it to services which might require it.

## AWS Organizations

* Manages multiple accounts under the same umbrella. You pick one account that is standard to become the Management account. It then creates a Organization that includes this management account. You can then invite pre-existing standard accounts to join the organization. Or you can create accounts directly under the organization from the beginning.
* The Organization will create a "Organizaton Root" which is basically just a container that includes all accounts within the Org. You can then create other containers called Organizational Units that can include accounts but also other OU. You can then define a hierarchy.
* Members account have their billing methods removed and segregated into the mgmt account, which only has its billing methods. The bills of other accounts are consolidated in the mgmt one.
* Things that benefits from usage (for ex. using X amount of resource Y gives discount) are pooled, so all accounts within an Org contributes to that.
* Organization also features SCP (service control policies), which restricts what accounts in the Org can do.
* Best practice is not to have IAM users inside all other member accounts. You actually use a single account to login to (either the mgmt account or another one). This single accounts contain all identities we can login to. Enterprise can also use ID Federation. In this case in works like the following:
    - Use ID Federation (with on-prems AD) to access the single identity account.
    - Role switch from this single identity account to roles in the other member accounts (and get the associated permissions). You're assuming a role in the other member account.

## Service Control Policies

* SCP can be attached to OU (Organizational Units), the Root container or even individual accounts. They basically put the boundaries of what an account is allowed to do. They are applied hierarchically to accounts, within the OU structure (concatenated and applied).
* The management account is left unaffected by SCP.
* SCP limits what the account can do (including the account root user) but they don't grant any permissions themselves. They just set the boundaries.
* SCP work either in a allow or deny list fashion. By default, it's a deny list, since when activated, AWS create a FullAWSAccess permission per default. So it becomes effectively a deny list where admins need to explicitely state which actions to deny (deny, allow, deny).
* If we remove the FullAwsAccess on the SCP, it becomes a standard allow list (deny, allow, deny) and the admin needs to explicitely state which actions to allow.
* Deny lists have less maintenance overhead but are less secure than allow lists.

## CloudWatch Logs

* Public Service usable from AWS or on-premises.
* Store, monitor, and access logging data.
* A way to interact with CloudWatch logs for a service is to either:
    - Have AWS Integration directly: EC2, VPC Flow Logs, Lambda, Cloudtrail, R53, etc.
    - Use the unified CloudWatch agent.
* Can generate metrics based on logs (for ex. number of failed ssh access) -> Metric Filter.
* CloudWatch Logs works per region. Logging sources events are stored inside log stream, which correspond to different source (for ex. log stream for /var folder in machineA). Those logs streams can then be grouped inside log groups (for ex. /var log group). On this log group, this is were we can define retention policies, permissions and potential metric filters.

# CloudTrail

* Logs API calls/activites as a CloudTrail Event.
* 90 days stored by default in the Event History.
* Enabled by default, no cost for 90 days history.
* If need to customize the service, need to create 1 or more Trails.
* Logs management events (EC2 instance creation, etc.) and data event (S3 file accessed ,etc.)
* Trails can be configured to be stored in S3 and/or sent to CWLogs (when further processing can be done.)
* It logs management events only by default.
* Can either create:
    - One region trail: the trail only stores event in this region.
    - "All region" trail: it's actually a collections of trails bound together in a logical global trail. CloudTrail is fundamentally a regional service.
* Global services like IAM, STS, CloudFront are stored in us-east-1.
* CloudTrail doesn't log in real time. There is up to 15 min delay.
