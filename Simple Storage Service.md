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

## S3 Bucket Keys

* When we use KMS to encrypt objects during PUT operations in S3, the KMS key generates a DEK (data encryption key) for each PUT objects.
* Because KMS has a cost, and a throttle, if we do a LOT of PUT operations per second, it can become extremely costly or we can hit a limit.
* To circumvent that, it possible to generate a temporary Bucket key: the KMS key generate a temporary key that will be common for all PUT oeration objects in the bucket.
* It is not retroactive: it only impacts the encryption of objects after the bucket key has been enabled.
* Side effects of that is that obviously, KMS cloudtrail event will show less logs (because less calls). Events will also show the bucket arn instead of the object arn. Encryption is preserved for replication.

## S3 Object Storage Classes

* S3 Standard: The default, balanced one in terms of costs and features. You pay for storage (GB/month), transfer OUT (GB) and a price per 1000 requests. No specific retrieval fee, no minimum duration, no minimum size. Data replicated accross at least 3AZ, with eleven 9 durability. Objects stored returns a 200OK. Instant retrieval. *Use S3 Standard for frequently accessed data which is important and non replaceable.*
* S3 Standard IA (Infrequent Access): Like S3 standard, but storage fee is much cheaper. The flip side of this is that you are billed for a minimum duration of 30 days, a minimum capacity charge of 128KB per objects and for every retrieval you pay a fee. Instant retrieval. *Use S3 Standard-IA for long-lived data, which is important but access infrequent.*
* S3 One Zone-IA: Similar to S3 Standard-IA but way cheaper. As a flip side, data only resides in one AZ. Still have 11 nines, but if the AZ fails, data is unaccessible. Instant Retrieval. *Use thise for long-lived data, which is non-critical and repleaceable and where access is infrequent.*
* S3 Glacier - Instant: Like S3 Standard-IA, but cheaper storage with more expensive retrieval and longer minimums. *Should be used for long-lived data accessed once per quarter with millisecond access.*
* S3 Glacier - Flexible: Replicated in 3AZ, cheap storage. However, objects caanot be made accessible. Need to pay a retrieval fee: expedited (data available within 1-5 min, $$$), standard (data available within 3-5 hours, $$), bulk (available in 5-12 hours, $). As such, first byte latency is minute or hours. 40KB min size, 90 days min duration. *Use it for archival data where frequent or realtime access isn't needed (ex. yearly). Minutes-hours retrieval.*
* S3 Glacier - Deep Archive: Similar than Flexible glacier but cheaper with much longer retrieval times: standard (12 hours), Buld (up to 48 hours). 40KB min size, 180 days min duration. *Use it for archival data that rarely if ever needs to be accessed - hours or days for retrieval e.g. leval or regulation data storage.*
* S3 Intelligent Tiering is a special storage class. It is basically splits into 5: frequent access (like Standard), Infrequent Access (Like IA), Archive Instant access (Like Glacier instant), Archive Access (like Flexible) and Deep Archive (Like Deep Archive). S3 will monitor objects in these storage class and move them to the different categories, with the associated costs, based on their usage patterns. Note that for Archive and Deep Archive access, those are optionals and need to be opt-in explicitely since there is a byte latency implication (retrieval time). It manages costs efficiently but comes with an additional monitoring and automation costs per 1000 objects. *Use S3 Intelligent Tiering for long-lived data with changing or unknown patterns.*

## S3 Lifecycle Configuration

* It's a set of rules that consists of actions on a bucket or a groups of objects in a bucket. Either transition actions (move one object from one storage class to another) or expiration action (delete that object or object version).
* These rules are triggered per duration (ie. after X days do Y) but NOT per access pattern. This is done by S3 Intelligent Tiering.
* Transitions can only go from top to bottom, in this order: Standard -> Standard-IA -> Intelligent-Tiering -> One Zone-IA -> Glacier instant -> Glacier flexible -> Deep Archive. All objects in class C can be move to any C-n classes below (except One-zone IA that can only be moved to flexible or deep archive).
* Note: be careful when setting lifecycle configuration from Standard Class: smaller objects in Standard can get billed to upper ranks due to the minimum object size billable requirements. in lower classes.
* Note: A rule cannot trigger a transition from Standard to Infrequent or 1Zone before 30 days.
* Note: A single rule cannot transition to IA for 1Zone and then to glacier classes within 30 days (duration minimums).

## S3 Replication

* Two types: Cross-Region Replication (CRR) and Same-Region Replication (SRR).
* If replication is in the same account, the process will create a role assumed by S3 that will read from a source bucket and which would have permissions to read to a destination bucket. The replication is then done via SSL.
* If replication is in a different account, in addition to the previous mechanism, it will also create a bucket policy on the destination bucket in the foreign account that would trust the role in the local account.
* The options for the S3 replications are that: it is either applied on all objects or a subset in a bucket, we can change the storage class during replication (default is to maintain), we can change the ownership (default is the source account), and we can set Replication Time Control (RTC, default is best effort, if set, we have a 15min guaranteed for object to be in sync in local and destination bucket).
* Important for the exam:
    - Replication is not retroactive (does not apply on existing objects) and versioning needs to be ON.
    - One way replication from source to destination.
    - Only works with unencrypted, SSE-S3 and SSE-KMS (with extra config) - doesn't work with SSE-C.
    - Source bucket owner needs permissions on the objects.
    - Does not replicate system events (like lifecycle rules), or objects in Glacier or Glacier Deep Archive.
    - Does not replicate Delete markers.
* Use cases for replication:
    - For SSR: Log aggregations: puts logs from different buckets into one bucket.
    - For SSR: Have data sync between a PROD and a DEV/TEST account.
    - For SSR: Data resilience with strict sovereignty (move data from one account to another put keep the region because of data sovereignty constraints).
    - For CRR: Global resilience improvements (same as above but without the data sovereignty restriction).
    - For CRR: Latency reduction: put data is buckets closer to end user for example.

## Presigned URL

* Can be used to access an object in a private S3 bucket, with the access rights of the identity which generates them.
* They are time-limited and they encode all of the authentication information needed inside.
* Can be used to upload objects and download objects from S3.
* You can create a URL for an object you have no access to.
* When using the URL, the permissons match the identity which generated it. Access denied could mean the generating ID never had access, or doesn't have *now*.
* It's usually a bad idea to generate with a role. URL stops working when temporary credentials expire.

## S3 Select and Glacier Select

* S3 can store HUGE objects (up to 5TB), you often want to retrieve the entire object.
* However, retrieving an entire 5TB object takes time, and use 5TB of bandwidth and bills.
* Filtering at the client side, doesn't reduce this since the object has been transfered already.
* S3/Glacier select let you use SQL-like statements to select parts of the object, prefiltered by S3.
* Compatible formats: CSV, JSON, Parquet, BZIP2 for CSV and JSON.
* It is then faster and cheaper.

## S3 Events

* Notifications generated when events occur in a bucket, can be deliver to SNS, SQS and Lambda functions.
* Events include:
    - Object Created (put, post, copy, etc.).
    - Object Delete (*, delete, deletemarkercreated, etc.).
    - Object Restore (retrieve from Glacier).
    - Replication (operationmissedthreshold, etc.).
* Events are JSON formatted. A probably better alternative is to use EventBridge, which is more recent and supports more services and events.

## S3 Access Logs

* Send logs access details from a source bucket (objects accessed, by who, etc.) to a target bucket.
* Best effort log delivery, within a few hours. Access logging can be enabled via the consule UI or via the PUT bucket logging.

## S3 Object Lock

* Only enabled on 'new' buckets (need support to enable in existing buckets).
* Useful for Write-Once-Read-Many type of workload. NO DELETE, NO OVERWRITE.
* Requires versioning - individual versions are locked.
* Two types of object lock available:
    - Retention Period
    - Legal Hold
* They can be applied both at once, one or the other, or none.
* A bucket can have a default object lock setting.
* For Retention:
    - Specify days & year for the retention period.
    - The first mode, COMPLIANCE, *can't be adjusted, deleted, overwritten, even by the account root user, until retention expires*
    - The second mode, GOVERNANCE, allow special permissions to be granted allowing lock settings and underlying object to be adjusted.
    - In this case, you need to have the s3:BypassGovernanceRetention action and to pass the header x-amz-bypass-governance-retention:true in your request header.
* For Legal Hold:
    - Set on an object version: ON or OFF, it's a toggle, no retention period needed.
    - Cannot do ANY DELETES or CHANGES until removed.
    - Action s3:PutObjectLegalHold is required to add or remove. Used to prevent accidental deletion of critical object versions.

## S3 Access Point
* Simplify access to S3 bucket.
* Rather than 1 bucket with 1 policy, create many access points, each with different access policies, different network access control and each with its own endpoint address.
* Create via webUI or console with command: aws s3control create-access-point --name secretcats --account-id 1234556 --bucket catpics
* For one bucket, you create access points and relative access point policies, which:
    - Has similar functionality as bucket policy,
    - Has unique DNS address for network access.
* Access point can be:
    - With internet origin (ex.: end-user accessing bucket from internet),
    - Or VPC origin. Here, we also need to specify a VPC endpoint, where additional policies can be set.
* Regarding access point access, a common scenario is to allow wide on the bucket for each endpoint (ex: allow \*) and let the endpoint policy filter what the user can actually access.
