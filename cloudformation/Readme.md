
## **CloudFormation**

* CloudFormation is a declarative way of outlining your AWS Infrastructure, for any resources (most are supported)
* CloudFormation creates declared resources, in the **right order**, with the exact configuration you specify

### Benefits of AWS CloudFormation

* Infrastructure as Code
  * No resources are manually created, which is great for control
  * The code can be version controlled for example, using git
  * Changes to the infrastructure are reviewed through code
* Cost
  * CloudFormation as a service is free, you pay for the resources in the stack
  * Each resources within the stack is stagged with an identifier so you can easily see how much a stack costs you
  * You can estimate the costs of your resources using the CloudFormation template
  * Savings strategy: In Dev, you could automation deletion of templates at 5 PM and recreated at 8 AM, safely
* Productivity
  * Ability to destroy and re-create an infrastructure on the cloud on the fly
  * Automated generation of Diagram for your templates!
  * Declarative programming (no need to figure out ordering and orchestration)
* Separation of concern: create many stacks for many apps, and many layers. Ex:
  * VPC stacks
  * Network stacks
  * APP stacks
* Don't re-invent the wheel
  * Leverage existing templates on the web
  * Leverage the documentation

### How CloudFormation Works

* Templates have to be uploaded in S3 then referenced in CloudFormation
* To update a template, we can't edit previous ones. We have to re-upload a new version of the template to AWS
* Stacks are identified by a name
* Deleting a stack deletes every single artifact that was created by CloudFormation

### Deploying CloudFormation templates

* Manual way:
  * Editing templates in the CloudFormation Designer
  * Using the console to input parameters, etc
* Automated way:
  * Editing templates in a YAML file
  * Using the AWS CLI to deploy the templates
  * Recommended way when fully want to automate your flow

### CloudFormation Building Blocks

* Template components:
  * **Resources: your AWS resources declared in the template (Required)**
  * Parameters: the dynamic inputs for your template
  * Mappings: the static variables for your template
  * Outputs: References to what has been created
  * Conditionals: List of conditions to perform resource creation
  * Metadata
* Template helpers:
  * References
  * Functions

### CloudFormation Resources

* The core of the CloudFormation template (**mandatory**)
* Tehy represent the different AWS Components that will be created and configured
* Resources are declared and can reference each other
* AWS handles creation, updates and deletion of resources for us
* Resource types identifiers are of the form:
  * **AWS::aws-product-name::data-type-name**
* All the resources available can be found at: <https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html>

### CloudFormation Parameters

* What are CloudFormation Parameters?
  * Parameters are a way to provide inputs to your AWS CloudFormation template
  * Useful for:
    * If you want to **reuse** your templates across the company
    * Some inputs can not be determined ahead of time
  * Parameters are extremely powerful, controlled, and can prevent errors from happening in your templates thanks to types
* When should you use a parameter?
  * Ask:
    * Is this CloudFormation resource configuration likely to change in the future? If so, make it a parameter
  * You don't have to re-upload a template to change a parameter
* Parameter Settings: Parameters can be controlled by these settings:
  * Type:
    * String
    * Number
    * CommaDelimitedList
    * List,Type>
    *  AWS Parameter (to help catch invalid values -  match against existing values in the AWS Account)
  * Description
  * Constraints
  * ConstraintDescription (String)
  * Min/MaxLenght
  * Min/MaxValue
  * Defaults
  * AllowedValues (array)
  * AllowedPattern (regexp)
  * NoEcho (Boolean)
* How to reference a Parameter
  * The **Fn::Ref** (in YAML **!Ref**) function can be leveraged to reference parameters
  * Parameters can be used anywhere in a template
  * The function can also reference other elements within the template
  * Concept: Pseudo Parameters
    * AWS Offers us pseudo parameters in any CloudFormation template that are enabled by default and can be used at any time.
    * Format for the list: Reference Value : Example Return Value
      * AWS::AccountID : 1234567890
      * AWS::NotificationARNs : [arn:aws:sns:us-east-1:123456789012:MyTopic]
      * AWS::NoValue : Does not return a value.
      * AWS::Region : us-east-2
      * AWS::StackId : arn:aws:cloudformation:us-east-1:123456789012:stack/MyStack/1c2fa620-982a-11e3-aff7-50e2416294e0
      * AWS::StackName : MyStack

### CloudFormation Mappings

* What are mappings?
  * Mappings are fixed variables within your CloudFormation template (Hard coded within the template)
  * Useful to differentiate between types of environments (dev vs prod), regions (AWS regions), AMI types, etc.
* Mappings vs Parameters
  * Mappings are great when you know in advance all the values that can be taken and that they can be deduced from variables such as 
    * Region
    * AZ
    * AWS Account
    * Envirnonment (dev vs prod)
    * Etc...
  * Mappings allow safer control over the template
  * Use Parameters when the values are user specific or case specific
* Acessing Mapping Values
  * **Fn::FindInMap** to return a named value from a specific key (in YAML shorthand: **!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]**)

### CloudFormation Outputs

* What are outputs?
  * The Outputs section declares *optional* output values that we can import into other stacks (if you export them first)
  * You can view the outputs in the AWS Console or using the AWS CLI
  * They're very useful for example, if you define a network CloudFormation, and output the variables such as VPC ID and your Subnet IDs
  * It's the best way to perform some collaboration cross stack, as you let each team handle their own part of the slack
  * **You can't delete a CloudFormation Stack if its outputs are being referenced by another CloudFormation stack**

### CloudFormation Conditions

* What are conditions used for?
  * Conditions are used to control the creation of resources or outputs based on a condition
  * Common conditions are:
    * Environment (dev / test / prod)
    * AWS Region
    * Parameter value
  * Each condition can reference another condition, parameter value or mapping
* How to define a condition? Here's an example: 
``` YAML
  Conditions:

  CreateProdResources: !Equals [ !Ref EnvType, prod] 
```
* The logical ID is defined by you, it's how you name the condition
* The intrinsic function (logical) can be any of the following:
  * Fn::And
  * Fn::Equals
  * Fn::If
  * Fn::Not
  * Fn::Or
* Conditions can be applied to resources / outputs / etc...

### CloudFormation Intrinsic Functions

* Must know intrinsic Functions
  * Fn::Ref | !Ref
    * Parameters => returns the value of the parameter
    * Resources => returns the physical ID of the underlying resource
  * Fn::GetAtt | !GetAtt
    * The best practice is to check the documentation of each resource and find out each attribute available for it
  * Fn::FindInMap | !FindInMap [MapName, TopLevelKey, SecondLevelKey ]
  * Fn::ImportValue | !ImportValue
    * Import values that are exported in other templates
  * Fn::Join | !Join
    * Join values with a delimiter
  * Fn::Sub | !Sub
    * Used to substitute variables from a text. 
    * **String** must contain ${VariableName} and will substitute them
  * Condition Functions (listed in the section above)

### CloudFormation Rollbacks

* Stack Creation Fails:
  * Default: everything rolls back (gets deleted). We can check the logs
  * Option to disable rollback and troubleshoot the event
* Stack Update Fails:
  * The stack automatically rolls back to the previous known working state
  * Ability to see in the log what happened and error messages

### ChangeSets

* When you update a stack, you need to know what changes before it happens for greater confidence
* ChangeSets won't say if the update will be successful

### Nested stacks

* Nested stacks are part of other stacks
* They allow you to isolate repeated patterns / common components in separate stacks and call them from other stacks
* Example:
  * Load Balancer configuration that is re-used
  * Security Group that is re-used
* Nested stacks are considered best practice
* To update a nested stack, update the parent (root stack)

#### Cross Stacks vs Nested Stacks

* Cross Stacks
  * Helpful when stacks have different lifecycles
  * Use Outputs Exports and Fn::ImportValue
  * When you need to pass export values to many stacks
* Nested Stacks
  * Helpful when components must be re-used
  * The nested stack is only important to the higher level stack (not shared)