---
title: "AWS Global Infrastructure"
draft: false
---

[AWS]({{< relref "20230314144515-aws.md" >}})

An AWS Region is a physical location in the world where AWS have multiple AZs.

AZs consist of one or more discrete data centers, each with redundant power, networking, and connectivity, housed in separate facilities.

Each region is completely independent. Each Availability Zone is isolated, but the Availability Zones in a region are connected through low-latency links.


## Regions {#regions}

A region is a geographical area.

Each region consists of 2 or more availability zones.

Each Amazon Region is designed to be completely isolated from the other Amazon Regions.

Each AWS Region has multiple Availability Zones and data centers.

You can replicate data within a region and between regions using private or public Internet connections.

You retain complete control and ownership over the region in which your data is physically located, making it easy to meet regional compliance and data residency requirements.

Note that there is a charge for data transfer between regions. When you launch an EC2 instance, you must select an AMI that’s in the same region. If the AMI is in another region, you can copy the AMI to the region you’re using.

Regions and Endpoints:

-   When you work with an instance using the command line interface or API actions, you must specify its regional endpoint.
-   To reduce data latency in your applications, most Amazon Web Services offer a regional endpoint to make your requests.
-   An endpoint is a URL that is the entry point for a web service.
-   For example, <https://dynamodb.us-west-2.amazonaws.com> is an entry point for the Amazon DynamoDB service.


## Availability Zones {#availability-zones}

Availability Zones are physically separate and isolated from each other.

AZs span one or more data centers and have direct, low-latency, high throughput, and redundant network connections between each other.

Each AZ is designed as an independent failure zone.

When you launch an instance, you can select an Availability Zone or let AWS choose one for you.

If you distribute your EC2 instances across multiple Availability Zones and one instance fails, you can design your application so that an instance in another Availability Zone can handle requests.

You can also use Elastic IP addresses to mask the failure of an instance in one Availability Zone by rapidly remapping the address to an instance in another Availability Zone.

An Availability Zone is represented by a region code followed by a letter identifier; for example, us-east-1a.

To ensure that resources are distributed across the Availability Zones for a region, AWS independently map Availability Zones to names for each AWS account.

For example, the Availability Zone us-east-1a for your AWS account might not be the same location as us-east-1a for another AWS account.

To coordinate Availability Zones across accounts, you must use the AZ ID, which is a unique and consistent identifier for an Availability Zone.

AZs are physically separated within a typical metropolitan region and are in lower risk flood plains.

AZs use discrete UPS and onsite backup generation facilities and are fed via different grids from independent facilities.

AZs are all redundantly connected to multiple tier-1 transit providers.

The following graphic shows three AWS Regions each of which has three Availability Zones:

{{< figure src="https://res.cloudinary.com/dkvj6mo4c/image/upload/v1698296479/aws/2023-10-26-00_58_44-screenshot_eymyoc.png" >}}


## Local Zones {#local-zones}

AWS Local Zones place compute, storage, database, and other select AWS services closer to end-users.

With AWS Local Zones, you can easily run highly demanding applications that require single-digit millisecond latencies to your end-users.

Each AWS Local Zone location is an extension of an AWS Region where you can run your latency sensitive applications using AWS services such as Amazon Elastic Compute Cloud,Amazon Virtual Private Cloud, Amazon Elastic Block Store, Amazon File Storage, and Amazon Elastic Load Balancing in geographic proximity to end-users.

AWS Local Zones provide a high-bandwidth, secure connection between local workloads and those running in the AWS Region, allowing you to seamlessly connect to the full range of in-region services through the same APIs and tool sets.


## Wavelength Zones {#wavelength-zones}

AWS Wavelength enables developers to build applications that deliver single-digit millisecond latencies to mobile devices and end-users.

AWS developers can deploy their applications to Wavelength Zones, AWS infrastructure deployments that embed AWS compute and storage services within the telecommunications providers’ datacenters at the edge of the 5G networks, and seamlessly access the breadth of AWS services in the region.

AWS Wavelength brings AWS services to the edge of the 5G network, minimizing the latency to connect to an application from a mobile device.


### reference {#reference}

1.  <https://aws.amazon.com/wavelength/>
2.  <https://www.geeksforgeeks.org/what-is-aws-wavelength/>


## AWS Outposts {#aws-outposts}

AWS Outposts bring native AWS services, infrastructure, and operating models to virtually any data center, co-location space, or on-premises facility.

You can use the same AWS APIs, tools, and infrastructure across [Private Cloud (on-premises)]({{< relref "2023-10-23-225546-private_cloud_on_premises.md" >}}) and the AWS cloud to deliver a truly consistent hybrid experience.

AWS Outposts is designed for connected environments and can be used to support workloads that need to remain on-premises due to low latency or local data processing needs.

AWS Outposts is a fully managed service that offers the same AWS infrastructure, AWS services, APIs, and tools to virtually any datacenter, co-location space, or on-premises facility for a truly consistent hybrid experience.

AWS Outposts is ideal for workloads that require low latency access to on-premises systems, local data processing, data residency, and migration of applications with local system interdependencies.

AWS compute, storage, database, and other services run locally on Outposts, and you can access the full range of AWS services available in the Region to build, manage, and scale your on-premises applications using familiar AWS services and tools.

Outposts is available as a 42U rack that can scale from 1 rack to 96 racks to create pools of compute and storage capacity.

Services you can run on AWS Outposts include:

-   [AWS EC2]({{< relref "2023-10-28-183931-aws_compute.md" >}})
-   [Amazon Elastic Block Store (Amazon EBS)]({{< relref "2023-10-28-230812-amazon_elastic_block_store_amazon_ebs.md" >}})
-   [Amazon Simple Storage Service (S3)]({{< relref "2023-10-28-222841-amazon_simple_storage_service_s3.md" >}})
-   [AWS virtual private cloud (VPC)]({{< relref "2023-10-29-194450-aws_virtual_private_cloud_vpc.md" >}})
-   [Amazon Elastic Container Service (ECS)]({{< relref "2023-10-28-203025-amazon_elastic_container_service_ecs.md" >}})/[Amazon Elastic Kubernetes Service (EKS)]({{< relref "2023-10-29-211859-amazon_elastic_kubernetes_service_eks.md" >}}).
-   RDS.
-   EMR.


## Edge locations and regional edge caches {#edge-locations-and-regional-edge-caches}

Edge locations are [Content Delivery Network (CDN)]({{< relref "2023-10-26-011430-content_delivery_network_cdn.md" >}}) endpoints for [Amazon CloudFront]({{< relref "2023-11-05-002049-amazon_cloudfront.md" >}}).

There are many more edge locations than regions.

Currently there are over 200 edge locations.

Regional Edge Caches sit between your CloudFront Origin servers and the Edge Locations.

A Regional Edge Cache has a larger cache-width than each of the individual Edge Locations.

The following diagram shows CloudFront Edge locations:
![](https://res.cloudinary.com/dkvj6mo4c/image/upload/v1698297253/aws/2023-10-26-01_13_31-screenshot_i9x5vo.png)
