## **AWS Serverless: Lambda**

### Lambda Overview

* Virtual **functions** - no servers to manage
* Limited by time - **short executions**
* Run **on-demand**
* **Scalingis automated**
* Benefits of lambda:
  * Easy pricing, pay per request and compute time
  * Integrated with the whole AWS suite of services
  * Integrated with many programming languages
  * Easy monitoring through AWS CloudWatch
  * Easy to get more resources per functions (up to 3GB of RAM)
  * Increasing RAM will also improve CPU and Network
* Main Lambda integrations:
  * API Gateway
  * Kinesis
  * DynamoDB
  * S3
  * CloudFront
  * CloudWatch Events EventBridge
  * CloudWatch Logs
  * SNS
  * SQS
  * Cognito

### Lambda - Synchronous Invocations

* Synchronous: CLI, SDK, API Gateway, Application Load Balancer
  * Results are returned right away
  * Error handling must happen client side (retries, exponential backoff, etc...)
* Synchronous Invocations - Services 
  * User Invoked:
    * Amazon API Gateway
    * Amazon CloudFront (Lambda@Edge)
    * Elastic Load Balancing
    * Amazon S3 Batch
  * Service Invoked:
    * Amazon Cognito
    * AWS Step Functions
  * Other Services
    * Amazon Lex
    * Amazon Alexa
    * Amazon Kinesis Data Firehose

### Lambda Integration with ALB

* To expose a Lambda function as an HTTP(S) endpoint you can use the ALB or API Gateway
* The Lambda function must be registered in a **target group**
* ALB Multi-Header Values
  * ALB can support multi header values (ALB setting)
  * When you enable multi-value headers, HTTP headers and query string parameters that are sent with multiple values are shown as arrays within the AWS Lambda event and response objects. 

### Lambda@Edge

* You have deployed a CDN using CloudFront
* What if you wanted to run a global AWS Lambda alongside, or if you ned to implement request filtering before reaching your application? You can use Lambda@Edge to deploy Lambda alongside CloudFront CDN
  * Build more responsive applications
  * No servers to manage, Lambda is deployed globally
  * Customize the CDN content
  * Pay for what you use
* You can use Lambda to change CloudFront requests and responses:
  * After CloudFront receives a request from a viewer (viewer request)
  * Before CloudFront forwards the request to the origin (origin request)
  * After CloudFront receives the response from the origin (origin response)
  * Before CloudFront forwards the response to the viewer (viewer response)
  * You can also generate responses to viewers without ever sending the request to the origin
* Lambda@Edge: Use Cases
  * Website Security and Privacy
  * Dynamic Web Application at the Edge
  * Search Engine Optimization (SEO)
  * Intelligently Route Across Origins and Data Centers
  * Bot Mitigation at the Edge
  * Real-time Image Transformation
  * A/B Testing
  * User Authentication and Authorization
  * User Priorization
  * User Tracking and Analytics

### Lambda - Asynchronous Invocations

* S3, SNS, CloudWatch Events...
* The events are placed in an **Event Queue**
* Lambda attempts to retry on errors
  * 3 tries total: 1 minute wait after 1st attempt, then 2 minutes wait
* Make sure the processing is idempotent (in case of retries, same results)
* If the function is retried, you will see **duplicate logs entries in CloudWatch Logs**
* Can define a DLQ - SNS or SQS -  for failed processing
* Asynchronous invocations allow you to speed up the processing if you don't need to wait for the result (ex: if you need 1000 files processed)
* Lambda Asyn chronous Invocations - Services
  * Amazon S3
  * Amazon SNS
  * Amazon CloudWatch Events / EventBride
  * AWS CodeCommit (CodeCommit Trigger: new branch, new tag, new push)
  * AWS CodePipeline (invoke Lambda during the pipeline, Lambda must callback)
  * Amazon CloudWatch Logs (log processing)
  * Amazon Simple Email Service (SES)
  * AWS CloudFormation
  * AWS Config
  * AWS IoT
  * AWS IoT Events

### Lambda - Event Source Mapping

* Kinesis Data Streams
* SQS & SQS FIFO Queue
* DynamoDB Streams
* Common denominator: records need to be polled from the source
* Lambda is invoked synchronously

#### Streams & Lambda (Kinesis & DynamoDB)

* An event source mapping creates an iterator for each shard, processes items in order
* Start with new items, from the beginning or from timestamp
* Low traffic: use batch window to accumulate records before processing
* Possible to process multiple batches in parallel
  * Up to 10 batches per shard
  * in-order processing is still guaranteed for each partition key

#### Streams & Lambda - Error Handling 

* By default, if your function returns an error, the entire batch is reprocessed until the function succeeds, or the items in the batch expire
* To ensure in-order processing, processing for affected shard is paused until the error is resolved
* You can configurethe event mapping to:
  * Discard old events
  * Restrict the number of retries
  * Split the batch on error (to work around Lambda timeout issues)
* Discarted events can go to a **Destination**

#### Lambda - Event Source Mapping

* Event Source Mapping will poll SQS (Long Polling)
* Specify batch size (1-10 messages)
* Recommended: Set the queue visibility timeout to 6x the timeout of your Lambda function
* To use a DLQ:
  * **Set-up on the SQS queue, not Lambda** (DLQ for Lambda is only for async invocations)
  * Or use a Lambda destination for failures

#### Queues & Lambda

* Lambda also supports in-order processing for FIFO queues, scalig up to the number of active message groups
* For standard queues, items aren't necessarily processed in order
* Lambda scales up to process a standard queue as quickly as possible
* When an error occurs, batches are returned to the queue as individual items and might be processed in a different grouping than the original batch
* Occasionally, the event source mapping might receive the same item from the queue twice, even if no function error occurred.
* Lambda deletes items from the queue after they're processed  successfully.
* You can configure the source queue to send items to a dead-letter queue if they can't be processed

#### Lambda Event Mapping Scaling

* **Kinesis Data Streams & DynamoDB Streams**:
  * One lambda invocation per stream shard
  * If you use parallelization, up to 10 batches processed per shard simultaneosly
* **SQS Standard**:
  * Lambda adds 60 more instances per minute to scale up
  * Up to 1000 instances of messages processed simultaneously
* **SQS FIFO**:
  * Messages with the same GroupID will be processed in order
  * The Lambda function scales to the number of active message groups

### Lambda Destinations

* **Asynchronous Invocations** - can define destinations for successful and failed event:
  * SQS
  * SNS
  * AWS Lambda
  * Amazon EventBridge Bus
* AWS recommends you use destinations instead of DLQ now (however, both can be used at the same time)
* **Event Source Mapping**: for discarded event batches
  * SQS
  * SNS

### Lambda Execution Role (IAM Role)

* Grants the Lambda function permissions to AWS services/rosources
* When you use an event source mapping to invoke a function, Lambda uses the execution role to read event data
* Best practice: create one Lambda Execution Role per function

#### Lambda Resource Based Policies

* Use resource-based policies to give other accounts and AWS services permission to use your Lambda resources
* Similar to S3 bucket policies for S3 bucket
* An IAM principal can access Lambda:
  * If the IAM policy attached to the principal authorizes it (user access)
  * Or if the resource-based policy authorizes (service access)
* When an AWS service calls your Lambda (e.g.: S3), the resource-based policy gives it access

### Lambda Environment Variables

* Environment variable = key / value pair in "String" form
* Adjust the function behavior without updating code
* The environment variables are available to your code
* Lambda Service adds its own system environment variables as well
* Helpful to store secrets (encrypted by KMS)
* Secrets can be encrypted by the Lambda service key, or your own CMK

### Lambda Logging & Monitoring

* CLoudWatch Logs
  * AWS Lambda execution logs are stored in AWS CloudWatch Logs
  * Lambda execution role needs to have IAM Policy capable to write to CloudWatch
* CloudWatch Metrics
  * AWS Lambda metrics are displayed in CloudWatch Metrics
  * Invocations, Durations, Concurrent Executions
  * Error count, Success Rates, Throttles
  * Async Delivery Failures
  * Iterator Age (Kinesis & DynamoDB Streams)

#### Lambda TRacing with X-Ray

* Enable in Lambda configuration (Active Tracing)
* Runs the X-Ray daemon for you
* Use AWS X-Ray SDK in code
* Ensure Lambda Function has a correct IAM Execution Role
  * The managed policy is called AWSXRayDaemonWriteAccess
* Environment variables to communicate with X-Ray
  * **_X_AMAZN_TRACE_ID**: contains the tracing header
  * **AWS_XRAY_CONTEXT_MISSING**: by default, LOG_ERROR
  * **AWS_XRAY_DAEMON_ADDRESS**: the X-Ray Daemon IP_ADDRESS:PORT

### Lambda in VPC

* By default, your Lambda function is launched outside your own VPC (in an AWS-owned VPC)
* Therefore it cannot access resources in your VPC (RDS, ElastiCache, internal ELB...)
* To place Lambda in a VPC, you must define: VPC ID, Subnets and Security Groups
* Lambda will create an ENI (Elastic Network Interface) in your subnets
* **AWSLambdaVPCAccessExecutionRole**

#### Lambda in VPC - Internet Access

* A Lambda function in your VPC does not have internet access
* Deploying a Lambda function in a public subnet does not give it internet access or a public IP
* Deploying a Lambda function in a private subnet gives it internet access if you have a Nat Gateway / Instance
* **You can use VPC endpoints to privately access AWS services without a NAT**

### Lambda Function Configuration

* **Ram**:
  * Scale from 128 to 3008MB in 64MB increments
  * The more RAM you add, the more vCPU credits you get
  * At 1792 MB, a function has the equivalent of one full vCPU
  * After 1792 MB, you get more than one CPU, and need to use multi-threading in your code to benefit from it
  * Therefore, if your application is CPU-bound (computation heavy), increase RAM
* **Timeout**:
  * Default is 3 seconds, max is 900 seconds

#### Lambda Execution Context

* The execution context is a temporary runtime environment that initializes any external dependencies of your lambda code
* Great for database connections, HTTP clients, SDK clients, etc...
* The execution context is maintained for some time in antecipation of another Lambda function invocation, so the next function invocation can "re-use" the context to execution time and save time initializing connections objects
* The execution context includes the /tmp directory

#### Lambda Functions /tmp space

* Use the /tmp directory:
  * If your Lambda function needs to download a big file to work
  * If your Lambda function needs disk space to perform operations
* Max size is 512 MB
* The directory content remains when the execution context is frozen, providing transient cache that can be used for multiple invocations (helpful to checkpoint your work)
* For permanent persistence of object, use S3

### Lambda Concurrency and Throttling

* Concurrency limit: up to 1000 concurrent executions
* Can set a "reserved concurrency" at the function limit (=limit)
* Each invocation over the concurrency limit will trigger a "Throttle"
* Throttle behavior:
  * If synchronous invocation => return ThrottleError - 429
  * If asynchronous invocation => retry automatically and then go to DLQ
* If you need a higher limit, open a support ticket
* Concurrency limit applies to all lambdas in the account

#### Concurrency and Asynchronous Invocations

* If the function doesn't have enough concurrency available to process all events, additional requests are throttled.
* For throttling errors (429) and system errors (500-series), Lambda returns the event to the queue and attempts to run the function again for up to 6 hours
* The retry interval increases exponentially from 1 second