---
title: "Small Language Models (SLMs)"
date: 2025-11-04
draft: false
---

## Overview {#overview}

Small Language Models (SLMs) are relatively lightweight language models designed to run efficiently in resource-constrained environments (e.g., laptops, smartphones, embedded/edge devices, or low-power servers).

Compared to frontier [Large Language Models (LLMs)]({{< relref "2023-12-01-141658-llms.md" >}}), SLMs typically use fewer parameters (commonly from ~1B up to ~10B) and smaller runtime footprints, while still supporting core NLP capabilities such as text generation, summarization, translation, and question answering.


## Terminology {#terminology}

> Some practitioners dislike the term “Small Language Model” because a billion parameters is not “small” in an absolute sense. Alternatives like “small LLM” exist, but “SLM” is widely used in practice.


## Why Use SLMs? {#why-use-slms}

-   Lower inference cost (memory, compute, power)
-   Lower latency (especially on-device)
-   Easier deployment in constrained environments
-   Potentially improved privacy when running locally (depending on the application)


## Trade-offs {#trade-offs}

-   Lower ceiling on capability compared to larger models (reasoning, long-context, tool use)
-   More sensitivity to data quality and training recipe
-   Smaller models may require tighter prompting, finetuning, or retrieval to match task requirements


## Examples {#examples}

Examples of commonly cited SLMs (~1–4B parameters):

-   `Llama 3.2 1B` (Meta)
-   `Qwen 2.5 1.5B` (Alibaba)
-   `DeepSeek-R1 1.5B` (DeepSeek; distilled from Qwen 2.5)
-   `SmolLM2 1.7B` (HuggingFaceTB)
-   `Phi-3.5 Mini 3.8B` (Microsoft)
-   [Gemma]({{< relref "2025-04-19-013506-google_gemma.md" >}}) (e.g., 4B-class variants)
-   [minimind]({{< relref "2025-11-04-234842-minimind.md" >}})

Other **small but strong** models sometimes mentioned include `Mistral 7B`, `Gemma 9B`, and `Phi-4 14B` (depending on your definition of “small”).


## References {#references}

-   [Hugging Face blog: Small language model](https://huggingface.co/blog/jjokah/small-language-model)
