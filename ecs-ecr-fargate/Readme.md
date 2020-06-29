## **ECS**

### ECS Clusters Overview

* ECS Clusters are logical grouping of EC2 instances
* EC2 instances run the ECS agent (Docker container)
* The ECS agents registers the instance to the ECS cluster
* The EC2 instances run a special AMI, made specifically for ECS

### ECS Task Definitions

* Task definitions are metadata in JSON form to tell ECS how to run a Docker Container
* It contains crucial information around:
  * Image Name
  * Port Binding for Container and Host
  * Memory and CPU required
  * Environment variables
  * Networking information

### ECS Service

* ECS Service help define how many tasks should run and how they should be run
* They ensure that the number of tasks desired is running accross our fleet of EC2 instances.
* They can be linked to ELB / NLB / ALB if needed

### ECR 

* ECR is a private Docker image repository
* Access is controlled through IAM (permission errors  => policy)
* AWS CLI v1 login command (may be asked at the exam)
  * $(aws ecr get-login --no-include-email --region us-east-1)
* AWS CLI v2 login command (may be asked at the exam)
  * aws ecr get-login-password --region -us-east-1 | docker login --username AWS -- password-stdin 1234567890.dkr.ecr.us-east-1.amazonaws.com
* Docker Push & Pull:
  * docker push 1234567890.dkr.us-east-1.amazonaws.com/demo:latest
  * docker pull 1234567890.dkr.us-east-1.amazonaws.com/demo:latest

### Fargate

* When launching an ECS Cluster, we have to create EC2 instances
* If we need to scale, we need to add EC2 instances
* With Fargate, you create the task definitions and AWS runs the containers
* To scale, just increase the task number.

### ECS IAM Roles

#### EC2 Instance Profile:

* Used by the ECS agent
* Makes API calls to ECS service
* Sends container logs to CloudWatch Logs
* Pull Docker image from ECR

#### ECS Task Role:

* Allows each task to have a specific role
* Use different roles for the different ECS Services you run
* Task role is defined in the task definition

### ECS Task Placement

* When a task of type EC2 is launched, ECS must determine where to place it, with the constraints of CPU, memory and available port
* Similarly, when a service scales in, ECS needs to determine which task to terminate. 
* To assist with this, you can define a task placement strategy and task placement constraints

#### ECS Task Placement Process

* Task placement strategies are a best effort
* When Amazon ECS places tasks, it uses the following process:
  * Identify the instances that satisfy the CPU, memory and port requirements in the task definition
  * Identify the instances that satisfy the task placement constraints
  * Identify the instances that satisfy the task placement strategies
  * Select the instances for task placement

#### ECS Task placement Strategies

* **Binpack**
  * Place tasks based on the least available amount of CPU or memory
  * This minimizes the number of instances in use (cost savings)
* **Random**
  * Place the task randomly
* **Spread**
  * Place the task evenly based on the specified value
  * Example: InstanceId, attribute:ecs.availability-zone

#### ECS Task Placement Constraints

* distinctInstance: place each task on a different container instance
* memberOf: places task on instance sthat satisfy an expression
  * Uses the Cluster Query Language

### ECS - Service Auto Scaling

* CPU and RAM is tracked in CloudWatch at the ECS service level
* **Target Tracking**: target a specific CloudWatch metric
* **Step Scaling**: scale based on CloudWatch alarms
* **Scheduled Scaling**: based on predictable changes
* ECS Service Scaling (task level) =/= EC2 Auto Scaling (instance level)
* Fargate Auto Scaling is much easier to setup

#### ECS - Cluster Capacity Provider

* A Capacity Provider is used in association with a cluster to determine the infrastructure that a task runs on
  * For ECS and Fargate users, the FARGATE and FARGATE_SPOT capacity providers are added automatically
  * For Amazon ECS on EC2, you need to associate the capacity provider with an auto-scaling group
* When you run a task or a service, you define a capacity provider strategy, to prioritize in which provider to run
* This allows the capacity provider to automatically provision infrastructure for you

