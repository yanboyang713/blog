---
title: "Amazon Simple Notification Service (Amazon SNS)"
draft: false
---

Amazon Simple Notification Service (Amazon SNS) is a web service that makes it easy to set up, operate, and send notifications from the cloud.

Amazon SNS is used for building and integrating loosely-coupled, distributed applications.

SNS provides instantaneous, push-based delivery (no polling).

SNS concepts:

-   Topics – how you label and group different endpoints that you send messages to.
-   Subscriptions – the endpoints that a topic sends messages to.
-   Publishers – the person/alarm/event that gives SNS the message that needs to be sent.

{{< figure src="https://res.cloudinary.com/dkvj6mo4c/image/upload/v1700965510/aws/2023-11-25-21_22_54-screenshot_oxsq1f.png" >}}

SNS usage:

-   Send automated or manual notifications.
-   Send notification to email, mobile (SMS), SQS, and HTTP endpoints.
-   Closely integrated with other AWS services such as CloudWatch so that alarms, events, and actions in your AWS account can trigger notifications.

Uses simple APIs and easy integration with applications.

Flexible message delivery is provided over multiple transport protocols.

Offered under an inexpensive, pay-as-you-go model with no up-front costs.

The web-based AWS Management Console offers the simplicity of a point-and-click interface.

Data type is JSON.

SNS supports a wide variety of needs including event notification, monitoring applications, workflow systems, time-sensitive information updates, mobile applications, and any other application that generates or consumes notifications.

SNS Subscribers:

-   HTTP.
-   HTTPS.
-   Email.
-   Email-JSON.
-   SQS.
-   Application.
-   Lambda.

SNS supports notifications over multiple transport protocols:

-   HTTP/HTTPS – subscribers specify a URL as part of the subscription registration.
-   Email/Email-JSON – messages are sent to registered addresses as email (text-based or JSON-object).
-   [Simple Queue Service (SQS)]({{< relref "2023-11-25-204942-simple_queue_service_sqs.md" >}}) – users can specify an SQS standard queue as the endpoint.
-   SMS – messages are sent to registered phone numbers as SMS text messages.

Topic names are limited to 256 characters.

SNS supports CloudTrail auditing for authenticated calls.

SNS provides durable storage of all messages that it receives (across multiple AZs).
