---
title: "handle long text when doing extraction"
date: 2025-04-20
draft: false
---

When working with files, like PDFs, you're likely to encounter text that exceeds your language model's context window. To process this text, consider these strategies:

1.  Change LLM Choose a different LLM that supports a larger context window.
2.  Brute Force Chunk the document, and extract content from each chunk.
3.  RAG Chunk the document, index the chunks, and only extract content from a subset of chunks that look "relevant".

Keep in mind that these strategies have different trade off and the best strategy likely depends on the application that you're designing!


## Reference List {#reference-list}

1.  <https://python.langchain.com/docs/how_to/extraction_long_text/>
