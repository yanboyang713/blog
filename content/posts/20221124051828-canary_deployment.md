---
title: "Canary Deployment"
draft: false
---

## Why need Canary Deployment {#why-need-canary-deployment}

[Murphy's law]({{< relref "2026-07-02-165446-murphy_law.md" >}})


## What Is Canary Deployment {#what-is-canary-deployment}

Canary deployments are a staged release practice. We roll out software updates to a small group of users first so they can test them and provide feedback. Once the changes are accepted, the update will be rolled out to the remaining users.

Itis often combined with [Rolling Deployment]({{< relref "20230625163659-rolling_deployment.md" >}}) to create an approach that Brings together the best of both worlds.

<https://semaphoreci.com/blog/what-is-canary-deployment#:~:text=In%20software%20engineering%2C%20canary%20deployment,the%20rest%20of%20the%20users>.

Canary deployments are a method of staged release [1]. To test the upgrades and get user input, we release them to a select sample of users first. The remaining users will receive the update after the modifications are approved. Canary deployments might be accomplished extremely well with [mesh service]({{< relref "20221124053019-mesh_service.md" >}}) ([istio]({{< relref "20230105194250-istio.md" >}})).

The most well-known open source service mesh software is called Istio. Service-to-service communication is handled by a separate infrastructure layer called a service mesh. It is in charge of ensuring that requests are delivered reliably through the intricate web of services that makes up a contemporary, cloud-native application. In actual use, the service mesh is usually implemented as a collection of thin network proxies that are installed concurrently with application code without the requirement for the application to be aware of it.


## Reference List {#reference-list}

1.  Canary Deployment strategy for Kubernetes (n.d.). Retrieved November 26, 2022, from <https://learn.microsoft.com/en-us/azure/devops/pipelines/ecosystems/kubernetes/canary-demo?view=azure-devops&tabs=yaml>
