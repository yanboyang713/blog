---
title: "resource groups and tagging"
draft: false
---

[AWS]({{< relref "20230314144515-aws.md" >}})

Tags are key / value pairs that can be attached to AWS resources.
Tags contain metadata (data about data).
Tags can sometimes be inherited – e.g. resources created by Auto Scaling, CloudFormation or Elastic Beanstalk.
Resource groups make it easy to group resources using the tags that are assigned to them.
You can group resources that share one or more tags.
Resource groups contain general information, such as:

-   Region.
-   Name.
-   Health Checks.

And specific information, such as:

-   Public &amp; private IP addresses (for EC2).
-   Port configurations (for ELB).
-   Database engine (for RDS).
