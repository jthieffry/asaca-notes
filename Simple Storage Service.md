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