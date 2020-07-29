## AWS SES - Simple Email Service

* Send emails to people using
  * SMTP interface
  * AWS SDK
* Ability to receive email. Integrates with:
  * S3
  * SNS
  * Lambda
* Integrated with IAM for allowing to send emails

## Databases Summary

* RDS: Relational databases, OLTP
  * PostgreeSQL, MySQL, Oracle, etc...
  * Aurora + Aurora Serverless
  * Provisioned database
* DynamoDB: NoSQL DB
  * Managed, Key value, Document
  * Serverless
* ElastiCache: In memory DB
  * Redis / Memcached
  * Cache capability
* Redshift: OLAP - Analytic Processing
  * Data Warehousing / Data Lake
  * Analytics queries
* Neptune: Graph Database
* DMS: Database Migration Service
* DocumentDB: managed MongoDB for AWS

## AWS Certicate Manager (ACM)

* To host public SSL certificates in AWS:
  * Buy your own and upload them using the CLI
  * Have ACM provision and renew public SSL certificates for your (free of cost)
* ACM loads SSL certificates on the following integrations:
  * Load Balancers (including the ones created by EB)
  * CloudFront distributions
  * APIs on API Gateways
* SSL certificates are overall hard to manually manage, so ACM is a great advantage for the infrastructure