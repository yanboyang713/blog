---
title: "the twelve-factor app methodology"
draft: false
---

## Introduction {#introduction}

In the modern era, software is commonly delivered as a service: called web apps, or [Software as a Service (SaaS)]({{< relref "20230812180305-software_as_a_service_saas.md" >}}). The twelve-factor app is a methodology for building software-as-a-service apps that:

-   Use declarative formats for setup automation, to minimize time and cost for new developers joining the project;
-   Have a clean contract with the underlying operating system, offering maximum portability between execution environments;
-   Are suitable for deployment on modern cloud platforms, obviating the need for servers and systems administration;
-   Minimize divergence between development and production, enabling [Continuous Deployment]({{< relref "20230812181208-ci_cd.md#what-is-continuous-deployment" >}}) for maximum agility;
-   And can scale up without significant changes to tooling, architecture, or development practices.

The twelve-factor methodology can be applied to apps written in any programming language, and which use any combination of backing services (database, queue, memory cache, etc).


## Who should read this document? {#who-should-read-this-document}

Any developer building applications which run as a service. Ops engineers who deploy or manage such applications.


## The Twelve Factors {#the-twelve-factors}

-   Codebase: One codebase tracked in revision control, many deploys
-   Dependencies: Explicitly declare and isolate dependencies
-   [Config]({{< relref "20230812203622-software_configurable_variable.md" >}}): Store config in the environment
-   Backing services: Treat backing services as attached resources
-   Build, release, run: Strictly separate build and run stages
-   Processes: Execute the app as one or more stateless processes
-   Port binding: Export services via port binding
-   Concurrency: Scale out via the process model
-   Disposability: Maximize robustness with fast startup and graceful shutdown
-   Dev/prod parity: Keep development, staging, and production as similar as possible
-   [Logging]({{< relref "20230812203021-logging.md" >}}): Treat logs as event streams
-   Admin processes: Run admin/management tasks as one-off processes


## Reference List {#reference-list}

1.  <https://12factor.net/>
