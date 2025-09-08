---
title: "NetLLM: Adapting Large Language Models for Networking"
date: 2025-09-07
draft: false
---

[Large Language Models (LLMs)]({{< relref "2023-12-01-141658-llms.md" >}})

Problem:
Current deep learning (DL) approaches for networking tasks such as congestion control, adaptive bitrate streaming, and job scheduling face two major issues: high engineering costs for designing specialized deep neural networks (DNNs) for each task, and poor generalization to unseen network conditions

Motivation:
As networks grow in complexity and diversity, these limitations hinder deployment and scalability. Large Language Models (LLMs), with their pre-trained knowledge and emergent abilities, offer the potential for a single, general-purpose model to handle diverse networking tasks, reducing design effort and improving adaptability

Key Ideas:
The authors propose NetLLM, a framework for efficiently adapting LLMs to networking. It introduces three core components: (1) a multimodal encoder to process diverse inputs like time-series data and graphs, (2) a networking head that directly generates valid task-specific outputs without token-by-token prediction, and (3) a data-driven low-rank adaptation (DD-LRNA) method that reduces fine-tuning costs using pre-collected datasets and lightweight parameter updates

Limitations:
NetLLM’s success relies on high-quality pre-trained LLMs and curated datasets. Its evaluation covers only three networking tasks, leaving broader applicability untested.

Next Steps:
Future work could explore real-time deployment, scaling to additional network domains, and integrating with emerging multimodal LLMs for richer input handling.
