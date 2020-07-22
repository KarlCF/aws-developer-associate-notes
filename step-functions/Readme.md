## AWS Step Functions

* Build serverless visual workflow to orchestrate your Lambda functions
* Represent flow as a JSON **state machine**
* Features: sequence, parallel, conditions, timeouts, error handling, etc...
* Can also integrate with EC2, ECS, on premise servers, API Gateway
* Maximum execution time of 1 year
* Possibility to implement human approval feature
* Use cases
  * Order fulfillment
  * Data Processing
  * Web applications
  * Any workflow

### Step Functions - Error Handling

* Any state can encounter runtime errors for various reasons:
  * State machine definition issues (for example, no matching rule in a Choice state)
  * Task failures (for example, an exception in a Lambda function)
  * Transient issues (for example, network partition events)
* By default, when a state reports an error, AWS Step Functions causes the execution to fail entirely
* **Retrying failures - Retry**: IntervalSeconds, MaxAttempts, BackoffRate
* **Moving on - Catch**: ErrorEquals, Next
* Best practice is to include data in the error messages

#### Step Functions - Standard vs Express

* Standard
  * Max duration = 1 year
  * Longer duration, slower execution
* Express
  * Max duration = 5 minutes
  * Short duration, very fast execution

