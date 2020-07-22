## Amazon Cognito

* Used to give users (outside of AWS) an identity to interact with our applications
* **Cognito User Pools**
  * Sign in functionality for app users
  * Integrate with API Gateway & Application Load Balancer
* **Cognito Identity Pools (Federated identity)**
  * Provide AWS credentials to users so they can access AWS resources directly
  * Integrate with Cognito User Pools as an identity provider
* **Cognito Sync**
  * Synchronize data from device to Cognito
  * Deprecated, replaced by AppSync
* **Cognito vs IAM**: Use cognito in the scenarios ("hundreds of users", "mobile users", "authenticate with SAML")

### Cognito User Pools (CUP)

#### User Features

* **Create a serverless database of user for your web & mobile apps**
* Simple login: Username (or email) / password combination
* Password reset
* Email & Phone Number Verification
* MFA
* Federated identities: users from Facebook, Google, SAML, etc...
* Feature: block users if their credentials are compromised elsewhere
* Login sends back a JSON Web Token (JWT)

#### Integrations

* CUP integrates with **API Gateway** and **Application Load Balancer**

#### Lambda Triggers

* CUP can invoke a Lambda function synchronously on these triggers: https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools-working-with-aws-lambda-triggers.html

#### Hosted Authentication UI

* Cognito has a hosted authentication UI that you can add to your app to handle sign-up and sign-in workflows
* Using the hosted UI, you have a foundation for integration with social logins, OIDC or SAML
* Can customize with a custom logo and Custom CSS

### Cognito Identity Pools (Federated Identities)

* **Get identities for "users" so they obtain temporary AWS credentials**
* Your identity pool (e.g. identity source) can include:
  * Public Providers (Login with Amazon, Facebook, Google, Apple)
  * Users in an Amazon Cognito User pool
  * OpenID Connect Providers & SAML Identity Providers
  * Developer Authenticated Identities (custom login server)
  * Cognito Identity Pools allow for **unauthenticated (guest) access**
* **Users can then access AWS services directly or through API Gateway**
  * The IAM policies applied to the credentials are defined in Cognito
  * They can be customized based on the user_id for fine grained control

#### IAM Roles

* Default IAM roles for authenticated and guest users
* Define rules to choose the role for each user based on the user's ID
* You can partition your users access using **policy variables**
* IAM credentials are obtained by Cognito Identity Pools through STS
* The roles must have a "trust" policy of Cognito Identity Pools

### Cognito User Pools vs Identity Pools

#### Cognito User Pools 

* Database of users for your web and mobile application
* Allows to federate logins through Public Social, OIDC, SAML...
* Can customize the hosted UI for authentication
* Has triggers with AWS Lambda during the authentication flow

#### Cognito Identity Pools

* Obtain AWS credentials for your users
* Users can login through Public Social, OIDC, SAML & Cognito User Pools
* Users can be unauthenticated (guests)
* Users are mapped to IAM roles & policies, can leverage policy variables

##### CUP (Manage user / password) + CIP (access AWS services)

### Cognito Sync

* Deprecated serivce - use AWS AppSync now
* Stores preferences, configuration, state of app
* Cross device synchronization (any platform - iOS, Android, etc...)
* Offline capability (sync when back online)
* Store data in datasets (up to 1MB), up to 20 datasets to synchronize
* **Push Sync**: silently notify alcross all devices when identity data changes
* **Cognito Stream**: stream data from Cognito into kinesis
* **Cognito Events**: execute Lambda functions in response to events

