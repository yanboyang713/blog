---
title: "Big Bang Deployment"
draft: false
---

{{< figure src="https://res.cloudinary.com/dkvj6mo4c/image/upload/v1687725987/Big-Bang-Deployment_aop0mo.png" >}}

Big Bang Deployment is quite straightforward, where we just roll out a new version in one go with service downtime. Preparation is essential for this strategy. We roll back to the previous version if the deployment fails.

-   Have service downtime (short) causes have to shutdown the old system to switch on new one
-   roll back still might disrupt users and there could be data implication.

Big Bang is sometimes the only choice, for example, when an intricate database upgrade is involved.


## Reference List {#reference-list}

1.  <https://cesar-alcancio.medium.com/how-to-avoid-big-bang-deployment-4dc901f2c1ca>
