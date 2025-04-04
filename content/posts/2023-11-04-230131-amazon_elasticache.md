---
title: "Amazon ElastiCache"
draft: false
---

{{< figure src="https://res.cloudinary.com/dkvj6mo4c/image/upload/v1699153758/aws/2023-11-04-23_08_24-screenshot_r6f6sx.png" >}}

ElastiCache is a web service that makes it easy to deploy and run Memcached or [Redis]({{< relref "2023-10-12-235342-redis.md" >}}) protocol-compliant server nodes in the cloud.

The in-memory caching provided by ElastiCache can be used to significantly improve latency and throughput for many read-heavy application workloads or compute-intensive workloads.

Best for scenarios where the DB load is based on Online Analytics Processing (OLAP) transactions.

The following table describes a few typical use cases for ElastiCache:

| Use Case                  | Benefit                                                                                                                                                                     |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Web session store         | In cases with load-balanced web servers, store web session information in Redis so if a server is lost, the session info is not lost, and another web server can pick it up |
| Database caching          | Use Memcached in front of AWS RDS to cache popular queries to offload work from RDS and return results faster to users                                                      |
| Leaderboards              | Use Redis to provide a live leaderboard for millions of users of your mobile app                                                                                            |
| Streaming data dashboards | Provide a landing spot for streaming sensor data on the factory floor, providing live real-time dashboard displays                                                          |

ElastiCache EC2 nodes cannot be accessed from the Internet, nor can they be accessed by EC2 instances in other VPCs.

Can be on-demand or reserved instances too (but not Spot instances).

ElastiCache can be used for storing session state.

There are two types of ElastiCache engine:

-   Memcached – simplest model, can run large nodes with multiple cores/threads, can be scaled in and out, can cache objects such as DBs.
-   Redis – complex model, supports encryption, master / slave replication, cross AZ (HA), automatic failover and backup/restore.
