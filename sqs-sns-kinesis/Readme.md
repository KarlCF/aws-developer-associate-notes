## **AWS Integration & Messaging: SQS, SNS & Kinesis**

### Section introduction

* When we start deploying multiple applications, they will inevitably need to communicate with one another
* There are two patterns of application communicaton
  * Synchronous commucation (application to application)
    * Can be problematic if there are sudden spikes of traffic
  * Asynchronous / Event based (application to queue to application)
    * Better to handle sudden spikes:
      * Using SQS: Queue model
      * Using SNS: pub/sub model
      * using Kinesis: real-time streaming model

### AWS SQS (Simple Queue Service)

* AWS SQS - Standard Queue
  * Oldest offering (over 10 y.o.)
  * Fully managed
  * Scales from 1 message per second to 10000s per second
  * Default retention of messages: 4 days, maximum of 14 days
  * No limit to how many messages can be in the queue
  * Low latency (<10ms on publish and receive)
  * Horizontal scaling in terms of number of consumers
  * Can have duplicate messages (at least once delivery, occasionally)
  * Can have out of order messages (best effort ordering)
  * Limitation of 256KB per message sent
* AWS SQS - Delay Queue
  * Delay a message (consumers don't see it immediately) up to 15 minutes
  * Default is 0 seconds (message available right away)
  * Can set a default at queue level
  * Can override the default using the DelaySeconds parameter
* SQS - Producing Messages
  * Define body (string, up to 256 KB)
  * Add message attributes (metadata - optional)
  * Provide Delay Delivery (optional)
  * After the message is sent, you receive:
    * Message identifier
    * MD5 hash of the body
* SQS - Consuming messages
* Consumers:
  * Poll SQS for messages (receive up to 10 messages at a time)
  * Process the message within the visibility timeout
  * Delete the message using the message ID & receipt handle
* SQS - Visibility timeout
  * When a consumer polls a message from a queue, the message is "invisibile" to other consumers for a defined period, the **Visibility Timeout**
    * Between 0 seconds and 12 hours (default is 30s)
    * If too high (15 minutes) and consumer fails to process the message, you must wait a long time before processing the message again
    * If too low (30s) and consumer needs to process the message (2m), another consumer will receive the message and it will be processed more than once
  * **ChangeMessageVisibility** API to change the visibility while processing a message
  * **DeleteMessage** API to tell SQS the message was successfully processed
* AWS SQS - Dead Letter Queue
  * If a consumer fails to process a message within the Visibility Timeout, the message goes back to the queue
  * We can set a threshold of how many times a message can go back to the queue - it's called a "redrive policy"
  * After the message threshold is exceeded, the message goes into a dead letter queue (DLQ)
  * We have to create a DLQ first and then designate it dead letter queue
  * Make sure to process the messages in the DLQ before they expire
* AWS SQS - Long Polling
  * When a consumer requests message from the queue, it can optionally "wait" for messages to arrive if there are none in the queue - this is called Long Polling
  * **LongPolling decreasese the number of API calls made to SQS while increasing the efficiency and latency of your application**
  * The wait time can be between 1s to 20s (20s preferable)
  * Long Polling is preferable to Short Polling
  * Long polling can be enabled at the queue level or at the API level using **WaitTimeSeconds**

### AWS SQS - FIFO Queue

* Newer offering - Not available in all regions
* Name of the queue must end in .fifo
* Lower throughput (up to 3000 per second with batching, 300/s without)
* Messages are processed in order by the consumer
* Messages are sent exactly once
* No per message delay (only once per queue delay)
* SQS FIFO - Features
  * **Deduplication**: (not send the same message twice)
    * Provide a MessageDeduplicationId with your message
    * De-duplication interval is 5 minutes, so if in 5 minutes there are repeated messages 
    * Content based duplication: the MessageDeduplicationId is generated as the SHA-256 of the message body (not the attributes)
  * **Sequencing**:
    * To ensure strict ordering between messages, specify a **MessageGroupId**
    * Messages with different Group ID may be received out of order
    * E.G. to order messages for a user, you could use the "user_id" as a group id
    * Messages with the same Group ID are delivered to one consumer at a time
* SQS Extended Client
  * Message size limit is 256KB, how to send large messages? Using the SQS Extended Client (Java Library)
  * The producer sends the content to a S3 bucket and a small metadata message to the Queue, the consumer then receives the metadata message, and pulls the larger file from S3.(Access to the S3 bucket is necessary)
* AWS SQS Security
  * Encryption in flight using the HTTPS endpoint
  * Can enable SSE (Server Side Encryption) using KMS
    * Can set the CMK (Customer Master Key) we want to use
    * Can set the data key reuse period (between 1 minute and 24 hours)
      * Lower and KMS API will be used often (higher security, higher cost)
      * Higher and KMS will be used less (Lower Cost, lower security)
    * SSE only encrypts the body, not the metadata (message ID, timestamp, attributes)
  * IAM policy must allow the usage of SQS
  * SQS queue access policy
    * Finer grained control over IP
    * Control over the time the requests come in
  * No VPC Endpoint, must have internet access to access SQS
* SQS - Must know API
  * CreateQueue, DeleteQueue
  * PurgeQueue: delete all the messages in queue
  * SendMessage, ReceiveMessage, DeleteMessage
  * ChangeMessageVisibility: change the timeout
  * Batch APIs for SendMessage, DeleteMessage, ChangeMessageVisibility, helps decrease your costs (BatchSendMessage, BatchDeleteMessage, etc...)
  * No BatchReceiveMessage, you can receive up to 10 messages

### AWS SNS

* The "event producer" only sends message to one SNS topic
* As many "event receivers" (subscriptions) as we want to listen to the SNS topic notifications
* Each subscriber to the topic will get all the messages(new feature to filter messages, might not be present in the exam)
* Up to 10000000(10 million) subscriptions per topic
* 100000 topics limit
* Subscribers can be:
  * SQS
  * HTTP / HTTPS (with delivery retries - how many times)
  * Lambda
  * Emails
  * SNS messages
  * Mobile Notifications
* SNS integrates with several Amazon products:
  * Some services can send data directly to SNS for notifications
  * CloudWatch (for alarms)
  * ASG notifications
  * Amazon S3 (on bucket events)
  * CloudFormation (upon stage changes => failed to build, etc)
  * Etc...
* AWS SNS - How to publish
  * Topic Publish (within your AWS Server - using the SDK)
    * Create a topic
    * Create a subscription (or many)
    * Publish to the topic
  * Direct Publish (for mobile apps SDK)
    * Create a platform application
    * Create a platform endpoint
    * Publish to the platform endpoint
    * Works with Google GCM, Apple APNS, Amazon ADM...
* SNS + SQS: Fan Out
  * Push once in SNS, receive in many SQS
  * Fully decoupled
  * No data loss
  * Ability to add receivers of data later
  * SQS allows for delayed processing (SNS needs instant processing)
  * SQS allows for retries of work
  * May have many workers on one queue and one worker on the other queue

### AWS Kinesis

* **Kinesis** is a managed alternative to Apache Kafka
* Great for:
  * Application logs, metrics, IoT, clickstreams
  * "Real-time" big data
  * Streaming processing frameworks (Spark, NiFi, etc...)
  * Data is automatically replicated to 3 AZ
* **Kinesis Streams**: low latency streaming ingest at scale
  * Streams are divided in ordered Shards / Partitions (scaling up addings shards)
  * **Kinesis Stream Shards**
    * One stream is made of many different shards
    * 1MB/s or 1000messages/s at write **PER SHARD**
    * 2MB/s at read **PER SHARD**
    * Billing is per shard provisioned, can have as many shards as you want
    * Batching available or per message calls
    * The number of shards can evolve over time (reshard / merge)
    * **Records are ordered per shard**
  * Data retention is 1 day by default, can go up to 7 days
  * Ability to reprocess / replay data
  * Multiple applications can consume the same stream
  * Real-time processing with scale of throughput
  * Once data is inserted in Kinesis, it can't be deleted (immutability)
* **Kinesis Analytics**: perform real-time analytics on streams using SQL
* **Kinesis Firehose**: load streams into S3, Redshift, ElasticSearch...
* AWS Kinesis API
  * **AWS Kinesis API - Put Records**
    * PutRecord API + Partition key that gets hashed
    * The same key goes to the same partition (helps with ordering for a specific key)
    * Choose a partition key that is highly distributed ( helps prevent "hot partition")
      * user_id if many users
      * **Not** country_id if 90% of the users are in one country
    * Use Batching with PutRecords to reduce costs and increase throughput
    * **AWS Kinesis API - ProvisionedThroughputExceeded** if we go over the limits
    * Can use CLI, AWS SDK, or producer libraries from various frameworks
  * **AWS Kinesis API - Exceptions**
    * ProvisionedThroughputExceeded Exceptions
      * Happens when sending more data (exceeding MB/s or TPS for any shard)
      * Make sure you don't have a hot shard (such as your partition key is bad and too much data goes to that partition)
      * Solution:
        * Retries with backoff
        * Increase shards (scaling)
        * Ensure your partition key is a good one
  * **AWS Kinesis API - Consumers**
    * Can use a normal consumer (CLI, SDK, etc...)
    * Can use Kinesis Client Library (in Java, Node, Python, Ruby, .Net)
      * KCL uses DynamoDB to checkpoint offsets
      * KCL uses DynamoDB to track other workers and share the work amongst shards

### Kinesis KCL

* Kinesis Client Library (KCL) is a Java library that helps read record for a Kinesis Streams with distributed applications sharing the read workload
* **Rule: each shard is to be read by only one KCL instance (but a KCL instance can read more than one Shard)**
* Means 4 shards =  max 4 KCL instances
* Progress is checkpointed into DynamoDB
* KCL can run on EC2, Elastic Beanstalk, on Premise Application
* **Records are read in order at the shard level**

### Kinesis Security

* Control access / authorization using IAM policies
* Encryption in flight using HTTPS endpoints
* Possibility to encrypt / decrypt data client side (harder)
* VPC Endpoints available for Kinesis to access within VPC

### AWS Kinesis Data Analytics

* Perform real-time analytics on Kinesis Streams using SQL
* Kinesis Data Analytics:
  * Auto Scaling
  * Managed: no servers to provision
  * Continuous: real time
* Pay for actual consumption rate
* Can create streams out of the real-time queries

### AWS Kinesis Firehose

* Fully Managed Service, no administration
* Near Real Time (60s latency)
* Load data into Redshift / Amazon S3 / ElasticSearch / Splunk 
* Automatic scaling
* Support many data format (pay for conversion)
* Pay for the amount of data going through Firehose)