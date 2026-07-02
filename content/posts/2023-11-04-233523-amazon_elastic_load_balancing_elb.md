---
title: "Amazon Elastic Load Balancing (ELB)"
draft: false
---

[AWS]({{< relref "20230314144515-aws.md" >}})

ELB automatically distributes incoming application traffic across multiple targets, such as Amazon EC2 instances, containers, and IP addresses.

ELB can handle the varying load of your application traffic in a single Availability Zone or across multiple Availability Zones.

ELB features high availability, automatic scaling, and robust security necessary to make your applications fault tolerant.

There are four types of Elastic Load Balancer (ELB) on AWS:

-   [Application Load Balancer (ALB)]({{< relref "2023-11-05-000028-application_load_balancer_alb.md" >}}) – layer 7 load balancer that routes connections based on the content of the request.
-   [Network Load Balancer (NLB)]({{< relref "2023-11-05-000155-network_load_balancer_nlb.md" >}}) – layer 4 load balancer that routes connections based on IP protocol data.
-   Classic Load Balancer (CLB) – this is the oldest of the three and provides basic load balancing at both layer 4 and layer 7 (not on the exam anymore).
-   Gateway Load Balancer (GLB) – distributes connections to virtual appliances and scales them up or down (not on the exam).
