## AWS Fundamentals: IAM + EC2 + EBS + EFS

### **EC2**

* **To SSH into linux: chmod 0400 ec2keypair.pem**
* ssh -i ec2keypair.pem ec2-user@255.255.255.255
* It's good to maintain one separate security group for SSH access (best practice)
* Limit of 5 Elastic IP's per account (if you need to increase, just contact AWS)
* Try to avoid using Elastic IP:
  * They often reflect poor architectural decisions
  * Instead, use a random public IP and register a DNS name on it
  * Use a load balancer and no public IP (best pattern)
* Bootstrapping = EC2 User Data (only runs once at the first start)
* EC2 User Data is automaticaly run with the sudo command

### Elastic Network Interfaces (ENI)

* Logical component in a VPC that represents a virtual network card
* The ENI can have the following attributes:
  * Primary private (IPv4), one or more secondary IPv4
  * One Elastic IP (Ipv4) per private IPv4
  * One Public IPv4
  * One or more security groups
  * A MAC address
* You can create ENI independently and attach them on the fly (move them) on EC2 instances for failover
* Bound to a specific AZ


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
* Only GP2 and IO1 can be used as boot volumes
* On GP2, you get 3 IOPS per GB. The minimum of IOPS is 100 and the maximum is 16k.
* On IO1, you can get a ratio of up to 50:1 IOPS:GB. The max for normal instances is 32K, on nitro instances it is 64k.

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

### EFS - Elastic File System

* Managed NFS (Network File System) that can be mounted on multiple EC2.
* EFS is multi-AZ
* Highly available and scalable. The usage cost is: 3 x (gp2's cost).
* Only pay for the use, not the available storage.
* Uses SG to control access to EFS
* Use cases:
  * Content management
  * Web Serving
  * Data Sharing
  * Wordpress
* Uses NFSv4.1 protocol
* **Compatible only with Linux based AMI**
* Encryption at rest using KMS
* POSIX file system (Linux) that has a standard file API
* File system scales automatically, pay-per-use, no capacity planning required
* **EFS Scale**
  * Thousands of concurrent NFS clients, 10GB/s + throughput
  * Grow to Petabyte-scale NFS, automatically
* **Performance mode (nset at a EFS creation time)**
  * General purpose (default): latency-sensitive use cases (web server, CMS, etc...)
  * Max I/O - higher latency, throughput, highly parallel (big data, media processing)
* **Storage Tiers (lifecycle management feature)**
  * Standard: for frequently accessed files
  * Infrequent access (EFS-IA): cost to retrieve files, lower price to store

### IAM

#### AWS STS - Security Token Service

* Allows to grant limited and temporary access to AWS resources ( up to 1 hour)
* **AssumeRole**: Assume roles within your account or cross account
* **AssumeRoleWithSAML**: return credentials for users logged with SAML
* **AssumeRoleWithWebIdentity**:
  * return creds for users logged in with an IdP (Facebook Login, Google Login, OIDC compatible...)
  * AWS recommends against using this, and using **Cognito Identity Pools** instead
* **GetSessionToken**: for MFA, from a user or AWS account root user
* **GetCallerIdentity**: return details about the IAM user or role used in the API call
* **DecodeAuthorizationMessage**: decode error message when an AWS API is denied

#### Using STS to Assume a Role

* Define an IAM Role within your account or cross-account
* Define which principals can access this IAM Role
* Use AWS STS (Security Token Service) to retrieve credentials and impersonate the IAM Role you have access to (**AssumeRole API**)
* Temporary credentials can be valid between 15 minutes to 1 hour

 #### STS with MFA

* Use **GetSessionToken** from STS
* Appropriate IAM policy using IAM Conditions
* **aws:MultiFactorAuthPresent:true**
* GetSessionToken returns;
  * Access ID
  * Secret Key
  * Session Token

#### IAM - Authorization Model

1. If there's an explicit DENY, end decision and Deny
2. If there's an allow, end decision with Allow
3. Else, Deny

#### IAM Policies & S3 Policies

* When evaluating if an IAM Principal can perform an operation X on a bucket, the **union** of its assigned IAM Policies and S3 Bucket Policies will be evaluated

#### Dynamic Policies with IAM

* How to assign each user a /home/<user> folder in an S3 bucket?
  * Option 1:
    * One IAM policy for each user
  * Option 2:
    * Dynamic policy with IAM
      * ${aws:username}, (e.g: {"Action" ["s3:*"], "Effect":"Allow","Resource":["arn:aws:::my-company/home/${aws:username}/*"]})

#### Inline vs Managed Policies

* AWS Managed Policy
  * Maintained by AWS
  * Good for users and administrators
  * Updated in case of new services / new APIs
* Customer Managed Policy
  * Best Practice, re-usable, can be applied to many principals
  * Version Controlled + rollback, central change management
* Inline
  * Strict one-to-one relationship between policy and principal
  * Policy is deleted if you delete the IAM principal

#### Granting a User Permissions to Pass a Role to an AWS Service

* To configure many AWS services, you must pass an IAM role to the service (this happens only once during setup), the service will then assume the role and perform actions
* For this, IAM permission iam:PassRole is needed
  * It often comes with iam:GetRole to view the role being passed
  * A trust policy for the role allows the service to allow the role

#### Directory Services

* **Microsoft Active Directory**
  * Found on any Windows Server with AD Domain Services
  * Database of **objects**: User Accounts, Computers, Printers, File Shares, Security Groups
  * Centralized security management, create account, assign permissions
  * Objects are organized in **trees**
  * A group of trees is a **forest**
* **AWS Directory Services**
  * **AWS Managed Microsoft AD**
    * Create your own AD in AWS, manage users locally, supports MFA
    * Enablish "trust" connections with your on-premise AD
  * **AD Connector**
    * Directory Gateway (proxy) to redirect to on-premise AD
    * Users are managed on the on-premise AD
  * **Simple AD**
    * AD-compatible managed directory on AWS
    * Cannot be joined with on-premise AD