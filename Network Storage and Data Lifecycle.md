# Key Notes

## EFS Architecture
* EFS is an implementation of NFSv4 -> provide network filesystem access (while EBS is block storage, SAN).
* EFS FS can be mounted in Linux only, and be shared between multiple EC2 instances.
* It's a private service. Conceptually, to make it available within a VPC, we specify subnets, and EFS creates mount point (reserve IP addr) in those subnets that EC2 instance can then use to mount NFS.
* On-pre can also leverage this service by accessing the VPC using DX or VPN.
* EFS offers general purpose or MAX I/O performance modes. General Purpose OK for 99.9% of cases (default).
* MAX IO might be used for parallel processing (scientific work, etc). High IO but also higher latencies.
* Offers also bursting and provisioned throughput modes (similar to gp3 vs io in EBS). Bursting is default.
* Offers standard (default) and infrequent access (IA) classes. Lifecycle policies can be used with classes, similar to S3.

## AWS Backup
* Fully managed data-protection (backup/restore) service.
* Consolidate management into one place, accross accounts and regions.
* Supports a wide range of aws products (EC2, EBS, EFS, Aurora, RDS, DynDB, DOcumentDB, etc, S3).
* You can define backup plans:
    - Frequency: daily, monthly, etc
    - Window: time window to execute backup
    - Lifecycle: move between storage class
    - Vault: destination container - assign KMS key for encryption and can set up lock to prevent deletion for compliance (Write once, read-many)
    - Region copy: can copy backup to other regions
* Backups can also be executed on-demand.
* Can also execute PITR (point in time recovery) with continuous backup.