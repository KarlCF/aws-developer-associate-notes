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
* 