---
title: "Network Load Balancer (NLB)"
draft: false
---

NLB is best suited for load balancing of TCP traffic where extreme performance is required.

Operating at the connection level (Layer 4), Network Load Balancer routes traffic to targets within [AWS virtual private cloud (VPC)]({{< relref "2023-10-29-194450-aws_virtual_private_cloud_vpc.md" >}}) and is capable of handling millions of requests per second while maintaining ultra-low latencies.

Network Load Balancer is also optimized to handle sudden and volatile traffic patterns.

{{< figure src="https://res.cloudinary.com/dkvj6mo4c/image/upload/v1699157033/aws/2023-11-05-00_03_04-screenshot_ok9imp.png" >}}
