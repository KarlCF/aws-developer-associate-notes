## SSM Parameter Store

* Secure storage for configuration and secrets
* Optional Seamless Encryption using KMS
* Serverless, scalable, durable, easy SDK
* Version tracking of configurations / secrets
* Configuration management using path & IAM
* Notification with CloudWatch Events
* Integration with CloudFormation

#### Hierarchy (example):

* /my-department/
  * my-app/
    * dev/
      * db-url
      * db-password
    * prod/
      * db-url
      * db-password
  * other-app/
* other-department/
* /aws/reference/secretsmanager/secret_ID_in_Secrets_Manager
* /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2

#### Parameters Policies (for advanced parameters)

* Allow to assign a TTL to a parameter (expiration date) to force updating or deleting sensitive data such as passwords
* Can assign multiple policies at a time

## Secrets Manager

* Capability to force rotation of secrets every X days
* Automate generation of secrets on rotation (uses Lambda)
* Integration with **Amazon RDS** (MySQL, PostgreeSQL, Aurora)
* Secrets are encrypted using KMS
* Mostly meant for RDS integration

### Parameter Store vs Secrets Manager

* Secrets Manager(Expensive)
  * Automatic rotation of secrets with AWS Lambda
  * Integration with RDS, Redshift, DocumentDB
  * KMS encryption is mandatory
  * Integration with CloudFormation
* SSM Parameter Store (Cheaper)
  * Simple API
  * No secret rotation
  * KMS encryption is optional
  * Integration with CloudFormation
  * Can pull a Secrets Manager secret using the SSM Parameter Store API

