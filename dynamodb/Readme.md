## DynamodDB

### NoSQL Databases

* NoSQL datagbases are non-relational databases and are **distributed**
* NoSQL databases include MongoDB, DynamoDB, etc.
* NoSQL databases do nbot support join
* All the data that is needed for a query is present in one row
* NoSQL databases don't perform aggregations such as "SUM"
* **NoSQL databases can scale horizontally**
* There's no "right" or "wrong" for NoSQL vs SQL, they just require to model the data differently and think about user queries differently

### DynamoDB

* Fully Managed, Highly available with replication across 3 AZ
* NoSQL database - not a relational database
* Scales to massive workloads, distributed database
* Milions of requests per seconds, trillions of row, 100s of TB of storage
* Fast and consistent in performance (low latency on retrieval)
* Integrated with IAM for security, authorization and administration
* Enables event driven programming with DynamoDB Streams
* Low cost and auto scaling capabilities

#### DynamoDB - Basics

* DynamoDB is made of **tables**
* Each table has a **primary key** (must be decided at creation time)
* Each table can have infinite number of items (=row)
* Each item has **attributes** ( can be added over time - can be null)
* Maximum size of a item is 400KB
* Data types supported are:
  * Scalar Types: String, Number, Binary, Boolean, Null
  * Document Types: List, Map
  * Set Types: String Set, Number Set, Binary Set

#### DynamoDB - Primary Keys

* Option 1: Partition Key only (HASH)
  * Partition Key must be unique for each item
  * Partition key must be "diverse" so that the data is distributed
  * **Example**: user_id for a users table
* Option 2: Partition Key + Sort Key
  * The combination must be unique
  * Data is grouped by partition key
  * Sort key == range key
  * **Example**: users-games table
    * user_id for the partition key
    * game_id for the sort key

### DynamoDB - Provisioned Throughput

* Table must have provisioned read and write capacity units
* **Read Capacity Units (RCU)**: throughput for reads
* **Write Capacity Units (WCU)**: throughput for writes
* Option to setup auto-scaling of throughput to meet demand
* Throughput can be exceeded temporarily using "burst credit"
* If burst credit are empty, you'll get a "ProvisionedThroughputException"
* It's then advised to do an exponential back-off retry

#### DynamoDB - WCU

* One write capacity units represent one write per second for an item up to 1KB in size
* If the items are larger than 1KB, more WCU are consumed
* **Example 1:** to write 10 objects per second of 2Kb each > 2 * 10 = **20 WCU needed**
* **Example 2:** to write 6 objects per second of 4.5 KB each > 6 * 5 (4.5 gets rounded up) = **30 WCU needed**
* **Example 3:** to write 120 objects per minute of 2 KB each > (120/60) * 2 = **4 WCU needed**

#### DynamoDB Strongly Consistent Read vs Eventually Consistent Read

* **Eventually Consistent Read:** If we read just after write, it's possible we'll get unexpected response because of replication
* **Strongly Consistent Read:** If we read just after a write, we will get the correct data
* **By default:** DynamoDB uses Eventually Consistent Reads, but GetItem, Query & Scan provide a "ConsistentRead" parameter you can set to True

#### DynamoDB - RCU

* One RCU represents one strongly consistent read per second, or two eventually consistent reads per second, for an item up to 4 KB in size.
* If the items are larger than 4 KB, more RCU are consumed
* **Example1:** 10 strongly consistent reads per second of 4 KB each > 10 * (4 / 4) = **10 RCU needed** 
* **Example2:** 16 eventually consistent reads per second of 12 KB each > (16 / 2) * (12/4) = **24 RCU needed**
* **Example3:** 10 strongly consistent reads per seconds of 6 KB each > 10 * (8 / 4) (It needs to be rounded up from 6 to 8) = **20 RCU needed**

#### DynamoDB - Partitions Internal

* Data is divided in partitions
* Partition keys go thorugh a hashing algorithm to know which partition they go to
* To compute the number of partitions:
  * By capacity: (TOTAL RCU / 3000) + (TOTAL WCU / 1000)
  * By size: Total Size / 10 GB
  * Total partitions = CEILING(MAX(Capacity, Size))
* **WCU and RCU are spread evently between partitions**

#### DynamoDB Throttling

* If we exceed our RCU or WCU, we get **ProvisionedThroughputExceededExceptions**
* Reasons:
  * Hot keys: one partition key is being read too many times (popular items)
  * Hot Partitions
  * Very large items: RCU and WCU depends on size of items
* Solutions:
  * Exponential back-off when exception is encountered (already in SDK)
  * Distribute partition keys as much as possible
  * If RCU issue, we can use DynamoDB Accelerator (DAX)

### DynamoDB Basic APIs

#### DynamoDB - Writing Data

* **PutItem** - Write data to DynamoDB (create data or full replace)
  * Consume WCU
* **UpdateItem** - Update data in DynamoDB (partial update of attributes)
  * Possibility to use Atomic Counters and increase them
* **Conditional Writes**:
  * Accept a write / update only if conditions are respected, otherwise reject
  * Helps with concurrent access to items
  * No performance impact

#### DynamoDB - Deleting Data

* **DeleteItem**
  * Delete an individual row
  * Ability to perform a conditional delete
* **DeleteTable**
  * Delete a whole table and all its items
  * Much quicker deletion than calling DeleteItem on all items

#### DynamoDB - Batching Writes

* **BatchWriteItem**
  * Up to 25 **PutItem** and / or **DeleteItem** in one call
  * Up to 16 MB of data written
  * Up to 400 KB of data per item
* Batching allows you to save in latency by reducing the number of API calls done against DynamoDB
* Operations are done in paralllel for better efficiency
* It's possible for part of a batch to fail, in which case we have to retry the failed items (using exponential back-off algorithm)

#### DynamoDB - Reading Data

* **GetItem**:
  * Read based on Primary key
  * Primary Key = HASH or HASH-RANGE
  * Eventually consistent read by default
  * Option to use strongly consistent reads (more RCU - might take longer)
  * **ProjectionExpression** can be specified to include only certain attributes
* **BatchGetItem**:
  * Up to 100 items
  * Up to 16 MB of data
  * Items are retrieved in parallel to minimize latency

#### DynamoDB - Query

* **Query** returns items based on:
  * PartitionKey value (**must be = operator**)
  * SortKey value (=, <, <=, >, >=, Between, Begin) - optional
  * FilterExpression to further filter (client side filtering)
  * **Returns:**
    * Up to 1 MB of Data
    * Or number of items specified in **Limit**
* Able to do pagination on the results
* Can query table, a local secondary index, or a global secondary index

#### DynamoDB - Scan

* **Scan** the entire table and then filter out data (inefficient)
* Returns up to 1 MB of data - use pagination to keep reading
* Consumes a lot of RCU
* Limit impact using Limit or reduce the size of the result and pause
* For faster performance, use **parallel scans**:
  * Multiple instances scan multiple partitions at the same time
  * Increases the throughput and RCU consumed
  * Limit the impact of parallel scans just like you would for Scans
* Can use a **ProjectionExpression + FilterExpression** (no change to RCU)

### DynamoDB - LSI (Local Secondary Index)

* Alternate range keys for your table, **local to the hash key**
* Up to five local secondary indexes per table
* The sort key consists of exacly one scalar attribute
* The attribute that you choose must be scalar String, Number, or Binary
* **LSI must be defined at table creation time**

### DynamoDB - GSI (Global Secondary Index)

* To speed-up queries on non-key attributes, use a Global Secondary Index
* GSI = partition key + optional sort key
* The index is a new "table" and we can project attributes on it
  * The partition key and sort key of the original table are always projected (KEYS_ONLY)
  * Can specify extra attributes to protect (INCLUDE)
  * Can use all attributes from main table (ALL)
* Must define RCU / WCU for the index
* **Possibility to add / modify GSI (not LSI)**

### DynamoDB Indexes and Throttling

* GSI: 
  * **If the writes are throttled on the GSI, then the main table will be throttled**
  * Even if the WCU on the main tables are fine
  * Choose your GSI partition key carefully!
  * Assign your WCU capacity carefully
* LSI:
  * Uses the WCU and RCU of the main table
  * No special throttling considerations

### DynamoDB Concurrency

* DynamoDB has a feature called "Conditional Update / Delete"
* That means that you can ensure an item hasn't changed before altering it
* That makes DynamoDB an **optimistic locking / concurrency** database

### DynamoDB - DAX

* DAX = DynamoDB Accelerator
* Seamless cache for DynamoDB, no application re-write
* Writes go through DAX to DynamoDB
* Micro seconds latency for cached reads & queries
* Solves the Hot Key problem (too many reads)
* 5 minutes TTL for cache by default
* Up to 10 nodes in the cluster
* Multi AZ (3 nodes minimum recommended for production)
* Secure (Encryption at rest with KMS, VPC, IAM, CloudTrail...)

#### DynamoDB - DAX vs ElastiCache

* Dax is better suited for individual objects cache, Query / Scan cache
* ElastiCache is more suited more advanced use, like for data Aggregation Results

### DynamoDB Streams

* Changes in DynamoDB (Create, Update, Delete) can end up in a DynamoDB Stream
* This stream can be read by AWS Lambda & EC2 instances, and we can then:
  * React to changes in real time (welcome email to new users)
  * Analytics
  * Create derivative tables / views
  * Insert into ElasticSearch
* Could implement cross region replication using Streams
* Stream has 24 hours of data retention
* Choose the information that will be written to the stream whenever the data in the table is modified:
  * KEYS_ONLY - Only the key attributes of the modified item
  * NEW_IMAGE - The entire item, as it appears after it was modified
  * OLD_IMAGE - The entire item, as it appeared before it was modified
  * NEW_AND_OLD_IMAGES - Both the new and the old images of the item
* DynamoDB Streams are made of shards, just like Kinesis Data Streams
* You don't provision shards, this is automated by AWS
* **Records are not retroactively populated in a stream after enabling it**

#### DynamoDB Streams & Lambda

* You need to define an **Event Source Mapping** to read from a DynamoDB Streams
* You need to ensure the Lambda function has the appropriate permissions
* **Your lambda function is invoked synchronously**

### DynamoDB - TTL

* TTL = automatically delete an item after an expiry date / time
* TTL is provided at no extra cost, deletions do not use WCU / RCU
* TTL is a background task operated by the DynamoDB service itself
* Helps reduce storage and manage the table size over time
* Helps adhere to regulatory norms
* TTL is enabled per row (you define a TTL column, and add a date there)
* DynamoDB typically deletes expired items within 48 hours of expirations
* Deleted items due to TTL are also deleted in GSI / LSI
* DynamoDB Streams can help recover expired items

### DynamoDB - CLI

* --projection-expression : attributes to retrieve
* --filter-expressions : filter results
* General CLI pagination options including DynamoDB / S3:
  * Optimization:
    * --page-size : full dataset is still received but each API call will request less data (helps avoid timeouts)
  * Pagination:
    * --max-items : max number of results return by the CLI. Returns NextToken
    * --starting-token : specify the last received NextToken to keep on reading

### DynamoDB Transactions

* Transaction = Ability to Create / Update / Delete multiple rows in different tables at the same time
* It's an "all or nothing" type of operation
* Write Modes: Standard, Transactional
* Read Modes: Eventual Consistency, Strong Consistency, Transactional
* Consume 2x of WCU / RCU
* **A transaction is a write to both tables, or none**

### DynamoDB as Session State Cache

* It's common to use DynamoDB to store session state

#### DynamoDB vs ElastiCache

* ElasticCache is in-memory, but DynamoDB is serveless
* Both are key/value stores

#### DynamoDB vs EFS:

* EFS must be attached to EC2 instances as a network drive

#### DynamoDB vs EBS & Instance Store

* EBS & Instance Store can only be used for local caching, not shared caching

#### DynamoDB vs S3

* S3 has higher latency, and not meant for small objects

### DynamoDB Write Sharding

* When designing an application with few partitions keys, we might run into partitions issues
* Solution: add a suffix (either random suffix or calculated)

### DynamoDB - Write Types

#### Conccurent Write

* Last write overrides the first (when both are ocurring in a short period)

#### Conditional Writes

* Usually first write will override the condition on the second write and it will fail (fixes the problem of concurrent writes)

#### Atomic Writes 

* Works with increases or decreases of value, both writes would work

#### Batch Writes

* Write / Update many items at a time


### DynamoDB - Patterns with S3

#### DynamoDB - Large Objects Pattern

* DynamoDB has a limit of 400 KB per item, so when storing Large files you can send the file metadata from S3 to DynamoDB, then request that data on DynamoDB and use it to retrieve the file from S3

#### DynamoDB - Indexing S3 objects metadata

* A write on Amazon S3, triggers a Lambda, which will write object metadata on a DynamoDB table

### DynamoDB Operations

* **Table Cleanup**:
  * Option 1: Scan + Delete > Very slow, expensive, consumes RCU & WCU
  * Option 2: Drop Table + Recreate Table > fast, cheap, efficient
* **Copying a DynamoDB Table**:
  * Option 1: Use AWS DataPipeline (uses EMR)
  * Option 2: Create a backup and restore the backup into a new table name (can take some time)
  * Option 3: Scan + Write > write own code

### DynamoDB Security

* VPC Endpoints available to access DynamoDB without internet
* Access fully controled by IAM
* Encryption at rest using KMS
* Encryption in transit using SSL / TLS

### DynamoDB Backup and Restore feature

* Point in time restore like RDS
* No performance impact

### DynamoDB Global Tables

* Multi region, fully repplicated, high performance

#### Amazon DMS can be used to migrate to DynamoDB (from Mongo, Oracle, MySql, S3, etc...) 

#### You can launch a local DynamoDB on your computer for development purposes