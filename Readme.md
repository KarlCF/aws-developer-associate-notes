# AWS Developer Exam Notes

These are my notes while studying for the AWS Certified Developer Exam, I used the Stephane Mareek udemy course:
 <https://www.udemy.com/course/aws-certified-developer-associate-dva-c01/>

The objective of writing along with the course and making it public is to help others learn and follow through the same content.

To do:

1. Review notes after finishing the course
2. Add links to segments and AWS Faqs links to each Service
3. Add notes after taking the practice exam and certification exam

## AWS Fundamentals Part 1: IAM + EC2

## **EC2**

### **SSH into linux: chmod 0400 ec2keypair.pem**

* ssh -i ec2keypair.pem ec2-user@255.255.255.255
* It's good to maintain one separate security group for SSH access (best practice)
* Limit of 5 Elastic IP's per account (if you need to increase, just contact AWS)
* Try to avoid using Elastic IP:
  * They often reflect poor architectural decisions
  * Instead, use a random public IP and register a DNS name on it
  * Use a load balancer and no public IP (best pattern)
* Bootstrapping = EC2 User Data (only runs once at the first start)
* EC2 User Data is automaticaly run with the sudo command

## **AWS Fundamentals Part 2: ELB + ASG + EBS**

## Load balancers

* Application Load Balancer (v2)
  * Load balancing multiple HTTP application accros machines (target groups)
  * Load balancing to multiple applications on the same machine (ex: containers)
  * Load balancing based on route in URL
  * Load balancing based on hostname URL
  * Has a port mapping feature to redirect to a dynamic port
  * Great for micro services & container-based application (ex: Docker & Amazon ECS)

  * ### **Good to know:**

  * Stickiness can be enabled at the target group level
    * Same requests to the same instance
    * Stickiness is directly generated by the ALB (not the application)
  * ALB supports HTTP/HTTPS & Websockets protocols
  * The application servers don't see the IP of the client directly
    * The true IP of the client is inserted in the header X-Forwarded-For
    * We can also get Port (X-Forwarded-Port) and proto (X-Forwarded-Proto)

* Network Load Balancer (v2)
  * Forward TCP traffict to your instances
  * Handle millions of requests per seconds
  * Support for static IP or elastic IP
  * Less latency: ~ 100 ms (vs 400 ms for ALB)
  * Mostly used for extreme performance, should not be the default load balancer you chose

### **Good to know:**

* Classic Load Balancers are Depreacted
  * ALB for HTTP / HTTPs & Websocket
  * Network Load Balancer for TCP
* CLB and ALB support SSL certificates and provide SSL termination
* All Load Balancers have health check capability
* ALB can route based on hostname / path
* ALB is a great fit with ECS (Docker)
* Any Load Balancer (CLB, ALB, NLB) has a static host name. Do not resolve and use underlying IP
* LBs can scale but not instantaneously - contact AWS beforehand for a "warm-up"
* NLB directly see the client IP
* 4xx errors are client induced errors
* 5xx errors are application induced errors
  * Load Balancer Errors 503 means at capacity or no registered target
* If the LB can't connect to your application, check your security groups!
* Monitoring
  * ELB access logs will log all access requests (so you can debug per request)
  * CloudWatch Metrics will give you aggregate statistics (ex: connections count)

## Auto Scaling Group

* Handles scaling (in and out) to a minimum and maximum configured
* Automatically register new instances to a load balancer
* It is possible to scale an ASG based on CloudWatch alarms

### ASGs have the following attributes

* A launch configuration
  * AMI + Instance type
  * EC2 User data
  * EBS Volumes
  * Security Groups
  * SSH Key Pair
* Min Size / Max Size / Initial Capacity
* Network + Subnets Information
* Load Balancer Information (Or target group info)
* Scaling Policies

### Auto Scaling new Rules

* It is now possible to define more refined auto scale rules that are directly managed by EC2
  * Target Average CPU Usage
  * Number of requests on the ELB per instance
  * Average Network In
  * Average Network Out
* These rules are easier to set up and can make more sense

### **Good to Know:**

* Scaling policies can be any CloudWatch metrics (Native or custom) and even based on schedule
* ASGs use Launch configurations and you update an ASG by providing a new launch configuration
* IAM roles attached to an ASG will get assigned to EC2 instances
* Instances marked as Unhealthy are terminated by the ASG and replaced

___

## EBS Volumes

* It's a network drive
  * It uses network to communicate the instance, so there might be some latency
  * It can be detached from one instance and attached to another quickly
* It's locked to an **AZ**
  * An EBS from AZ A cannot be attached to AZ B, to move accros you first need to snapshot it
* Have provisioned capacity (in size and IOPS) and you are billed accordingly (not for the use, but for the provision by itself)
* EBS Volumes come in 4 types:
  * GP2 (SSD): General purpose SSD volume that balances price and performance, various workloads
  * IO1 (SSD): Highest-performance SSD volume for mission-critical low-latency or high-throughput workloads
  * ST1 (HDD): Low cost HDD volume designed for frequently accessed, throughput intensive workloads
  * SC1 (HDD): Lowest cost HDD volume designed for less frequently accessed workloads
* EBS are characterized in Size | Throughput | IOPS
* As of Feb. 2017, you can resize EBS volumes
  * Size (any volume type)
  * IOPS (only IO1)
  * After resizing an EBS volume, you need to repartition your driver on the EC2 instance

* ### EBS Encryption

* When you create an encrypted EBS volume, you'll get the following:
  * Data at rest is encrypted inside the volume
  * All the data in flight (moving between instance and volume) is encrypted
  * All Snapshots are encrypted
  * All volumes created from snapshots are also encrypted
  * Encryption and decryption are transparent (no user interaction is needed)
  * Encryption has minimal iompact on latency
  * EBS Encryption utilizes key from KMS (AES-256)
  * Copying an unencrypted snapshot allows encryption

* ### EBS Snapshots

* EBS can be backed up using snapshots
* Snapshots only take the used space on your EBS volume
* Snapshots are used for:
  * Backups (disaster recovery)
  * Volume migration
    * Resizing a volume down
    * Changing the volume type
    * Encrypt a volume

## **AWS Fundamentals Part 3: Route 53 + ElastiCache + VPC**

## Route 53

* Most common records are:
  * A: URL to IPv4
  * AAAA: URL to IPv6
  * CNAME: URL to URL
  * Alias: URL to AWS resource
* Route53 can use:
  * Public domain names you own
  * Private domain names that can be resolved by your VPCs. (ex: application1.company.internal)
* Route53 has advanced features such as:
  * Load Balancing (Through DNS - also called client load balancing)
  * Health checks (although limited...)
  * Routing policy: simple, failover, geolocation, geoproximity, latency, weighted
* Prefer Alias over CNAME for AWS resources (bettter performance)

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
* RDS vs Aurora
  * Aurora is proprietary technology from AWS (not open sourced)
  * Postgres and MySQL are both supported as Aurora DB (your drivers will work as if Aurora was a Postgres or MySQL database)
  * Aurora is "AWS cloud optimized" and claims 5x performance improvement over MySql on RDS and 3x over Postgres on RDS
  * Aurora's storage automatically grows of increments of 10GB, up to 64 TB
  * Can have 15 replicas, while MySQL has 5, and the replication is faster (below 10 ms replica lag)
  * Failover in Aurora is instantaneous. It's High Available natively
  * Costs more than RDS (about 20%), but more efficient

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

## VPC

* Public Subnets usually contains:
  * Load Balancers
  * Static Websites
  * Files
  * Public Authentication Layers
* Private Subnets usually contains:
  * Web application servers
  * Databases
* Public and private subnets can communicate if they're in the same VPC
* It's possible to use a VPN to connect to a VPC
* VPC Flow Logs allow you to monitor the traffic within, in and out of your VPC

## **AWS Fundamentals Part 4: S3**

* Buckets are defined at the region level
* Must have globally unique names
* Objects (files) have a Key. The key is the ***FULL*** path:
  * <my_bucket>/my_file.txt
  * <my_bucket>/my_folder1/another_folder/my_file.txt
* There's no concept of "directories" within buckets (althought the UI might make it appear like there is), just keys with very long names that contain slashes ("/")
* Object Values are the content of the body:
  * Max Size is 5TB
  * If uploading more than 5GB, must use "multi-part upload"
* Objects come with Metadata (list of text key / value pairs - system or user metadata)
* Tags (Unicode key / value pair - up to 10) - useful for security / lifecycle
* Version ID (if versioning is enabled)

### AWS S3 - Versioning

* Versioning is enabled at the bucket level
* It's best practice to version your buckets
  * Protect against unintended deletes
  * Easy roll back to previous version
* Any file not versioned prior to enabling versioning will have version "null"

### S3 Encryption

* There are 4 methods of encrypting objects in S3:
  * SSE-S3: encrypts S3 objects using keys handled & managed by AWS
    * Object is encrypted server side
    * AES-256 encryption type
    * Must set header: "x-amz-server-side-encryption":"AES256"
  * SSE-KMS: leverage AWS KMS to manage encryption (can use CMK)
    * Advantages: user control + audit trail
    * Object is encrypted server side
    * Must set header: "x-amz-server-side-encryption":"aws:kms"
  * SSE-C: when you use your own keys to encrypt data
    * Amazon S3 does not store the encryption key you provide
    * **HTTPS must be used**
    * Encryption key must be provided in HTTP headers, for every HTTP request made
  * Client Side Encryption: data is encrypted client side
    * Client use a library such as the Amazon S3 Encryption Client
    * Clients must encrypt data themselves before sending to S3
    * Clients must decrypt data themselves when retrieving from S3
    * Customer fully manages the key and encryption cycle
* Encryption in transit (SSL):
  * AWS S3 exposes:
    * HTTP endpoint: non encrypted
    * HTTPS endpoint: encryption in flight
  * You're able to use the endpoint you want, but HTTPS is reccomended
  * HTTPS is mandatory for SSE-C
  * Encryption in flight is also called SSL / TLS

* ### S3 Security & Bucket policies

* User based:
  * IAM policies - which API calls should be allowed for a specific user from IAM console
* Resource based:
  * Bucket Policies - bucket wide rules from S3 console - allows cross account
  * Object Access Control List (ACL) - finer grain
  * Bucket Access Control List (ACL) - less common

### S3 Bucket Policies

* JSON based policies
  * Resources: buckets and objects
  * Actions: Set of API to Allow or Deny
  * Effect: Allow / Deny
  * Principal: The account or user to apply the policy to
* We can use S3 bucket policy to:
  * Grant public access to the bucket
  * Force objects to be encrypted at upload
  * Grant access to another account (Cross Account)

### S3 Security

* Networking:
  * Supports VPC Endpoints (for instances in VPC without internet)
* Logging and Audit:
  * S3 access logs can be stored in other S3 bucket
  * API calls can be logged in AWS CloudTrail
* User Security:
  * MFA can be required in versioned buckets to delete objects
  * Signed URLs: URLs that are valid only for a  limited time

### S3 Websites

* If you get a 403 (Forbidden) error, check if the bucket policy allows public reads

### S3 CORS

* Cross Origin Resource Sharing allows you to limit the number of websites that can request your files in S3 (and limit your costs)
* If you request data from another S3 bucket, you need to enable CORS

### S3 - Consistency Model

* **Read after write consistency for PUTS of new objects**
  * As soon as an object is written, it can be retrieved.
  ex: (PUT 200 -> GET 200)
  * The exception is that if a GET is done before the object exists, the result will be cached and after writing the object it will be *eventually consistent*
  ex: (GET 404 -> PUT 200 -> GET 404)
* **Eventual Consistency for DELETES and PUTS of existing objects**
  * If we read an object after updating, we might get the older version
  * If we delete an object, we might still be able to retrieve it for a short time.

### S3 - Performance

* Before July 17th 2018
  * When you had > 100 TPS (transactions per second) S3 performance could degrade
  * Behind the curtains, each object goes to an S3 partition and for the best performance, we want the highest partition distribution
  * ***In the exam and in the past:*** It is reccomended to have random prefix in front of your key name to optimize perfomrance (never rerccomended to use date prefixes, as the pertitions would be very similar)
* After July 17th 2018 (Current state, but not in exam)
  * We can scale up to 3500 RPS for PUT and 5500 RPS for GET for EACH PREFIX
  * For reference: <https://aws.amazon.com/about-aws/whats-new/2018/07/amazon-s3-announces-increased-request-rate-performance/>
  * As stated in the news: 'This S3 request rate performance increase removes any previous guidance to randomize object prefixes to achieve faster performance.'
* Faster upload of large objects (>=100mb), use multipart upload:
  * Parallelizes PUTs for greater throughput
  * Maximize your network bandwidth and efficiency
  * Decrease time to retry in case a part fails
  * MUST use a multi-part if object size is greater than 5GB
* Use CloudFront to cache S3 objects around the world
* S3 Transfer Acceleration (uses edge locations) - just needs to change the endpoint you write to, not the code
* If using SS3-KMS encryption, you may be limited to your AWS limits for KMS usage (~100s - 1000s downlaods / uploads per second)

### S3 & Glacier Select

* If you retrieve data in S3 and Glacier, you may only want a **subset** of it
* If you retrieve all the data, the network cost may be high
* WIth S3 Select / Glacier Select, you can use SQL **SELECT** queries to let S3 or Glacier know exactly which attributes / filters you want
  * select * from s3object s where s.\"Country (Name)\" like %United States&'
* Save cost up to 80% and increase performance by up to 400%
* The "SELECT" happens "within" S3 or Glacier
* Works with files in CSV, JSON or Parquet format
* Files can be compressed with GZIP or BZIP2
* No subqueries or Joins are supported

___

## Developing in the AWS: AWS CLI, SDK, IAM Roles & Policies

## AWS CLI

* AWS CLI on EC2 instances: Never place your credentials on EC2, always use IAM Roles attached to it
* Some AWS CLI commands (not all) conmtain a --dry-run option to simulate API calls
* When API calls fail, you can get a long error message, which can be decoded using the **STS** command line(IAM user or Role must have STS write permission: decode message):
  * sts **decode-authorization-message**
* EC2 Instance Metadata:
  * Allows EC2 instances to "learn about themselves" without using an IAM Role for that purpose
  * The URL is <http://169.254.169.254/latest/meta-data> (ex: `curl http://169.254.169.254/latest/meta-data`)
    * You can retrieve IAM Role name from the metadata, but you ***cannot*** retrieve the IAM Policy
  * Metadata = info about the EC2 Instance
  * Userdata = launch script of the EC2 Instance
* AWS Profile
  * You can use `aws configure --profile` to configure multiple credentials on the same machine
  * When running commands, they will still run on the default profile, if you need to use an specific profile you should finish the command with `--profile {your-profile}`

## AWS SDK overview

* When you want perform actions on AWS directly from your application without using the CLI, you can use an SDK (software development kit)
* We have to use AWS SDK when coding against AWS Services such as DynamoDB
* If you don't specify or configure a default region, then us-east-1 will be chosen by default
* It's recommended to use the **default credential provider chain**
  * The **default credential provider chain** works seamlessly with:
    * AWS credentials at ~/.aws/credentials (only on our machine / on premises)
    * Instance Profile Credentials using IAM Roles
    * Environment variables (AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY)
  * ***NEVER STORE AWS CREDENTIALS ON YOUR CODE***
* Any API that fails because of too many calls needs to be retried with Exponential Backoff
  * These apply to rate limited API
  * Retry mechanism included in SDK API calls

___

## **AWS Elastic Beanstalk**

### Elastic beanstalk overview

* ElasticBeanstalk is a developer centric view of deploying applications on AWS
* It uses all the components we've seen before:
  * EC2, ASG, ELB, RDS, etc...
* We still have full control over the configuration
* BeanStalk is free but you will pay for the underlying resources
* Managed service
  * Instance configuration / OS is handled by beanstalk
  * Deployment strategy is configurable but performed by ElasticBeanstalk
* Just the application code is responsibility of the developer
* Three architecture models:
  * Single Instance deployment: good for dev
  * LB + ASG: great for production or pre-production web applications
  * ASG only: great for non-web apps in productions (workers, etc...)
* ElasticBeanstalk has three components:
  * Application
  * Application version: each deployment gets assigned a version
  * Environment name (dev, test, prod...): free naming
* You deploy application versions to environments and can promote application versions to the next environment
* Rollback feature to previous application version
* Full control over lifecycle of environments
* Support for many platforms, and if not support you can write your custom platform (advanced)

### Elastic Beanstalk Deployment Modes

* Single Instance - Great for Dev
* High Availability with Load Balancer - Great for Prod

### Beanstalk Deployment Options for Updates

* All at once (deploy all in one go) - Fastest, but instances aren't available to serve traffic for a bit (has downtime)
  * Fastest deployment
  * Application has downtime
  * Great for quick iterations in development environment
  * No additional cost
* Rolling: Update a few instances at a time (also called buckets), and then move onto the next bucket once the first bucket is healthy
  * Application is running below capacity
  * Can set the bucket size
  * Application is running both versions simultaneously
  * No additional cost
  * Long deployment
* Rolling with additional batches: like rolling, but spins up new instances to move the batch (so that old application is still available and at full capacity)
  * Application is still rolling at full capacity
  * Can set the bucket size
  * Application is running both versions simultaneously
  * Small additional cost
  * Additional batch is removed at the end of the deployment
  * Longer deployment
  * Good for prod
* Immutable: spins up new instances in a new ASG, deploys version to these instances, and then swaps all the instances when everything is healthy
  * New Code is deployed to new instances on a temporary ASG
  * High cost, double capacity
  * Longest deployment
  * Quick rollback in case of failures (just terminate new ASG)
  * Great for Prod

### Elastic Beanstalk Deployment Blue / Green

* Not a "direct feature" of Elastic Beanstalk
* Zero downtime and release facility
* Create a new "stage" environment and deploy v2 there
* The new environment (green) can be validated independently and roll back if issues occur
* Route53 can be setup using weighted policies to redirect a little bit of traffict to the stage environment
* Using Beanstalk, "swap URLs" when done with the environment test

### Elastic Beanstalk Extensions

* A zip file containing your code must be deployed to Elastic Beanstalk
* All the parameters set in the UI can be configured with code using files
  * Requirements:
    * In the .ebextensions/ directory in the root of the source code
    * YAML / JSON format
    * .config extensions (example: logging.config)
    * Able to modify some default settings using: option_settings
    * Ability to add resources such as RDS, ElastiCache, DynamoDB, etc...

### Elastic Beanstalk CLI

* We can install an additional CLI called the "EB cli" which makes working with Beanstalk from the CLI easier
* Basic commands are:
  * eb create
  * eb status
  * eb health
  * eb events
  * eb logs
  * eb open
  * eb deploy
  * eb config
  * eb terminate
* Very helpful when automating deployment pipelines

### Elastic Beanstalk under the hood

* Elastic Beanstalk relies on CloudFormation

### Elastic Beanstalk Deployment Mechanism

* Describe dependencies (requirements.txt for Python, package.json for Node.js)
* Package code as zip
* Zip file is uploaded to each EC2 machine
* Each EC2 machine resolves dependencies (slow if there are a lot of dependencies or if they are big)
* Optimization in case of long deployments: Package dependencies with source code to improve deployment performance and speed

### Elastic Beanstalk Exam Tips

* Beanstalk can work with HTTPS
  * Idea: Load the SSL onto the Load Balancer
  * Can be done from the Console ( EB console, load balancer configuration)
  * Can be done from the code: .ebextensions/securelistener-alb.config
  * SSL Certificate can be provisioned using ACM (AWS Certificate Manager) or CLI
  * Must configure security group to allow incoming port 443 (HTTPS port)
* Beanstalk redirect HTTP to HTTPS
  * Configure your instances to redirect HTTP to HTTPS
  * Or configure Application Load Balancer (ALB only) with a rule
  * Make sure health checks are not redirected (so they keep sending 200 OK)
* Elastic Beanstalk Lifecycle Policy
  * Elastic Beanstalk can store at most 1000 application versions
  * If you don't remove old versions, you won't be able to deploy newer ones
  * To phase out old application versions, use a lifecycle policy
    * Based on time (old versions are removed)
    * Based on space (when you have too many versions)
  * Versions that are currently used won't be deleted
  * Even if the version is deleted, you have the option not to delete the source bundle in S3 to prevent data loss
* Web Server vs Worker Environment
  * If your application tasks that are long to complete, offload these tasks to a dedicated worker environment
  * Decoupling your application into two tiers in common
    * Example: processing a video, generating a zip file, etc
  * You can define periodic tasks in a file cron.yaml
* RDS with Elastic Beanstalk
  * Rds can be provisioned with Beanstalk, which is great for dev / test, but not justifiable for Prod as it can result in data loss
  * The best for prod is to separately create an RDS database and provide our EB application with the connection string
  * Steps to migrate from RDS coupled in EB to standalone RDS:
    * Take an RDS DB Snapshot
    * Enable deletion protection in RDS
    * Create a new environment without an RDS, pointing to the existing old RDS
    * Perform blue/green deployment and swap new and old environments
    * Terminate the old environment (RDS won't be deleted, as the deletion protect is activated)
    * Delete CloudFormation stack (will be in DELETE_FAILED state, as the RDS still remains)

___

## **AWS CI/CD: CodeCommit, CodePipeline, CodeBuild, CodeDeploy

* AWS CodeCommit: Storing our code
* AWS CodePipeline: automating our pipeline from code to ElasticBeanstalk (?)
* AWS CodeBuild: building and testing our code
* AWS CodeDeploy: deploying the code to EC2 fleets (not Beanstalk)

### Codemmit Overview

* Private Git repositories
* No size limit on repositories (scale seamlessly)
* Fully managed, highly available
* Code only in AWS Cloud account => increased security and compliance
* Secure (Encryption, access control, etc...)
* Integrated with Jenkins / CodeBuild / and other CI tools

### CodeCommit Security

* Interactions are done using Git (standard)
* Authentication in Git:
  * SSH Keys: AWS Users can configure SSH keys in their IAM Console
  * HTTPS: Done through the AWS CLI Authentication helper or Generating HTTPS credentials
  * MFA (multi factor authentication) can be enabled for extra safety
* Authorization in Git:
  * IAM Policies manage user / roles rights to repositories
* Encryption:
  * Repositories are automatically encrypted at rest using KMS
  * Encrypted in transit (can only use HTTPS or SSH - both secure)
* Cross Account access:
  * Use IAM Role in your AWS Account and use AWS STS (with AssumeRole API)

### CodeCommit vs GitHub

* Similarities:
  * Both are git repositories
  * Both support code review (pull requests)
  * GitHub and CodeCommit can be integrated with AWS CodeBuild
  * Both support HTTPS and SSH method of authentication
* Differences:
  * Security:
    * User administration:
      * GitHub: GitHub Users
      * CodeCommit: AWS IAM users & roles
  * Hosted:
    * GitHub: hosted by GitHub
    * GitHub Enterprise: self hosted on your servers
    * CodeCommit: managed & hosted by AWS
  * UI:
    * GitHub Ui is fully featured
    * CodeCommit UI is minimal

### CodeCommit Notifications

* You can trigger notifications in CodeCommit using AWS SNS, AWS Lambda or AWS CloudWatch Event Rules
* Use cases for SNS / AWS Lambda notifications:
  * Deletion of branches
  * Trigger for pushes that happens in master branch
  * Notify external Build System
  * Trigger AWS Lambda function to perform codebase analysis (maybe credentials got commited to the code)
* Use cases for CloudWatch Event Rules:
  * Trigger for pull request updates (created / updated / deleted / commented)
  * Commit comment events
  * CloudWatch Event Rules goes into an SNS Topic

### CodePipeline Overview

* Continuous delivery
* Visual Workflow
* Source: GitHub / CodeCommit / Amazon S3
* Build: COdeBuild / Jenkins / Etc.
* Load Testing: 3rd party tools
* Deploy: AWS CodeDeploy / Beanstalk / CloudFormation / ECS ...
* Made of Stages:
  * Each stage can have **sequential actions and/ or parallel actions**
  * Stage examples: Build / Test / Deploy / Load Test /etc...
  * Manual approval can be defined at any stage

### AWS CodePipeline Artifacts

* Each pipeline stage can create "artifacts"
* Artifacts are passed stored in Amazon S3 and passed on to the next stage

### CodePipeline Troubleshooting

* CodePipeline state changes happen in **AWS CloudWatch Events**, which can in return create SNS notifications
  * Ex: you can create events for failed pipelines
  * Ex: you can create events for cancelled stages
* If CodePipeline fails a stage, your pipeline stops and you can get information in the console
* AWS CloudTrail can be used to audit AWS API calls
* If Pipeline can't perform an action, make sure the "IAM Service Role" attached does have enough permissions (IAM Policy)

### CodeBuild Overview

* Fully managed build service
* Alternative to other build tools such as Jenkins
* Continuous scaling (no servers to manage or provision - no build queue)
* Pay for usage: the time it takes to complete the builds
* Leverages Docker under the hood for reproducible builds
* Possibility to extend capabilities leveraging our own Docker images
* Secure: Integration with KMS for encryption of build artifacts, IAM for build permissions, VPC for network security, and CloudTrail for API calls logging
* Source Code from GitHub / CodeCommit / CodePipeline / S3 ...
* Build instructions can be defined in code (buildspec.yml file)
* Output logs to Amazon S3 & CloudWatch Logs
* Metrics to monitor CodeBuild statistics
* Use CloudWatch Alarms to detect failed builds and trigger notifications (SNS included)
* CloudWatch Events / AWS Lambda as a Glue
* Ability to reproduce CodeBuild locally to troubleshoot in case of errors
* Builds can be defined within CodePipeline or CodeBuild itself
* **CodeBuild supports a lot of environments natively, but in case your environment is not supported, you can generate a docker image and run any environment you like**

### CodeBuild BuildSpec

* buildspec.yml file must be at the **root** of your code
* Define environment variables:
  * Plaintext variables
  * Secure secrets: use SSM Parameter store
* Phases (specify commands to run):
  * Install: install dependencies you may need for your build
  * Pre build: final commands to execute before build
  * **Build: actual build commands**
  * Post build: finishing touches (zip output for example)
* Artifacts: What to upload to S3 (encrypted with KMS)
* Cache: Files to cache (usually dependencies) to S3 for future build speedup

### CodeBuild Local Build

* In case of need of deep troubleshooting beyond logs you can run CodeBuild locally on your desktop
  * Needs to have Docker and CodeBuild Agent installed

### CodeDeploy Overview

* AWS CodeDeploy is a service that automates code deployments to any instance, including Amazon EC2 instances and instances running on-premises
* Each EC2 Machine (or on-prem) must be running the CodeDeploy Agent
* The agent is continuously polling AWS CodeDeploy for work to do
* CodeDeploy sends appspec.yml file (must be at the root of the source code)
* Application is pulled from GitHub or S3
* CodeDeploy Agent will report of success / failure of deployment on the instance
* EC2 instances are grouped by deployment group (dev / test / prod)
* Lots of flexibility to define any kind of deployments
* CodeDeploy can be chained into CodePipeline and use artifacts from there
* CodeDeploy can re-use existing setup tools, works with any application, auto scaling integration
* Blue / Green deployments only works with EC2 instances (not on premises)
* Support for AWS Lambda deployments
* CodeDeploy does not provision resources

## AWS CodeDeploy Primary Components

* **Application**: unique name
* **Compute platform:** EC2/On-Premise or Lambda
* **Deployment configuration:** Deployment rules for success / failures
  * EC2/On-Premise: you can specify the minimum number of healthy instances for the deployment. 
  * AWS Lambda: specify how traffic is routed to your updated Lambda functions versions.
* **Deloyment group:** group of tagged instances (allows to deploy gradually)
* **Deployment type:** In-place deployment or Blue/green deployment
* **IAM instance profile:** need to give EC2 the permissions to pull from S3 / Github
* **Application revision:** application code + appspec.yml file
* **Service role:** Role for CodeDeploy to perform what it needs
* **Target Revision:** Target deployment application version

## AWS CodeDeploy AppSec

* File section: how to source and copy from S3 / GitHub to filesystem
* Hooks: set of instructions to do to deploy the new version (hooks can have timeouts). The order is:
  * ApplicationStop
  * DownloadBundle
  * BeforeInstall
  * AfterInstall
  * ApplicationStart
  * **ValidateService: really important**

## AWS CodeDeploy Deployment Config

* Configs:
  * One at a time: one instance at a time, one instance fails => deployment stops
  * Half at a time: 50%
  * All at once: quick but no healthy host, downtime. Good for dev environments
  * Custom: min healthy host = 75%
* Failures:
  * Instances stay in "failed state"
  * New deployments will first be deployed to "failed state" instances
  * To Rollback: redeploy old deployment or enable automated rollback for failures
* Deployment Targets:
  * Set of EC2 instances with tags
  * Directly to an ASG
  * Mix of ASG / Tags so you can build deployment segments
  * Customization in scripts with DEPLOYMENT_GROUP_NAME environment variables

## CodeStar Overview

* CodeStar is an integrated solution that regroups: GitHub, CodeCommit, CodeBuild, CodeDeploy, CloudFormation, CodePipeline, CloudWatch
* 