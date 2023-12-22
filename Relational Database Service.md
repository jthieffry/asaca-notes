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