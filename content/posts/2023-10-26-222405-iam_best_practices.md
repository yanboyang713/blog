---
title: "IAM Best Practices"
draft: false
---

[AWS Identity and Access Management (IAM)]({{< relref "2023-10-26-011820-aws_identity_and_access_management_iam.md" >}})

Lock away the AWS root user access keys.
Create individual IAM users.
Use AWS defined policies to assign permissions whenever possible.
Use groups to assign permissions to IAM users.
Grant least privilege.
Use access levels to review IAM permissions.
Configure a strong password policy for users.
Enable MFA.
Use roles for applications that run on AWS EC2 instances.
Delegate by using roles instead of sharing credentials.
Rotate credentials regularly.
Remove unnecessary credentials.
Use policy conditions for extra security.
Monitor activity in your AWS account.
