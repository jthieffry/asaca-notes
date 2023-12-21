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