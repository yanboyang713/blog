---
title: "Amazon CloudFront"
draft: false
---

Amazon CloudFront is a [Content Delivery Network (CDN)]({{< relref "2023-10-26-011430-content_delivery_network_cdn.md" >}}) that allows you to store (cache) your content at “edge locations” located around the world.

This allows customers to access content more quickly and provides security against DDoS attacks.

CloudFront can be used for data, videos, applications, and APIs.

CloudFront benefits:

-   Cache content at Edge Location for fast distribution to customers.
-   Built-in Distributed [Denial of Service (DDoS)]({{< relref "2023-11-05-002954-denial_of_service_ddos.md" >}}) attack protection.
-   Integrates with many AWS services ([Amazon Simple Storage Service (S3)]({{< relref "2023-10-28-222841-amazon_simple_storage_service_s3.md" >}}), [AWS EC2]({{< relref "2023-10-28-183931-aws_compute.md" >}}), [Amazon Elastic Load Balancing (ELB)]({{< relref "2023-11-04-233523-amazon_elastic_load_balancing_elb.md" >}}), [Amazon Route 53]({{< relref "2023-11-05-001514-amazon_route_53.md" >}}), [AWS Lambda]({{< relref "2023-10-28-204435-aws_lambda.md" >}})).

Origins and Distributions:

-   An origin is the origin of the files that the CDN will distribute.
-   Origins can be either an S3 bucket, an EC2 instance, an Elastic Load Balancer, or Route 53 – can also be external (non-AWS).
-   To distribute content with CloudFront you need to create a distribution.
-   There are two types of distribution: Web Distribution and RTMP Distribution.

CloudFront uses Edge Locations and Regional Edge Caches:

-   An edge location is the location where content is cached (separate to AWS regions/AZs).
-   Requests are automatically routed to the nearest edge location.
-   Regional Edge Caches are located between origin web servers and global edge locations and have a larger cache.
-   Regional Edge caches aim to get content closer to users.

The diagram below shows where Regional Edge Caches and Edge Locations are placed in relation to end users:
![](https://res.cloudinary.com/dkvj6mo4c/image/upload/v1699158828/aws/2023-11-05-00_33_07-screenshot_vfvjr2.png)


## [Pricing]({{< relref "2023-11-05-021535-aws_billing_and_pricing.md" >}}) {#pricing--2023-11-05-021535-aws-billing-and-pricing-dot-md}

CloudFront pricing is determined by:

-   Traffic distribution – data transfer and request pricing, varies across regions, and is based on the edge location from which the content is served.
-   Requests – the number and type of requests (HTTP or HTTPS) and the geographic region in which they are made.
-   Data transfer out – quantity of data transferred out of CloudFront edge locations.
-   There are additional chargeable items such as invalidation requests, field-level encryption requests, and custom SSL certificates.
