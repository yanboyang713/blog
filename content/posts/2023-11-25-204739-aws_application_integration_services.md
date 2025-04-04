---
title: "AWS application integration services"
draft: false
---

The AWS application integration services are a family of services that enable decoupled communication between applications.

These services provide decoupling for microservices, distributed systems, and serverless applications.

AWS application integration services allow you to connect apps, without needing to write custom code to enable interoperability.

Decoupled applications can interoperate whilst being resilient to the failure or overload of any individual component.

The following services are involved with application integration:

| Service                                                                                                                                | What it does                                                                                                                                                         | Example use cases                                                                              |
|----------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| [Simple Queue Service (SQS)]({{< relref "2023-11-25-204942-simple_queue_service_sqs.md" >}})                                           | Messaging queue; store and forward patterns                                                                                                                          | Building distributed / decoupled applications                                                  |
| [Amazon Simple Notification Service (Amazon SNS)]({{< relref "2023-11-25-205039-amazon_simple_notification_service_amazon_sns.md" >}}) | Set up, operate, and send notifications from the cloud                                                                                                               | Send email notification when CloudWatch alarm is triggered                                     |
| [AWS Step Functions]({{< relref "2023-11-25-205139-aws_step_functions.md" >}})                                                         | Out-of-the-box coordination of AWS service components with visual workflow                                                                                           | Order processing workflow                                                                      |
| [Amazon Simple Workflow Service (SWF)]({{< relref "2023-11-25-205232-amazon_simple_workflow_service_swf.md" >}})                       | Need to support external processes or specialized execution logic                                                                                                    | Human-enabled workflows like an order fulfilment system or for procedural requests             |
| [Amazon MQ]({{< relref "2023-11-25-205335-amazon_mq.md" >}})                                                                           | Message broker service for [Apache Active MQ]({{< relref "2023-11-25-205526-apache_active_mq.md" >}}) and [RabbitMQ]({{< relref "2023-11-25-205459-rabbitmq.md" >}}) | Need a message queue that supports industry standard APIs and protocols; migrate queues to AWS |
