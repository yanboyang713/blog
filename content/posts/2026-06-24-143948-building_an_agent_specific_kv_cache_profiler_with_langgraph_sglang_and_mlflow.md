---
title: "Agent-Specific KV-Cache Profiler with LangGraph, SGLang, and MLflow"
date: 2026-06-24
tags: ["observability", "kv-cache", "langgraph", "sglang", "mlflow"]
draft: false
---

[Observability in LLM Inference Serving Engines: Comparing Metrics, Logging, Tracing, and Profiling]({{< relref "2026-06-23-224027-observability_in_llm_inference_serving_engines_comparing_metrics_logging_tracing_and_profiling.md" >}})

[Observability for AI Agent Frameworks: Comparing MLflow, LangSmith, Phoenix, Langfuse, Braintrust, W&amp;B Weave, and OpenTelemetry]({{< relref "2026-06-24-132210-observability_for_ai_agent_frameworks_comparing_mlflow_langsmith_phoenix_langfuse_braintrust_w_b_weave_and_opentelemetry.md" >}})

[KV-cache]({{< relref "2026-06-23-204356-kv_cache.md" >}})
[vLLM]({{< relref "2025-04-22-085318-vllm.md" >}})
[KVFlow: Efficient Prefix Caching for Accelerating LLM-Based Multi-Agent Workflows]({{< relref "2026-06-19-175540-kvflow.md" >}})


## Objective {#objective}

Multi-agent LLM applications contain more structure than ordinary independent inference requests. A planner may be invoked repeatedly, an executor may consume changing tool outputs, and a verifier may share substantial context with earlier agents. These differences affect prompt length, prefix reuse, prefill computation, KV-cache occupancy, and workflow latency.

A global cache-hit rate cannot explain these behaviors. It cannot tell us:

-   which agent benefited from cache reuse;
-   which workflow transition caused a cache miss;
-   whether one workflow evicted another workflow's useful prefixes;
-   whether the critical-path latency is dominated by queueing, prefill, decode, tools, or cache misses.

The project is to build an agent-specific KV-cache profiler using:

-   [LangGraph]({{< relref "2025-04-20-031839-langgraph.md" >}}) to define and identify agents, workflow state, turns, branches, loops, and transitions;
-   [SGLang]({{< relref "2026-06-17-143142-sglang.md" >}}) to serve the model and expose request, prefix-cache, queue, and timing telemetry;
-   [MLflow]({{< relref "2023-11-30-224657-mlflow.md" >}}) to collect traces, attach custom measurements, organize experiments, and compare configurations.

The initial objective is characterization rather than cache-policy optimization. Before designing a new eviction, compression, prefetch, or routing policy, the profiler should explain how real multi-agent workflows use the KV cache.


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

This information does not naturally exist inside an inference server. SGLang sees tokenized requests, but it does not inherently know that one request belongs to a planner and another belongs to a verifier.


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

MLflow is the trace and experiment-management layer. It is not the KV-cache profiler by itself. The cache measurements must come from SGLang, KVFlow, or custom backend instrumentation.


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
Planner -> Planner
Planner -> Executor
Executor -> Verifier
Verifier -> Planner
Tool Agent -> Executor
```

The unit of analysis is an ordered pair \\(A\_i \rightarrow A\_j\\), where \\(A\_i\\) is the previous agent and \\(A\_j\\) is the current agent.


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

\\[
T<sub>\text{workflow}</sub>
=
&sum;_i
\left(
T<sub>\text{queue},i</sub>

-

T<sub>\text{prefill},i</sub>

-

T<sub>\text{decode},i</sub>

-

T<sub>\text{tool},i</sub>
\right)
\\]

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
verifier
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
workflow               = planner-executor-verifier
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
  -> verifier
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

\\[
N\_{\text{new-prefill}}
=
\max(N\_{\text{prompt}} - N\_{\text{cached}}, 0)
\\]


### Reported request cache-hit ratio {#reported-request-cache-hit-ratio}

\\[
H\_{\text{request}}
=
\frac{N\_{\text{cached}}}{N\_{\text{prompt}}}
\\]

Name this `reported_cache_hit_ratio` because SGLang's cache alignment and engine-specific accounting may affect the exact denominator. Initially, total prompt tokens can be used as the denominator. Later, the calculation should account for tokens that are not cache-eligible.


### Workflow-level weighted hit ratio {#workflow-level-weighted-hit-ratio}

A simple average of request hit ratios can be misleading. A ten-token request and a ten-thousand-token request should not have equal weight.

\\[
H\_{\text{workflow}}
=
\frac{\sum\_i N\_{\text{cached},i}}
{\sum\_i N\_{\text{prompt},i}}
\\]


### Recompute burden {#recompute-burden}

\\[
B\_{\text{recompute}}
=
\sum\_i N\_{\text{new-prefill},i}
\\]

This may correlate more directly with workflow latency than average hit ratio.


### Time to first token {#time-to-first-token}

\\[
TTFT
=
t\_{\text{first-token}} - t\_{\text{request-submitted}}
\\]

Measure TTFT with streaming responses. Do not estimate TTFT by dividing total latency by token count.


### Time per output token {#time-per-output-token}

For responses with more than one output token:

\\[
TPOT
=
\frac{
t\_{\text{last-token}} - t\_{\text{first-token}}
}{
N\_{\text{output}} - 1
}
\\]


### End-to-end latency {#end-to-end-latency}

\\[
T\_{\text{E2E}}
=
t\_{\text{response-complete}} - t\_{\text{request-submitted}}
\\]


### Cache pressure {#cache-pressure}

\\[
P\_{\text{cache}}
=
\frac{N\_{\text{used-cache-tokens}}}{N\_{\text{cache-capacity}}}
\\]

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

Suggested repository structure:

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
-   Do not log full prompts and outputs unless the workload is synthetic and non-sensitive.
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
  -> Verifier
  -> END
```

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
    verification: str | None
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

Cache reuse depends on the final tokenized sequence, not only on apparent prompt text.

Record:

```text
prompt_template_version
chat_template_name
prompt_text_hash
token_id_hash
fixed_prefix_tokens
dynamic_suffix_tokens
```

Avoid storing raw production prompts unless necessary.

A useful prompt decomposition is:

\\[
P<sub>\text{agent}</sub>
=
P<sub>\text{shared}</sub>

-

P<sub>\text{role}</sub>

-

P<sub>\text{fixed-examples}</sub>

-

P<sub>\text{dynamic-state}</sub>

-

P<sub>\text{current-input}</sub>
\\]

where:

-   \\(P\_{\text{shared}}\\) is shared among agents;
-   \\(P\_{\text{role}}\\) identifies one agent;
-   \\(P\_{\text{fixed-examples}}\\) remains constant;
-   \\(P\_{\text{dynamic-state}}\\) changes with workflow execution;
-   \\(P\_{\text{current-input}}\\) contains the current task or message.

The profiler should identify the token index where two agent prompts diverge. This gives an application-side prediction of the maximum possible prefix-cache hit.


## Validation tests {#validation-tests}


### Test 1: cold request {#test-1-cold-request}

Restart SGLang and send one request.

Expected result:

```text
cached_tokens approximately 0
```

Some template-level or startup behavior may produce small deviations, so inspect the exact tokenized prompt.


### Test 2: exact repeated request {#test-2-exact-repeated-request}

Send the identical request twice.

Expected result:

```text
second.cached_tokens > first.cached_tokens
second.new_prefill_tokens < first.new_prefill_tokens
second.ttft_ms < first.ttft_ms
```


### Test 3: appended conversation {#test-3-appended-conversation}

Send a second request whose prompt contains the first request as an exact prefix plus additional messages.

Expected result:

```text
cached_tokens approximately equal to the reusable earlier prefix
```


### Test 4: one-token divergence {#test-4-one-token-divergence}

Construct two prompts that differ near the beginning.

Expected result:

```text
cached prefix ends near the first token difference
```


### Test 5: identifier completeness {#test-5-identifier-completeness}

Every model invocation must have:

```text
thread_id
workflow_run_id
agent_id
turn_id
request_uuid
MLflow span
```

Target:

```text
100% request-to-span join rate
```


### Test 6: cache-report sanity {#test-6-cache-report-sanity}

For every request:

\\[
0 \leq N\_{\text{cached}} \leq N\_{\text{prompt}}
\\]

and:

\\[
N\_{\text{new-prefill}}
=
N\_{\text{prompt}} - N\_{\text{cached}}
\\]


### Test 7: timing consistency {#test-7-timing-consistency}

For every request:

\\[
TTFT \leq T\_{\text{E2E}}
\\]

and, for streaming:

\\[
t\_{\text{submit}}
\leq
t\_{\text{first-token}}
\leq
t\_{\text{last-token}}
\leq
t\_{\text{complete}}
\\]


### Test 8: instrumentation overhead {#test-8-instrumentation-overhead}

Execute the same workload with:

-   profiling disabled;
-   MLflow autotracing only;
-   MLflow plus metrics scraper;
-   full SGLang request tracing;
-   source-level cache instrumentation.

Report the overhead rather than assuming it is negligible. A reasonable target for the normal application-level profiler is below roughly 5% latency overhead, but the measured value should decide whether the design is acceptable.


## Initial experimental matrix {#initial-experimental-matrix}


### Experiment A: Prefix caching enabled versus disabled {#experiment-a-prefix-caching-enabled-versus-disabled}

Run the same workflows with:

```text
radix cache enabled
radix cache disabled
```

Use SGLang's `--disable-radix-cache` option for the no-prefix-reuse baseline.

Measure:

-   workflow completion time;
-   per-agent TTFT;
-   new prefill tokens;
-   token throughput;
-   reported cache-hit ratio.

This experiment quantifies the total value of prefix caching.


### Experiment B: Cold versus warm workflows {#experiment-b-cold-versus-warm-workflows}

For cold-cache execution, restart or explicitly flush the server between workflow runs.

For warm-cache execution, invoke the same workflow repeatedly without clearing the cache.

This reveals:

-   cache warm-up time;
-   steady-state cache-hit ratio;
-   which agents benefit after repeated execution;
-   whether benefits stabilize after several runs.


### Experiment C: Self-agent reuse {#experiment-c-self-agent-reuse}

Invoke one agent repeatedly while extending its own history:

```text
Planner turn 1
Planner turn 2
Planner turn 3
```

This isolates reuse caused by an agent's own previous context.


### Experiment D: Cross-agent shared prefix {#experiment-d-cross-agent-shared-prefix}

Create controlled prompts where two agents share the same initial prefix:

```text
[shared application context]
[shared task description]
[agent-specific instruction]
```

Execute Agent A before Agent B and measure Agent B's cached tokens. Then reverse the execution order.


### Experiment E: Prompt-layout sensitivity {#experiment-e-prompt-layout-sensitivity}

Compare two prompt layouts.

Agent-first layout:

```text
You are the planner.
Large shared document...
```

Shared-prefix-first layout:

```text
Large shared document...
You are the planner.
```

Both prompts contain the same information, but only the second layout exposes the shared document as a reusable prefix across differently instructed agents.

Measure:

-   cached tokens;
-   new prefill tokens;
-   TTFT;
-   workflow latency.


### Experiment F: Workflow topology {#experiment-f-workflow-topology}

Compare:

```text
Sequential:
Planner -> Executor -> Verifier

Loop:
Planner -> Executor -> Verifier -> Planner

Branch:
Planner -> Executor A or Executor B -> Verifier

Parallel:
Planner -> {Executor A, Executor B} -> Aggregator
```

The objective is to determine how topology affects temporal reuse distance and cache survival.


### Experiment G: Cache pressure {#experiment-g-cache-pressure}

Gradually increase:

-   prompt length;
-   number of agents;
-   workflow concurrency;
-   number of distinct workflows.

Observe when cache-hit ratios begin to decline:

```text
capacity sufficient
  -> partial eviction
  -> heavy cache thrashing
```


### Experiment H: Inter-workflow interference {#experiment-h-inter-workflow-interference}

Run Workflow A alone and record its cache behavior. Then run Workflow A concurrently with unrelated Workflow B workloads.

\\[
\text{interference penalty}
=
T<sub>A,\text{concurrent}</sub>

-

T<sub>A,\text{isolated}</sub>
\\]

Also compare A's cached-token ratio under both conditions.


### Experiment I: Eviction-policy comparison {#experiment-i-eviction-policy-comparison}

Run identical workloads with each available `--radix-eviction-policy` option:

```text
lru
lfu
slru
priority
```

Compare:

-   global cache-hit rate;
-   per-agent cache-hit rate;
-   recompute burden;
-   p50 and p95 TTFT;
-   workflow completion time;
-   fairness among workflows.

A globally higher hit rate does not necessarily imply lower workflow completion time.


## Analysis and visualizations {#analysis-and-visualizations}


### Per-agent summary {#per-agent-summary}

For each agent, calculate:

-   request count;
-   mean and median prompt length;
-   mean and median cached tokens;
-   p50, p90, and p95 cache-hit ratio;
-   p50 and p95 TTFT;
-   p50 and p95 end-to-end latency;
-   total newly computed tokens.


### Agent-transition matrix {#agent-transition-matrix}

Construct a matrix where rows are previous agents and columns are current agents.

Each cell should contain:

-   number of transitions;
-   average cached tokens;
-   average cache-hit ratio;
-   average TTFT;
-   average new prefill tokens.

Example:

| Transition              | Hit ratio | New prefill | TTFT   |
|-------------------------|-----------|-------------|--------|
| Planner -&gt; Planner   | 82%       | 410 tokens  | 115 ms |
| Planner -&gt; Executor  | 56%       | 1180 tokens | 184 ms |
| Executor -&gt; Verifier | 22%       | 2340 tokens | 301 ms |
| Verifier -&gt; Planner  | 9%        | 3020 tokens | 372 ms |


### Cache reuse versus TTFT {#cache-reuse-versus-ttft}

Plot:

```text
x-axis: reported cache-hit ratio
y-axis: TTFT
color: agent
```

This reveals whether cache reuse produces similar benefits for all agents.


### Prompt tokens versus new prefill tokens {#prompt-tokens-versus-new-prefill-tokens}

Plot:

```text
x-axis: prompt tokens
y-axis: new prefill tokens
```

The distance between the diagonal and each point represents reused work.


### Cache pressure curve {#cache-pressure-curve}

Plot:

```text
x-axis: logical cache utilization
y-axis: per-agent cache-hit ratio
```

This reveals which agents are most sensitive to memory pressure.


### Workflow critical path {#workflow-critical-path}

Display the LangGraph trace with:

-   agent duration;
-   TTFT;
-   cached tokens;
-   new prefill tokens;
-   tool duration.

This shows whether cache optimization would materially shorten the critical path.


## Phase 8: Source-level SGLang instrumentation {#phase-8-source-level-sglang-instrumentation}

Application-level profiling answers how much cache was reused. It does not reveal cache provenance or exact eviction behavior.

The next stage is to modify SGLang or extend KVFlow-style metadata paths to emit cache lifecycle events.

Recommended event types:

```text
prefix_match
prefix_insert
cache_access
cache_lock
cache_unlock
cache_evict
cache_offload
cache_prefetch_start
cache_prefetch_complete
```

Example prefix-match event:

```json
{
  "timestamp_ns": 1840000123456,
  "event": "prefix_match",
  "request_uuid": "f47c...",
  "workflow_run_id": "incident-0042-run-01",
  "agent_id": "planner",
  "matched_tokens": 3072,
  "prompt_tokens": 4096,
  "cache_node_ids": [18, 27, 44]
}
```

Example eviction event:

```json
{
  "timestamp_ns": 1840000789012,
  "event": "cache_evict",
  "cache_node_id": 27,
  "evicted_tokens": 256,
  "last_access_request_uuid": "a12b...",
  "last_access_agent_id": "verifier",
  "residency_ms": 1832.4
}
```

Events should be emitted asynchronously or buffered to minimize scheduler overhead.


### Do not assign one permanent owner to a shared cache node {#do-not-assign-one-permanent-owner-to-a-shared-cache-node}

A radix-tree node may be reused by several agents and workflows.

Instead of:

```text
node.owner = planner
```

maintain an event history:

```text
node created by request A
node accessed by planner
node accessed by executor
node accessed by verifier
node evicted
```

Offline analysis can reconstruct:

-   first creator;
-   most recent accessor;
-   number of accesses by agent;
-   cross-agent reuse count;
-   residency time;
-   reuse distance.

This avoids imposing an incorrect single-owner model on shared cache nodes.


### Cache provenance metrics {#cache-provenance-metrics}

With cache-node metadata, calculate:

\\[
R\_{\text{self}}
=
\frac{\text{tokens reused from the same agent}}
{\text{all cached tokens}}
\\]

\\[
R\_{\text{cross-agent}}
=
\frac{\text{tokens reused from other agents}}
{\text{all cached tokens}}
\\]

\\[
R\_{\text{cross-workflow}}
=
\frac{\text{tokens reused from other workflows}}
{\text{all cached tokens}}
\\]

This instrumentation should remain optional because per-node access metadata can introduce substantial overhead.


## Integrating SGLang request traces {#integrating-sglang-request-traces}

SGLang can export request traces using OpenTelemetry by enabling `--enable-trace` and configuring `--otlp-traces-endpoint`. This can expose request-stage information around tokenizer and scheduler paths, and SGLang documents extension APIs for adding trace slices.

The first implementation should keep SGLang traces in Jaeger, Phoenix, Grafana Tempo, or another OpenTelemetry backend and correlate them with MLflow through `request_uuid`.

A later implementation can investigate unified distributed tracing:

```text
MLflow LangGraph span
  -> propagated trace context
  -> SGLang request span
     -> scheduler span
     -> prefill span
     -> decode span
```

MLflow supports distributed tracing through W3C TraceContext headers when both services log to the same tracking server and experiment. However, SGLang's native OpenTelemetry exporter and MLflow's trace ingestion should be validated together before assuming one unified trace. Request-ID correlation is simpler and should be completed first.


## Relationship to KVFlow {#relationship-to-kvflow}

KVFlow is relevant because it is explicitly workflow-aware. It models agent execution with an Agent Step Graph, uses steps-to-execution to guide fine-grained cache eviction, and introduces overlapped KV prefetching for agents expected in the next step.

This profiler can reuse the same conceptual direction but focus first on measurement:

```text
KVFlow-style metadata:
client_id
agent_id
steps_to_execution
fixed_prompt_boundary

Profiler metadata:
workflow_run_id
thread_id
turn_id
request_uuid
MLflow trace_id
MLflow span_id
```

Source-level instrumentation can then emit:

-   matched prefix length;
-   fixed-prefix hit length;
-   dynamic-suffix hit length;
-   inserted cache tokens;
-   evicted cache tokens;
-   GPU residency time;
-   CPU residency time;
-   prefetch duration.

That would produce more accurate measurements than relying only on OpenAI-compatible response usage.


## Privacy and logging safety {#privacy-and-logging-safety}

Prompts, model outputs, tool results, and workflow state may contain sensitive information.

Default policy:

-   log identifiers and token counts rather than raw prompt text;
-   use metadata-only SGLang request logging;
-   hash prompt templates when exact content is unnecessary;
-   redact credentials and tool outputs;
-   separate profiling metadata from application data;
-   define trace-retention policy;
-   disable full payload logging in shared deployments;
-   use synthetic data for initial experiments.

The profiler should include a redaction function before any text reaches MLflow.


## Reproducibility requirements {#reproducibility-requirements}

Every MLflow run should log:

-   source-code commit;
-   dependency lock file;
-   SGLang launch command;
-   model and tokenizer revisions;
-   chat template;
-   workflow graph definition;
-   prompt templates;
-   workload dataset or generation seed;
-   hardware information;
-   server logs;
-   raw request records;
-   sampled server metrics;
-   analysis output.

A benchmark result without its prompt template, model revision, cache configuration, and request order is not fully reproducible.


## Milestones {#milestones}


### Milestone 1: Basic request profiler {#milestone-1-basic-request-profiler}

Deliverables:

-   three-node LangGraph workflow;
-   SGLang cache reporting;
-   MLflow LangGraph traces;
-   per-request JSONL records;
-   cold-versus-warm validation.


### Milestone 2: Agent-transition characterization {#milestone-2-agent-transition-characterization}

Deliverables:

-   per-agent cache metrics;
-   transition matrix;
-   self-agent and controlled cross-agent experiments;
-   prompt-layout comparison.


### Milestone 3: Cache-pressure characterization {#milestone-3-cache-pressure-characterization}

Deliverables:

-   concurrency experiments;
-   cache-utilization sampling;
-   interference measurements;
-   eviction-policy comparison.


### Milestone 4: Source-level cache events {#milestone-4-source-level-cache-events}

Deliverables:

-   prefix-match and insertion events;
-   eviction events;
-   request-level event correlation;
-   event-overhead evaluation.


### Milestone 5: Cache provenance {#milestone-5-cache-provenance}

Deliverables:

-   self-reuse attribution;
-   cross-agent reuse attribution;
-   cross-workflow reuse attribution;
-   cache-residency analysis.


## Development schedule {#development-schedule}


### Week 1: Environment and baseline {#week-1-environment-and-baseline}

-   deploy MLflow locally;
-   launch SGLang with cache reporting and metrics;
-   build a three-node LangGraph workflow;
-   verify one successful inference request;
-   freeze configurations.


### Week 2: Request-level profiler {#week-2-request-level-profiler}

-   add workflow and request identifiers;
-   implement manual MLflow inference spans;
-   collect prompt, cached, output, and latency fields;
-   export the canonical request table.


### Week 3: Streaming and timing {#week-3-streaming-and-timing}

-   add TTFT and TPOT measurement;
-   validate timestamp ordering;
-   measure tracing overhead;
-   add error and cancellation handling.


### Week 4: Server metrics {#week-4-server-metrics}

-   implement the Prometheus scraper;
-   record logical cache utilization;
-   join server time series with request intervals;
-   create initial MLflow artifacts and plots.


### Week 5: Controlled experiments {#week-5-controlled-experiments}

-   cold versus warm cache;
-   self-agent reuse;
-   cross-agent reuse;
-   prompt reordering;
-   workflow interleaving.


### Week 6: Cache pressure and concurrency {#week-6-cache-pressure-and-concurrency}

-   vary cache capacity;
-   vary workflow concurrency;
-   construct agent-transition matrices;
-   identify workload regimes with eviction pressure.


### Week 7: Source-level instrumentation {#week-7-source-level-instrumentation}

-   add prefix-match, insertion, and eviction events;
-   propagate application identifiers into SGLang;
-   measure instrumentation overhead.


### Week 8: Analysis and report {#week-8-analysis-and-report}

-   finalize datasets;
-   generate plots;
-   document limitations;
-   compare findings with KVFlow;
-   prepare the research report or paper outline.


## Minimum viable profiler {#minimum-viable-profiler}

The minimum viable profiler is complete when it can:

1.  display a LangGraph workflow as an MLflow trace;
2.  identify every agent and model invocation;
3.  report prompt tokens and cached tokens for every request;
4.  calculate newly computed prompt tokens and reported cache-hit ratio;
5.  measure TTFT and end-to-end latency;
6.  group traces by workflow thread;
7.  export one canonical request table;
8.  compare cold-cache and warm-cache experiments;
9.  generate per-agent and transition-level summaries;
10. reproduce results from a saved configuration.

Eviction provenance, cache-node residency, and CPU-GPU cache movement are not requirements for the minimum viable version.


## Expected outcome {#expected-outcome}

The first outcome should be a characterization dataset rather than a new cache policy.

That dataset should reveal:

-   which agents generate the largest prompts;
-   which agents obtain the most cache reuse;
-   which transitions share token prefixes;
-   which prompt layouts reduce reuse;
-   when cache pressure begins to degrade performance;
-   whether global cache metrics hide agent-level imbalance;
-   how cache reuse affects total workflow completion time.

After these behaviors are understood, the same platform can support later work on:

-   workflow-aware eviction;
-   agent-aware cache priorities;
-   cache prefetching;
-   cache compression;
-   RL-based cache management;
-   replica-affinity routing;
-   prompt-layout optimization.


## Conclusion {#conclusion}

LangGraph, SGLang, and MLflow provide complementary components for agent-specific KV-cache profiling.

LangGraph supplies the logical workflow structure. SGLang supplies prefix-cache behavior and inference timing. MLflow supplies trace organization, experiment tracking, custom metadata, artifacts, and reproducible comparison.

The implementation should begin with per-request cached-token reporting, client-side timing, and application-side identifiers. It should then add server-level cache pressure and, only after those measurements are validated, extend SGLang or KVFlow with cache lifecycle and provenance events.

This staged approach minimizes engineering risk while producing useful results early. It also prevents the project from prematurely optimizing cache policies before the underlying multi-agent workload has been accurately characterized.


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
10. <https://opentelemetry.io/docs/concepts/context-propagation/>
