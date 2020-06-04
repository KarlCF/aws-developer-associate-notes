## **AWS Monitoring & Audit: CloudWatch, X-Ray and CloudTrail**

### Monitoring in AWS

* AWS CloudWatch:
  * Metrics: Collect and track key metrics
  * Logs: Collect, monitor, analyze and store log files
  * Events: Send notifications when certain events happen in your AWS
  * Alarms: React in real-time to metrics / events
* AWS X-Ray:
  * Troubleshooting application performance and errors
  * Distributed tracing of microservices
* AWS CloudTrail:
  * Internal monitoring of API calls being made
  * Audit changes to AWS Resources by your users

### AWS CloudWatch Metrics

* CloudWatch provides metrics for every services in AWS
* **Metric** is a variable to monitor (CPUUtilization, NetworkIn...)
* Metrics belongs to namespaces
* **Dimension** is an attribute of a metric (instance id, environment, etc...)
* Up to 10 dimensions per metric
* Metrics have **timestamps**
* AWS CloudWatch EC2 Detailed monitoring
  * EC2 instances metrics are by default updated every 5 minutes, detailed monitoring decreases it to 1 minute, but costs more
  * Use detailed monitoring if you want to more promptly scale your ASG
  * EC2 Memory usage is not a default metric, but a custom one.
* AWS CloudWatch Custom Metrics
  * Possibility to define and send your own custom metrics to CloudWatch
  * Ability to use dimnensions (attributes) to segment metrics
    * Instance.id
    * Environment.name
  * Metric Resolution:
    * Standard: 1 minute
    * High Resolution: up to 1 second (**StorageResolution** API parameter) - Higher cost
  * Use API call **PutMetricData**
  * Use exponential back off in case of throttle errors

### AWS CloudWatch Alarms

* Alarms are used to trigger notifications for any metric
* Alarms can go to Auto Scaling, EC2 Actions, SNS notifications
* Various options (sampling, %, max, min, etc...)
* Alarm States:
  * OK
  * INSUFFICIENT_DATA
  * ALARM
* Period:
  * Lenght of time in seconds to evaluate the metric
  * High resolution custom metrics can only choose 10 or 30 sec

### CloudWatch Logs

* Applications can send logs to CloudWatch using the SDK
* CloudWatch can collect log from:
  * ElasticBeanstalk: collection of logs from application
  * ECS: collection from containers
  * AWS Lambda: collection from function logs
  * VPC Flow Logs: VPC specific logs
  * API Gateway
  * CloudTrail based on filter
  * CloudWatch log agents: for example on EC2 machines or on premises
  * Route53: Log DNS queries
* CloudWatch Logs can go to:
  * Batch exporter to S3 for archival
  * Stream to ElasticSearch cluster for further analytics
* CloudWatch Logs can use filter expressions
* Logs storage architecture:
  * Log groups: arbitrary name, usually representing an application
  * Log stream: instances within application / log files / containers
* Can define log expiration policies (never expire, 30 days, etc...)
* Using the AWS CLI we can tail CloudWatch logs
* To send logs to CloudWatch, make sure IAM permissions are correct!
* Security: encryption of logs using KMS are at the Group Level

### AWS CloudWatch Events

* Schedule: Cron jobs
* Event Pattern: Event rules to react to a service doing something
* Triggers to Lambda functions, SQS / SNS / Kinesis Messages
* CloudWatch Event creates a small JSON document to give information about the change

### AWS X-Ray

* AWS X-Ray provides visual analysis of our applications
* AWS X-Ray advantages:
  * Troubleshooting performance (bottlenecks)
  * Understand dependencies in a microservice architecture
  * Pinpoint service issues
  * Review request behavior
  * Find errors and exceptions
  * Are we meeting time SLA
  * Where I am throttled?
  * Identify users that are impacted
* X-Ray compatibility
  * AWS Lambda
  * Elastic Beanstalk
  * ECS
  * ELB
  * API Gateway
  * EC2 instances or any application servers (even on premises)
* AWS X-Ray Leverages Tracing
  * Tracing is an end to end way to following a "request"
  * Each component dealing with the request adds its own "trace"
  * Tracing is made of segments (+sub segments)
  * Annotations can be added to traces to provide extra-information
  * Ability to trace:
    * Every request
    * Sample request (as a ¢ for example or a rate per minute)
  * X-Ray Security:
    * IAM for authorization
    * KMS for encryption at rest
* How to enable X-Ray:
  * Your code (Java, Python, Go, Node.js, .NET) must import the AWS X-Ray SDK
  * The application SDK will thencapture:
    * Calls to AWS services
    * HTTP / HTTPS requests
    * Database Calls (MySQL, PostgreSQL, DynamoDB)
    * Queue calls (SQS)
  * Install the X-Ray daemon or enable X-Ray AWS Integration
    * X-Ray daemon works as a low level UDP packet interceptor
    * AWS Lambda / Each application must have the IAM rights to write data to X-Ray
  * To enable on AWS Lambda:
    * Ensure it has an IAM execution role with proper policy (AWSX-RayWriteOnlyAccess)
    * Ensure that X-Ray is imported in the code

### X-Ray Exam tips

* The X-Ray daemon / agent has a config to send traces cross account. This allows to have a central account for all your application tracing:
  * Make sure that the IAM permissions are correct -  the agent will assume the role.
* **Segments**: each application / service will send them
* **Trace**: segments collected together to form an end-to-end trace
* **Sampling**: decrease the amount of requests sent to X-Ray, reduce costs
* **Annotations**: Key Value pairs used to index traces and use with filters
* **Metadata**: Key value pairs, **not indexed**, not used for searching
* Code must be instrumented to use AWS X-Ray SDK (interceptors, handlers, http clients)
* IAM role must be correct to send traces to X-Ray
* **X-Ray on EC2 / on Premises**:
  * Linux system must run the X-Ray daemon
  * IAM instance role if EC2, other AWS credentials on on-premise instance
* **X-Ray on Lambda**:
  * Make sure X-Ray integration is checked on Lambda
* **X-Ray on Beanstalk**:
  * Set configuration on EB console
  * Or use a beanstalk extension (.ebsextensions/xray-daemon.config)
* **X-Ray on ECS / EKS / Fargate (Docker)**:
  * Create a Docker image that runs the Daemon / or use the official X-Ray Docker image
  * Ensure port mappings & network settings are correct and IAM task roles are defined

### AWS CloudTrail

* Provides cgovernance, compliance and audit for your AWS Account
* CloudTrail is enabled by default
* Get an histopy of events / API calls made within your AWS Account by
  * Console
  * SDK
  * CLI
  * AWS Services
* Can put logs from CloudTrail into CloudWatch Logs
* If a resource is deleted in AWS, check CloudTrail

### CloudTrail vs CloudWatch vs X-Ray

* CloudTrail
  * Audit API calls made by users / services / AWS console
  * Useful to detect unauthorized calls or root cause of changes
* CloudWatch:
  * CloudWatch Metrics over time for monitoring
  * CloudWatch Logs for storing application log
  * CloudWatch Alarms to send notifications in case of unexpected metrics
* X-Ray:
  * Automated Trace Analysis & Central Service Map Visualization
  * Latency, Errors and Fault analysis
  * Request tracking accross distributed systems
