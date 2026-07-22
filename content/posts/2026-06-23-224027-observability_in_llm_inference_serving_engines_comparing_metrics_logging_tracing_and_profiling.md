---
title: "Observability in LLM Inference Serving Engines: Comparing Metrics, Logging, Tracing, and Profiling"
date: 2026-06-23
draft: false
---

[Large Language Models (LLMs)]({{< relref "2023-12-01-141658-llms.md" >}})


## Why observability matters {#why-observability-matters}

[Large Language Model (LLM) Inference Serving]({{< relref "2025-10-31-155338-large_language_model_llm_inference_engines.md" >}}) systems are often evaluated by throughput, latency, memory efficiency, model coverage, and hardware compatibility. Those measurements are necessary, but they do not fully answer the operational question:

> When an LLM service becomes slow, inefficient, unstable, or expensive, can we determine why?

An inference server may become slow because of request queueing, oversized batches, long prompt prefill, KV-cache pressure, GPU memory exhaustion, prefix-cache misses, request preemption, CPU-GPU transfers, distributed communication, or inefficient kernels. A server that reports only aggregate throughput can show that performance degraded without explaining the cause.

Observability is therefore a core property of an LLM serving engine. A highly observable engine should expose enough telemetry to explain the whole inference lifecycle:

```text
Request received
  -> Tokenization
  -> Queueing and scheduling
  -> Prefix-cache lookup
  -> Prompt prefill
  -> Token-by-token decoding
  -> Detokenization and streaming
  -> Request completion
```

This comparison covers:

-   [vLLM]({{< relref "2025-04-22-085318-vllm.md" >}})
-   mini-vLLM / Nano-vLLM-style minimal implementations
-   [SGLang]({{< relref "2026-06-17-143142-sglang.md" >}})
-   [Mini-SGLang]({{< relref "2026-06-17-143216-mini_sglang.md" >}})
-   TensorRT-LLM
-   Hugging Face Text Generation Inference (TGI)
-   LMDeploy
-   MLC-LLM
-   [Ollama]({{< relref "2025-04-14-230239-ollama.md" >}})
-   Ray Serve
-   [DeepSpeed]({{< relref "2026-05-18-203245-deepspeed.md" >}})
-   [Hugging Face Accelerate]({{< relref "2026-05-18-203211-hugging_face_accelerate.md" >}})

The goal is not to identify one universally superior engine. The goal is to evaluate which runtime behaviors each system makes visible, how easily those measurements can be exported and correlated, and how difficult it is to add new instrumentation when the built-in view is insufficient.


## Observability dimensions {#observability-dimensions}


### Request-level metrics {#request-level-metrics}

Request-level metrics describe individual inference calls from the user's point of view. Important measurements include:

-   input or prompt token count;
-   generated token count;
-   queueing delay;
-   time to first token (TTFT);
-   time per output token (TPOT);
-   inter-token latency;
-   prefill latency;
-   decoding latency;
-   end-to-end latency;
-   request cancellation and failure;
-   streaming duration.

These metrics show whether latency is introduced before model execution, during prompt processing, or during token generation.


### Service-level metrics {#service-level-metrics}

Service-level metrics describe aggregate deployment behavior:

-   requests per second;
-   tokens per second;
-   active requests;
-   queued requests;
-   batch size;
-   prefill and decode batch composition;
-   request admission failures;
-   scheduler utilization;
-   error rate;
-   percentile latency;
-   model loading state.

These measurements are the basis for production alerting and capacity planning.


### Resource metrics {#resource-metrics}

Resource observability should include:

-   GPU utilization;
-   GPU memory utilization;
-   CPU utilization;
-   host memory consumption;
-   cache capacity and occupancy;
-   memory fragmentation;
-   PCIe or NVLink transfer activity;
-   tensor-parallel communication;
-   network traffic;
-   disk or remote-cache activity.

Generic GPU monitoring is useful, but inference-engine metrics are needed to connect resource usage to requests, batches, cache state, and scheduling decisions.


### [KV-cache]({{< relref "2026-06-23-204356-kv_cache.md" >}}) observability {#kv-cache--2026-06-23-204356-kv-cache-dot-md--observability}

KV-cache behavior is central to LLM serving. Useful measurements include:

-   total KV-cache capacity;
-   occupied KV-cache tokens or blocks;
-   available cache capacity;
-   prefix-cache hit rate;
-   cached prompt tokens;
-   newly computed prompt tokens;
-   cache allocation failures;
-   request preemption caused by memory pressure;
-   cache eviction;
-   CPU offloading;
-   cache loading and prefetching;
-   external KV-cache transfers.

Raw GPU memory usage is not enough because many engines preallocate KV-cache memory. Logical cache occupancy can change substantially while total GPU memory usage remains almost constant.


### Logging {#logging}

A production-ready serving engine should support configurable structured logs for:

-   request arrival and completion;
-   request identifiers;
-   model configuration;
-   scheduling decisions;
-   warnings and failures;
-   out-of-memory events;
-   cache activity;
-   distributed-worker state;
-   model loading;
-   API errors.

JSON logs are especially useful for Elasticsearch, Loki, Splunk, and cloud logging platforms. Logging must also be configurable because prompts and outputs may contain sensitive data.


### Distributed tracing {#distributed-tracing}

Distributed tracing connects operations across service boundaries. A single LLM request may pass through:

```text
API gateway
  -> Application service
  -> Inference router
  -> Model server
  -> Distributed GPU workers
  -> External KV-cache service
```

[OpenTelemetry]({{< relref "2025-09-16-194840-opentelemetry.md" >}}) trace propagation can reveal how much time is spent in each component. Useful spans include tokenization, queueing, scheduling, cache lookup, cache loading, prefill, decoding, streaming, model forwarding, and inter-worker communication.

Tracing becomes especially important with multiple replicas, routers, disaggregated prefill/decode workers, or external cache systems.


### Runtime and kernel profiling {#runtime-and-kernel-profiling}

Operational metrics show what is happening. Profiling tools explain why it is happening. Relevant profilers include:

-   PyTorch Profiler;
-   NVIDIA Nsight Systems;
-   NVIDIA Nsight Compute;
-   CUDA profiling tools;
-   NVTX annotations;
-   CPU profilers;
-   Python profilers;
-   distributed communication profilers.

An engine is easier to analyze when it provides profiling endpoints, NVTX ranges, reproducible workloads, and a clear separation between scheduling logic and GPU execution.


### Export and integration {#export-and-integration}

Observability data becomes more valuable when exported through standard interfaces:

-   [Prometheus]({{< relref "20230602142910-prometheus.md" >}})
-   OpenTelemetry Protocol (OTLP);
-   structured JSON logs;
-   CSV or JSONL request records;
-   trace exporters;
-   profiling outputs;
-   Grafana dashboards;
-   experiment-tracking integrations.

Standard interfaces reduce the effort required to integrate a serving engine into an existing monitoring stack.


### Extensibility {#extensibility}

Advanced production and research environments often need custom instrumentation:

-   custom request metadata;
-   scheduler event hooks;
-   cache lifecycle instrumentation;
-   model-layer timing;
-   custom Prometheus metrics;
-   OpenTelemetry spans;
-   request-to-worker mapping;
-   experimental scheduling or caching policies.

Extensibility matters most when built-in metrics are not specific enough to explain a production incident or research result.


## Comparison {#comparison}


### Summary table {#summary-table}

| Engine                  | Visible without source changes                                                         | Attribution                                                                       | Standard export                                                       | Custom instrumentation                                      | Best fit                                               |
|-------------------------|----------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------|-----------------------------------------------------------------------|-------------------------------------------------------------|--------------------------------------------------------|
| vLLM                    | Strong request, service, cache, and profiling surfaces                                 | Request and aggregate metrics; tracing improves request correlation               | Prometheus, Grafana/Perses dashboards, OTLP traces, profiler outputs  | Moderate to high; Python internals make extension practical | Production monitoring and performance debugging        |
| SGLang                  | Strong metrics, cache visibility, request tracing, request dump/replay                 | Request traces plus scheduler/cache/service metrics                               | Prometheus, Grafana, OpenTelemetry/Jaeger, logs                       | High; designed for serving-system experimentation           | Production monitoring, debugging, and research         |
| TensorRT-LLM            | Runtime-iteration metrics, GPU memory, in-flight batching, deep CUDA profiling         | Batch/runtime iteration and GPU-worker level; weaker high-level request causality | HTTP metrics endpoint, Nsight, NVTX, PyTorch profiler traces          | Moderate; lower-level C++/CUDA stack increases effort       | Kernel and NVIDIA-stack performance debugging          |
| Hugging Face TGI        | Strong HTTP, queue, batch, token, prefill/decode, and latency metrics                  | Request histograms and batch-level method labels                                  | Prometheus and Grafana                                                | Moderate; less convenient for kernel-level changes          | Production monitoring                                  |
| LMDeploy                | Prometheus metrics, Grafana, DP-rank awareness, PyTorch/Nsight/Ray profiling           | API-server, replica, and rank level; limited native tracing                       | Prometheus, Grafana, PyTorch profiler, Nsight, Ray timeline           | Moderate                                                    | Production monitoring and targeted profiling           |
| MLC-LLM                 | REST/OpenAI-compatible server and request event logging                                | Request event level when tracing is enabled                                       | Limited built-in standard export; custom integration likely           | High for compiler/runtime research                          | Edge deployment and systems research                   |
| Ollama                  | Per-response timing and token statistics, model process state, logs                    | Per response and loaded model                                                     | Limited built-in export; external observability integrations          | Moderate                                                    | Local development and lightweight application serving  |
| Ray Serve               | Broad platform metrics, request IDs, logs, replica/router metrics, autoscaling metrics | Deployment, replica, router, app, and request ID                                  | Prometheus, Grafana, Loki, dashboard, custom metrics                  | High                                                        | Production platform observability around model servers |
| DeepSpeed               | FLOPS, latency, throughput, module profiling, communication logging                    | Layer, operator, and distributed-rank level                                       | Profiler reports and logs; experiment integration by application code | High in model/runtime code                                  | Distributed model and communication profiling          |
| Hugging Face Accelerate | PyTorch profiling and experiment tracking                                              | Code block, operator, device, and process level                                   | Chrome trace, profiler output, experiment trackers                    | High in Python code                                         | Experiment instrumentation, not serving observability  |
| mini-vLLM / Nano-vLLM   | Minimal built-in telemetry                                                             | Mostly whatever the researcher adds                                               | Custom                                                                | Very high; compact codebase                                 | Teaching and research instrumentation                  |
| mini-SGLang             | Minimal production telemetry, but realistic serving features                           | Mostly whatever the researcher adds                                               | Custom                                                                | Very high; compact SGLang-style codebase                    | Teaching, scheduler/cache research                     |


### [vLLM]({{< relref "2025-04-22-085318-vllm.md" >}}) {#vllm--2025-04-22-085318-vllm-dot-md}

vLLM is one of the strongest general-purpose choices when production monitoring and serving-specific performance debugging are both required.

Without modifying source code, its OpenAI-compatible server exposes Prometheus metrics at `/metrics`. The documented metrics and dashboards cover request and token throughput, latency, scheduler state, prefix-cache behavior, KV-cache usage, and model-serving health. vLLM also documents Grafana and Perses dashboards, offline metrics access, OpenTelemetry trace export, and profiling workflows based on PyTorch Profiler.

Attribution is good at the aggregate and request-latency level, and better when OpenTelemetry is enabled. The strongest built-in view is service and scheduler behavior; for detailed per-kernel analysis, the profiling path is required.

Best use:

-   production monitoring;
-   queue, scheduler, prefill/decode, and cache debugging;
-   research that needs realistic vLLM behavior but can tolerate a larger codebase.


### SGLang {#sglang}

SGLang has a strong observability story for both production serving and systems research.

Its production metrics include request count, prompt tokens, generation tokens, token usage, cache hit rate, TTFT, TPOT, end-to-end latency, running requests, queued requests, used tokens, and generation throughput. It can also expose MFU-related estimates when enabled. Request tracing uses OpenTelemetry and can be visualized through collectors such as Jaeger. Runtime trace levels can be changed without restarting, and request dump/replay plus crash dump/replay make it easier to reproduce failures.

SGLang is especially useful when the question involves cache behavior, scheduling policy, or request-level tracing. It gives more direct serving-system observability than engines that focus mainly on HTTP metrics or kernel profiling.

Best use:

-   production monitoring with Prometheus and OpenTelemetry;
-   cache, scheduling, and latency breakdown analysis;
-   research instrumentation on serving policies.


### TensorRT-LLM {#tensorrt-llm}

TensorRT-LLM is strongest when the main question is low-level GPU performance.

The `trtllm-serve` command exposes an OpenAI-compatible server and a metrics endpoint with runtime-iteration statistics such as GPU memory usage and in-flight batching details. Its performance tooling is deep: Nsight Systems, Nsight Compute, CUDA profiler toggling, PyTorch profiler integration, and NVTX markers are documented.

Its weaker side is high-level, application-style observability. It is excellent at explaining why a kernel, graph, or runtime path is expensive, but production request causality, trace propagation, and structured request lifecycle telemetry usually require more surrounding infrastructure.

Best use:

-   NVIDIA GPU kernel and runtime analysis;
-   optimized production serving when paired with external monitoring;
-   debugging in-flight batching and GPU execution efficiency.


### Hugging Face Text Generation Inference {#hugging-face-text-generation-inference}

TGI is a strong production-monitoring engine.

It exposes Prometheus metrics that cover request count, request duration, queue duration, validation duration, input length, generated tokens, request success, time per token, inter-token latency, batch size, and forward duration split by prefill and decode. Its monitoring documentation is oriented toward Prometheus and Grafana.

The main limitation is depth below the serving layer. TGI is good at showing HTTP, queue, request, and batch behavior. It is less direct than TensorRT-LLM for kernel-level diagnosis and less direct than SGLang for request tracing and cache-centric experimentation.

Best use:

-   production dashboards and alerting;
-   latency and batching analysis;
-   environments that want a mature OpenAI-compatible text-generation server with Prometheus metrics.


### LMDeploy {#lmdeploy}

LMDeploy provides production metrics and useful profiling hooks, especially for PyTorchEngine deployments.

Metrics can be enabled for the API server and scraped by Prometheus, with Grafana support documented. For data-parallel deployments, each DP rank has an API server endpoint that may need to be scraped separately. Profiling support includes PyTorch Profiler, Nsight Systems, and Ray timeline output for distributed execution.

LMDeploy is a balanced option: more production-oriented than minimal research engines and more profiling-aware than systems that only expose HTTP metrics. Native tracing and request-causal observability are less prominent than in vLLM or SGLang.

Best use:

-   production monitoring for LMDeploy deployments;
-   targeted PyTorch and distributed profiling;
-   debugging across API-server and DP-rank boundaries.


### MLC-LLM {#mlc-llm}

MLC-LLM is best understood as a compiler/runtime and deployment system rather than a production observability platform.

It offers REST and OpenAI-compatible serving modes and includes a tracing flag for request event logging. Its major strength is portability across server, browser, mobile, and local runtimes. That portability makes it useful for systems research and application embedding, but standard production observability such as Prometheus metrics, Grafana dashboards, or OpenTelemetry traces is not the center of the documented interface.

Best use:

-   cross-platform deployment experiments;
-   edge and local inference;
-   runtime/compiler instrumentation where custom hooks are acceptable.


### Ollama {#ollama}

Ollama prioritizes local usability and simple application integration.

The API returns useful per-request statistics such as total duration, load duration, prompt evaluation count and duration, and generation evaluation count and duration. It also exposes running model state. These statistics are useful for local troubleshooting and basic performance checks.

Ollama is weaker as a built-in production observability target. Standard metrics and tracing usually come from wrappers, gateways, or external observability integrations rather than from a rich native telemetry surface.

Best use:

-   local development;
-   lightweight application serving;
-   coarse latency and token-throughput inspection.


### Ray Serve {#ray-serve}

Ray Serve is not an LLM engine by itself, but it is a strong production serving platform around LLM engines.

It exposes broad platform metrics for request lifecycle, HTTP and gRPC proxy behavior, routing, batching, replicas, autoscaling, model multiplexing, event-loop scheduling, and controller state. Metrics are labeled by deployment and replica, and Ray supports custom metrics inside deployments. Logs, request IDs, dashboard integration, Loki, and memory profiling with memray are also documented.

Ray Serve is strongest for multi-replica service observability: routers, deployments, autoscaling, request routing, platform failures, and model-server wrappers. It does not replace engine-native metrics for KV-cache occupancy, prefill/decode behavior, or CUDA kernel profiling.

Best use:

-   production platform observability;
-   replica and autoscaling analysis;
-   adding request, business, and routing metrics around an LLM engine.


### DeepSpeed {#deepspeed}

DeepSpeed is primarily a distributed training and inference runtime, not a complete request-serving observability stack.

Its strengths are model and distributed-runtime profiling. The FLOPS profiler reports latency, throughput, FLOPS, parameters, MACs, and per-module forward latency. DeepSpeed configuration also supports wall-clock breakdowns and communication logging for distributed operations.

DeepSpeed can explain model-layer and communication costs well, but it does not provide LLM-server concepts such as HTTP request queueing, TTFT, TPOT, streaming duration, or KV-cache hit rate unless the application serving layer adds them.

Best use:

-   model-level performance analysis;
-   distributed communication profiling;
-   research or custom serving stacks that already use DeepSpeed internally.


### Hugging Face Accelerate {#hugging-face-accelerate}

Accelerate is an application instrumentation and distributed-execution library, not an inference server.

It integrates with PyTorch Profiler and can export Chrome traces. It also supports experiment tracking integrations. This makes it useful for measuring custom inference scripts, research experiments, and model code paths.

It does not provide built-in request queues, HTTP serving metrics, KV-cache telemetry, or production dashboards. Those must come from the application or serving system built around Accelerate.

Best use:

-   profiling custom Python inference code;
-   experiment tracking;
-   lightweight distributed instrumentation outside a full serving engine.


### mini-vLLM / Nano-vLLM-style implementations {#mini-vllm-nano-vllm-style-implementations}

Small vLLM-style implementations are usually poor production observability targets and excellent research instrumentation targets.

They typically do not expose rich Prometheus metrics, OpenTelemetry traces, or dashboards. Their advantage is readability: the scheduler, block manager, prefix cache, CUDA graph path, or generation loop can be instrumented directly with a small amount of source modification.

Best use:

-   teaching how vLLM-style serving works;
-   adding custom event logs around scheduling and cache behavior;
-   experiments where code clarity matters more than production hardening.


### Mini-SGLang {#mini-sglang}

Mini-SGLang plays a similar role for SGLang-style systems.

It preserves important serving ideas such as radix cache, chunked prefill, overlap scheduling, tensor parallelism, and structured outputs in a compact codebase. It is therefore useful for research on scheduler and cache behavior. Like other minimal engines, it should not be treated as a production observability platform without additional telemetry.

Best use:

-   scheduler and cache research;
-   readable reproduction of SGLang-style serving behavior;
-   custom instrumentation for experiments.


## Assessment by use case {#assessment-by-use-case}


### Production monitoring {#production-monitoring}

The strongest choices are vLLM, SGLang, TGI, LMDeploy, and Ray Serve.

-   vLLM is strong when operators need Prometheus metrics, dashboards, cache visibility, and optional OpenTelemetry traces.
-   SGLang is strong when production monitoring must include cache, queue, TTFT/TPOT, request tracing, and replay tools.
-   TGI is strong when the production requirement is a mature Prometheus/Grafana text-generation server.
-   LMDeploy is strong when the deployment already uses LMDeploy and needs Prometheus plus profiling hooks.
-   Ray Serve is strong when the main concern is service-platform observability across replicas, routing, autoscaling, and wrappers around model servers.

TensorRT-LLM can be production-grade, but its observability strength is lower-level runtime and GPU execution. It often benefits from external application and tracing layers. Ollama and MLC-LLM are better treated as local, embedded, or custom-instrumented systems unless wrapped by a production observability stack.


### Performance debugging {#performance-debugging}

The best tool depends on the suspected bottleneck.

-   Queueing, scheduling, prefill/decode split, and KV-cache pressure: vLLM or SGLang.
-   Kernel, CUDA graph, and NVIDIA runtime efficiency: TensorRT-LLM with Nsight and NVTX.
-   Request batching and HTTP-serving latency: TGI, vLLM, SGLang, or LMDeploy.
-   Replica routing, autoscaling, or event-loop blocking: Ray Serve.
-   Model-layer and communication costs: DeepSpeed or Accelerate.
-   Local model-load and token-generation timing: Ollama.
-   Compiler/runtime behavior across deployment targets: MLC-LLM.


### Research instrumentation {#research-instrumentation}

The most extensible choices are mini-SGLang, mini-vLLM/Nano-vLLM-style implementations, SGLang, vLLM, DeepSpeed, and Accelerate.

Minimal engines are best when source readability is more important than production completeness. SGLang and vLLM are better when experiments need realistic production behavior. DeepSpeed and Accelerate are useful when the research question is inside the model, distributed runtime, or profiling loop rather than inside a complete HTTP inference server.


## Conclusion {#conclusion}

There is no single best observability engine.

For production monitoring, vLLM, SGLang, TGI, LMDeploy, and Ray Serve provide the most direct integration paths. For root-cause analysis of LLM serving behavior, vLLM and SGLang expose the most useful scheduler, queue, cache, and token-latency signals. For low-level GPU performance, TensorRT-LLM is the strongest option. For local development, Ollama is simple and exposes useful per-response timing, but it needs external tooling for production observability. For research instrumentation, the minimal vLLM/SGLang-style implementations are easiest to modify, while full vLLM and SGLang provide more realistic production behavior.

The practical choice depends on the question being asked:

-   "Is the service healthy?" favors Prometheus metrics, dashboards, logs, and alerting.
-   "Why is this request slow?" favors request-level traces, TTFT/TPOT, queue time, prefill/decode timing, and cache telemetry.
-   "Why is the GPU inefficient?" favors profiling, NVTX, Nsight, and model/runtime instrumentation.
-   "Can I test a new scheduling or caching idea?" favors a compact or highly extensible codebase.


## Reference List {#reference-list}

1.  <https://docs.vllm.ai/en/latest/usage/metrics/>
2.  <https://docs.vllm.ai/en/latest/examples/observability/prometheus_grafana/>
3.  <https://docs.vllm.ai/en/latest/examples/observability/opentelemetry/>
4.  <https://docs.vllm.ai/en/latest/examples/features/profiling/>
5.  <https://docs.sglang.io/docs/advanced_features/observability>
6.  <https://docs.sglang.io/docs/references/production_metrics>
7.  <https://docs.sglang.io/docs/references/production_request_trace>
8.  <https://huggingface.co/docs/text-generation-inference/en/basic_tutorials/monitoring>
9.  <https://huggingface.co/docs/text-generation-inference/en/reference/metrics>
10. <https://nvidia.github.io/TensorRT-LLM/commands/trtllm-serve.html>
11. <https://nvidia.github.io/TensorRT-LLM/performance/perf-analysis.html>
12. <https://lmdeploy.readthedocs.io/en/latest/advance/metrics.html>
13. <https://lmdeploy.readthedocs.io/en/latest/advance/pytorch_profiling.html>
14. <https://docs.ray.io/en/latest/serve/monitoring.html>
15. <https://docs.ray.io/en/latest/ray-observability/index.html>
16. <https://mlc.ai/mlc-llm/docs/deploy/rest.html>
17. <https://github.com/ollama/ollama/blob/main/docs/api.md>
18. <https://www.deepspeed.ai/tutorials/flops-profiler/>
19. <https://www.deepspeed.ai/docs/config-json/>
20. <https://huggingface.co/docs/accelerate/en/usage_guides/profiler>
21. <https://huggingface.co/docs/accelerate/en/usage_guides/tracking>
22. <https://github.com/sgl-project/mini-sglang>
23. <https://github.com/GeeeekExplorer/nano-vllm>
