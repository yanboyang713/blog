---
title: "AWS organizations and consolidated billing"
draft: false
---

[AWS]({{< relref "20230314144515-aws.md" >}})
[AWS Billing and Pricing]({{< relref "2023-11-05-021535-aws_billing_and_pricing.md" >}})

AWS organizations allows you to consolidate multiple AWS accounts into an organization that you create and centrally manage.

Available in two feature sets:

-   Consolidated Billing.
-   All features.

Includes root accounts and organizational units.
Policies are applied to root accounts or OUs.

Consolidated billing includes:

-   Paying Account – independent and cannot access resources of other accounts.
-   Linked Accounts – all linked accounts are independent.

Consolidated billing has the following benefits:

-   One bill – You get one bill for multiple accounts.
-   Easy tracking – You can track the charges across multiple accounts and download the combined cost and usage data.
-   Combined usage – You can combine the usage across all accounts in the organization to share the volume pricing discounts and Reserved Instance discounts. This can result in a lower charge for your project, department, or company than with individual standalone accounts.
-   No extra fee – Consolidated billing is offered at no additional cost.

Limit of 20 linked accounts (by default).
One bill for multiple AWS accounts.
Easy to track charges and allocate costs.
Volume pricing discounts can be applied to resources.
Billing alerts enabled on the Paying account include data for all Linked accounts (or can be created per Linked account).
Consolidated billing allows you to get volume discounts on all your accounts.
Unused reserved instances (RIs) for EC2 are applied across the group.
CloudTrail is on a per account basis and per region basis but can be aggregated into a single bucket in the paying account.

Best practices:

-   Always enable multi-factor authentication (MFA) on the root account.
-   Always use a strong and complex password on the root account.
-   The Paying account should be used for billing purposes only. Do not deploy resources into the Paying account.
