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