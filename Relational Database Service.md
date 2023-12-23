# Key Notes

## Database Refresher
* Relational vs non-relational:
    1. Relational (RDBMS) are also abusively called SQL. They have a rigid structure defined in advance that defines the model of the tables and between the tables (the schema). Hard to change after initial setting.
    2. Non-relational (NoSQL) are everything that are not relational. They have no or less rigid structure. Multiple type exist:
        * KV: No schema/structure, scalable and really fast. You have one key that correspond to one value.
        * Wide-column store: Has tables and either one key (partition) or two (partition+sort) which need to be unique. These are defined for all records but that's the only restriction. Fields for these key can be multiple or none. Doesn't have to be the same for all records. DynamoDB is the star product in AWS for that. 
        * Document: Like KV but the V is actually document (like json or yaml) that can change dynamically easily. DB engine is aware of the document and can execute complex adhoc nested query within the data (document). This DB is good for interacting with whole documents or deep attribute interactions.
        * Column: Column DB acts on column instead of row. In traditional SQL, you act on row, ie records. It's perfect for transaction type of scenario (ie. item A got sold on X for price P). But it's not that efficient if you want to have the price for all items because you need to fetch all records first. Column DB shift the paradigm and actually stores Data by consedring the column first. It makes it really good for auditing and reporting (ie. generating reports based on existing SQL DB). AMazon Redshit, which is a datawarehouse used for reporting, is an example of a column DB.
        * Graph: Those DB stores relationship betwen table/items in addition to the items themselves. It's really good for relationship that are dynamic and changes often. Queries are efficient. Particularly useful for social network type of DB.

## ACID vs. BASE
* These are two DB transaction models. 
* According to CAP theorem, a multi-node DB is either consistent ot available. 
* ACID ensures consistency while BASE focuses on availability. 
* ACID:
 1. Atomic: every part of the transaction succeed or fail. 
 2. Consistent: every requests take into account previous request. 
 3. Isolated: every requests are made like they are the only ones happening. 
 4. Durable: once executed, requests remain and survives power off. 
* When you hear ACID, think RDS. It is good but scalability is limited. 
* BASE:
 1. Basically available: Data is replicated asap and
always available. 
 2. Soft State: Consistency is not guaranteed and must be managed by the app. 
 3. Eventually consistent: State will eventually consistent but might take time. 
* When you hear BASE, think DynamoDB. Highly scalable. DynamoDB can however enforce consistency thanks to DynamoDB transactions. 

## DB on EC2
* DB on EC2 is eithet one EC2 that has the whole stack in it (app + web + DB) or a standalone EC2 with the DB only. 
* Why it could be justifiable:
 1. Need to have access to DB OS. 
 2. Advanced DB tuning (DBROOT). 
 3. Vendor demands. 
 4. Need a DB or version that AWS dont provide. 
 5. Specific OS/DB combo that AWS dont provide. 
 6. Architecture AWS dont provide (replication/resilience). 
 7. Decision makers who just want it. 
* Why it is a overall bad idea:
 1. Admin overhead (need to manage ec2 and dbhost). 
 2. Need to manage backup and DR plan. 
 3. EC2 is single AZ. 
 4. Missing out on AWSDB features. 
 5. EC2 is on/off. No serverless or easy scaling. 
 6. Replication requires time, skill, monitoring etc. 
 7. Performances: AWS optimize its DB better. 

## RDS Architecture
* RDS is more like a DatabaseServerasAService (DBSaaS) because you can have multiple DB per DB server (instance).
* You have a choice for the engine: MySQL, MariaDB, PostgreSQL, Oracle, Microsoft SQL Server.
* Be careful, Amazon Aurora is a different product, it's not RDS, even though it has compatibility with some of the above engine.
* RDS is a managed service: no access to OS or SSH access (most of the time, there is actually an option we'll see later that can do this).
* When you create RDS instance, you first need to define a subnet group = what subnets RDS can use.
* If you have a multi-AZ deployment, RDS will have a primary and standby scheduled in different AZ.
* Between primary and standby, there is synchronous replication.
* You can also decide to have read replicas => they are used only for read, and can be scheduled in different regions or AZ.
* With read replicas, it's asynchronous replication from the standby.
* Backups and snapshots are made from the standy and pushed to an AWS-managed S3 bucket (that you don't see).
* RDS has its data backed by EBS volumes. It's a dedicated storage per instance.
* One RDS instance is at least 1 DB (can be multiple).

## RDS Multi AZ - Instance and Cluster modes
* For Instance mode:
    - A primary DB is deployed in one AZ, and is the HOT instance (R/W).
    - A StandBy DB (ONE only) is deployed in another AZ and is COLD (no R/W). There is synchronous (AZ) replication from Primary to Standby.
    - Primary and Standby are in the same region but different AZ.
    - Backups are made from the standby: no perf impact on the primary. Backups and snapshots are uploaded to S3 in multiple AZ.
    - A database CNAME DNS records point to the primary. In a failover scenario, the CNAME points to secondary. Can take 60~120s.
* For Cluster mode:
    - A writer instance is in on AZ. It does R/W. Other readers are deployed in different AZ.
    - Readers are RO. Data is replicated to them asynchronously (ReadReplica) from Writer. Data is commited (confirmed) when at least 1 reader finishes writing.
    - Cluster Endpoint points at the writer instance. Used for reads/writes and admin.
    - Reader endpoint points to any available read (reader + writer).
    - Instance endpoint points at a specific instance. Generally only used for debugging.
    - Hardware is much faster wrt instance mode. Readers are used for read, allowing read scaling. Replication is via transaction logs, it's more efficient. Harware used Graviton + local nvme ssd storage. Fast writes to localstorage that are then flushed to EBS.
    - Failover is faster: ~35s.

## RDS - Backups and Restore
* In RDS, there is two type of backups:
    - Automatic Backups
    - Snapshots
* Both of these are stored in S3 but in buckets managed by AWS and replicated in multiple AZ: you don't see those buckets in your account.
* Backups incur an IO pause, so it's noticeable from the instance, unless you're in a multi-AZ configuration where the operation is done on the standby.
* Snapshots are not automatic: they need to be triggered manually. First snapshot is a full copy, subsequent ones are incremental so technically quicker.
* Snapshots are managed by you so they also can last forever if you don't delete them manually.
* Automated backups occur once per day automatically. Similar to snapshots, the first one is full copy, subsequents are incremental.
* You define a backup window for the automated backup, or let AWS decide at random.
* Every 5 min a transaction logs is also written to S3. It allows you to replay transactions from an automated backup. So you have a 5 min max loss of potential data.
* Retention for automated backup is between 0 to 35 days (0 is disabled automated backup). 35 is hard limit, it will expire after that.
* RDS can replicate backups to another region. Charges apply and it's not the default. Need to enable manually.
* The restore operation creates a NEW RDS instance -> you need to update the endpoint on your application to the new RDS address.
* WIth automated backups, you have 5 min point in time recovery. But the restore isn't a fast operation and can take time to restore.

## Read Replicas
* The "new" Multi-AZ mode (cluster mode) is like the one Multi-AZ (instance mode) with the addition of read replicas.
* They are their own separated DB in a way that they have their own endpoint that needs to be communicated to the end application.
* Kept in sync with primary via asynchronous operation. Can be a small lag between primary and read replicas.
* ReadReplicas can be created in the same or in a different region as the primary. If different region, AWS takes care of the networking part.
* ReadReplicas allow read performance improvements:
    - Can have max 5X read-replicas per DB instance, each providing an additional instance of read performance.
    - ReadReplicas can have readReplicas but the async replication lag starts to become a problem.
    - Global perf improvements.
* ReadReplicas also bring RPO/RTO improvements:
    - Near 0 RPO (since data is replicated to them async)
    - In case of failure, ReadReplica can be promoted quickly to primary: low RTO.
    - Only good for failure, if data corruption, readReplica won't help much.
    - ReadReplicas are read only until promoted.
    - Global resilience improvement.

## RDS Security
* SSL/TLS(in transit) is available for RDS and can be made mandatory.
* RDS supports EBS volume encryption via KMS (at rest). This is handled at the HOST/EBS level (not DB level).
* Data keys generated by either AWS (aws-managed) or customer (custom kms) are used for encryption operations.
* Storage, logs, snapshots and replicas are encrypted. Once encryption is set, it cannot be undone.
* Some DB engine has other at rest capabilities, such as MicrosoftSQL and Oracle: they support TDE (TransparentDataEncryption).
* TDE is handled within the DB engine, not at the host or ebs level. Oracle also supports integration with CloudHSM so encryption keys are provided by the user and not by AWS -> completely opaque to AWS, much stronger key control.
* IAM can be used for authentication to the DB:
    - A local user is created on the DB and is configured to use AWS Authentication Token.
    - We attach special policy to an IAM identity (either IAM user or IAM role - which can be used by an EC2 instance as profile).
    - This policy maps above IAM identity with the local RDS user. It allows the identity to use the generate-db-auth-token operation.
    - This auth-token operation generates a 15 min valid token that can be used as user password for the local RDS user.
    - WARNING: once again, this method only allows AUTHENTICATION. AUTHORIZATION is still managed locally at the DB level based on the level of permission the local user has.

## RDS Custom
* Fills the gap between RDS and EC2 running a DB engine. Really niche scenario.
* Only works for MSSQL and Oracel for now.
* Can connect to the instance using SSH, RDP or Session Manager.

## Aurora Architecture (Provisioned)
* Aurora arch is very different from RDS.
* Aurora concept revoles around the cluster, which is a single primary instance and 0+ replicas.
* Aurora doesn't use local storage but a cluster volume (~CEPH).
* Aurora has faster provisioning and improved availability and performance.
* The cluster volume can have max 128TiB, and replicated accross 6 nodes in different AZs. The primary read and writes to the cluster volume while replicas only read.
* Storage nodes are all SSD based: low latency, high IOPS. It is billed on what's used and currently is balled on the most used (high watermark) even though it is changing.
* Storage which is freed up can be reused. Replicas can be added and removed without requiring storage provisioning.
* You can have up to 15 replicas. You have the cluster endpoint, pointing to the primary (RW) and the reader endpoints, load balancing between primary and readers (RO). You can also create custom endpoints.
* Aurora has no Free tier option. Doesn't support micro instance. Compute is hourly charge per second, 10 min minimum. Storage price is GB/month consumed, IO cost per request. However, 100X DB size in backups are included in pricing (ex. if DB uses 100G of data, you have 100G free backup).
* Backups in Aurora are similar to traditional RDS (snapshots, automatic backups). And a restore also creates a new cluster.
* New features include backtrack: allows in-place rewinds to a previous point in time (need to be enabled). No need to create new cluster.
* New features also include fast clones, which make a new database MUCH faster than copying all the data (leveraging copy-on-write).

## Aurora Serverless
* Aurora serverless is the equivalent of Fargate for ECS (while provisioned is the equivalent of managed nodes).
* You don't choose instance type, but scalable ACU: Aurora Capacity Units. You choose a min & max ACU per cluster.
* Cluster adjusts this number based on load. It can go to 0 and be paused.
* You pay the consumption billing per second basis. Storage wise, resilience is same as provisioned (6 copies accross AZs).
* As ACUs are dynamically added and replaced, they access the cluster volume the same way as provisioned.
* However, between the application and ACU, we have a proxy fleet managed by AWS that dynamically redirects cluster endpoint traffic to available ACUs.
* Aurora Serverless is great for:
    - Infrequently used application (since it can be paused).
    - New application and variable workloads/unpredictable.
    - Dev and test DB
    - Multi-tenant apps where the more the tenant use, the more you can bill him.

## Aurora Global Database
* Allow you to have RO replicas in other regions.
* Your main region is a traditional Aurora cluster. But with Global Database you can replicate the storage to up to 5 different regions and deploy up to 16 RO replicas in all of these regions.
* Storage replication time ~1s.
* Aurora Global DB usecase include:
    - Cross-region DR and BCP.
    - Global Read Scaling if consumers are international.
    - NO impact on DB perfs. Secondary regions can be promoted to RW.

## Aurora Multi-master
* In MM, all instances are R/W.
* No concept of endpoints, application connects directly either to one instance or multiple instance (no abstracted LB layer).
* When write is executed on one instance, data is written to cluster volume if a quorum of Aurora instance agrees (confirm that there is no conflicts).
* The writing node also liaise with other RW instance to update their local cache with this new write operation.
* In term of failover, if the app is designed to write to multiple DB at the same time, if one instance fails, the app can still write seamlessly or so to the other available nodes. It is almost fault-tolerant.
* In opposition, in traditional Aurora, if the RW fails, one read replica needs to be promoted to RW and it takes time and disrupts the app.

## RDS Proxy
* Fully managed DB proxy for RDS/Aurora
* Auto-scaling, highly available by default.
* Provides connection pooling - reduces DB load.
* Only accessible from a VPC.
* Accessed via Proxy Endpoint - no app changes required.
* Can enforce TLS/SSL.
* Can reduce failover time by ~60%.
* Abstracts failure away from the app.

## Database Migration Service (DMS)
* Managed database migration service. It runs using a replication instance.
* Uses source and destination endpoints which point at source and target databases.
* ONE ENDPOINT MUST BE ON AWS.
* source > | source endpoint > replication instance (with replication tasks) > destination endpoint | > target
* Migration jobs can be:
    - Full load (one-off migration of all data)
    - Full load + CDC (first full load, then change data capture wich replicates live data)
    - CDC only (if you want to do the full load using another tool, such as vendor tooling or snowball)
* If SRC and DST DB engines are not the same, can use SCT (schema conversion tool) to assist in the translation.
* SCT is separate from DMS but also provided by AWS and is used to convert one DB ENGINE to ANOTHER.
* It is NOT used when migrating between DB with the same engine.