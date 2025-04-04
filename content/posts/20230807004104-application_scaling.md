---
title: "application scaling"
draft: false
---

There are two common approaches to scaling an application:

-   [horizontally]({{< relref "20230807004425-horizontally_scaling.md" >}}): Where you start a new instance of application
-   [Vertically]({{< relref "20230807004333-vertically_scaling.md" >}}): Where you improve an independent application layer that has a bottleneck

The simplest way to scale a backend is to start another instance of the server. This will solve the issue, but in many cases it is a waste of hardware resources. For example, imagine you have a bottleneck in an application that collects or logs statistics. This might only use 15% of your CPU, because logging might include multiple IO operations but no intensive CPU operations. However, to scale this auxiliary function, you will have to pay for the whole instance.
