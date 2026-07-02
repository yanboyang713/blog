---
title: "ServiceRouter: Hyperscale and Minimal Cost Service Mesh at Meta"
draft: false
---

ServiceRouter (SR), Meta’s global [service mesh]({{< relref "20221124053019-mesh_service.md" >}})

Hyperscale systems are designed to operate at such large volumes, providing high availability, efficient load distribution, and low-latency service across geographically dispersed data centers.

First, SR is designed for hyperscale and currently uses millions of L7 routers to route tens of billions of requests per second across tens of thousands of services.
Second, while SR adopts the common approach of using sidecar or remote proxies to route 1% of RPC requests in our fleet, it employs a routing library that is directly linked into service executables to route the remaining 99% directly from clients to servers, without the extra hop of going through a proxy.

Third, SR provides built-in support for sharded services, which account for 68% of RPC requests in our fleet, whereas existing general-purpose service meshes do not support sharded services.


## Reference List {#reference-list}

1.  <https://www.usenix.org/system/files/osdi23-saokar.pdf>
