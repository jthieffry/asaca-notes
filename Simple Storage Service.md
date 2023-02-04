# Key Notes

## S3 Security

* S3 is private by default.
* Resource Policies:
    - Are like identity policies, but attached to a resource.
    - It's a policy that takes the perspective of a resource in terms of permissions.
    - Acts on a *principal*: If you see a principal in the statement, it's most likely a resource policy, and not an identity one.
    - You can allow/deny on the same or different accounts and you can allow anynomous principals.
    - S3 Bucket policies are a resource policy.
* Block Public Access is a failsafe permission setting for a bucket that will override any other policies: it can in one click block/allow public access to you bucket.
* As a rule of thumb, you can use Identity Policy (IAM) when:
    - You are controlling multiple resources, not just one type (ex. EC2 + S3 + SQS, etc.)
    - You have a preference for IAM, and you reside *in the same account*.
* You can use bucket policy when:
    - You're just controlling one resource type (here S3).
    - You need anonymous or cross-account permissions
* Avoid using bucket ACL as much as possible, they are legacy and less flexible.

## S3 Static Hosting

* S3 is usually used via API calls, but with the Static Hosting mode, you allow access via HTTP.
* You need to set a index and error html documents.
* A Website endpoint is created, based on the name and the region of the bucket.
* You can also specify your own domain via R53, but the name of the bucket matters (if your domain is sub.domain.com, the bucket name has to be sub.domain.com).
* Three use cases for Static Hosting:
    - Static website (ex.: blogs, etc.).
    - Offloading (put your static assets/ folder from your dynamic site in S3 and points the img in your main html to S3).
    - OOB pages (when mainsite is in maintenance, create a maintenance page and redirect your DNS to S3 Static).
* S3 you pay for:
    - The storage (GB/month).
    - Data OUT.
    - Requests/month (ex: "GET", "LIST", etc.).

## Object Versioning and MFA Delete

* Versioning is originally disabled on a bucket. If switch to enabled, it can never be disabled again.
* However, you can put in suspend state and back to enabled anytime.
* With versioning enabled, older versions of an object are stored. All of these versions take space (and we are billed for it).
* We can "delete" an object with the delete marker, which actually only puts an "invisible" version to an object making it appears like it's deleted, but it's actually still present. We can "undelete" anytime.
* We can still though definitively delete a version of an object with version delete. Removing all versions of an object effectively delete the object.
* MFA Delete is a mechanism that we can add to a version-enabled bucket which requires MFA token whenever we change the state of a bucket (from enabled to suspended and vice-versa) and whenever we want to delete an object version.

## AWS Performance Optimization

* Multipart Upload:
    - Data is broken up. Minimum data size 100MB.
    - Can be broken up to 10.000 parts, each part ranging between 5MB to 5GB
    - If one part fails to be uploaded, can be individually restarted.
    - Increased transfer rate since all parts are being uploaded in parallel.
* S3 Transfer Acceleration:
    - Initially OFF. Can be enabled if the bucket doesn't have period in its name and have compatible DNS naming.
    - Leverages AWS Network to transfer more quickly from a remote upload location to a bucket destination, using Edge location as entrypoint.
    - Only need to go to public internet when accessing the Edge Location.

## Key Management Service (KMS)

* Regional and Public Service - Create, Store and Manage Keys (Symmetric and assymetric).
* Can do cryptographic operations (encrypt, decrypt, etc.).
* Keys never leave KMS - Provides FIPS 140-2(L2).
* KMS Keys are logical (ID, Date, policy, desc. & state) and backed by physical key material.
* They are generated or imported and can be used for up to 4kb of data.
* KMS can generate data keys (Data Encryption Keys - DEKs) that can works on > 4kb.
* It will generate a plaintext version for immediate use and that would be discarded, and a ciphertext version.
* The client (either user or AWS service of example), will use the plaintext key to encrypt data, then discard that plaintext, then store the encrypted key with data.
* KMS Keys are isolated to a region and never leave. There is two types: AWS Owned (works in the BG) and customer owned.
* Customer owner can be AWS managed or customer managed keys. Customer managed keys are more configurable.
* KMS keys support rotation, rotated keys are known as backing keys, and we can set aliases.
* Key policies are resource policies on the key. Every key has one ! To use a key, we need the combination of the key policies + IAM policies or Grants.

## Object Encryption in S3

* Buckets themselves aren't encrypted. Their objects are. Encryption at rest.
* Two types of encryption: Client-side encryption and Server-side encryption.
* In client side encryption, the client is responsible for the whole crypto stack: manages the keys, do the crypto operations, and upload the encrypted data to S3 directly.
* In Server side encryption, we have three different type:
    - Server-side encryption with Customer provided keys (SSE-C)
    - Server-side encryption with Amazon S3-Managed Keys (SSE-S3)
    - Server-side encryption with KMS Keys stored in KMS (SSE-KMS)
* For SSE-C, the customer is responsible of the keys. When he uploads data to S3, he uploads the plaintext document and the key to be used along with the encryption. S3 gathers it, encrypts the document with the key provided, creates a hash of the key, then stores the encrypted document with the hash and discard the key. For decryption, user needs to submit the key used and S3 will decrypt the document and send back.
* For SSE-S3, customer simply uploads the plaintext data. AWS will generate a root key (user has no control over). For every objects uploaded, S3 will generate a new key (user has no control over) that will be used to encrypt the data. Root key will encrypt that key and the encrypted key with the encrypted data will be stored. Plaintext key is discarded.
* For SSE-KMS, it's like SSE-S3 but the Root key is actually a KMS one. Either a AWS-Managed key if we don't specify, or a customer managed one.
* SSE-C we have full control of our keys but we delegate the encryption/decryption (CPU) process to S3.
* SSE-S3 is usually the default encryption method, but we don't have any control over the cryptography process (i.e. rotation schedule, etc.). It also means that a S3 admin can decrypt the data at will. *SSE-S3 uses AES256 for the cryptographic aspect algorith.*
* SSE-KMS has the added benefit over SSE-S3 that it provides role separation. If we uses a customer managed KMS keys, then we control the rotation and the permission, meaning who can decrypt our data. In heavily regulated environment, this might be necessary.
* We can also set a bucket default encryption. This is just a default, it means that if we don't specify anything, this encryption (or not) process will be used.
