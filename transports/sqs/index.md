---
title: Amazon SQS Transport
summary: A transport for Amazon Web Services Simple Queue Service.
component: SQS
reviewed: 2017-10-16
related:
 - samples/sqs/simple
tags:
- AWS
redirects:
- nservicebus/sqs/index
---

[Simple Queue Service (SQS)](https://aws.amazon.com/sqs/) is a message queue service provided by [Amazon Web Services](https://aws.amazon.com/).


## Advantages

 * Fully managed turn-key messaging infrastructure. SQS queues requires very little effort to set up, maintain and manage over time.
 * Integrates seamlessly with other services provided by AWS, for example, [IAM](https://aws.amazon.com/documentation/iam/), [CloudWatch](https://aws.amazon.com/cloudwatch/), [Lambda](https://aws.amazon.com/lambda/), etc. For organizations already committed to AWS, SQS is a natural choice.
 * Can be used as a gateway between endpoints that may not have direct connectivity to each-other.


## Disadvantages

 * Like other message brokers, there is no local store-and-forward mechanism available. If an endpoint cannot reach SQS, either due to network problems or if SQS is unavailable, the endpoint will not be able to send nor receive messages.
 * Can be relatively expensive when using larger volumes of messages.


## Getting Started

An [AWS IAM](http://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html) account with a pair of [Access Keys](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-getting-started.html) is required.

The IAM account requires the following:


#### [SQS permissions](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-api-permissions-reference.html)

 * CreateQueue
 * DeleteMessage
 * DeleteMessageBatch
 * GetQueueUrl
 * ReceiveMessage
 * SendMessage
 * SendMessageBatch
 * SetQueueAttributes
 * ChangeMessageVisibility
 * ChangeMessageVisibilityBatch
 * PurgeQueue


#### [S3 permissions](http://docs.aws.amazon.com/AmazonS3/latest/dev/using-with-s3-actions.html)

 * CreateBucket
 * DeleteObject
 * GetObject
 * PutObject
 * PutLifecycleConfiguration
 * GetLifecycleConfiguration
 * ListAllMyBuckets

By default, [AWS Access Key ID and AWS Secret Access Key](http://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys) are discovered from environment variables of the machine that is running the endpoint:

 * Access Key ID goes in `AWS_ACCESS_KEY_ID`
 * Secret Access Key goes in `AWS_SECRET_ACCESS_KEY`

snippet: SqsTransport


## Retries and Timeouts


The SQS transport uses the default [retries and timeouts](http://docs.aws.amazon.com/sdk-for-net/v3/developer-guide/retries-timeouts.html) values implemented by the [AWS SDK for .NET](https://aws.amazon.com/sdk-for-net/):

| Parameter          | Default value |
|--------------------|---------------|
| `MaxErrorRetries`  | 4             |
| `RequestTimeout`   | 100s          |
| `ReadWriteTimeout` | 300s          |

NOTE: NServiceBus will perform [immediate](/nservicebus/recoverability/#immediate-retries) and [delayed](/nservicebus/recoverability/#delayed-retries) retries in addition to retries performed internally by the SQS client.




## Troubleshooting


### AmazonSQSException: Request is throttled

When sending messages it can happen that the send operation is throttled. When that happens the following exception is logged.
```
2017-11-14 23:10:24,314|ERROR|18|NServiceBus.Transports.SQS.MessageDispatcher|Exception from Send.
Amazon.SQS.AmazonSQSException: Request is throttled. ---> Amazon.Runtime.Internal.HttpErrorResponseException: The remote server returned an error: (403) Forbidden. ---> System.Net.WebException: The remote server returned an error: (403) Forbidden.
```

When this happens while receiving a message then this can safely be ignored as [recoverability features](/nservicebus/recoverability/) should resolve it. When sending messages outside a message handler those message operations can fail. Sending messages in bulk might increase the chance of being throttled on send to avoid flodding the Amazon SQS infrastructure.

By batching multiple concurrent message operations together in a smart way it is possible to adhere to the throttling constraints. For example send a maximum of 100 messages concurrently as outlined in the [sending large amount of messages guidelines](/nservicebus/handlers/async-handlers.md#concurrency-large-amount-of-concurrent-message-operations).


