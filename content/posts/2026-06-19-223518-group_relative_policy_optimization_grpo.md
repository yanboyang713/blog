---
title: "Group Relative Policy Optimization (GRPO)"
date: 2026-06-19
draft: false
---

It is a [Reinforcement Learning]({{< relref "2023-11-08-013737-reinforcement_learning.md" >}}) algorithm that improves a policy by comparing several sampled behaviors for the same input.

GRPO is closely related to PPO, or Proximal Policy Optimization, but it estimates the quality of an action using rewards relative to a group of sampled alternatives.
