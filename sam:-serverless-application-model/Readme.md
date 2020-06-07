## **AWS Serverless Application Model (SAM)**

* Framework for developing and deploying serverless application
* All the configuration is YAML code
* Generate complex CloudFormation from simple YAML file
* Supports anything from CloudFormation: Outputs, Mappings, Parameters, Resources...
* Only two commands to deploy to AWS
* SAM can use CodeDeploy to deploy Lambda Functions
* SAM can help you to run Lambda, API Gateway, DynamoDB locally

### AWS SAM - Recipe

* Transform Header indicates it's a SAM template:
  * Transform: 'AWS::Serverless-2016-10-31'
* Write Code
  * AWS::Serverless::Function
  * AWS::Serverless::Api
  * AWS::Serverless::SimpleTable
* Package & Deploy:
  * aws cloudformation package / sam package
  * aws cloudformation deploy / sam deploy