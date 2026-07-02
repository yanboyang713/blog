---
title: "Observability for AI Agent Frameworks: Comparing MLflow, LangSmith, Phoenix, Langfuse, Braintrust, W&B Weave, and OpenTelemetry"
date: 2026-06-24
tags: ["observability", "ai-agents", "llm"]
draft: false
---

## Problem {#problem}

An inference server can report cache hits, memory utilization, latency, scheduler state, and token throughput. It does not normally know that a request came from a planner, executor, reviewer, verifier, retriever, or tool-using agent.

An agent framework such as [LangGraph]({{< relref "2025-04-20-031839-langgraph.md" >}}) understands the application workflow:

```text
workflow
  -> planner
  -> executor
  -> tool call
  -> verifier
```

The inference server understands the model execution:

```text
request
  -> queueing
  -> prefix-cache lookup
  -> prefill
  -> decoding
  -> cache insertion
```

Agent-specific KV-cache profiling requires these two views to be connected. A useful trace should answer both questions:

-   Which agent, node, branch, or tool caused this model request?
-   How did that request interact with the inference backend's KV cache, scheduler, and prefill/decode path?

This note compares [MLflow]({{< relref "2023-11-30-224657-mlflow.md" >}}), [LangSmith]({{< relref "2025-04-20-033220-langsmith.md" >}}), Phoenix, Langfuse, Braintrust, W&amp;B Weave, and [OpenTelemetry]({{< relref "2025-09-16-194840-opentelemetry.md" >}}) with OpenInference according to how effectively they trace agent workflows, store custom cache telemetry, support experiments, and correlate application activity with inference-backend telemetry.


## Requirements for agent-specific cache observability {#requirements-for-agent-specific-cache-observability}


### Workflow-level trace {#workflow-level-trace}

One trace should represent a complete multi-agent workflow:

```text
workflow_id = incident-0042
```

The workflow trace should carry stable identifiers such as `workflow.id`, `session.id`, `thread.id`, `tenant.id`, and `experiment.id`.


### Agent-level spans {#agent-level-spans}

Each agent invocation should produce a child span:

```text
planner
executor
verifier
```

Useful span attributes include `agent.id`, `agent.role`, `graph.node.name`, `graph.edge.from`, `graph.edge.to`, `turn.id`, and `previous_agent.id`.


### Model-call spans {#model-call-spans}

The actual inference request should be nested under the responsible agent:

```text
workflow
  -> planner
     -> model request
```

The model span should carry `request.id` so it can be joined with backend logs, Prometheus exemplars, serving traces, or request-level JSONL records.


### Custom cache attributes {#custom-cache-attributes}

[KV-cache]({{< relref "2026-06-23-204356-kv_cache.md" >}})
The model span should accept fields such as:

```text
kv.prompt_tokens
kv.cached_tokens
kv.new_prefill_tokens
kv.cache_hit_ratio
kv.cache_occupied_tokens
kv.evicted_tokens
kv.host_transfer_bytes
latency.ttft_ms
latency.e2e_ms
```

OpenInference already defines standard token-cache fields such as `llm.token_count.prompt_details.cache_read` and `llm.token_count.prompt_details.cache_write`. Deeper serving-engine fields, such as cache occupancy, eviction count, block movement, and host transfer bytes, should be treated as project-specific extension attributes until a shared convention exists.


### Distributed trace propagation {#distributed-trace-propagation}

Trace context should cross the HTTP or RPC boundary:

```text
LangGraph
  -> traceparent
  -> SGLang or vLLM
  -> scheduler and cache manager
```

This is where OpenTelemetry matters most. The agent platform can show application causality, but end-to-end root-cause analysis requires the inference server to continue the same trace or at least record a join key that maps back to it.


### Experiment comparison {#experiment-comparison}

The observability tool should make it possible to compare:

-   [SGLang]({{< relref "2026-06-17-143142-sglang.md" >}}) versus [vLLM]({{< relref "2025-04-22-085318-vllm.md" >}});
-   radix caching versus disabled prefix caching;
-   different prompt layouts;
-   different cache capacities;
-   different agent transition patterns;
-   alternative eviction policies;
-   workflow concurrency levels;
-   serving-engine versions.

This requires more than traces. It also requires parameters, metrics, datasets, artifacts, query/export paths, and repeatable evaluation inputs.


## Summary comparison {#summary-comparison}

| Tool                          | LangGraph integration                                                                             | Custom telemetry                                                                 | Experiments and evaluations                                                  | Self-hosting                          | Open standards                                                                | Best fit                                                   |
|-------------------------------|---------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------|------------------------------------------------------------------------------|---------------------------------------|-------------------------------------------------------------------------------|------------------------------------------------------------|
| MLflow                        | Native LangGraph auto-tracing through LangChain; JS/TS path can use OpenTelemetry ingestion       | Excellent: manual spans and custom attributes fit cache metrics well             | Excellent: runs, params, metrics, artifacts, datasets, evals, traces         | Yes                                   | OpenTelemetry-compatible; supports GenAI conventions                          | Systems research and reproducible experiment tracking      |
| LangSmith                     | Best native LangGraph experience; minimal setup for graph traces                                  | Good: metadata and manual tracing work, but backend cache metrics must be added  | Excellent: offline and online evaluations, datasets, experiment comparison   | Cloud, hybrid, or self-hosted options | Supports OpenTelemetry ingestion/export paths, but LangSmith model is primary | Fast LangGraph debugging and production agent monitoring   |
| Phoenix                       | First-class AI traces through OpenInference instrumentation; LangChain/LangGraph path is natural  | Excellent: OpenTelemetry attributes and OpenInference semantics fit custom spans | Strong: evals, datasets, experiments, prompt tools                           | Yes                                   | OpenTelemetry and OpenInference                                               | Open-source distributed tracing across app and backend     |
| Langfuse                      | Strong LangGraph callback integration with support for multi-agent and nested traces              | Excellent: metadata, tags, scores, sessions, custom trace IDs, OTLP ingestion    | Strong: evals, datasets, dashboards, prompt management                       | Yes                                   | OpenTelemetry-oriented; OTLP ingestion                                        | Open-source LLM application observability                  |
| Braintrust                    | LangChain callback and auto-instrumentation work with LangGraph                                   | Excellent: nested spans, metrics, metadata, OTel compatibility                   | Excellent: production traces, datasets, scorers, experiments, online scoring | Options available                     | Custom SDK plus OpenTelemetry interoperability                                | Evaluation-centric agent development                       |
| W&amp;B Weave                 | Generic LangChain tracing; LangGraph likely needs more manual wrapping than LangSmith or Langfuse | Excellent: ops, attributes, custom kinds, versioned models                       | Excellent: Weave evaluations, W&amp;B ecosystem, model and prompt versioning | Primarily W&amp;B ecosystem           | Weave data model; not primarily OTel                                          | Teams already using W&amp;B                                |
| OpenTelemetry + OpenInference | Instrumentation layer, not a complete product                                                     | Maximum flexibility                                                              | Requires another backend or experiment system                                | Yes                                   | Yes                                                                           | Cross-service research telemetry and vendor-neutral traces |


## Tool notes {#tool-notes}


### MLflow {#mlflow}

MLflow is a strong candidate when the project combines systems measurements with repeatable experiments.

MLflow documents automatic tracing for LangGraph through its LangChain integration. Enabling `mlflow.langchain.autolog()` captures graph execution into traces, and manual tracing APIs can add child spans inside graph nodes or tools. MLflow also supports OpenTelemetry-compatible trace ingestion and emphasizes tracing, evaluation, production monitoring, datasets, parameters, metrics, and artifacts in one system.

Custom spans and attributes can carry cache telemetry:

```python
with mlflow.start_span(name="sglang_request") as span:
    span.set_attributes(
        {
            "workflow.id": workflow_id,
            "agent.id": agent_id,
            "request.id": request_id,
            "kv.prompt_tokens": prompt_tokens,
            "kv.cached_tokens": cached_tokens,
            "kv.cache_hit_ratio": hit_ratio,
            "latency.ttft_ms": ttft_ms,
        }
    )
```

MLflow's broader experiment model is especially useful for comparing serving configurations. A run can record:

```text
serving_engine = sglang
model = qwen
cache_policy = radix
cache_capacity = 32768
agent_framework = langgraph
workflow_concurrency = 8
```

Artifacts can include:

-   request-level CSV or JSONL records;
-   profiler traces;
-   Prometheus snapshots;
-   prompt templates;
-   graph definitions;
-   plots and statistical summaries.

MLflow's main advantage is that traces, metrics, parameters, artifacts, datasets, and evaluations can be organized in the same experimental system. Its main limitation is that it does not automatically understand KV-cache internals. Cache telemetry must still be obtained from SGLang, vLLM, TensorRT-LLM, or custom inference-server instrumentation.

Best fit: academic and systems experiments requiring reproducibility, quantitative comparison, and custom cache metrics.


### LangSmith {#langsmith}

LangSmith provides the most direct observability experience for LangGraph.

Because LangGraph and LangSmith are in the same ecosystem, tracing can capture graph execution, nodes, model calls, tool calls, inputs, outputs, thread or conversation organization, metadata, tags, and feedback with minimal setup. The documented LangGraph path can be as simple as enabling `LANGSMITH_TRACING=true` and providing an API key when using LangChain/LangGraph integrations.

This makes LangSmith especially effective for answering application-level questions:

-   Which node executed?
-   Which branch did the graph select?
-   Which tool did the agent call?
-   Where did an error occur?
-   How did agent state change?
-   Which prompt was sent to the model?

LangSmith also supports evaluation workflows: curated datasets, online and offline evaluation, evaluators, experiment comparison, and feedback loops from production traces back into datasets.

Custom metadata can include cache values, but LangSmith does not obtain backend KV-cache internals by itself. Connecting low-level SGLang or vLLM events to the same trace requires custom instrumentation and trace-context propagation. LangSmith does support OpenTelemetry-based tracing, but its strongest interface remains the LangChain/LangGraph-native model.

Best fit: fastest path to detailed LangGraph debugging and production agent monitoring.


### Phoenix {#phoenix}

Phoenix is an open-source AI observability and evaluation platform built around OpenTelemetry and OpenInference.

Phoenix is useful for cache profiling because the application and inference server can use the same telemetry pipeline:

```text
LangGraph
  -> OpenInference spans
  -> OpenTelemetry Collector
  -> Phoenix

SGLang or vLLM
  -> OpenTelemetry spans
  -> OpenTelemetry Collector
  -> Phoenix
```

Phoenix accepts OTLP traces and is built on OpenTelemetry with OpenInference instrumentation. OpenInference adds AI-specific span kinds and attributes for LLM calls, agents, tools, retrieval, prompts, token usage, and graph nodes. This is directly relevant to agent-specific cache work because standard fields can represent agent spans, LLM spans, tool spans, prompt token counts, and cache-read/cache-write token counts.

Custom spans can represent:

-   prefix-cache lookup;
-   cache allocation;
-   eviction;
-   CPU offload;
-   GPU prefetch;
-   model prefill;
-   decoding.

Phoenix also provides evaluations, datasets, experiments, and prompt-development capabilities. Compared with MLflow, Phoenix is more directly centered on AI traces and OpenTelemetry. Compared with LangSmith, it is less tightly coupled to LangGraph and more suitable for heterogeneous frameworks and services.

Best fit: open-source, standards-based correlation between agent traces and inference-server telemetry.


### Langfuse {#langfuse}

Langfuse is an open-source LLM engineering and observability platform that can be self-hosted.

Its tracing model centers on traces, sessions, observations, generations, events, custom metadata, scores, and dashboards. Langfuse documents a LangGraph cookbook using the LangChain callback handler, trace names, run names, tags, metadata, user IDs, session IDs, multi-agent traces, nested agents, and custom trace IDs.

A Langfuse representation might be:

```text
session: workflow-0042
  -> trace: workflow invocation
     -> planner observation
        -> SGLang generation
     -> executor observation
     -> verifier observation
```

Custom metadata can hold cached-token and latency information. Evaluations and dashboards can then aggregate those measurements across agents or workflow types.

Langfuse also supports OpenTelemetry ingestion through OTLP endpoints and maps OTel attributes into its trace and observation model. This makes it a practical bridge when part of the system uses Langfuse SDKs and another part emits plain OpenTelemetry spans.

Langfuse offers a good balance between:

-   open-source deployment;
-   LLM-specific user interfaces;
-   custom instrumentation;
-   session-level organization;
-   agent graph visualization;
-   OpenTelemetry ingestion.

Its limitation is similar to the other application observability tools: low-level cache events require additional instrumentation in the serving backend.

Best fit: open-source LLM-specific tracing, sessions, dashboards, evaluations, and self-hosting.


### Braintrust {#braintrust}

Braintrust combines AI tracing, evaluation, datasets, experiments, and production feedback.

Its instrumentation captures nested spans for application logic, tools, LLM calls, token usage, latency, costs, errors, and custom metadata. The LangChain callback and auto-instrumentation path works with LangGraph, and Braintrust also provides OpenTelemetry integration and compatibility mode for distributed tracing between OTel-instrumented code and Braintrust spans.

For a cache project, Braintrust can support experiments such as:

```text
same workflow inputs
  -> SGLang LRU
  -> workflow-aware eviction
  -> RL-based eviction
```

Braintrust's strongest characteristic is the tight relationship between production traces and evaluations. A failed or slow workflow trace can become an evaluation example, and alternative configurations can be compared against the same data. It is therefore a strong choice when evaluation workflow matters as much as tracing.

The central tradeoff is that Braintrust is more evaluation-product oriented than Phoenix or raw OpenTelemetry. If the project prioritizes open telemetry plumbing and inference-backend research instrumentation, Phoenix or OpenTelemetry may be a cleaner baseline. If the project prioritizes continuous evaluation and trace-to-dataset workflows, Braintrust is strong.

Best fit: evaluation-centric development where production traces are continuously converted into experiments.


### W&amp;B Weave {#w-and-b-weave}

W&amp;B Weave traces LLM calls, custom functions, and agent workflows. Its `op` abstraction wraps application components and captures inputs, outputs, timing, code versions, and custom display properties.

Weave also provides:

-   trace visualization;
-   custom attributes;
-   operation kinds such as agent, LLM, tool, and search;
-   token and cost tracking;
-   datasets;
-   structured evaluations;
-   comparison of experimental runs;
-   model and prompt versioning;
-   integration with the broader Weights &amp; Biases ecosystem.

A LangGraph node could be wrapped as a Weave operation:

```python
@weave.op(kind="agent")
def planner_node(state):
    ...
```

Cache metrics could be returned from the operation, stored as attributes, or logged in the model-call result.

Weave is especially attractive when the research workflow already uses W&amp;B for model experiments. It can visualize relationships such as:

```text
cache-hit ratio versus TTFT
cached tokens versus prompt length
agent identity versus eviction count
```

The limitation is LangGraph specificity. Weave has strong support for LangChain tracing and documented support for several agent frameworks, including AutoGen. Dedicated LangGraph ergonomics are less prominent than LangSmith, MLflow, Phoenix, Langfuse, or Braintrust, so more manual instrumentation may be required.

Best fit: teams already invested in W&amp;B that want combined tracing, evaluation, code versioning, and visualization.


### OpenTelemetry and OpenInference {#opentelemetry-and-openinference}

OpenTelemetry is not a complete AI observability product. It is a standard and instrumentation ecosystem for producing, propagating, collecting, and exporting traces, metrics, and logs.

OpenInference adds AI-specific semantic conventions for:

-   model calls;
-   prompts and responses;
-   retrieval;
-   tools;
-   agents;
-   graph nodes;
-   token usage;
-   prompt-cache read and write token counts.

Together, they provide the most flexible architecture for connecting LangGraph to a serving backend:

```text
LangGraph planner span
  trace_id = abc
  -> HTTP traceparent
  -> SGLang request span
     -> prefix lookup span
     -> prefill span
     -> cache insertion span
```

The spans can then be exported to Phoenix, Jaeger, Grafana Tempo, Langfuse, MLflow, Braintrust, LangSmith, or another OpenTelemetry-compatible backend.

This approach provides maximum control and minimizes vendor lock-in. It also requires the most engineering because OpenTelemetry alone does not supply an AI-focused experiment interface, evaluation system, prompt manager, or complete trace-analysis product.

Best fit: research requiring end-to-end distributed tracing from the agent framework into a modified inference server.


## Which tool is best for which goal? {#which-tool-is-best-for-which-goal}


### Fastest LangGraph debugging {#fastest-langgraph-debugging}

Use LangSmith.

It offers the most direct understanding of graph nodes, branches, tools, and stateful agent executions.


### Reproducible systems experiments {#reproducible-systems-experiments}

Use MLflow.

It combines LangGraph traces with parameters, metrics, artifacts, evaluations, and experiment comparison. This is the most natural fit when the cache study needs tables, plots, raw artifacts, and repeated experiment runs.


### Open-source distributed tracing {#open-source-distributed-tracing}

Use Phoenix.

Its OpenTelemetry and OpenInference architecture is well suited to joining application spans with serving-engine spans.


### Open-source LLM product observability {#open-source-llm-product-observability}

Use Langfuse.

It provides sessions, nested observations, graph-oriented examples, dashboards, evaluations, OpenTelemetry ingestion, and self-hosting.


### Evaluation-first development {#evaluation-first-development}

Use Braintrust.

It provides a strong workflow from production traces to datasets, scorers, experiments, and online evaluation.


### Existing Weights &amp; Biases environment {#existing-weights-and-biases-environment}

Use W&amp;B Weave.

It offers custom operation tracing, evaluation, code versioning, model versioning, and flexible visualization inside the W&amp;B ecosystem.


### Maximum control {#maximum-control}

Use OpenTelemetry with OpenInference and select a separate storage and visualization backend.

This is the best baseline when the main research output is a trace schema or backend instrumentation rather than a product dashboard.


## Recommended bake-off {#recommended-bake-off}

Do not select a platform only from its documentation. Run a small bake-off with one LangGraph workflow:

```text
Planner -> Executor -> Verifier
```

Record the following fields for every model invocation:

```text
workflow_id
agent_id
turn_id
previous_agent_id
request_id
prompt_tokens
cached_tokens
new_prefill_tokens
cache_hit_ratio
ttft_ms
e2e_latency_ms
```

Evaluate each candidate with the same workflow and the same inference requests:

| Criterion              | Question                                                         |
|------------------------|------------------------------------------------------------------|
| Instrumentation effort | How many code changes are required?                              |
| LangGraph fidelity     | Are graph nodes and transitions represented correctly?           |
| Custom fields          | Can arbitrary cache metrics be stored and queried?               |
| Distributed tracing    | Can trace context reach the inference server?                    |
| Aggregation            | Can metrics be grouped by agent, transition, and backend?        |
| Export                 | Can raw traces be exported for statistical analysis?             |
| Experiments            | Can serving configurations be compared reproducibly?             |
| Self-hosting           | Can all telemetry remain on local infrastructure?                |
| Overhead               | Does tracing materially change latency?                          |
| Extensibility          | Can future cache events be added without redesigning the schema? |


## Practical shortlist for KV-cache research {#practical-shortlist-for-kv-cache-research}

The most relevant initial shortlist is:

```text
MLflow
Phoenix
Langfuse
LangSmith
```

MLflow should be tested for experiment management and custom numerical metrics.

Phoenix should be tested for OpenTelemetry-based correlation between LangGraph and SGLang.

Langfuse should be tested as an open-source LLM-specific observability system with sessions, dashboards, and OTLP ingestion.

LangSmith should be used as the LangGraph-native reference point.

A useful initial comparison is:

```text
LangGraph + SGLang + MLflow
LangGraph + SGLang + Phoenix
LangGraph + SGLang + Langfuse
LangGraph + SGLang + LangSmith
```

The same workflow, prompt set, concurrency level, and inference requests should be used in every trial.


## Conclusion {#conclusion}

Agent-framework observability and inference-server observability solve different parts of the problem.

LangGraph tells us:

-   which agent ran;
-   which branch was selected;
-   which tool was called;
-   how state moved through the workflow.

SGLang, vLLM, or another inference engine tells us:

-   how many tokens were cached;
-   how much prefill was required;
-   how long inference took;
-   how much cache capacity was used;
-   which cache entries were inserted or evicted.

The observability platform must join these layers.

MLflow is a strong candidate because of its experiment-management capabilities and support for custom spans. LangSmith provides the deepest native LangGraph experience. Phoenix and Langfuse are compelling open-source alternatives, especially when self-hosting and distributed tracing are important. Braintrust is strongest when the trace-to-evaluation workflow is central. Weave is attractive for teams already standardized on W&amp;B. OpenTelemetry with OpenInference is the vendor-neutral substrate that makes end-to-end research instrumentation possible.

The correct selection should be based on a small, controlled prototype rather than product feature lists alone.


## Reference List {#reference-list}

1.  <https://mlflow.org/docs/latest/genai/tracing/integrations/listing/langgraph/>
2.  <https://mlflow.org/docs/latest/genai/tracing/>
3.  <https://mlflow.org/docs/latest/genai/tracing/app-instrumentation/manual-tracing/>
4.  <https://mlflow.org/docs/latest/genai/eval-monitor/>
5.  <https://docs.langchain.com/langsmith/trace-with-langgraph>
6.  <https://docs.langchain.com/langsmith/observability-quickstart>
7.  <https://docs.langchain.com/langsmith/evaluation>
8.  <https://docs.langchain.com/langsmith/trace-with-opentelemetry>
9.  <https://arize.com/docs/phoenix>
10. <https://arize.com/docs/phoenix/integrations/python/langchain>
11. <https://arize-ai.github.io/openinference/spec/>
12. <https://arize-ai.github.io/openinference/spec/semantic_conventions.html>
13. <https://langfuse.com/guides/cookbook/integration_langgraph>
14. <https://langfuse.com/docs/observability/overview>
15. <https://langfuse.com/integrations/native/opentelemetry>
16. <https://langfuse.com/self-hosting>
17. <https://www.braintrust.dev/docs/integrations/sdk-integrations/langchain>
18. <https://www.braintrust.dev/docs/instrument>
19. <https://www.braintrust.dev/docs/evaluate>
20. <https://www.braintrust.dev/docs/integrations/sdk-integrations/opentelemetry>
21. <https://www.braintrust.dev/docs/admin/self-hosting>
22. <https://docs.wandb.ai/weave/guides/integrations/langchain>
23. <https://docs.wandb.ai/weave/guides/tracking/ops>
24. <https://docs.wandb.ai/weave/guides/core-types/models>
25. <https://docs.wandb.ai/weave/guides/core-types/evaluations>
26. <https://docs.wandb.ai/weave/guides/integrations/autogen>
27. <https://opentelemetry.io/docs/concepts/signals/traces/>
28. <https://opentelemetry.io/docs/concepts/context-propagation/>
