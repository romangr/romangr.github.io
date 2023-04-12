---
layout: post
title:  "Observability patterns for AWS Lambda"
date:   2023-04-11 21:18:00 +0300
categories: aws observability
description: Common observability patterns for AWS Lambda
---

### Covered cases

AWS has many manitoring and observability capabilities for AWS Lambda out of the box. For example, there are a lot of metrics
available in CloudWatch [by default](https://docs.aws.amazon.com/lambda/latest/dg/monitoring-metrics.html){:target="_blank"}.
Also there is an easily integrated tracing solution — [AWS X-Ray](https://docs.aws.amazon.com/lambda/latest/operatorguide/trace-requests.html){:target="_blank"}.
What is really missing by default — alarms. And it's reasonable because alarms should be adapted the particular use cases.
On the other hand there are some patterns that can be applied in most common cases for AWS Lambda based applications.

### Common patterns

#### SQS → Lambda integration

This is a case when Lambda function is triggered by SQS via [EventSourceMapping](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-eventsourcemapping.html){:target="_blank"}.
In this scenario retries are enabled by default via SQS configuration for Maximum Receives and Visibility Timeout so
there is no reason to fire an alarm when lambda fails during the processing. It makes sense to fire the alarm when
all retries have been exceeded. To achieve that alarm for SQS DLQ message visible metric can be created.

![](/assets/observability/sqs_lambda.png)

1. Lambda function is triggered via EventSourceMapping
2. When message has been received from the queue configured number of times and has not been deleted after that, message is placed to a DLQ
3. Alarm is triggered when there are visible messages in the DLQ
4. Operator redrives messages to the main queue when the issue is fixed

#### StepFunctions → Lambda (and other services) integration

When there is an integration state in a StepFunction (including Task state for AWS Lambda), retries can be configured on the StepFunction's side.
Therefore alarm should be triggered only when all this retries exceeded. Alarm for StepFunctions failed executions metric suits this purpose well.

![](/assets/observability/step_functions_other_services.png)

1. Lambda function fails when called from StepFunctions
2. Alarm is triggered by StepFunctions failed executions metric
3. Message with the StepFunctions execution input is sent to [DLQ mechanism](/aws/stepfunctions/2023/01/29/DLQ-for-stepfunctions.html){:target="_blank"}
4. Operator redrives messages to start a new StepFunctions execution when the issue is fixed

#### API Gateway → Lambda integration

When lambda is integrated with API gateway it's important that the gateway doesn't respond with server side errors.
To ensure that alarm is triggered by API Gateway 5XXError metric. This also covers errors in Authorizer lambdas and some other configuration issues.

![](/assets/observability/api_gateway_lambda.png)

1. API is called by a client
2. API Gateway calls a lambda function that fails
3. Alarm is triggered by API Gateway 5XXError metric

#### Scheduled EventBridge rule → Lambda integration

This type of integration works well when no specific input is needed for the function.
In this case DLQ mechanism is not needed since the function can be re-triggered without any specific input.
Therefore for such integration alarm can be triggered by Function Error metric.
However there are default retries applied for lambda [async invocations](https://docs.aws.amazon.com/lambda/latest/dg/invocation-async.html){:target="_blank"}, so the alarm configuration should take it into account and don't fire on the first failure of the lambda.

![](/assets/observability/scheduled_event_bridge_lambda.png)

1. Lambda function is triggered by scheduled EventBridge rule and fails
2. Alarm is triggered by Function Error metric