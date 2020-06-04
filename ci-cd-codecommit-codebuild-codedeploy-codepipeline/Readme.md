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
  * Canary: 
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
* Helps quickly create "CICD-ready" projects for EC2, Lambda, Beanstalk
* Supported languages: C#, Go, HTML 5, Java, Node.js, PHP, Python, Ruby
* Issue tracking integration with: JIRA / GitHub Issues
* Ability to integrate with Cloud9 to obtain a web IDE (not all regions)
* One dashboard to view all your components
* Free service, pay only for the underlying services and resources
* Limited Customization
