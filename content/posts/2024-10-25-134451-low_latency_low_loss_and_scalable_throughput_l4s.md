---
title: "Low Latency, Low Loss, and Scalable Throughput (L4S)"
draft: false
---

L4S (Low Latency Low Loss Scalable Throughput) is a network architecture and protocol aimed at significantly improving latency for interactive applications while maintaining high throughput. It is an enhancement to existing congestion control mechanisms, especially for real-time and sensitive traffic such as gaming, video conferencing, and AR/VR. Here’s a breakdown of what it involves:

Key Concepts:
Dual-Queue Coupled AQM (Active Queue Management):

L4S relies on the use of two separate queues:
One queue for traditional TCP traffic.
Another queue for L4S-enabled flows that are latency-sensitive.
This separation helps avoid interference between the two types of traffic, ensuring low latency for real-time flows.
ECN (Explicit Congestion Notification):

L4S makes heavy use of ECN to signal congestion before it leads to packet loss.
It operates proactively by notifying senders of incipient congestion, giving them a chance to slow down transmission rates gracefully without causing delay or packet drops.
Scalable Congestion Control Algorithms:

L4S expects applications to implement congestion control algorithms that respond more quickly and smoothly to network feedback, such as DCTCP (Data Center TCP) or newer congestion control mechanisms like TCP Prague.
These algorithms adjust sending rates based on finer-grained ECN markings to maintain both low latency and high throughput.
