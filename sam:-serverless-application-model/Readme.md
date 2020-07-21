## **AWS Serverless Application Model (SAM)**

* Framework for developing and deploying serverless application
* All the configuration is YAML code
* Generate complex CloudFormation from simple YAML file
* Supports anything from CloudFormation: Outputs, Mappings, Parameters, Resources...
* Only two commands to deploy to AWS
* SAM can use CodeDeploy to deploy Lambda Functions
* SAM can help you to run Lambda, API Gateway, DynamoDB locally
* SAM requires the **Transform** and **Resources** sections


### AWS SAM - Recipe

* Transform Header indicates it's a SAM template:
  * Transform: 'AWS::Serverless-2016-10-31'
* Write Code
  * AWS::Serverless::Function (Lambda)
  * AWS::Serverless::Api (API Gateway)
  * AWS::Serverless::SimpleTable (DynamoDB)
* Package & Deploy:
  * aws cloudformation package / sam package
  * aws cloudformation deploy / sam deploy

### SAM Policy Templates

* List of templates to apply permissions to your Lambda Functions
* Full list available: https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-policy-templates.html
* Important examples:
  * **S3ReadPolicy**: Gives read only permissions to objects in S3
  * **SQSPollerPolicy**: Allows to poll an SQS queue
  * **DynamoDBCrudPolicy**: CRUD = Create Read Update Delete

### SAM with CodeDeploy

* SAM framework natively uses CodeDeploy to update Lambda functions
* Traffic Shifting feature
* Pre and Post traffic hooks features to validate deployment (before the traffic shift starts and after it ends)
* Easy & automated rollback using CloudWatch Alarms


### Commands:

* sam build: fetch dependencies and create local deployment artifacts
* sam package: package and upload to amazon S3, generate CF template
* sam deploy: deploy to CloudFormation