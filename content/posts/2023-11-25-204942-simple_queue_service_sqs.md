---
title: "Amazon Simple Queue Service (SQS)"
draft: false
---

Amazon Simple Queue Service (SQS) is a distributed queue system.

Amazon SQS enables you to send, store, and receive messages between software components.

An Amazon SQS queue is a temporary repository for messages that are awaiting processing.

The SQS queue acts as a buffer between the component producing and saving data, and the component receiving the data for processing.

{{< figure src="https://res.cloudinary.com/dkvj6mo4c/image/upload/v1700965803/aws/2023-11-25-21_28_53-screenshot_de9xmj.png" >}}

This is known as decoupling / loose coupling and helps to enable elasticity for your application.

Amazon SQS is pull-based, not push-based (like [Amazon Simple Notification Service (Amazon SNS)]({{< relref "2023-11-25-205039-amazon_simple_notification_service_amazon_sns.md" >}})).
