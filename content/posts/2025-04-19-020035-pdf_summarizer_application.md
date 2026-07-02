---
title: "PDF summarizer application"
date: 2025-04-19
draft: false
---

## Objective {#objective}

PDF summarizer application that downloads, extracts, and summarizes research papers(<https://arxiv.org/>) locally using [Gemma3]({{< relref "2025-04-19-013506-google_gemma.md" >}}) (4b) + [LangChain]({{< relref "2024-07-10-054216-langchain.md" >}}) + [Ollama]({{< relref "2025-04-14-230239-ollama.md" >}}).


## How it works ? {#how-it-works}

PDF Summarization application is designed to process research papers efficiently using Gemma 3, split them into manageable chunks, and generate structured summaries.


## Key Components of this PDF Summarizer {#key-components-of-this-pdf-summarizer}

-   User Uploads PDF — The user provides an ArXiv PDF URL in Streamlit, which sends a request to the FastAPI backend.
-   Text Extraction — The backend downloads the PDF and extracts text using PyMuPDF (Fitz).
-   Chunking with LangChain — RecursiveCharacterTextSplitter divides the text into manageable chunks for processing
-   Summarization with Gemma 3 — Each chunk is sent to Ollama, where Gemma 3 generates summaries in parallel for efficiency.
-   Final Output in Streamlit — Summaries are merged into a structured format using Pydantic , displayed in the UI, and available for download.


## Implementation Guide {#implementation-guide}

Let’s go through a step-by-step breakdown of this PDF extract and Summarization application.


### Step 1. Create a new directory for your project {#step-1-dot-create-a-new-directory-for-your-project}

```bash
mkdir pdfSummarizerApp
cd pdfSummarizerApp
```


### Step 2: Initialize the project using [uv]({{< relref "2025-04-19-021805-uv.md" >}}) (this creates pyproject.toml and .venv) {#step-2-initialize-the-project-using-uv--2025-04-19-021805-uv-dot-md----this-creates-pyproject-dot-toml-and-dot-venv}

```bash
uv init
```


### Step 3: Add dependencies to pyproject.toml {#step-3-add-dependencies-to-pyproject-dot-toml}

```bash
uv add fastapi uvicorn requests langchain pydantic pymupdf streamlit ollama httpx
```

-   FastAPI: For building the backend API.
-   Uvicorn: An ASGI server to run FastAPI applications.
-   Requests: For handling HTTP requests.
-   LangChain: For managing text processing and interaction with the language model.
-   Pydantic: For data validation within FastAPI.
-   PyMuPDF: For extracting text from PDFs.
-   Streamlit: For creating the frontend user interface.
-   Ollama: For running the Gemma 3 model locally.​[using 27B for demo]
-   httpx: For making asynchronous HTTP requests.


### Step 4: Activate the virtual environment (Linux/macOS) {#step-4-activate-the-virtual-environment--linux-macos}

```bash
source .venv/bin/activate
```


### Step 5: Run a quick test {#step-5-run-a-quick-test}

```bash
python -c "import fastapi, uvicorn, requests, langchain, pydantic, fitz, streamlit, httpx; print('All good!')"
```


### Step 6: Setup [ollama]({{< relref "2025-04-14-230239-ollama.md" >}}) and Download Gemma3 {#step-6-setup-ollama--2025-04-14-230239-ollama-dot-md--and-download-gemma3}

```console
yanboyang713@Meta-Scientific-Linux ~/projects/pdfSummarizerApp (git)-[master] % ollama list
NAME         ID              SIZE      MODIFIED
gemma3:4b    a2af6cc3eb7f    3.3 GB    8 hours ago
```

```console
yanboyang713@Meta-Scientific-Linux ~/projects/pdfSummarizerApp (git)-[master] % ollama show gemma3:4b
  Model
    architecture        gemma3
    parameters          4.3B
    context length      131072
    embedding length    2560
    quantization        Q4_K_M

  Capabilities
    completion
    vision

  Parameters
    stop           "<end_of_turn>"
    temperature    1
    top_k          64
    top_p          0.95

  License
    Gemma Terms of Use
    Last modified: February 21, 2024
```


### import packages {#import-packages}

```python
import os
import logging
import requests
import fitz
import asyncio
import json
import httpx
from concurrent.futures import ThreadPoolExecutor
from fastapi import FastAPI
from pydantic import BaseModel
from langchain.text_splitter import RecursiveCharacterTextSplitter
import ollama
```

[handle long text when doing extraction]({{< relref "2025-04-20-015209-handle_long_text_when_doing_extraction.md" >}})


## Reference List {#reference-list}

1.  <https://medium.com/google-cloud/running-googles-gemma-3-llm-langchain-locally-with-ollama-with-full-code-a57c94754393>
2.  <https://github.com/arjunprabhulal/gemma3_pdf_summarizer>
