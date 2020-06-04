## **AWS Fundamentals: RDS + Aurora + ElastiCache**

--- 

## RDS

* RDS Read Replicas for read scalability
  * Up to 5 Read Replicas
  * Within AZ, Cross AZ or Cross Region
  * Replication is ASYNC, so reads are eventually consistent
  * Replicas can be promoted to their own DB
  * Applications must update the connection string to leverage read replicas
* RDS Multi AZ (Disaster Recovery)
  * SYNC replication
  * One DNS name - automatic app failover to standby
  * Increases availability
  * Failover in case of loss of AZ, loss of network, instance or storage failure
  * No manual intervention in apps
  * Not used for scaling
* RDS Backups
  * Automatically enabled in RDS
  * Automated backups:
    * Daily full snapshot of the database
    * Capture transaction logs in real time => ability to restore to any point in time
    * 7 days retention (default) that can be increased to 35 days
  * DB Snapshots:
    * Manually triggered by the user
    * Retention of backup for as long as you want
* RDS Encryption
  * Encryption at rest capability with AWS KMS - AES-256 encryption
  * SSL certificates to encrypt data to RDS in flight
  * To enforce SSL:
    * PostgreeSQL: rds.force_ssl=1 in AWS RDS Console (Parameter Groups)
    * MySQL: Within the DB:
    GRANT USAGE ON \*.\* TO 'mysqluser'@'%' **REQUIRE SSL**;
  * To connect using SSL:
    * Provide the SSL Trust certificate (download from AWS)
    * Provide SSL options when connecting to database
* RDS Security
  * RDS databases are usually deployed within a private subnet, not in a public one
  * RDS Security works by leveraging security groups - it controls who can communicate with RDS
  * IAM policies help control who can manage AWS RDS
  * Traditional Username and Password can be used to login to the database
  * IAM users can now be used too (MySQL / Aurora - New feature)

---

## Aurora

* Aurora is proprietary technology from AWS (not open sourced)
* Postgres and MySQL are both supported as Aurora DB (your drivers will work as if Aurora was a Postgres or MySQL database)
* Aurora is "AWS cloud optimized" and claims 5x performance improvement over MySql on RDS and 3x over Postgres on RDS
* Aurora's storage automatically grows of increments of 10GB, up to 64 TB
* Can have 15 replicas, while MySQL has 5, and the replication is faster (below 10 ms replica lag)
* Failover in Aurora is instantaneous. It's High Available natively
* Costs more than RDS (about 20%), but more efficient
* **Aurora High Availability**
  * 6 copies of your data accross 3 AZ:
    * 4 copies out of 6 needed for writes
    * 3 copies out of 6 needed for reads
    * Self Healing with peer-to-peer replication
    * Storage is striped across hundreds of volumes
  * One Aurora instance takes writes (Master)
  * Automated failover for master in less than 30 seconds
  * Master + up to 15 Aurora Read Replicas serve reads
  * Support for Cross Region Replication
* **Aurora DB Cluster**
  * Writer Endpoint: points to the master automatically (even if changes on failover) 
  * Reader Endpoint: Connection load balancing for read.
* **Features of Aurora**
  * Automatic fail-over
  * Backup and Recover
  * Isolation and security
  * Industry compliance
  * Push-button scaling
  * Automated Patching with Zero Downtime
  * Advanced Monitoring
  * Routine Maintenance
  * Backtrack: restore data at any point of time without using backups
* **Aurora Security**
  * **See RDS Security listed above**
* **Aurora Serverless**
  * Automated database instantiation and auto-scaling based on actual usage
  * Good for: intermittent, infrequent or unpredictable workload
  * No capacity planning needed
  * **Pay per second**: can be more cost-effective
* **Global Aurora**
  * Aurora Cross Region Read Replicas:
    * Useful for disaster recovery
    * Simple to put in place
  * Aurora Global Database (recommended):
    * 1 Primary Region (read / write)
    * Up to 5 secondary (read-only) regions, replication lag is less than 1 second
    * Up to 16 Read Replicas per secondary region
    * Help for decreasing latency
    * Promoting another region (**DR**) has an RTO of < 1 minute

---

## ElastiCache

* Managed Redis or Memcached
* Caches are in-memory databases with high performance, low latency
* Reduce load off of databases for read intensive workloads
* Helps make your application stateless
* Write Scaling using sharding
* Read Scaling using Read Replicas
* Multi AZ with Failover Capability
* AWS manages OS maintenance / patching optimizations, setup, configuration, monitoring, failure recovery and backups
  * ElastiCache Solution Architecture - DB Cache
  * Applications queries ElastiCache, if not available, gets the data from RDS and writes it back to ElastiCache, making future reads quicker
  * Helps relieve load in RDS
  * Cache must have an invalidation strategy to make sure that only the latest data is used there (is done in the application side)
* Redis Overview
  * Is a in-memory key-value store
  * Low latency (under ms)
  * Cache survive reboots by default (it's called persistence)
  * Great to host
    * User sessions
    * Leaderboard (for gaming)
    * Distributed states
    * Relieve pressure on databases (such as RDS)
    * Pub / Sub capability for messaging
  * Multi AZ with Automatic Failover for disaster recovery if you don't want to lose your cache data
  * Support for Read Replicas
* Memcached Overview
  * Is an in-memory object store
  * Cache doesn't survive reboots
  * Use cases:
    * Quick retrieval of objects from memory
    * Cache often accessed objects
  * ***Non exam notes:*** Overall, Redis has better feature sets and has grown in popularity.
* ElastiCache Patterns
  * ElastiCache is helpful for read-heavy application workloads
    * Social networks, gaming, media sharing, Q&A portals
  * *compute-intensive* workloads (recommendation engines)
  * There are two pattern / cache strategies for ElastiCache (may be different based on the kind of application you have)
    * Lazy Loading - Load only when necessary
      * Pros:
        * Only requested data is cached (cache isn't filled with unused data)
        * Node failures aren't fatal (just increased latency on to "warm" the cache)
      * Cons:
        * Cache miss results in 3 round trips, noticeable delay for that request
        * Stale data: data can be updated in the database and outdated in the cache (one way you can solve this by applying a ttl in the cache)
    * Write Through - Add or Update cache whenever database is updated
      * Pros:
        * Data is never stale in the cache
        * Write penalty vs Read penalty (each write requires 2 calls)
      * Cons:
        * Missing Data until it is added / updated in the DB. Can be mitigated by implementing Lazy Loading strategy together
        * Cache churn - a lot of data that is rarely read, causing the cache size to be sometimes as big as the DB