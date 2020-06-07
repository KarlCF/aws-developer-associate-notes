## **Developing in the AWS: AWS CLI, SDK, IAM Roles & Policies**

#### **AWS CLI**

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

#### **MFA with CLI**

* To use MFA with the CLI, you must create a temporary session. To do so, you must run the STS GetSessionToken API call
  * **aws sts -getsession-token** --serial-number arn-of-the-mfa-device --token-code code-from-token --duration-seconds 3600

#### **AWS SDK**

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

#### **AWS Limites (Quotas)**

* **API Rate Limits**
  * DescribeInstances API for EC2 has a limit of 100 calls/s
  * GetObject on S3 has a limit of 5500 GET/s
  * For Intermittent Errors: implement Exponential Backoff
  * For Consistent Errors: request an API throttling limit incrase
* **Service Quotas (Service Limits)**
  * Running On-Demand Standard Instances: 1152vCPU
  * You can request a service limit increase by opening a ticket
  * You can request a service quota increase by using the Service Quotas API

 #### **Exponential Backoff (any AWS serivce)**

 * If you get **ThrottlingExpection** intermittently, use exponential backoff
 * Retry mechanism, included in SDK API calls
 * Must implement yourself if using the API as is or in specific cases.

#### **AWS CLI Credentials Provider Chain**

* The CLI will look for credentials in this order:
  1. Command line options - --region, --output, and --profile
  2. Environment variables - AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY and AWS_SESSION_TOKEN
  3. CLI credentials file -aws configure (~/.aws/credentials on Linux/Mac & C:\User\user\\.aws\credentials on Windows)
  4. CLI configuration file -aws configure (~/.aws/configure on Linux/Mac & C:\User\user\\.aws\config on Windows)
  5. Container credentials - for ECS tasks
  6. Instance profile credentials - for EC2 Instance Profiles

#### **AWS SDK Default Credentials Provider**

* The Java SDK (example) will look for credentials in this order:
  1. Environment variables
  2. Java system properties - aws.accessKeyId and aws.secretKey
  3. The default credential profiles file - ~/.aws/credentials, shared by many SDK
  4. Amazon ECS container credentials - for ECS containers
  5. Instance profile credentials - used on EC2 instances