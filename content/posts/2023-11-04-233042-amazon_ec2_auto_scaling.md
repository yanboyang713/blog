---
title: "Amazon EC2 Auto Scaling"
draft: false
---

{{< figure src="https://res.cloudinary.com/dkvj6mo4c/image/upload/v1699155211/aws/2023-11-04-23_32_43-screenshot_rzr3j1.png" >}}

Amazon EC2 Auto Scaling automates the process of launching (scaling out) and terminating (scaling in) Amazon EC2 instances based on the traffic demand for your application.

Auto Scaling helps to ensure that you have the correct number of EC2 instances available to handle the application load.

[AWS EC2]({{< relref "2023-10-28-183931-aws_compute.md" >}}) Auto Scaling provides elasticity and scalability.

You create collections of EC2 instances, called an Auto Scaling group (ASG).

You can specify the minimum number of instances in each ASG, and AWS Auto Scaling will ensure the group never goes beneath this size.

You can also specify the maximum number of instances in each ASG, and the group will never go above this size.

A desired capacity can be configured, and AWS Auto Scaling will ensure the group has this number of instances.

You can also specify scaling policies that control when Auto Scaling launches or terminates instances.

Scaling policies determine when, if, and how the ASG scales and shrinks (on-demand/dynamic scaling, cyclic/scheduled scaling).

Scaling Plans define the triggers and when instances should be provisioned/de-provisioned.

A launch configuration is the template used to create new EC2 instances and includes parameters such as instance family, instance type, AMI, key pair, and security groups.
