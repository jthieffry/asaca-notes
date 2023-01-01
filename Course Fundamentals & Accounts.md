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
