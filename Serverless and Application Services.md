# Key Notes

## Event-driven Architecture 
* No constant running or waiting for things. 
* Producers generate events when something happen. 
* Events are delivered to consumers. 
* Actions are taken and the system returns to waiting. 
* Mature event-driven architecture only consumes resources while handling events. 

## Lambda
* Function as a service. Short running (max 15min), focussed. 
* Functions are loaded and run in a runtime env. Env has direct memory (indirect cpu) allocation. 
* Common runtimes: python, ruby, java, go, c#. 
* Custom runtimes like rust are possible using layers. 
* 512 mb storage available as /tmp. Up to 10240. 
* Lambda usage scenarios:
    - Serverless apps (s3, apigw, lambda)
    - File processing (S3, s3 events, lambda)
    - Serverless cron (eventsbridge/cwevents, lambda)
    - RT stream data processing (kinesis, lambda)
* Lambda allows two connection modes:
    1. Public lambda: by default. Can access public aws services and public internet. Has no access to vpc by default. Offers best perfs. 
    2. VPC lambda: obey all VPC rules (so potentially no access to internet but access to vpc). 
* For VPC networking mode, AWS creates one ENI per SG/Subnet pair selection within Lambda. All lambda with the same pair are multiplexed to this ENI which is injected in the target vpc. 
* The above takes approx. 90s for the setup but only happens once during initial invocation. 
* Lambda needs execution roles to do things (including writing logs to cloudwatch). 
* Lambda resources policies (only accessible via api and cli) controls what svc and accounts can invoke lambda fn. 
* For logging, lambda uses cloudwatch (metrics), cloudwatch logs (logs) and xray (distributed tracing). 
* Lambda invocations:
    1. Synchronous:
        - CLI/API/Clients invoke a lambda, pass data in and wait for response. 
        - Results (success or failure) returned during the request. 
        - Errors or retries have to be handled within the client. 
    2. Asynchronous:
        - Calling svc isnt waiting for any kind of response. 
        - If processing event fails, lambda will retry between 0 and 2 times. 
        - Events can be sent to dead letter queue after repeated failure processing. 
        - Lambda has to be idempotent. 
        - Lambda supports destination (sqs,sns,lambda,eventsbridge) where success and fail events can be sent. 
    3. Event Source Mapping:
        - Used on streams or queue which dont support event generation to invoke lambda (kinesis, dyndb streams, sqs)
        - An event source mapping outside the lambda fn retrieves stream from a source and compiles them in batches, generating events made of batches and send to lambda. 
        - The source mapping need to have access to the source and shares the same perms as the lambda fn so even if the lamda fn doesnt touch directly the source, it still needs at least read perms to it. 
        - sqs queues or sns topics can be used for any discarded failed event batches. 
* Lambda fn have versions: v1, v2 etc. 
* A version is the code + the config of the lambda fn. 
* It is immutable and has its own arn. 
* $Latest points at the latest ver. 
* Aliases (dev stage prod) point at a version and can be changed. 
* Contexts:
    - An environment context is the environment a lambda function runs in. A cold start is a full ~100ms creation and config including code download. 
    - A lambda fn can reuse an environment context but has to assume it can't. Concurrent exec uses multiple ctx. 
    - Ctx are removed after awhile without being used. 
    - Provisioned concurrency can be used. AWS will create and keep x ctx warm and ready to use, improving start speed. 