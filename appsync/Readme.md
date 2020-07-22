## AWS AppSync

* **AppSync** is a managed service that uses **GraphQL**
* **GraphQL** makes it easy to applications to get exactly the data they need
* This includes combining data from one or more sources
  * NoSQL data stores, Relational databases, HTTP APIs, etc...
  * Integrates with DynamoDB, Aurora, Elasticearch & others
  * Custom sources with AWS Lambda
* Retrieve data in **real-time with WebSocket or MQTT on WebSocket**
* For mobile apps: local data access & data synchronization
* It all starts with uploading one **GraphQL schema**

### AppSync - Security

* There are four ways you can authrorize applications to interact with your AWS AppSync GraphQL API
  * API_KEY
  * AWS_IAM: IAM users / roles / cross-account access
  * OPENID_CONNECT: OpenID Connect provider / JSON Web Token
  * AMAZON_COGNITO_USER_POOLS
* For custom domain & HTTPS, use CloudFront in front of AppSync
* 