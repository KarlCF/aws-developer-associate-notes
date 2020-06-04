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
