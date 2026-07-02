---
title: "AWS Elastic File System (EFS)"
draft: false
---

EFS is a fully managed service that makes it easy to set up and scale file storage in the Amazon Cloud.

Good for big data and analytics, media processing workflows, content management, web serving, home directories etc.

EFS uses the NFS protocol.

Pay for what you use (no pre-provisioning required).

Can scale up to petabytes.

EFS is elastic and grows and shrinks as you add and remove data.

Can concurrently connect 1 to 1000s of EC2 instances, from multiple AZs.

A file system can be accessed concurrently from all AZs in the region where it is located.

By default, you can create up to 10 file systems per account.

On-premises access can be enabled via Direct Connect or AWS VPN.

Can choose General Purpose or Max I/O (both SSD).

The VPC of the connecting instance must have DNS hostnames enabled.

EFS provides a file system interface, file system access semantics (such as strong consistency and file locking).

Data is stored across multiple AZs within a region.

Read after write consistency.

Need to create mount targets and choose AZs to include (recommended to include all AZ’s).

Instances can be behind an ELB.

There are two performance modes:

-   “General Purpose” performance mode is appropriate for most file systems.
-   “Max I/O” performance mode is optimized for applications where tens, hundreds, or thousands of EC2 instances are accessing the file system.

Amazon EFS is designed to burst to allow high throughput levels for periods of time.
![](https://res.cloudinary.com/dkvj6mo4c/image/upload/v1698551164/aws/2023-10-28-23_45_29-screenshot_l0zbay.png)
