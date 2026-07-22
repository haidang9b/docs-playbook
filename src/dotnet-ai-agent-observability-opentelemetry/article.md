---
title: "AI Agent Observability in .NET with OpenTelemetry"
description: ".NET AI agent observability with OpenTelemetry's GenAI semantic conventions: trace agents, tools, tokens, and cost natively in C# — no Python sidecar."
focus_keyphrase: ".NET AI agent observability"
keywords:
  - ".NET AI agent observability"
  - "OpenTelemetry GenAI semantic conventions"
  - "Semantic Kernel telemetry"
  - "Microsoft.Extensions.AI OpenTelemetry"
  - "Microsoft Agent Framework tracing"
author: ""
date: "2026-07-22"
slug: "dotnet-ai-agent-observability-opentelemetry"
---

# AI Agent Observability in .NET with OpenTelemetry

You shipped an LLM agent in C#, and now it is a black box. Which tool call blew the latency budget? How many tokens did that three-step plan actually burn, and what did it cost? When you search for **.NET AI agent observability**, almost every tutorial, SDK, and community library assumes you are writing Python or TypeScript. That gap is real — but the conclusion most developers draw from it is wrong.

Here is the good news, up front: **.NET is one of the most OpenTelemetry-native platforms in existence**, and the modern .NET AI stack already emits **OpenTelemetry GenAI semantic conventions** out of the box. You do not need a Python sidecar, a proxy, or a bespoke SDK. You need about 20 lines of wiring. This article shows you exactly what to turn on, how to wire the exporter, and how to read a multi-step agent trace — with every version-sensitive detail called out so your telemetry does not break on the next upgrade.

---

## Why .NET AI observability feels like a second-class citizen

The perception is not imaginary. The OpenTelemetry GenAI effort **started Python-first**: the very first instrumentation library targeted the OpenAI *Python* API, with other languages explicitly slated "to follow" [REF-17]. Follow-up guidance on agent observability continued to describe coverage as landing "in Python and other languages" [REF-18].

The tooling numbers make the skew concrete:

- OpenTelemetry's own Python-contrib repository ships a dedicated `instrumentation-genai/` directory with **eight GenAI instrumentation packages** (OpenAI, Anthropic, Google GenAI, VertexAI, LangChain, and more). The `opentelemetry-dotnet` and `opentelemetry-js` repos have **no equivalent GenAI instrumentation directory at all** [REF-25].
- **OpenLLMetry** (Traceloop), the leading community GenAI-instrumentation project, publishes SDKs for **Python and JavaScript/TypeScript only — no .NET/C#** — while covering 16 LLM providers [REF-24].

So if you measure the *third-party ecosystem*, .NET looks under-served. But that is the wrong place to look.

### The plot twist: instrumentation is built into the frameworks

In .NET, GenAI telemetry does not live in a separate instrumentation package — it is emitted **directly by the frameworks themselves**. Microsoft.Extensions.AI, Semantic Kernel, the OpenAI .NET SDK, and Microsoft Agent Framework all publish `gen_ai.*` spans and metrics natively through the base class library's diagnostics APIs [REF-4][REF-7][REF-8][REF-15]. The OpenTelemetry .NET SDK simply subscribes to them.

The reason this works so cleanly is architectural: in .NET, the tracing API *is* `System.Diagnostics.ActivitySource` / `Activity`, and the metrics API *is* `System.Diagnostics.Metrics.Meter` — these are the concrete OpenTelemetry API implementations for the platform [REF-9]. There is no shim. That is the empowering message: **you are not waiting for the ecosystem to catch up; the platform was OTel-native before "GenAI observability" was a phrase.**

---

## The OpenTelemetry GenAI semantic conventions in 60 seconds

**Semantic conventions** are the agreed-upon names for spans, attributes, and metrics so that any backend can interpret your telemetry the same way. The GenAI conventions define how an LLM call, a tool execution, or an agent invocation should look on the wire.

### Status and the 2026 repo move (read this before you pin anything)

Two facts you must internalize before writing any assertions in tests or dashboards:

1. **The GenAI conventions are experimental.** Every span and metric definition carries a stability badge of **"Development"** — the OpenTelemetry project's label for conventions that can still change shape between versions [REF-21]. Microsoft's own `OpenTelemetryChatClient` documents that it implements "SemConv for GenAI v1.37" and is experimental [REF-6].
2. **They moved repositories in 2026.** As of a v1.42.0 release (June 2026), all `gen_ai.*` attributes, spans, and metrics were split out of the main `open-telemetry/semantic-conventions` repo into a dedicated **`open-telemetry/semantic-conventions-genai`** repo [REF-2]. The old spec pages now show a "moved" notice and mark attributes deprecated-in-place, though the canonical landing URL that Microsoft docs link to still resolves [REF-1][REF-3].

**Practical takeaway:** treat the telemetry *shape* as a moving target. Pin your library versions, and do not hard-code attribute-name assertions across major upgrades.

### The span, attribute, and metric model

The conventions center on one attribute, `gen_ai.operation.name`, whose allowed values map to the operations an agent performs [REF-2]:

| Operation | Meaning |
| --- | --- |
| `chat` | A chat-completion request to a model |
| `embeddings` | An embeddings request |
| `execute_tool` | A tool/function invocation |
| `create_agent` / `invoke_agent` | Agent lifecycle and a single agent turn |
| `invoke_workflow` | A multi-agent workflow step |

Span names follow the pattern `{operation} {model-or-agent-name}` (for example, `chat gpt-4o` or `invoke_agent TravelPlanner`) [REF-2]. The attributes you will read most often:

- `gen_ai.provider.name` — the provider (**older emitters use `gen_ai.system`**)
- `gen_ai.request.model` and `gen_ai.response.model`
- `gen_ai.usage.input_tokens` / `gen_ai.usage.output_tokens`
- `gen_ai.agent.name` / `gen_ai.agent.id`, `gen_ai.tool.name`
- `gen_ai.response.finish_reason`, `gen_ai.response.id`

Prompts and completions do **not** ride on span attributes by default — they are attached as span *events* (`gen_ai.content.prompt` / `gen_ai.content.completion`) and are **off by default** because they can contain sensitive data [REF-2].

For metrics, two histograms matter [REF-21]:

- `gen_ai.client.operation.duration` (seconds)
- `gen_ai.client.token.usage` (tokens)

Both carry the **Development** stability badge and a requirement level of **Recommended** — *not* Required [REF-21]. Do not assume they are always present; verify against your library version.

> **Attribute drift is the #1 gotcha.** A Semantic Kernel console sample from ~2024 emits `gen_ai.system`, `gen_ai.operation.name: chat.completions`, and `gen_ai.response.prompt_tokens`. Newer emitters use `gen_ai.provider.name`, `chat`, and `gen_ai.usage.input_tokens` / `output_tokens` [REF-3][REF-7][REF-21]. Exact names depend on the library version — always confirm against the telemetry you actually see.

---

## How the .NET AI stack emits telemetry

Each library has its own on-switch and its own **ActivitySource** name. Here is the whole surface.

### Microsoft.Extensions.AI — `UseOpenTelemetry`

`Microsoft.Extensions.AI` is the common abstraction layer (`IChatClient`) across providers. You add telemetry as **middleware** in the client pipeline, which wraps the underlying client in an `OpenTelemetryChatClient` that emits GenAI spans and token-usage metrics [REF-4][REF-5].

```csharp
using Microsoft.Extensions.AI;

// Pick a source name you control — you'll subscribe to it later.
const string sourceName = "MyAgentApp";

IChatClient client = baseChatClient
    .AsBuilder()
    .UseOpenTelemetry(
        sourceName: sourceName,
        loggerFactory: loggerFactory,
        configure: o => o.EnableSensitiveData = false) // keep prompts OFF in prod
    .Build();
```

The `configure` callback exposes `EnableSensitiveData`; leave it `false` in production so prompt/completion content is not captured [REF-5][REF-6].

### Semantic Kernel — experimental diagnostics switches

Semantic Kernel emits telemetry from the `Microsoft.SemanticKernel*` sources and meters, but only after you flip an **experimental switch**. Enable it via environment variable or `AppContext` [REF-7]:

```bash
# Non-sensitive telemetry (spans, token counts — no prompt content)
export SEMANTICKERNEL_EXPERIMENTAL_GENAI_ENABLE_OTEL_DIAGNOSTICS=true

# Sensitive telemetry (adds prompts/completions) — dev/debug only
export SEMANTICKERNEL_EXPERIMENTAL_GENAI_ENABLE_OTEL_DIAGNOSTICS_SENSITIVE=true
```

```csharp
// Equivalent in-process switch (set before building the kernel)
AppContext.SetSwitch(
    "Microsoft.SemanticKernel.Experimental.GenAI.EnableOTelDiagnostics", true);
```

Then subscribe with `AddSource("Microsoft.SemanticKernel*")` and `AddMeter("Microsoft.SemanticKernel*")` [REF-7].

### OpenAI .NET SDK and Azure.AI.OpenAI

The official OpenAI .NET SDK is also gated behind an experimental switch [REF-8]:

```csharp
AppContext.SetSwitch("OpenAI.Experimental.EnableOpenTelemetry", true);
// or set env var: OPENAI_EXPERIMENTAL_ENABLE_OPEN_TELEMETRY=true
```

Subscribe via `AddSource("OpenAI.*")` / `AddMeter("OpenAI.*")`. Note that today only the non-streaming `OpenAI.ChatClient` path is instrumented [REF-8].

`Azure.AI.OpenAI` wraps the OpenAI SDK; Microsoft's OpenTelemetry distro registers the `Azure.AI.OpenAI*` ActivitySource alongside the others, so subscribe with `AddSource("Azure.AI.OpenAI*")` [REF-22].

### The full source/meter list

Microsoft's OpenTelemetry distro documents the complete set of activity sources the .NET AI stack uses [REF-22]:

| Library | ActivitySource / Meter |
| --- | --- |
| OpenAI .NET SDK | `OpenAI.*` |
| Azure.AI.OpenAI | `Azure.AI.OpenAI*` |
| Semantic Kernel | `Microsoft.SemanticKernel*` |
| Microsoft.Extensions.AI | `Experimental.Microsoft.Extensions.AI` |
| Microsoft Agent Framework | `Experimental.Microsoft.Agents.AI*` |

Because all of these are plain `ActivitySource` / `Meter` instances, the OTel .NET SDK bridges them with nothing more than `AddSource` / `AddMeter` [REF-9][REF-22].

---

## Wiring it up: TracerProvider, MeterProvider, and the OTLP exporter

With the emitters enabled, you register OpenTelemetry and point an **OTLP exporter** at a backend. In a host-based app, `AddOpenTelemetry()` gives you `WithTracing` and `WithMetrics` builders [REF-10][REF-11].

```csharp
using OpenTelemetry.Metrics;
using OpenTelemetry.Resources;
using OpenTelemetry.Trace;

builder.Services.AddOpenTelemetry()
    .ConfigureResource(r => r.AddService("MyAgentApp"))
    .WithTracing(tracing => tracing
        // subscribe to every AI activity source you enabled
        .AddSource("MyAgentApp")
        .AddSource("Experimental.Microsoft.Extensions.AI")
        .AddSource("Experimental.Microsoft.Agents.AI*")
        .AddSource("Microsoft.SemanticKernel*")
        .AddSource("OpenAI.*")
        .AddSource("Azure.AI.OpenAI*")
        .AddOtlpExporter())            // defaults to http://localhost:4317
    .WithMetrics(metrics => metrics
        .AddMeter("Microsoft.SemanticKernel*")
        .AddMeter("OpenAI.*")
        .AddMeter("Experimental.Microsoft.Extensions.AI")
        .AddOtlpExporter());
```

`AddOtlpExporter()` targets `http://localhost:4317` (gRPC) by default; override it with the `OTEL_EXPORTER_OTLP_ENDPOINT` environment variable or in the configure callback [REF-11][REF-12]. For a console app without generic-host DI, build a `TracerProvider` / `MeterProvider` directly with `Sdk.CreateTracerProviderBuilder()` — the `AddSource` / `AddMeter` / `AddOtlpExporter` calls are identical [REF-13].

---

## Seeing your traces

You do not need a hosted backend to start. Because everything speaks **OTLP**, any OTLP receiver works.

### Standalone Aspire Dashboard (fastest local option)

The .NET Aspire Dashboard runs as a **standalone container** and acts as an OTLP receiver for any language — no AppHost or Aspire project required [REF-13][REF-14]:

```bash
docker run --rm -it \
  -p 18888:18888 \
  -p 4317:18889 \
  mcr.microsoft.com/dotnet/aspire-dashboard:latest
```

Point your app's exporter at `http://localhost:4317`, run it, then open the dashboard UI at `http://localhost:18888`. GenAI spans show up in the **Traces** view and token/duration histograms in **Metrics** [REF-13].

### Jaeger (drop-in OTLP alternative)

Jaeger ingests OTLP natively on ports **4317** (gRPC) and **4318** (HTTP), enabled by default in the Jaeger v2 all-in-one image — so a .NET app pointed at `http://localhost:4317` appears with no collector in between [REF-23]:

```bash
docker run --rm -it \
  -p 16686:16686 -p 4317:4317 -p 4318:4318 \
  jaegertracing/jaeger:latest
```

### Azure Monitor (managed backend)

For production, **Azure Monitor / Application Insights** is the managed OTLP backend. Add `AddAzureMonitorTraceExporter` / `AddAzureMonitorMetricExporter` (or use Microsoft's distro) to ship the same spans to App Insights, where multi-agent runs surface in the "Agents (preview)" view [REF-20][REF-16].

---

## Instrumenting a multi-step agent

This is where GenAI conventions prove their value. **Microsoft Agent Framework** emits a nested span tree so you can see an agent's full reasoning path [REF-15]. Enable it on the agent builder [REF-15]:

```csharp
using Microsoft.Agents.AI;

AIAgent observableAgent = agent
    .AsBuilder()
    .UseOpenTelemetry(
        sourceName: "Experimental.Microsoft.Agents.AI",
        configure: o => o.EnableSensitiveData = false)
    .Build();
```

Sensitive-data capture is controlled by `EnableSensitiveData` or the `OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT` environment variable — keep it off in production [REF-15].

### Reading the trace tree

For a multi-agent travel planner, the official end-to-end tutorial produces exactly the structure you want to debug against [REF-16]:

```text
invoke_agent TravelPlanner              gen_ai.agent.id=agt_01…  (root)
├─ chat gpt-4o                          gen_ai.usage.input_tokens=812
│                                       gen_ai.usage.output_tokens=143
├─ execute_tool search_flights          gen_ai.tool.name=search_flights
├─ execute_tool get_weather             gen_ai.tool.name=get_weather
└─ invoke_agent BudgetChecker           gen_ai.agent.id=agt_02…
   └─ chat gpt-4o                       gen_ai.usage.input_tokens=291
                                        gen_ai.usage.output_tokens=64
```

Three things this trace gives you for free:

- **Stable per-agent identity.** Each agent carries a `gen_ai.agent.id`, so you can group spans and compare turns across runs [REF-16].
- **Per-tool timing and counts.** Every `execute_tool` span is timed independently; Agent Framework also emits an `agent_framework.function.invocation.duration` metric for tool calls [REF-15].
- **Token and cost tracking.** Summing `gen_ai.usage.input_tokens` and `gen_ai.usage.output_tokens` across the tree gives total token consumption per run; multiply by your model's per-token price to attribute cost per agent or per tool path [REF-16].

Because these are standard `gen_ai.*` attributes, the same trace renders in Aspire Dashboard, Jaeger, or App Insights without any backend-specific mapping [REF-16][REF-21].

---

## Production considerations

Getting a trace locally is easy; keeping it safe and stable in production takes three deliberate choices.

- **Keep sensitive-data capture OFF.** Prompt and completion content is opt-in for a reason — it frequently contains PII or secrets. Leave `EnableSensitiveData=false` and avoid the `..._SENSITIVE` / `CAPTURE_MESSAGE_CONTENT` switches outside of debugging [REF-7][REF-15].
- **Expect churn — pin versions.** Everything `gen_ai.*` is **Development**-status and moved repos in 2026, and Microsoft.Extensions.AI tracks a specific SemConv version (v1.37) [REF-4][REF-6][REF-21]. Pin your AI-library versions, and treat dashboards or alerts built on exact attribute names as version-coupled.
- **Sample deliberately.** LLM calls are verbose and expensive; use OpenTelemetry sampling and an OTLP collector to control volume and cost before data reaches your backend [REF-10][REF-16].

---

## Next Steps

You now have the full path: enable the per-library switch, subscribe with `AddSource` / `AddMeter`, attach an OTLP exporter, and read the trace in a dashboard you spun up with one `docker run`.

1. **Start local.** Run the standalone Aspire Dashboard and wire `UseOpenTelemetry` into a Microsoft.Extensions.AI client — you will see your first `chat` span in minutes [REF-13][REF-14].
2. **Trace a real agent.** Work through the official multi-agent monitoring tutorial to see nested `invoke_agent → chat → execute_tool` spans with token and cost breakdowns [REF-16].
3. **Track the spec.** Bookmark the `semantic-conventions-genai` repo so you catch attribute changes before they break your dashboards [REF-2][REF-21].

Vendor-neutral, production-grade **.NET AI agent observability** is not something you are waiting on — it is a switch and a subscription away.

---

*Sources Consulted:*

- [REF-1] Gen AI — OpenTelemetry Semantic Conventions (canonical spec landing) — https://opentelemetry.io/docs/specs/semconv/gen-ai/
- [REF-2] OpenTelemetry GenAI Semantic Conventions repository — https://github.com/open-telemetry/semantic-conventions-genai
- [REF-3] Gen AI attributes registry (`gen_ai.*` definitions) — https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/
- [REF-4] Microsoft.Extensions.AI libraries — .NET — https://learn.microsoft.com/en-us/dotnet/ai/microsoft-extensions-ai
- [REF-5] OpenTelemetryChatClientBuilderExtensions.UseOpenTelemetry Method — https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.ai.opentelemetrychatclientbuilderextensions.useopentelemetry
- [REF-6] OpenTelemetryChatClient Class — https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.ai.opentelemetrychatclient
- [REF-7] Inspection of telemetry data with the console — Semantic Kernel — https://learn.microsoft.com/en-us/semantic-kernel/concepts/enterprise-readiness/observability/telemetry-with-console
- [REF-8] openai-dotnet Observability.md — https://github.com/openai/openai-dotnet/blob/main/docs/Observability.md
- [REF-9] Instrumentation — OpenTelemetry .NET — https://opentelemetry.io/docs/languages/dotnet/instrumentation/
- [REF-10] Getting Started — OpenTelemetry .NET — https://opentelemetry.io/docs/languages/dotnet/getting-started/
- [REF-11] OTLP Exporter for OpenTelemetry .NET (README) — https://github.com/open-telemetry/opentelemetry-dotnet/blob/main/src/OpenTelemetry.Exporter.OpenTelemetryProtocol/README.md
- [REF-12] Exporters — OpenTelemetry .NET — https://opentelemetry.io/docs/languages/dotnet/exporters/
- [REF-13] Example: Use OpenTelemetry with OTLP and the standalone Aspire Dashboard — .NET — https://learn.microsoft.com/en-us/dotnet/core/diagnostics/observability-otlp-example
- [REF-14] Run the Aspire dashboard standalone — https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/dashboard/standalone
- [REF-15] Observability — Microsoft Agent Framework — https://learn.microsoft.com/en-us/agent-framework/agents/observability
- [REF-16] Monitor a multi-agent app with OpenTelemetry and Application Insights (.NET) — https://learn.microsoft.com/en-us/azure/app-service/tutorial-ai-agent-monitoring-dotnet
- [REF-17] OpenTelemetry for Generative AI (OTel blog) — https://opentelemetry.io/blog/2024/otel-generative-ai/
- [REF-18] AI Agent Observability - Evolving Standards and Best Practices (OTel blog) — https://opentelemetry.io/blog/2025/ai-agent-observability/
- [REF-20] Azure Monitor OpenTelemetry Exporter client library for .NET — https://learn.microsoft.com/en-us/dotnet/api/overview/azure/monitor.opentelemetry.exporter-readme
- [REF-21] GenAI metrics semantic conventions — semantic-conventions-genai — https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-metrics.md
- [REF-22] Microsoft OpenTelemetry distro for .NET — Azure Monitor getting started — https://github.com/microsoft/opentelemetry-distro-dotnet/blob/main/docs/azure-monitor-getting-started.md
- [REF-23] Getting Started — Jaeger — https://www.jaegertracing.io/docs/latest/getting-started/
- [REF-24] OpenLLMetry (Traceloop) repository — https://github.com/traceloop/openllmetry
- [REF-25] opentelemetry-python-contrib `instrumentation-genai/` directory — https://github.com/open-telemetry/opentelemetry-python-contrib/tree/main/instrumentation-genai
