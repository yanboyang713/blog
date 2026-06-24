---
title: "hierarchical radix cache"
date: 2026-06-19
draft: false
---

A hierarchical radix cache is a prefix KV cache that combines:

-   a radix tree for sharing common prompt prefixes, and
-   a memory hierarchy, typically GPU memory as the fast first level and CPU memory as the larger, slower second level.


## what is a radix cache? {#what-is-a-radix-cache}

Suppose several prompts begin with overlapping token sequences:

```text
Prompt 1: You are an expert programmer. Write Python code.
Prompt 2: You are an expert programmer. Review Python code.
Prompt 3: You are an expert mathematician. Solve this equation.
```

A radix tree stores their shared prefixes only once:

```text
You are an expert
├── programmer
│   ├── Write Python code
│   └── Review Python code
└── mathematician
    └── Solve this equation
```

Each tree node represents a segment of tokens and stores the corresponding KV tensors.
Therefore, the KV tensors for:

```text
You are an expert
```

do not need to be duplicated for every prompt.
When a new request arrives, the serving system walks down the tree and finds the longest matching prefix.
