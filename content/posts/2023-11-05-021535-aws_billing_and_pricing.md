---
title: "AWS Billing and Pricing"
draft: false
---

[AWS]({{< relref "20230314144515-aws.md" >}})

AWS works on a pay as you go model in which you only pay for what you use, when you are using it.

If you turn off resources, you don’t pay for them (you may pay for consumed storage).

There are no upfront charges, and you stop paying for a service when you stop using it.

Aside from EC2 reserved instances you are not locked into long term contracts and can terminate whenever you choose to.

Volume discounts are available so the more you use a service the cheaper it gets (per unit used).

There are no termination fees.

The three fundamental drivers of cost with AWS are: compute, storage, and outbound data transfer.

In most cases, there is no charge for inbound data transfer or for data transfer between other AWS services within the same region (there are some exceptions).

Outbound data transfer is aggregated across services and then charged at the outbound data transfer rate.

Free tier allows you to run certain resources for free.

Free tier includes offers that expire after 12 months and offers that never expire.

Pricing policies include:

-   Pay as you go.
-   Pay less when you reserve.
-   Pay even less per unit when using more.
-   Pay even less as AWS grows.
-   Custom pricing (enterprise customers only).

Free services include:

-   Amazon VPC.
-   Elastic Beanstalk (but not the resources created).
-   CloudFormation (but not the resources created).
-   Identity Access Management (IAM).
-   Auto Scaling (but not the resources created).
-   OpsWorks.
-   Consolidated Billing.

Fundamentally charges include:

1.  Compute.
2.  Storage.
3.  Data out.


## AWS COST &amp; USAGE REPORT {#aws-cost-and-usage-report}

Publish AWS billing reports to an Amazon S3 bucket.
Reports break down costs by:

-   Hour, day, month, product, product resource, tags.

Can update the report up to three times a day.
Create, retrieve, and delete your reports using the AWS CUR API Reference.


## AWS PRICE LIST API {#aws-price-list-api}

Query the prices of AWS services.
Price List Service API (AKA the Query API) – query with JSON.
AWS Price List API (AKA the Bulk API) – query with HTML.
Alerts via Amazon SNS when prices change.


## AWS BUDGETS {#aws-budgets}

Used to track cost, usage, or coverage and utilization for your Reserved Instances and Savings Plans, across multiple dimensions, such as service, or Cost Categories.
Alerting through event-driven alert notifications for when actual or forecasted cost or usage exceeds your budget limit, or when your RI and Savings Plans’ coverage or utilization drops below your threshold.
Create annual, quarterly, monthly, or even daily budgets depending on your business needs.
