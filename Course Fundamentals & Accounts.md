# Key Points

## AWS Accounts

* AWS Account only needs unique e-mail address and a credit card (not necessarily unique).
* AWS Account is a container for identities and resources.
* By default, everything outside this container is denied access (unless explicitely granted).
* AWS Account ROOT user has all control over the AWS Account. This can't be modified.
* AWS Accounts are good for segregating workloads and ensuring minimal business impact in case of incident.

## MFA

* Adds another layer of security to your accounts, in addition to username and password.
* Secret Key & User ID encoded by AWS within a QR code, that you have to register to your virtual MFA device.

## Budgets

* Good to track spending, usage, etc. accross all or a subset of AWS services.
* Multiple budget types available: spending, usage*, savings plans*, reservation* (*: need to enable Cost Explorer first).
* Can create an alert threshold for the budget and add an e-mail recipient to be alerted before the threshold is reached.

## Creating a New Account - Checklist

* Add MFA to Root account.
* Add a budget with threshold alarm.
* Enable IAM User & Role Access to billing.
* Review and enable monthly costs reports (PDF version, etc.).

## IAM Basics

* Identity & Access Management: fully trusted by the AWS account.
* Global service with a global resilience & has all permissions the account and root user has.
* IAM can create three entities: User (individuals), Group (group of users), Role (when the number is unknown).
* IAM can also create Policy that we attach to user / group / role & that defines what is allowed or denied for AWS Services access.
* IAM is also fundamentally three things: an ID Provider (manages identities), Authenticate (prove who you claim to be), Authorize (allow or deny resource access).
* IAM is FREE but has quota restrictions, it allows or denies its identities on its AWS account, but has no direct control on external accounts or users.
* IAM also allows Identity Federation and MFA.
