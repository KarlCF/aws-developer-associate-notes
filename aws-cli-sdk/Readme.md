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
