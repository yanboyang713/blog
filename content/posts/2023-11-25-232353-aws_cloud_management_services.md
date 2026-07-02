---
title: "AWS Cloud Management services"
draft: false
---

The AWS Cloud Management services can be used for account management, configuration compliance, application delivery, and systems management.


## AWS organizations {#aws-organizations}

AWS organizations allows you to consolidate multiple AWS accounts into an organization that you create and centrally manage.

Available in two feature sets:

-   Consolidated Billing.
-   All features.

Includes root accounts and organizational units.

Policies are applied to root accounts or OUs.

Consolidated billing includes:

-   Paying Account – independent and cannot access resources of other accounts.
-   Linked Accounts – all linked accounts are independent.

{{< figure src="https://res.cloudinary.com/dkvj6mo4c/image/upload/v1700972826/aws/2023-11-25-23_25_50-screenshot_abr0m4.png" >}}


## AWS Control Tower {#aws-control-tower}

Simplifies the process of creating multi-account environments.

Sets up governance, compliance, and security guardrails for you.

Integrates with other services and features to setup the environment for you including:

-   AWS Organizations, SCPs, OUs, AWS Config, [AWS CloudTrail]({{< relref "2023-11-05-020331-aws_cloudtrail.md" >}}), [Amazon Simple Storage Service (S3)]({{< relref "2023-10-28-222841-amazon_simple_storage_service_s3.md" >}}), [Amazon Simple Notification Service (Amazon SNS)]({{< relref "2023-11-25-205039-amazon_simple_notification_service_amazon_sns.md" >}}), AWS CloudFormation, AWS Service Catalog, AWS Single Sign-On (SSO).

Examples of guardrails AWS Control Tower can configure for you include:

-   Disallowing public write access to [Amazon Simple Storage Service (S3)]({{< relref "2023-10-28-222841-amazon_simple_storage_service_s3.md" >}}) buckets.
-   Disallowing access as a root user without multi-factor authentication.
-   Enabling encryption for [Amazon Elastic Block Store (Amazon EBS)]({{< relref "2023-10-28-230812-amazon_elastic_block_store_amazon_ebs.md" >}}) volumes attached to [AWS EC2]({{< relref "2023-10-28-183931-aws_compute.md" >}}) instances.


## AWS Config {#aws-config}

AWS Config is a fully managed service that provides you with an AWS resource inventory, configuration history, and configuration change notifications to enable security and regulatory compliance.

With AWS Config, you can discover existing and deleted AWS resources, determine your overall compliance against rules, and dive into configuration details of a resource at any point in time.

AWS Config enables compliance auditing, security analysis, resource change tracking, and troubleshooting.

{{< figure src="https://res.cloudinary.com/dkvj6mo4c/image/upload/v1700973135/aws/2023-11-25-23_31_00-screenshot_wvst7b.png" >}}


## AWS Service Catalog {#aws-service-catalog}

AWS Service Catalog allows organizations to create and manage catalogs of IT services that are approved for use on AWS.

AWS Service Catalog allows you to centrally manage commonly deployed IT services.

IT services can include virtual machine images, servers, software, and databases and multi-tier application architectures.

Enables users to quickly deploy only the approved IT services they need.


## AWS Systems Manager {#aws-systems-manager}

Manages many AWS resources including Amazon EC2, Amazon S3, Amazon RDS etc.

Systems Manager Components:

-   Automation.
-   Run Command.
-   Inventory.
-   Patch Manager.
-   Session Manager.
-   Parameter Store.


## AWS Personal Health Dashboard {#aws-personal-health-dashboard}

AWS Personal Health Dashboard provides alerts and remediation guidance when AWS is experiencing events that may impact you.

Personal Health Dashboard gives you a personalized view into the performance and availability of the AWS services underlying your AWS resources.

The dashboard displays relevant and timely information to help you manage events in progress.

Also provides proactive notification to help you plan for scheduled activities.

Alerts are triggered by changes in the health of AWS resources, giving you event visibility, and guidance to help quickly diagnose and resolve issues.

You get a personalized view of the status of the AWS services that power your applications, enabling you to quickly see when AWS is experiencing issues that may impact you.

Also provides forward looking notifications, and you can set up alerts across multiple channels, including email and mobile notifications, so you receive timely and relevant information to help plan for scheduled changes that may affect you.

Alerts include remediation details and specific guidance to enable you to take immediate action to address AWS events impacting your resources.

Can integrate with Amazon CloudWatch Events, enabling you to build custom rules and select targets such as AWS Lambda functions to define automated remediation actions.

The AWS Health API allows you to integrate health data and notifications with your existing in-house or third-party IT Management tools.


## Service Health Dashboard {#service-health-dashboard}

AWS publishes up-to-the-minute information on service availability.

This information is not personalized to you (unlike Personal Health Dashboard).


## AWS OpsWorks {#aws-opsworks}

AWS OpsWorks is a configuration management service that provides managed instances of Chef and Puppet.

Updates include patching, updating, backup, configuration, and compliance management.


## AWS Trusted Advisor {#aws-trusted-advisor}

AWS Trusted Advisor is an online tool that provides you real time guidance to help you provision your resources following AWS best practices.

Trusted Advisor checks help optimize your AWS infrastructure, improve security and performance, reduce your overall costs, and monitor service limits.

AWS Basic Support and AWS Developer Support customers get access to 6 security checks (S3 Bucket Permissions, Security Groups – Specific Ports Unrestricted, IAM Use, MFA on Root Account, EBS Public Snapshots, RDS Public Snapshots) and 50 service limit checks.

AWS Business Support and AWS Enterprise Support customers get access to all 115 Trusted Advisor checks (14 cost optimization, 17 security, 24 fault tolerance, 10 performance, and 50 service limits) and recommendations.


## AWS CloudFormation {#aws-cloudformation}

AWS CloudFormation provides a common language for you to describe and provision all the infrastructure resources in your cloud environment.

CloudFormation allows you to use a simple text file to model and provision, in an automated and secure manner, all the resources needed for your applications across all regions and accounts.

This file serves as the single source of truth for your cloud environment.

You can use JSON or YAML to describe what AWS resources you want to create and configure.
