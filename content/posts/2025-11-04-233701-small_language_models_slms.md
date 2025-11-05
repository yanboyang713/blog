---
title: "Small Language Models (SLMs)"
date: 2025-11-04
draft: false
---

## What are Small Language Models? {#what-are-small-language-models}

Small Language Models (SLMs) are lightweight versions of traditional language models designed to operate efficiently on resource-constrained environments such as smartphones, embedded systems, or low-power computers. While large language models have hundreds of billions—or even trillions—of parameters, SLMs typically range from 1 million to 10 billion parameters. The small language models are significantly smaller but they still retain core NLP capabilities like text generation, summarization, translation, and question-answering.

"Some practitioners don't like the term "Small Language Model", because a billion parameter is not small by any means. They prefer "Small [Large Language Models]({{< relref "2023-12-01-141658-llms.md" >}})", which sounds convoluted. But the majority went with Small Language Model, so SLM it is. By the way, note that it is only small in comparison with the large models.


## Examples of Small Language Models {#examples-of-small-language-models}

Several small yet powerful language models have emerged, proving that size isn’t everything. The following examples are SLMs ranging from 1-4 billion parameters:

-   Llama3.2-1B – A Meta-developed 1-billion-parameter variant optimized for edge devices.
-   Qwen2.5-1.5B – A model from Alibaba designed for multilingual applications with 1.5 billion parameters.
-   DeepSeeek-R1-1.5B - DeepSeek's first-generation of reasoning model distilled from Qwen2.5 with 1.5 billion parameters.
-   SmolLM2-1.7B – From HuggingFaceTB, a state-of-the-art "small" (1.7 billion-parameter) language model trained on specialized open datasets (FineMath, Stack-Edu, and SmolTalk).
-   Phi-3.5-Mini-3.8B – Microsoft's tiny-but-might open model with 3.8 billion-parameters optimized for reasoning and code generation.
-   [Gemma]({{< relref "2025-04-19-013506-google_gemma.md" >}})3-4B - Developed by Google DeepMind, this light but powerful 4 billion-parameter model is multilingual and multimodal.
-   [minimind]({{< relref "2025-11-04-234842-minimind.md" >}})

Here are other more powerful small language models available out there: Mistral 7B, Gemma 9B, and Phi-4 14B (though I'm not sure if Phi-4 with 14 Billion parameters still qualifies as "small" but it's so capable :)


## Reference List {#reference-list}

1.  <https://huggingface.co/blog/jjokah/small-language-model>
