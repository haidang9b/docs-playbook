# References — dotnet-ai-agent-observability-opentelemetry

## Synthesis

**Core thesis is well-supported by primary sources.** You can get production-grade, vendor-neutral observability for .NET AI agents *today*: the OpenTelemetry GenAI semantic conventions are emitted natively by the .NET AI stack (Microsoft.Extensions.AI, Semantic Kernel, Microsoft Agent Framework, and the OpenAI .NET SDK) through `System.Diagnostics.ActivitySource`/`Meter`, which the OpenTelemetry .NET SDK bridges directly. No Python sidecar is required. [REF-4][REF-5][REF-7][REF-8][REF-15][REF-16]

### Key findings

1. **GenAI semantic conventions are still experimental — status label is literally "Development" — and they *moved repos* in 2026.** As of a v1.42.0 release (June 2026), all `gen_ai.*` attributes/spans were split out of the main `open-telemetry/semantic-conventions` repo into a dedicated `open-telemetry/semantic-conventions-genai` repo (docs live at `docs/gen-ai/gen-ai-spans.md`, `gen-ai-metrics.md`, `gen-ai-agent-spans.md`, etc.). The old spec pages now show "moved" notices and mark attributes deprecated-in-place, but the *canonical spec URL* `https://opentelemetry.io/docs/specs/semconv/gen-ai/` is still what Microsoft docs link to. Every span type carries a blue **Development** stability badge — the writer must state prominently that telemetry shape can change. [REF-1][REF-2][REF-3][REF-21]

2. **The span/attribute model is consistent across the stack.** `gen_ai.operation.name` allowed values (from the current spec): `chat`, `embeddings`, `text_completion`, `generate_content`, `retrieval`, `execute_tool`, `create_agent`, `invoke_agent`, `invoke_workflow`, `plan`, plus a family of `*_memory` operations. Span name convention: `{gen_ai.operation.name} {gen_ai.request.model}` (or agent/tool name). Key attributes: `gen_ai.operation.name`, `gen_ai.provider.name` (older emitters use `gen_ai.system`), `gen_ai.request.model`, `gen_ai.usage.input_tokens` / `gen_ai.usage.output_tokens`, `gen_ai.agent.name` / `gen_ai.agent.id`, `gen_ai.tool.name`, `gen_ai.response.finish_reason`, `gen_ai.response.id`. Prompts/completions ride on span *events* (`gen_ai.content.prompt` / `gen_ai.content.completion`) and are OFF by default (sensitive data). [REF-3][REF-7][REF-15][REF-16][REF-21]

3. **Metrics.** Two GenAI metrics matter: `gen_ai.client.operation.duration` (histogram, seconds) and `gen_ai.client.token.usage` (histogram, tokens). Per the current `gen-ai-metrics.md`, both carry the **Development** stability badge and a requirement level of **Recommended** (not strictly required). Agent Framework additionally emits `agent_framework.function.invocation.duration` for tool calls. [REF-15][REF-21]

4. **Enabling telemetry is per-library and gated behind experimental switches:**
   - **Microsoft.Extensions.AI**: `ChatClientBuilder.UseOpenTelemetry(...)` wraps the client in `OpenTelemetryChatClient` (implements GenAI SemConv v1.37); emits spans + token-usage metrics; takes an optional source name and `ILoggerFactory`. [REF-4][REF-5][REF-6]
   - **Semantic Kernel (.NET)**: subscribe to `AddSource("Microsoft.SemanticKernel*")` / `AddMeter("Microsoft.SemanticKernel*")`; enable via env var `SEMANTICKERNEL_EXPERIMENTAL_GENAI_ENABLE_OTEL_DIAGNOSTICS(=true)` (non-sensitive) or `..._SENSITIVE(=true)` (prompts/completions), or in C# `AppContext.SetSwitch("Microsoft.SemanticKernel.Experimental.GenAI.EnableOTelDiagnostics(Sensitive)", true)`. ActivitySource name observed: `Microsoft.SemanticKernel.Diagnostics`. [REF-7]
   - **OpenAI .NET SDK**: `AppContext.SetSwitch("OpenAI.Experimental.EnableOpenTelemetry", true)` or env var `OPENAI_EXPERIMENTAL_ENABLE_OPEN_TELEMETRY=true`; subscribe via `AddSource("OpenAI.*")` / `AddMeter("OpenAI.*")`. Only `OpenAI.ChatClient` non-streaming is instrumented today. [REF-8]
   - **Azure.AI.OpenAI**: wraps the OpenAI SDK; Microsoft's OpenTelemetry distro registers the `Azure.AI.OpenAI*` ActivitySource (alongside `OpenAI.*`, `Microsoft.SemanticKernel*`, `Experimental.Microsoft.Extensions.AI`, and `Experimental.Microsoft.Agents.AI*`) — subscribe with `AddSource("Azure.AI.OpenAI*")`. [REF-22]
   - **Microsoft Agent Framework**: `agent.AsBuilder().UseOpenTelemetry(sourceName, o => o.EnableSensitiveData = ...)` (default source `Experimental.Microsoft.Agents.AI`); emits `invoke_agent` / `chat` / `execute_tool` spans with `gen_ai.agent.*` attributes. Sensitive-data capture controlled by `EnableSensitiveData` or env `OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT`. [REF-15][REF-16]

5. **.NET is the OTel-native language.** The tracing API *is* `System.Diagnostics.ActivitySource`/`Activity`, and metrics *is* `System.Diagnostics.Metrics.Meter` — the OTel .NET SDK just subscribes via `AddSource`/`AddMeter` and attaches an OTLP exporter. This reinforces the "no sidecar" angle: instrumentation is first-class BCL. [REF-9][REF-10][REF-11][REF-12]

6. **Viewing traces locally is trivial with the standalone Aspire Dashboard** (`docker run ... mcr.microsoft.com/dotnet/aspire-dashboard`, OTLP on 4317, UI on 18888) — no AppHost/Aspire project required; works for any OTLP source. **Jaeger** is a drop-in alternative: since v1.35 (and by default in Jaeger v2) it ingests OTLP natively on ports 4317 (gRPC) / 4318 (HTTP), so a .NET app pointed at `http://localhost:4317` shows up with no collector in between. Azure Monitor (App Insights) is the managed OTLP backend. [REF-13][REF-14][REF-16][REF-20][REF-23]

7. **Agent-specific concerns are directly addressed by an official end-to-end tutorial** (Azure App Service, Microsoft Agent Framework, multi-agent travel planner): stable per-agent IDs (`gen_ai.agent.id`), nested `invoke_agent → chat / execute_tool` spans, per-tool call counts, and token/cost breakdown via `gen_ai.usage.input_tokens`/`output_tokens`, surfaced in the App Insights "Agents (preview)" view. Great scaffold for the "multi-step agent traces + cost tracking" section. [REF-16]

### The gap / why .NET (article's core angle)
The OTel GenAI effort *started* Python-first: the first instrumentation library targeted the OpenAI **Python** API, with other languages "to follow." **Hard numbers back the skew:** OpenTelemetry's own Python-contrib repo ships a dedicated `instrumentation-genai/` directory with **8 GenAI instrumentation packages** (OpenAI, Anthropic, Google GenAI, VertexAI, LangChain, openai-agents-v2, claude-agent-sdk, weaviate) — the `opentelemetry-dotnet` and `opentelemetry-js` repos have **no equivalent GenAI instrumentation directory at all**. The leading community project, **OpenLLMetry (Traceloop)**, publishes SDKs for **Python and JavaScript/TypeScript only — no .NET/C#** — while instrumenting 16 LLM providers. .NET support, while now first-class *in the frameworks themselves* (M.E.AI, SK, Agent Framework, OpenAI SDK), is under-served by the broader OTel-for-AI ecosystem — exactly the gap this article fills. [REF-17][REF-18][REF-24][REF-25]

### Contradictions / caveats for the writer
- **Attribute naming drift.** Older emitters (SK console sample, ~2024) show `gen_ai.system`, `gen_ai.operation.name: chat.completions`, and `gen_ai.response.prompt_tokens`; newer conventions/emitters use `gen_ai.provider.name`, `chat`, and `gen_ai.usage.input_tokens`/`output_tokens`. Call out that exact attribute names depend on the library version. [REF-3][REF-7][REF-15][REF-16][REF-21]
- **Everything is experimental / Development.** M.E.AI cites "SemConv for GenAI v1.37"; the spec's spans and metrics all carry the Development badge. Telemetry output can change between versions — recommend pinning versions in examples. [REF-2][REF-4][REF-21]
- **Sensitive data is opt-in.** Prompts/completions are not captured unless explicitly enabled; recommend keeping it off in production.

### Recommended H2/H3 outline for the writer
- **H2: Why .NET AI observability feels like a second-class citizen** (the Python/TS skew with hard numbers; BLUF that .NET is actually OTel-native) [REF-17][REF-18][REF-24][REF-25][REF-9]
- **H2: The OpenTelemetry GenAI semantic conventions in 60 seconds**
  - H3: Status & the 2026 repo move (experimental / Development; where the spec lives now) [REF-1][REF-2][REF-3][REF-21]
  - H3: The span/attribute/metric model (`gen_ai.*`, operation names, token metrics, Recommended level) [REF-3][REF-15][REF-21]
- **H2: How the .NET AI stack emits telemetry** (the instrumentation surface)
  - H3: Microsoft.Extensions.AI — `UseOpenTelemetry` / `OpenTelemetryChatClient` [REF-4][REF-5][REF-6]
  - H3: Semantic Kernel — diagnostics env switches [REF-7]
  - H3: OpenAI .NET SDK / Azure.AI.OpenAI — experimental switch + `OpenAI.*` / `Azure.AI.OpenAI*` sources [REF-8][REF-22]
  - H3: ActivitySource + Meter: the .NET-native OTel bridge [REF-9][REF-10]
- **H2: Wiring it up** (TracerProvider/MeterProvider, `AddSource`/`AddMeter`, OTLP exporter) [REF-10][REF-11][REF-12][REF-13]
- **H2: Seeing your traces** (standalone Aspire Dashboard; Jaeger native OTLP; Azure Monitor) [REF-13][REF-14][REF-20][REF-23]
- **H2: Instrumenting a multi-step agent** (nested spans, tool spans, token/cost tracking — Agent Framework tutorial) [REF-15][REF-16]
- **H2: Production considerations** (sensitive-data toggles, experimental churn, sampling) [REF-7][REF-16]
- **Next Steps CTA + Sources Consulted**

---

## References
<!-- Confirmed:   [REF-N] Title — https://url -->
<!-- Placeholder: [REF-N] TODO: verify — what is still needed -->

- [REF-1] Gen AI — OpenTelemetry Semantic Conventions (canonical spec landing; shows 2026 repo-move notice) — https://opentelemetry.io/docs/specs/semconv/gen-ai/
- [REF-2] OpenTelemetry GenAI Semantic Conventions repository (dedicated repo as of 2026; spans, metrics, events, MCP, provider conventions) — https://github.com/open-telemetry/semantic-conventions-genai
- [REF-3] Gen AI attributes registry (`gen_ai.*` attribute definitions; now deprecated-in-place/moved) — https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/
- [REF-4] Microsoft.Extensions.AI libraries — .NET (UseOpenTelemetry extension, telemetry middleware) — https://learn.microsoft.com/en-us/dotnet/ai/microsoft-extensions-ai
- [REF-5] OpenTelemetryChatClientBuilderExtensions.UseOpenTelemetry Method (API reference) — https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.ai.opentelemetrychatclientbuilderextensions.useopentelemetry
- [REF-6] OpenTelemetryChatClient Class (API reference; implements GenAI SemConv v1.37, experimental) — https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.ai.opentelemetrychatclient
- [REF-7] Inspection of telemetry data with the console — Semantic Kernel (SEMANTICKERNEL_EXPERIMENTAL_GENAI_ENABLE_OTEL_DIAGNOSTICS(_SENSITIVE); Microsoft.SemanticKernel* source/meter) — https://learn.microsoft.com/en-us/semantic-kernel/concepts/enterprise-readiness/observability/telemetry-with-console
- [REF-8] openai-dotnet Observability.md (OpenAI.Experimental.EnableOpenTelemetry switch / OPENAI_EXPERIMENTAL_ENABLE_OPEN_TELEMETRY; AddSource/AddMeter "OpenAI.*") — https://github.com/openai/openai-dotnet/blob/main/docs/Observability.md
- [REF-9] Instrumentation — OpenTelemetry .NET (ActivitySource/Activity and Meter as the OTel API implementation) — https://opentelemetry.io/docs/languages/dotnet/instrumentation/
- [REF-10] Getting Started — OpenTelemetry .NET (AddOpenTelemetry, WithTracing/WithMetrics, exporters) — https://opentelemetry.io/docs/languages/dotnet/getting-started/
- [REF-11] OTLP Exporter for OpenTelemetry .NET (README; AddOtlpExporter on TracerProviderBuilder/MeterProviderBuilder, config) — https://github.com/open-telemetry/opentelemetry-dotnet/blob/main/src/OpenTelemetry.Exporter.OpenTelemetryProtocol/README.md
- [REF-12] Exporters — OpenTelemetry .NET (exporter options incl. OTLP) — https://opentelemetry.io/docs/languages/dotnet/exporters/
- [REF-13] Example: Use OpenTelemetry with OTLP and the standalone Aspire Dashboard — .NET (docker run, OTLP 4317, TracerProvider/MeterProvider wiring, viewing traces/metrics) — https://learn.microsoft.com/en-us/dotnet/core/diagnostics/observability-otlp-example
- [REF-14] Run the Aspire dashboard standalone (standalone container as OTLP receiver for any language) — https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/dashboard/standalone
- [REF-15] Observability — Microsoft Agent Framework (UseOpenTelemetry/WithOpenTelemetry, EnableSensitiveData, invoke_agent/chat/execute_tool spans, gen_ai.client.token.usage & gen_ai.client.operation.duration metrics, default source Experimental.Microsoft.Agents.AI) — https://learn.microsoft.com/en-us/agent-framework/agents/observability
- [REF-16] Monitor a multi-agent app with OpenTelemetry and Application Insights (.NET) — Azure App Service (end-to-end multi-agent tutorial; per-agent IDs, gen_ai.agent.*, gen_ai.usage.*, tool spans, cost/token breakdown) — https://learn.microsoft.com/en-us/azure/app-service/tutorial-ai-agent-monitoring-dotnet
- [REF-17] OpenTelemetry for Generative AI (OTel blog; first instrumentation library targets OpenAI Python, other languages "to follow") — https://opentelemetry.io/blog/2024/otel-generative-ai/
- [REF-18] AI Agent Observability - Evolving Standards and Best Practices (OTel blog; instrumentation coverage "in Python and other languages") — https://opentelemetry.io/blog/2025/ai-agent-observability/
- [REF-19] Configuring OpenTelemetry in Application Insights — Azure Monitor (Azure Monitor OTLP/exporter configuration for .NET) — https://learn.microsoft.com/en-us/azure/azure-monitor/app/opentelemetry-configuration
- [REF-20] Azure Monitor OpenTelemetry Exporter client library for .NET (AddAzureMonitorTraceExporter/MetricExporter) — https://learn.microsoft.com/en-us/dotnet/api/overview/azure/monitor.opentelemetry.exporter-readme
- [REF-21] GenAI metrics semantic conventions — semantic-conventions-genai (gen-ai-metrics.md; `gen_ai.client.token.usage` and `gen_ai.client.operation.duration` both **Development** + **Recommended**) — https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-metrics.md  (companion spans doc, **Development** status + operation-name list: https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-spans.md )
- [REF-22] Microsoft OpenTelemetry distro for .NET — Azure Monitor getting started (registers `Azure.AI.OpenAI*`, `OpenAI.*`, `Microsoft.SemanticKernel*`, `Experimental.Microsoft.Extensions.AI`, `Experimental.Microsoft.Agents.AI*` activity sources) — https://github.com/microsoft/opentelemetry-distro-dotnet/blob/main/docs/azure-monitor-getting-started.md
- [REF-23] Getting Started — Jaeger (native OTLP ingestion on ports 4317/gRPC and 4318/HTTP, enabled by default in the Jaeger v2 all-in-one image) — https://www.jaegertracing.io/docs/latest/getting-started/
- [REF-24] OpenLLMetry (Traceloop) repository (community GenAI instrumentation; SDKs for Python and JavaScript/TypeScript only — no .NET/C#; 16 LLM providers instrumented) — https://github.com/traceloop/openllmetry
- [REF-25] opentelemetry-python-contrib `instrumentation-genai/` directory (8 GenAI instrumentation packages in Python; no equivalent directory in opentelemetry-dotnet or opentelemetry-js) — https://github.com/open-telemetry/opentelemetry-python-contrib/tree/main/instrumentation-genai
