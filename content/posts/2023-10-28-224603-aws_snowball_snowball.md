---
title: "AWS Snowball (Snowball)"
draft: false
---

[AWS]({{< relref "20230314144515-aws.md" >}})

With AWS Snowball (Snowball), you can transfer hundreds of terabytes or petabytes of data between your [Private Cloud (on-premises)]({{< relref "2023-10-23-225546-private_cloud_on_premises.md" >}}) and [Amazon Simple Storage Service (S3)]({{< relref "2023-10-28-222841-amazon_simple_storage_service_s3.md" >}}).

Uses a secure storage device for physical transportation.

AWS Snowball Client is software that is installed on a local computer and is used to identify, compress, encrypt, and transfer data.

Uses 256-bit encryption (managed with the AWS KMS) and tamper-resistant enclosures with TPM.

The table below describes the AWS Snow offerings at a high-level:

| Service        | What it Is                                                                             |
|----------------|----------------------------------------------------------------------------------------|
| AWS Snowball   | Bulk data transfer, edge storage, and edge compute                                     |
| AWS Snowmobile | A literal shipping container full of storage (up to 100PB) and a truck to transport it |
| AWS Snowcone   | The smallest device in the range that is best suited for outside the data center       |

Snowball can import to S3 or export from S3.

Import/export is when you send your own disks into AWS – this is being deprecated in favor of Snowball.

Snowball must be ordered from and returned to the same region.

To speed up data transfer it is recommended to run simultaneous instances of the AWS Snowball Client in multiple terminals and transfer small files as batches.


## [Pricing]({{< relref "2023-11-05-021535-aws_billing_and_pricing.md" >}}) {#pricing--2023-11-05-021535-aws-billing-and-pricing-dot-md}

Pay a service fee per data transfer job and the cost of shipping the appliance.
Each job allows use of Snowball appliance for 10 days onsite for free.
Data transfer into AWS is free and outbound is charged (per region pricing).
