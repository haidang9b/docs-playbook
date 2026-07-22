---
topic: dotnet-ai-agent-observability-opentelemetry
title: "AI Agent Observability in .NET with OpenTelemetry"
status: publish
updated: 2026-07-22
---

## Goal
Show intermediate-to-advanced .NET developers how to instrument AI agents with OpenTelemetry — capturing GenAI traces, spans, token usage, and cost — using first-class C#/.NET tooling, closing the gap left by the almost entirely Python/TS OTel-for-AI ecosystem.

**Audience:** Intermediate–Advanced .NET developers building LLM/agent apps (Semantic Kernel, Microsoft.Extensions.AI, Azure OpenAI).
**BLUF:** You can get production-grade, vendor-neutral observability for .NET AI agents today using OpenTelemetry's GenAI semantic conventions — no need to reach for a Python sidecar.

## Progress
- [x] Research — references.md created (25 confirmed, 0 TODO placeholders)
- [x] Draft — article.md v1 written
- [x] Review — Reviewer Notes applied, references resolved (2026-07-22)
- [ ] Publish

## Next step
Human SME review + pre-publish checklist, then **publish** (fill `author`, run final link check). No content blockers remain.

## Notes / open questions
- Resolved: OTel GenAI semantic conventions have status label **"Development"** (experimental) as of mid-2026, and moved out of `open-telemetry/semantic-conventions` into the dedicated `open-telemetry/semantic-conventions-genai` repo (~v1.42.0, June 2026). Both `gen_ai.client.token.usage` and `gen_ai.client.operation.duration` are **Recommended** (not required). Canonical spec URL `opentelemetry.io/docs/specs/semconv/gen-ai/` still resolves (shows moved notice). [REF-1][REF-2][REF-3][REF-21]
- Resolved: .NET instrumentation surface verified — Microsoft.Extensions.AI (`UseOpenTelemetry`/`OpenTelemetryChatClient`), Semantic Kernel (`SEMANTICKERNEL_EXPERIMENTAL_GENAI_ENABLE_OTEL_DIAGNOSTICS[_SENSITIVE]`), OpenAI .NET SDK (`OpenAI.Experimental.EnableOpenTelemetry`), Azure.AI.OpenAI (`Azure.AI.OpenAI*` ActivitySource), Microsoft Agent Framework (`invoke_agent`/`chat`/`execute_tool` spans). [REF-4]–[REF-8][REF-15][REF-16][REF-22]
- Resolved: viewing/backends — standalone Aspire Dashboard and Jaeger native OTLP (ports 4317/4318, default in Jaeger v2) confirmed. [REF-13][REF-14][REF-23]
- Resolved: Python/TS skew backed by hard numbers — OpenLLMetry ships Python + JS/TS SDKs only (no .NET), 16 providers; OTel Python-contrib has an 8-package `instrumentation-genai/` dir with no equivalent in opentelemetry-dotnet or opentelemetry-js. [REF-24][REF-25]
- No open TODO placeholders remain; publish gate for references is clear.
- Draft (2026-07-22): article.md written; cites REF-1 through REF-25 except REF-19 (Azure Monitor config page not needed — REF-20 covered the exporter). Zero TODO placeholders relied on. Image prompt ideas saved to assets/images/IMAGE_PROMPTS.md.
- Watch-out for writer: attribute naming drifts by library version (`gen_ai.system` vs `gen_ai.provider.name`, `chat.completions` vs `chat`, `gen_ai.response.prompt_tokens` vs `gen_ai.usage.input_tokens`). Pin versions in code samples; everything GenAI is experimental/Development.
- Review (2026-07-22): PASS, no content blockers. Frontmatter/house-style/reference-completeness all pass; 24 REFs cited (REF-19 intentionally uncited), Sources Consulted mirrors exactly, 0 TODO. Development caveat, attribute-drift warning, metrics-as-Recommended, and sensitive-data opt-in all present and prominent. One safe edit applied (idiom "earn their keep" → "prove their value"). Non-blocking suggestions left for author/SME: (1) metrics `WithMetrics` block omits the Agent Framework meter present in the tracing block — verify/add the meter name if desired; (2) when a custom `sourceName` is passed to M.E.AI `UseOpenTelemetry`, spans emit under that name, not `Experimental.Microsoft.Extensions.AI` — subscribing to both is harmless but worth a one-line clarification. Human still owes SME review + pre-publish checklist before setting `published`.
