---
title: "AWS Instance store volumes"
draft: false
---

Instance store volumes are high performance local disks that are physically attached to the host computer on which an [AWS EC2]({{< relref "2023-10-28-183931-aws_compute.md" >}}) instance runs.

Instance stores are ephemeral which means the data is lost when powered off (non-persistent).

Instances stores are ideal for temporary storage of information that changes frequently, such as buffers, caches, or scratch data.

Instance store volume root devices are created from [Amazon Machine Image (AMI)]({{< relref "2023-10-28-183931-aws_compute.md#amazon-machine-image--ami" >}}) templates stored on [Amazon Simple Storage Service (S3)]({{< relref "2023-10-28-222841-amazon_simple_storage_service_s3.md" >}}).

Instance store volumes cannot be detached/reattached.
