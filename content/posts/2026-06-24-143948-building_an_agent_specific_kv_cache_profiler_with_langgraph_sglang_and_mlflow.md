---
title: "Agent-Specific KV-Cache Profiler with LangGraph, SGLang, and MLflow"
date: 2026-06-24
tags: ["observability", "kv-cache", "langgraph", "sglang", "mlflow"]
draft: false
---

## Objective {#objective}

Multi-agent LLM applications contain more structure than ordinary independent inference requests. This project use exactly same [KVFlow]({{< relref "2026-06-19-175540-kvflow.md" >}}) workflow, which uses a PEER-style four-agent cycle: Planner, Executor, Expresser, and Reviewer. These agents can have different fixed prompts, different dynamic state, and different reuse distance across workflow steps. Those differences affect prompt length, prefix reuse, prefill computation, KV-cache occupancy, and workflow latency.

A global cache-hit rate cannot explain these behaviors. It cannot tell us:

-   which agent benefited from cache reuse;
-   which workflow transition caused a cache miss;
-   whether one workflow evicted another workflow's useful prefixes;
-   whether the critical-path latency is dominated by queueing, prefill, decode, tools, or cache misses.

The project is to build an agent-specific [KV-cache]({{< relref "2026-06-23-204356-kv_cache.md" >}}) profiler using:

-   [LangGraph]({{< relref "2025-04-20-031839-langgraph.md" >}}) to define and identify agents, workflow state, turns, branches, loops, and transitions;
-   [SGLang]({{< relref "2026-06-17-143142-sglang.md" >}}) to serve the model and expose request, prefix-cache, queue, and timing telemetry;
-   [MLflow]({{< relref "2023-11-30-224657-mlflow.md" >}}) to collect traces, attach custom measurements, organize experiments, and compare configurations.

The initial objective is characterization rather than cache-policy optimization. Before designing a new eviction, compression, prefetch, or routing policy, the profiler should explain how real multi-agent workflows use the KV cache.


## KVFlow workflow {#kvflow-workflow}

This note uses the same multi-agent workflow abstraction as KVFlow. The workflow is the PEER cycle:

```text
Planner -> Executor -> Expresser -> Reviewer -> Planner
```

The agents play the following roles:

Planner
: decomposes the user problem, decides the next step, and prepares instructions for execution.

Executor
: performs the planned work, such as retrieval, tool use, calculation, or synthesis of intermediate evidence.

Expresser
: turns the intermediate result into a clear answer or report for the user-facing side of the workflow.

Reviewer
: reviews the expressed result, performs self-assessment, identifies missing or weak parts, and may send the workflow back to the Planner for another cycle.

In KVFlow, these agents are important because each agent has a fixed prompt and appears at a known position in the workflow. The serving layer can use the current agent identity and future steps-to-execution to decide which KV-cache entries should be retained, evicted, or prefetched. This article therefore uses `reviewer`, not `verifier`, as the fourth agent.

KVFlow's motivating example is cache behavior across this cycle: when the Executor is active, its KV cache may evict the Expresser's cache under a simple LRU policy; when the Expresser becomes active again, the cache miss increases prefill latency. The profiler should therefore measure cache reuse by PEER agent and by PEER transition, not only by global cache-hit rate.


## Why this stack {#why-this-stack}


### LangGraph supplies workflow semantics {#langgraph-supplies-workflow-semantics}

LangGraph knows which logical component is executing. It can identify:

-   the current graph node;
-   the current agent;
-   the previous agent;
-   the workflow thread;
-   the current turn;
-   branches and loops;
-   tool calls and state transitions.

This information does not naturally exist inside an inference server. SGLang sees tokenized requests, but it does not inherently know that one request belongs to the Planner, Executor, Expresser, or Reviewer.


### SGLang supplies cache and serving telemetry {#sglang-supplies-cache-and-serving-telemetry}

SGLang uses radix-tree-based prefix caching. When requests share an identical token prefix, they can reuse corresponding KV-cache entries.

The relevant documented SGLang features are:

-   `--enable-cache-report`, which returns cached-token counts in `usage.prompt_tokens_details` for OpenAI-compatible requests;
-   `--enable-metrics`, which exposes Prometheus metrics at `/metrics`;
-   metrics such as `sglang:cache_hit_rate`, `sglang:token_usage`, `sglang:num_used_tokens`, `sglang:num_running_reqs`, `sglang:num_queue_reqs`, `sglang:gen_throughput`, TTFT histograms, TPOT histograms, and end-to-end latency histograms;
-   `--export-metrics-to-file` and `--export-metrics-to-file-dir` for per-request performance exports;
-   `--log-requests` with `--log-requests-level` and `--log-requests-format` for metadata or payload logging;
-   `--disable-radix-cache` for a no-prefix-cache baseline;
-   `--radix-eviction-policy` with options such as `lru`, `lfu`, `slru`, and `priority`;
-   `--enable-trace` and `--otlp-traces-endpoint` for OpenTelemetry request tracing.

SGLang is also sufficiently extensible for later source-level cache-event and provenance instrumentation.
Details compare with [LLM Inference Serving Engines in Observability]({{< relref "2026-06-23-224027-observability_in_llm_inference_serving_engines_comparing_metrics_logging_tracing_and_profiling.md" >}}).


### MLflow connects traces and experiments {#mlflow-connects-traces-and-experiments}

MLflow can automatically trace LangGraph executions through `mlflow.langchain.autolog()`. It records graph execution as traces and can capture nested spans for graph nodes, tools, model calls, and custom code blocks.

MLflow also supports manual spans and arbitrary custom attributes. A model-request span can therefore include fields such as:

```text
agent.id
workflow.id
request.uuid
kv.prompt_tokens
kv.cached_tokens
kv.new_prefill_tokens
kv.cache_hit_ratio
latency.ttft_ms
latency.e2e_ms
```

MLflow can store the configuration of each experiment, including model revision, tokenizer revision, cache capacity, eviction policy, workflow topology, prompt layout, and concurrency level.
MLflow is the trace and experiment-management layer. It is not the KV-cache profiler by itself. The cache measurements must come from SGLang, [KVFlow]({{< relref "2026-06-19-175540-kvflow.md" >}}), or custom backend instrumentation.
Details compare with [AI Agent Frameworks in Observability]({{< relref "2026-06-24-132210-observability_for_ai_agent_frameworks_comparing_mlflow_langsmith_phoenix_langfuse_braintrust_w_b_weave_and_opentelemetry.md" >}}).


## Research questions {#research-questions}


### RQ1: Do agents exhibit different cache behavior? {#rq1-do-agents-exhibit-different-cache-behavior}

For each agent, measure:

-   prompt-length distribution;
-   cached prompt tokens;
-   newly computed prefill tokens;
-   output tokens;
-   reported cache-hit ratio;
-   TTFT;
-   TPOT;
-   end-to-end latency;
-   change across workflow turns.


### RQ2: How does workflow position affect cache reuse? {#rq2-how-does-workflow-position-affect-cache-reuse}

Compare transitions such as:

```text
Planner -> Executor
Executor -> Expresser
Expresser -> Reviewer
Reviewer -> Planner
Planner -> Planner
```

The unit of analysis is an ordered pair \\((A\_i \rightarrow A\_j)\\), where \\((A\_i)\\) is the previous agent and \\((A\_j)\\) is the current agent.


### RQ3: How does prompt organization affect cache reuse? {#rq3-how-does-prompt-organization-affect-cache-reuse}

Compare prompt layouts where shared content appears before or after agent-specific instructions.

Cache-friendly:

```text
[shared task context]
[shared documents]
[agent-specific role]
[dynamic input]
```

Less cache-friendly:

```text
[agent-specific role]
[shared task context]
[shared documents]
[dynamic input]
```

Prefix caching depends on exact token-prefix identity, not semantic similarity.


### RQ4: How does cache pressure affect each agent? {#rq4-how-does-cache-pressure-affect-each-agent}

Vary:

-   KV-cache capacity;
-   number of concurrent workflows;
-   fixed prompt length;
-   dynamic suffix length;
-   output length;
-   workflow interleaving.

Measure which agents lose reuse first as pressure increases.


### RQ5: What is the benefit of cross-agent reuse? {#rq5-what-is-the-benefit-of-cross-agent-reuse}

Compare configurations where agents:

-   share no prefix;
-   share only a system prompt;
-   share task descriptions;
-   share documents and few-shot examples;
-   reuse a complete earlier conversation branch.


### RQ6: How does cache behavior affect workflow completion time? {#rq6-how-does-cache-behavior-affect-workflow-completion-time}

The primary objective should not be only global cache-hit rate.

For the KVFlow PEER cycle, this means:

```text
total workflow time
= Planner time
+ Executor time
+ Expresser time
+ Reviewer time
+ any repeated cycle time
```

An agent may have a low cache-hit rate but contribute little to total latency. Another agent may be latency-critical even if it is invoked infrequently.


## Architecture {#architecture}

The proposed architecture has three observability layers:

```text
LangGraph application
  knows: workflow, agent, transition, turn, graph state
  sends: OpenAI-compatible request plus correlation metadata

SGLang server
  knows: tokenization, prefix lookup, prefill, decode, cache use
  emits: usage, Prometheus metrics, logs, traces, optional cache events

MLflow
  stores: workflow traces, agent spans, inference spans
  stores: experiment parameters, metrics, artifacts, tables, plots
```

No single component has a complete view:

-   LangGraph understands the application but not the physical cache.
-   SGLang understands the cache but not the logical agent workflow.
-   MLflow stores and analyzes the combined information but does not generate cache telemetry by itself.

The profiler joins these layers through stable identifiers.


## Profiling levels {#profiling-levels}


### Level 1: Request-level profiling without modifying SGLang {#level-1-request-level-profiling-without-modifying-sglang}

Collect:

-   workflow identity;
-   agent identity;
-   previous agent identity;
-   prompt tokens;
-   cached prompt tokens;
-   newly computed prompt tokens;
-   output tokens;
-   end-to-end latency;
-   TTFT and TPOT when streaming is enabled;
-   reported cache-hit ratio.

This level is sufficient to characterize basic per-agent reuse and should be the first working prototype.


### Level 2: Correlation with server-level cache state {#level-2-correlation-with-server-level-cache-state}

Add:

-   global cache-hit rate;
-   occupied cache tokens;
-   logical cache utilization;
-   running requests;
-   queued requests;
-   generation throughput;
-   server-side latency distributions.

This level reveals how per-agent behavior changes under concurrency and cache pressure.

These are server-level values. They should not be treated as exact per-agent ownership under concurrency.


### Level 3: Source-level SGLang cache instrumentation {#level-3-source-level-sglang-cache-instrumentation}

Add:

-   prefix-match events;
-   cache insertion events;
-   cache-node access events;
-   eviction events;
-   cache residency time;
-   cache provenance;
-   host offloading;
-   device prefetching;
-   cache-node sharing among requests.

This level is required for exact self-agent reuse, cross-agent reuse, cross-workflow reuse, eviction attribution, and cache-lifecycle analysis.


## Identifier design {#identifier-design}

Reliable correlation requires more than one identifier.


### `benchmark_run_id` {#benchmark-run-id}

Identifies the experimental condition:

```text
warm-cache-concurrency-8-run-03
```


### `thread_id` {#thread-id}

Represents a persistent workflow session or conversation. For LangGraph, this maps naturally to the configurable `thread_id`:

```text
thread_id = kubernetes-incident-0042
```

MLflow 3.6 and later records LangGraph thread IDs in trace metadata when the graph is invoked with:

```python
graph.invoke(inputs, {"configurable": {"thread_id": "incident-0042"}})
```


### `workflow_run_id` {#workflow-run-id}

Identifies one invocation or resume of the graph:

```text
workflow_run_id = 550e8400-e29b-41d4-a716-446655440000
```

Several workflow runs may belong to the same long-lived `thread_id`.


### `agent_id` {#agent-id}

Identifies the logical role of the current node:

```text
planner
executor
expresser
reviewer
```

Agent names alone are not globally unique because concurrent workflows may each contain an agent named `planner`.


### `turn_id` {#turn-id}

Identifies the logical iteration of the workflow or conversation:

```text
turn_id = 3
```


### `request_uuid` or `agent_call_id` {#request-uuid-or-agent-call-id}

Identifies every model invocation:

```text
request_uuid = 54ac4f10-90ec-4e12-9a9e-c6c41dff67de
```

This should be the primary join key across:

-   the LangGraph node;
-   the MLflow span;
-   the HTTP request metadata;
-   SGLang request logs;
-   request-level performance exports;
-   future cache-event logs.

The identifier hierarchy is:

```text
thread_id
  -> workflow_run_id
     -> turn_id
        -> agent_id
           -> request_uuid
```


## MLflow data organization {#mlflow-data-organization}


### Experiment {#experiment}

One MLflow experiment represents the complete research project:

```text
agent-specific-kv-cache-profiling
```


### Run {#run}

One MLflow run represents one benchmark configuration.

Example parameters:

```text
model                  = Qwen/Qwen2.5-3B-Instruct
serving_engine         = sglang
prefix_cache           = enabled
radix_eviction_policy  = lru
workflow               = planner-executor-expresser-reviewer
concurrency            = 8
fixed_prompt_tokens    = 1024
cache_capacity         = default
random_seed            = 42
```

A run may execute hundreds of workflow instances. Do not create one MLflow run per model request.


### Trace {#trace}

One MLflow trace represents one LangGraph workflow execution:

```text
workflow_run_id = 550e8400-...
```


### Span {#span}

Spans represent stages inside the workflow:

```text
workflow
  -> planner
     -> sglang_inference
  -> executor
     -> tool_call
     -> sglang_inference
  -> expresser
     -> sglang_inference
  -> reviewer
     -> sglang_inference
```


## Canonical request schema {#canonical-request-schema}

MLflow traces provide visualization, but the project should also maintain a canonical request-level table for statistical analysis. Each row should represent one LLM invocation.

| Category     | Fields                                                                                       |
|--------------|----------------------------------------------------------------------------------------------|
| Experiment   | `benchmark_run_id`, `experiment_name`, `configuration_hash`, `timestamp`                     |
| Workflow     | `thread_id`, `workflow_run_id`, `workflow_type`, `workflow_concurrency`, `turn_id`           |
| Agent        | `agent_id`, `previous_agent_id`, `graph_node`, `request_uuid`                                |
| Request      | `model_name`, `sglang_response_id`, `prompt_tokens`, `output_tokens`                         |
| Cache        | `cached_tokens`, `new_prefill_tokens`, `reported_cache_hit_ratio`                            |
| Timing       | `queue_ms`, `ttft_ms`, `tpot_ms`, `prefill_ms`, `decode_ms`, `e2e_ms`                        |
| Server state | `cache_used_tokens_before`, `cache_used_tokens_after`, `running_requests`, `queued_requests` |
| Prompt       | `prompt_template_version`, `chat_template_name`, `prompt_hash`, `token_id_hash`              |
| Result       | `status`, `error_type`, `finish_reason`                                                      |

Store the final table as Parquet and log it as an MLflow artifact. JSONL is useful during development, but Parquet preserves types and is more efficient for repeated analytical queries.


## Metric definitions {#metric-definitions}


### Newly computed prompt tokens {#newly-computed-prompt-tokens}

```text
new_prefill_tokens = max(prompt_tokens - cached_tokens, 0)
```


### Reported request cache-hit ratio {#reported-request-cache-hit-ratio}

```text
reported_cache_hit_ratio = cached_tokens / prompt_tokens
```

Name this `reported_cache_hit_ratio` because SGLang's cache alignment and engine-specific accounting may affect the exact denominator. Initially, total prompt tokens can be used as the denominator. Later, the calculation should account for tokens that are not cache-eligible.


### Workflow-level weighted hit ratio {#workflow-level-weighted-hit-ratio}

Workflow-level weighted hit ratio asks:

```text
Across the whole workflow, what fraction of all prompt tokens came from the KV cache?
```

A simple average of request hit ratios can be misleading. A small request and a very large request should not have equal weight.

```text
workflow_cache_hit_ratio
= sum(cached_tokens for all requests)
  / sum(prompt_tokens for all requests)
```

For example:

```text
Request 1:
prompt_tokens = 100
cached_tokens = 90
request_hit_ratio = 90%

Request 2:
prompt_tokens = 10000
cached_tokens = 1000
request_hit_ratio = 10%
```

The simple average is:

```text
(90% + 10%) / 2 = 50%
```

But this is misleading because Request 2 is much larger. The weighted workflow hit ratio is:

```text
(90 + 1000) / (100 + 10000)
= 1090 / 10100
= about 10.8%
```

For the KVFlow PEER cycle, calculate it across all agent calls in one workflow run:

```text
workflow_cache_hit_ratio
= cached tokens from Planner, Executor, Expresser, and Reviewer
  / prompt tokens from Planner, Executor, Expresser, and Reviewer
```

This gives the cache reuse of the whole workflow, not just one agent call.


### Recompute burden {#recompute-burden}

Recompute burden asks:

```text
How many prompt tokens had to be recomputed instead of reused from the KV cache?
```

For one request:

```text
new_prefill_tokens = prompt_tokens - cached_tokens
```

For the whole workflow:

```text
recompute_burden = sum(new_prefill_tokens for all requests)
```

For example:

```text
Planner:
prompt_tokens = 4000
cached_tokens = 3000
new_prefill_tokens = 1000

Executor:
prompt_tokens = 8000
cached_tokens = 2000
new_prefill_tokens = 6000

Expresser:
prompt_tokens = 5000
cached_tokens = 4500
new_prefill_tokens = 500

Reviewer:
prompt_tokens = 3000
cached_tokens = 1000
new_prefill_tokens = 2000
```

Then:

```text
recompute_burden
= 1000 + 6000 + 500 + 2000
= 9500 tokens
```

This may correlate more directly with workflow latency than average hit ratio. Cached tokens are relatively cheap to reuse, while new prefill tokens require GPU computation. A workflow can have a decent cache-hit ratio but still be slow if the uncached portion is large.


### Time to first token {#time-to-first-token}

```text
ttft_ms = first_token_time_ms - request_submitted_time_ms
```

Measure TTFT with streaming responses. Do not estimate TTFT by dividing total latency by token count.


### Time per output token {#time-per-output-token}

For responses with more than one output token:

```text
tpot_ms
= (last_token_time_ms - first_token_time_ms)
  / (output_tokens - 1)
```


### End-to-end latency {#end-to-end-latency}

```text
e2e_latency_ms = response_complete_time_ms - request_submitted_time_ms
```


### Cache pressure {#cache-pressure}

```text
cache_pressure = used_cache_tokens / cache_capacity_tokens
```

Prefer SGLang's logical cache metrics over raw `nvidia-smi` memory. An inference server may reserve a large memory pool at startup even when relatively few logical cache entries are occupied.


## Phase 0: Reproducible environment {#phase-0-reproducible-environment}

Before implementing profiling, freeze the software and hardware configuration.

Record:

-   operating system;
-   Python version;
-   CUDA version;
-   GPU model and memory;
-   NVIDIA driver;
-   PyTorch version;
-   SGLang commit or package version;
-   LangGraph version;
-   LangChain version;
-   MLflow version;
-   model identifier and revision;
-   tokenizer revision;
-   chat template;
-   model precision;
-   KV-cache precision;
-   maximum context length;
-   SGLang launch arguments.

The exact chat template is especially important. Prefix caching operates on token identity, so a change in role markers, whitespace, or message serialization can change cache behavior.

Repository structure:

```text
agent-kv-profiler/
  configs/
    sglang.yaml
    workflow.yaml
    experiments/
  profiler/
    identifiers.py
    mlflow_tracing.py
    sglang_client.py
    metrics_scraper.py
    schemas.py
  workflows/
    synthetic/
    kubernetes_aiops/
  sglang_instrumentation/
  experiments/
  analysis/
  tests/
  artifacts/
```

Start with one SGLang process, one GPU, one model, and one sequential LangGraph workflow. Distributed serving should not be introduced until the profiler is validated.


## Phase 1: Start MLflow and SGLang {#phase-1-start-mlflow-and-sglang}


### MLflow {#mlflow}

A local MLflow server is sufficient for the first prototype:

```bash
mlflow server \
  --host 0.0.0.0 \
  --port 5000
```


### SGLang {#sglang}

A development launch configuration should enable only the telemetry needed for the first profiling level:

```bash
mkdir -p artifacts/sglang/request_metrics
mkdir -p artifacts/sglang/request_logs

python -m sglang.launch_server \
  --model-path "$MODEL_PATH" \
  --served-model-name profiler-model \
  --host 0.0.0.0 \
  --port 30000 \
  --enable-cache-report \
  --enable-metrics \
  --enable-request-time-stats-logging \
  --export-metrics-to-file \
  --export-metrics-to-file-dir artifacts/sglang/request_metrics \
  --log-requests \
  --log-requests-level 0 \
  --log-requests-format json \
  --log-requests-target artifacts/sglang/request_logs
```

Important design choices:

-   `--enable-cache-report` supplies per-request cached-token counts in OpenAI-compatible usage records.
-   `--enable-metrics` exposes Prometheus metrics.
-   `--log-requests-level 0` keeps request logging at metadata level.
-   For this profiler, raw production prompts may be logged intentionally when prompt-level cache debugging requires the exact text.
-   Do not enable every debugging and tracing option simultaneously, because instrumentation can change latency.

Use separate modes:

```text
baseline mode
application profiling mode
server tracing mode
source instrumentation mode
```

The experiment runner should save the complete launch command as an MLflow artifact.


## Phase 2: Minimal LangGraph workflow {#phase-2-minimal-langgraph-workflow}

The first workflow should be deliberately simple:

```text
START
  -> Planner
  -> Executor
  -> Expresser
  -> Reviewer
  -> Planner (next cycle)
```

For a finite benchmark, stop after a fixed number of PEER cycles or after the Reviewer decides that no more revision is needed.

The state should contain at least:

```python
from typing import TypedDict

class WorkflowState(TypedDict):
    thread_id: str
    workflow_run_id: str
    benchmark_run_id: str
    turn_id: int
    previous_agent_id: str | None
    messages: list[dict[str, str]]
    plan: str | None
    execution_result: str | None
    expression: str | None
    review: str | None
```

The initial workflow should avoid external tools. Tool calls can be introduced after the inference-only profiling pipeline is validated.

Each node should call one common SGLang wrapper rather than implementing its own request logic. This guarantees consistent identifiers and measurements across agents.


## Phase 3: MLflow tracing for LangGraph {#phase-3-mlflow-tracing-for-langgraph}

Initialize MLflow before constructing or invoking the graph:

```python
import mlflow

mlflow.set_tracking_uri("http://localhost:5000")
mlflow.set_experiment("agent-specific-kv-cache-profiling")

mlflow.langchain.autolog()
```

For asynchronous LangGraph execution with manual spans inside graph nodes, test:

```python
mlflow.langchain.autolog(run_tracer_inline=True)
```

MLflow documents this option as useful for nesting manual spans under autologged LangGraph traces in async scenarios. It should be tested carefully because sequential async invocations can merge traces unexpectedly.

Invoke the graph with a stable LangGraph thread identifier:

```python
result = graph.invoke(
    initial_state,
    config={
        "configurable": {
            "thread_id": initial_state["thread_id"],
        }
    },
)
```

The `thread_id` represents the long-lived workflow session, while `workflow_run_id` remains unique to this invocation.


## Phase 4: Instrumented SGLang client {#phase-4-instrumented-sglang-client}

Do not rely only on automatic LangGraph tracing. Create a manual span around every SGLang inference request so cache measurements can be attached to the exact model invocation.

The wrapper should:

1.  create a `request_uuid`;
2.  add correlation metadata;
3.  start an MLflow child span;
4.  record the request start time;
5.  measure the first generated token for streaming requests;
6.  collect the final usage record;
7.  read `cached_tokens`;
8.  calculate derived cache metrics;
9.  attach measurements to the span;
10. append one canonical JSONL record.


### Non-streaming prototype {#non-streaming-prototype}

This initial implementation measures cache reuse and end-to-end latency. Add streaming in the next iteration for TTFT and TPOT.

```python
from __future__ import annotations

import json
import time
import uuid
from pathlib import Path
from typing import Any

import mlflow
from mlflow.entities import SpanType
from openai import OpenAI


class ProfiledSGLangClient:
    def __init__(
        self,
        *,
        base_url: str,
        model: str,
        api_key: str = "EMPTY",
        event_file: str = "artifacts/request_records/requests.jsonl",
    ) -> None:
        self.client = OpenAI(base_url=base_url, api_key=api_key)
        self.model = model
        self.event_file = Path(event_file)

    def invoke(
        self,
        *,
        messages: list[dict[str, str]],
        thread_id: str,
        workflow_run_id: str,
        benchmark_run_id: str,
        agent_id: str,
        turn_id: int,
        previous_agent_id: str | None,
        max_tokens: int = 256,
    ) -> dict[str, Any]:
        request_uuid = str(uuid.uuid4())

        headers = {
            "x-request-uuid": request_uuid,
            "x-thread-id": thread_id,
            "x-workflow-run-id": workflow_run_id,
            "x-agent-id": agent_id,
            "x-turn-id": str(turn_id),
        }

        with mlflow.start_span(
            name="sglang:chat-completion",
            span_type=SpanType.LLM,
        ) as span:
            span.set_attributes(
                {
                    "benchmark.run_id": benchmark_run_id,
                    "thread.id": thread_id,
                    "workflow.run_id": workflow_run_id,
                    "agent.id": agent_id,
                    "agent.previous_id": previous_agent_id or "START",
                    "agent.turn_id": turn_id,
                    "request.uuid": request_uuid,
                    "model.name": self.model,
                    "model.max_tokens": max_tokens,
                }
            )

            start = time.perf_counter()

            response = self.client.chat.completions.create(
                model=self.model,
                messages=messages,
                temperature=0,
                max_tokens=max_tokens,
                extra_headers=headers,
            )

            end = time.perf_counter()
            e2e_ms = (end - start) * 1000

            raw = response.model_dump()
            usage = raw.get("usage") or {}
            details = usage.get("prompt_tokens_details") or {}

            prompt_tokens = int(usage.get("prompt_tokens") or 0)
            cached_tokens = int(details.get("cached_tokens") or 0)
            output_tokens = int(usage.get("completion_tokens") or 0)

            new_prefill_tokens = max(prompt_tokens - cached_tokens, 0)
            hit_ratio = cached_tokens / prompt_tokens if prompt_tokens else 0.0

            content = response.choices[0].message.content or ""

            attributes = {
                "request.sglang_response_id": raw.get("id", ""),
                "prompt.tokens": prompt_tokens,
                "prompt.cached_tokens": cached_tokens,
                "prompt.new_prefill_tokens": new_prefill_tokens,
                "prompt.reported_cache_hit_ratio": hit_ratio,
                "output.tokens": output_tokens,
                "latency.e2e_ms": e2e_ms,
            }

            span.set_attributes(attributes)
            span.set_outputs(
                {
                    "output_tokens": output_tokens,
                    "cached_tokens": cached_tokens,
                    "cache_hit_ratio": hit_ratio,
                }
            )

            record = {
                "timestamp_start_ns": int(start * 1_000_000_000),
                "timestamp_end_ns": int(end * 1_000_000_000),
                "status": "success",
                "benchmark.run_id": benchmark_run_id,
                "thread.id": thread_id,
                "workflow.run_id": workflow_run_id,
                "agent.id": agent_id,
                "agent.previous_id": previous_agent_id or "START",
                "agent.turn_id": turn_id,
                "request.uuid": request_uuid,
                "model.name": self.model,
                **attributes,
            }

            self.event_file.parent.mkdir(parents=True, exist_ok=True)
            with self.event_file.open("a", encoding="utf-8") as file:
                file.write(json.dumps(record) + "\n")

            return {
                "content": content,
                "request_uuid": request_uuid,
                "sglang_response_id": raw.get("id", ""),
                "prompt_tokens": prompt_tokens,
                "cached_tokens": cached_tokens,
                "new_prefill_tokens": new_prefill_tokens,
                "reported_cache_hit_ratio": hit_ratio,
                "output_tokens": output_tokens,
                "e2e_ms": e2e_ms,
            }
```

The custom HTTP headers are useful correlation keys. Confirm what your deployed SGLang version records in request logs. If headers are not preserved in logs or request objects, add a small frontend patch before relying on them for backend joins.


### Streaming timing {#streaming-timing}

The streaming client should record:

```text
t_submit
t_first_token
t_last_token
t_complete
```

Then calculate:

```python
ttft_ms = (t_first_token - t_submit) * 1000
e2e_ms = (t_complete - t_submit) * 1000

if output_tokens > 1:
    tpot_ms = (
        (t_last_token - t_first_token)
        / (output_tokens - 1)
    ) * 1000
else:
    tpot_ms = None
```

The final stream event should be inspected for usage and cached-token information. Test the installed SGLang version explicitly because streaming usage behavior may differ across releases and API modes. SGLang also provides `--stream-response-default-include-usage` if usage should be included by default in streaming responses.


## Phase 5: Integrate the client into LangGraph nodes {#phase-5-integrate-the-client-into-langgraph-nodes}

The LangGraph state should carry profiling context:

```python
from typing import TypedDict

class AgentState(TypedDict):
    thread_id: str
    workflow_run_id: str
    benchmark_run_id: str
    turn_id: int
    previous_agent_id: str | None
    messages: list[dict[str, str]]
    result: str | None
```

A planner node can call the profiled client:

```python
def planner_node(state: AgentState) -> dict:
    result = sglang_client.invoke(
        messages=state["messages"],
        thread_id=state["thread_id"],
        workflow_run_id=state["workflow_run_id"],
        benchmark_run_id=state["benchmark_run_id"],
        agent_id="planner",
        turn_id=state["turn_id"],
        previous_agent_id=state["previous_agent_id"],
    )

    return {
        "result": result["content"],
        "previous_agent_id": "planner",
        "turn_id": state["turn_id"] + 1,
    }
```

Apply the same interface to every agent. Do not allow individual nodes to define incompatible profiling fields.


## Phase 6: Server-state sampling {#phase-6-server-state-sampling}

The SGLang metrics endpoint should be sampled throughout each experiment.

At minimum, collect:

```text
sglang:cache_hit_rate
sglang:token_usage
sglang:num_used_tokens
sglang:num_running_reqs
sglang:num_queue_reqs
sglang:gen_throughput
sglang:time_to_first_token_seconds
sglang:time_per_output_token_seconds
sglang:e2e_request_latency_seconds
```

Use a fixed sampling interval:

```text
250 ms for short controlled experiments
1 s for longer experiments
```

Write an append-only artifact:

```text
timestamp_ns
benchmark_run_id
metric_name
metric_labels
metric_value
```

Do not store every high-frequency sample as a top-level MLflow metric. That can create excessive tracking overhead.

Use:

-   MLflow span attributes for per-request measurements;
-   Parquet or JSONL artifacts for raw time series;
-   MLflow run metrics for summaries such as averages and percentiles.

For each inference call, optionally capture:

```text
cache.used_tokens_before
cache.used_tokens_after
cache.utilization_before
cache.utilization_after
running_requests_before
queued_requests_before
```

Under concurrency, before-and-after differences cannot be attributed exclusively to one request. Treat them as context rather than ownership.


## Phase 7: Prompt instrumentation {#phase-7-prompt-instrumentation}

Prompt instrumentation means recording metadata about the prompt before it is sent to the model. The goal is to explain why a request did or did not reuse KV cache.

Cache reuse depends on the final tokenized sequence, not only on apparent prompt text. Two prompts can look similar to a human but produce different token prefixes because of role order, chat template changes, whitespace, separators, or message serialization.

Record:

```text
prompt_template_version
chat_template_name
raw_messages_json
serialized_prompt_text
raw_prompt_text
prompt_text_hash
token_id_hash
fixed_prefix_tokens
dynamic_suffix_tokens
first_divergence_token_index
```

For this profiler, store raw production prompts when the experiment requires exact prompt-level cache debugging. The raw prompt record should include both the application message structure and the final serialized prompt that reaches the tokenizer:

```text
raw_messages_json = original chat messages before serialization
serialized_prompt_text = exact prompt text after applying the chat template
raw_prompt_text = prompt text stored for inspection and replay
```

The hash fields should still be stored because they make grouping and joins easier:

```text
prompt_text_hash = hash(serialized_prompt_text)
token_id_hash = hash(token_ids)
```

A useful prompt decomposition is:

```text
agent_prompt
= shared_context
+ agent_role
+ fixed_examples
+ dynamic_state
+ current_input
```

Where:

-   `shared_context` is shared by Planner, Executor, Expresser, and Reviewer.
-   `agent_role` identifies the current KVFlow agent.
-   `fixed_examples` remain constant across requests.
-   `dynamic_state` changes as the workflow executes.
-   `current_input` contains the current task, user message, or tool result.

The profiler should record the exact token sequence and a hash of it:

```text
token_ids = tokenize(agent_prompt)
token_id_hash = hash(token_ids)
```

It should also identify where two agent prompts first diverge:

```text
first_divergence_token_index
= first token position where prompt A and prompt B differ
```

This gives an application-side estimate of the maximum possible prefix-cache reuse:

```text
maximum_possible_reused_tokens = first_divergence_token_index
```

For example, this layout is cache-friendly because the shared context appears first:

```text
shared_context
fixed_examples
agent_role
dynamic_state
current_input
```

This layout is less cache-friendly across agents because the prompt diverges immediately at `agent_role`:

```text
agent_role
shared_context
fixed_examples
dynamic_state
current_input
```

Prompt instrumentation helps distinguish cache misses caused by prompt layout from misses caused by eviction, cache pressure, or backend scheduling.


## Reference List {#reference-list}

1.  <https://mlflow.org/docs/latest/genai/tracing/integrations/listing/langgraph/>
2.  <https://mlflow.org/docs/latest/genai/tracing/app-instrumentation/manual-tracing/>
3.  <https://mlflow.org/docs/latest/genai/tracing/app-instrumentation/distributed-tracing/>
4.  <https://mlflow.org/docs/latest/genai/tracing/>
5.  <https://docs.sglang.io/docs/advanced_features/server_arguments>
6.  <https://docs.sglang.io/docs/advanced_features/observability>
7.  <https://docs.sglang.io/docs/references/production_metrics>
8.  <https://docs.sglang.io/docs/references/production_request_trace>
9.  <https://arxiv.org/abs/2507.07400>
10. <https://github.com/PanZaifeng/KVFlow>
11. <https://arxiv.org/abs/2407.06985>
12. <https://opentelemetry.io/docs/concepts/context-propagation/>
