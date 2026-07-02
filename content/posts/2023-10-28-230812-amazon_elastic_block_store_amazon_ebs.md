---
title: "Amazon Elastic Block Store (Amazon EBS)"
draft: false
---

[AWS]({{< relref "20230314144515-aws.md" >}})

Amazon Elastic Block Store (Amazon EBS) provides persistent block storage volumes for use with [AWS EC2]({{< relref "2023-10-28-183931-aws_compute.md" >}}) instances in the AWS Cloud.

Each Amazon EBS volume is automatically replicated within its Availability Zone to protect you from component failure, offering high availability and durability.

Amazon EBS volumes offer the consistent and low-latency performance needed to run your workloads.

With Amazon EBS, you can scale your usage up or down within minutes – all while paying a low price for only what you provision.

The following EBS volumes appear most often on the AWS exams:

| Volume Type         | EBS Provisioned IOPS SSD (io1/io2)                                                    | EBS General Purpose SSD (gp2/gp3)                                                                        | Throughput Optimized HDD (st1)                                                        | Cold HDD (sc1)                                                         |
|---------------------|---------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------|------------------------------------------------------------------------|
| Short Description   | Highest performance SSD volume designed for latency-sensitive transactional workloads | General Purpose SSD volume that balances price performance for a wide variety of transactional workloads | Low-cose HDD volume, designed for frequently assessed. Throughput intensive workloads | Lowest cost HDD volume designed for less frequently assessed workloads |
| Use Cases           | I/O-intensive NoSQL and relational databases                                          | Boot volumes, low-latency interactive apps, dev &amp; test                                               | Big-data, data warehouses, log processing                                             | Colder data requiring fewer scans per day                              |
| Volume Size         | 4 GB -16 TB                                                                           | 1GB - 16TB                                                                                               | 125GB - 16TB                                                                          | 125GB - 16TB                                                           |
| Max IOPS / volume   | 64,000                                                                                | 16,000                                                                                                   | 500                                                                                   | 250                                                                    |
| Can be boot volume? | Yes                                                                                   | Yes                                                                                                      | Np                                                                                    | No                                                                     |
| EBS Multi-attach    | Supported                                                                             | Not Supported                                                                                            | Not Supported                                                                         | Not Supported                                                          |

EBS volume data persists independently of the life of the instance.

EBS volumes do not need to be attached to an instance.

You can attach multiple EBS volumes to an instance.

You cannot attach an EBS volume to multiple instances (use Elastic File Store instead).

EBS volumes must be in the same AZ as the instances they are attached to.

Termination protection is turned off by default and must be manually enabled (keeps the volume/data when the instance is terminated).

Root EBS volumes are deleted on termination by default.

Extra non-boot volumes are not deleted on termination by default.

The behavior can be changed by altering the “DeleteOnTermination” attribute.

EBS Snapshots:

-   Snapshots capture a point-in-time state of an instance.
-   Snapshots are stored on S3.
-   Does not provide granular backup (not a replacement for backup software).
-   If you make periodic snapshots of a volume, the snapshots are incremental, which means that only the blocks on the device that have changed after your last snapshot are saved in the new snapshot.
-   Even though snapshots are saved incrementally, the snapshot deletion process is designed so that you need to retain only the most recent snapshot to restore the volume.
-   Snapshots can only be accessed through the EC2 APIs.
-   EBS volumes are AZ specific, but snapshots are region specific.

{{< figure src="https://res.cloudinary.com/dkvj6mo4c/image/upload/v1698550280/aws/2023-10-28-23_30_30-screenshot_lb2r0q.png" >}}


## [Pricing]({{< relref "2023-11-05-021535-aws_billing_and_pricing.md" >}}) {#pricing--2023-11-05-021535-aws-billing-and-pricing-dot-md}

Pricing is based on three factors:

-   Volumes – volume storage for all EBS volumes type is charged by the amount of GB provisioned per month.
-   Snapshots – based on the amount of space consumed by snapshots in S3. Copying snapshots is charged on the amount of data copied across regions.
-   Data transfer – inbound data transfer is free, outbound data transfer charges are tiered.
