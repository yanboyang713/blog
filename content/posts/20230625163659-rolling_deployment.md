---
title: "Rolling Deployment"
draft: false
---

{{< figure src="https://res.cloudinary.com/dkvj6mo4c/image/upload/v1687726497/Rolling-Deployment_q7hhaj.png" >}}

Rolling Deployment applies phased deployment compared with [Big Bang Deployment]({{< relref "20230625163520-big_bang_deployment.md" >}}). The whole plant is upgraded one by one over a period of time.

It is more like a marathon than a sprint. This method lets us incrementally update different parts of the system over time. It's a staged rollout where we gradually deploy the new version of the application to the production envirment.


## Advantage {#advantage}

-   it usually prevents downtime. we are updating one server, the others are still up and running, serving our users.
-   we can spot and mitigate any issues early during the rollout. This reduces the rist of widespread problems. We're only ever exposing a small part of our system to the new at any one time.
-   popular choice for many teams, it balances risk and user impact in a controlled, methodical way.


## Disadvantage {#disadvantage}

-   rollong deployment is typically a slower process. while it reduces the risk of system-wide issues, it doesn't entirely eliminate it. If an issue slips out initial checks, it might still propagate as we update more servers.
-   This strategy does not support targeted rollouts. We cannot control which users get the new version during the rollout. All users will gradually see the new version as we update the servers. We can not direct the new version to specific users based on criteria like location, device type etc.
