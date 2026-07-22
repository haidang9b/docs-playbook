# Topic Ideas Backlog

Ranked blog topic ideas produced by the `blog-topic-scout` agent. Each entry is ready to hand to `/create-blog <slug>`.

---

## 2026-07-22 — Theme: C#/.NET x AI

Scope surveyed: dev.to and Medium community posts, plus official sources (Microsoft .NET Blog, Microsoft Foundry / Agent Framework blogs, Semantic Kernel docs, Microsoft Learn, MCP C# SDK, dotnet/ai-samples). Ranked best-first by opportunity = gap size x timeliness x audience demand as of July 2026.

**Landscape snapshot (what's already saturated vs. thin):**
- *Saturated / crowded:* "Intro to Microsoft.Extensions.AI", "Hello-world MCP server in C#", "Build a basic RAG with Ollama + .NET", "ML.NET vs Python" comparison posts, "Meet the Microsoft Agent Framework" announcement pieces. Many are shallow and version-lagging.
- *Timely context:* Microsoft Agent Framework hit 1.0 GA on 2026-04-03; Foundry Agent Service reached GA in July 2026; .NET 10 (Nov 2025) matured Microsoft.Extensions.AI, shipped MCP first-class support, and EF Core 10 added AI-ready vector search over SQL Server 2025 native vectors. Most community content still targets .NET 9 / preview APIs, so accurate .NET 10 + Agent Framework 1.0 content is undersupplied.

---

### AI agent observability in .NET: tracing, tokens, and cost with OpenTelemetry
- **Slug:** dotnet-ai-agent-observability-opentelemetry
- **Audience:** Intermediate
- **Angle / gap:** There is a large body of OpenTelemetry-for-AI content on dev.to, but it is almost entirely Python/TypeScript — the .NET story is nearly absent even though Microsoft.Extensions.AI and the Agent Framework emit OpenTelemetry `gen_ai` spans out of the box. This piece would be the go-to .NET-native walkthrough: wiring `OpenTelemetry` + the `gen_ai` semantic conventions into an `IChatClient`/agent, capturing token usage as both span attributes and metrics, and setting a hard budget cap to avoid the "runaway agent burns the daily token budget" failure mode. Ties directly to production readiness that intro posts skip.
- **Prior art:**
  - https://dev.to/thedailyagent/ai-agent-observability-the-4-pillars-that-keep-your-agents-from-burning-2000-at-3-am-24cn (Python-leaning, no .NET)
  - https://dev.to/x4nent/opentelemetry-genai-semantic-conventions-the-standard-for-llm-observability-1o2a (conventions, language-agnostic)
  - https://learn.microsoft.com/en-us/dotnet/ai/ (official .NET AI docs to ground the .NET APIs)
- **Est. effort:** Medium

### Kill the vector DB: RAG in .NET 10 with SQL Server 2025 native vectors and EF Core 10
- **Slug:** dotnet-rag-sql-server-2025-native-vectors
- **Audience:** Intermediate
- **Angle / gap:** RAG-in-.NET is heavily covered, but almost every existing post reaches for a separate vector store (Qdrant, Pinecone, Azure AI Search). The "you already have a vector DB — it's your relational DB" narrative is fresh and only lightly covered, and EF Core 10 + SQL Server 2025 native vector types landed with .NET 10, so most tutorials predate the built-in support. Angle: build a production-shaped RAG pipeline (chunk -> embed with `IEmbeddingGenerator` -> store/query vectors in SQL Server via EF Core 10 hybrid search) with no extra infrastructure, plus honest limits on when you *do* still want a dedicated vector store.
- **Prior art:**
  - https://vik-sharma.medium.com/stop-reaching-for-a-separate-vector-db-build-rag-in-net-with-sql-server-2025-9e7858db7030
  - https://medium.com/@murataslan1/building-a-retrieval-augmented-generation-rag-solution-with-net-ai-and-azure-sql-937941bd5f14
  - https://learn.microsoft.com/en-us/dotnet/ai/ (ground EF Core 10 / MEAI embedding APIs)
- **Est. effort:** Medium

### Evals as unit tests: gating LLM quality in CI with Microsoft.Extensions.AI.Evaluation
- **Slug:** dotnet-ai-evaluation-unit-tests-ci
- **Audience:** Intermediate
- **Angle / gap:** The evaluation libraries are documented by Microsoft and have a few intro write-ups, but there is a clear practitioner gap: treating evals as real unit tests wired into a CI pipeline (MSTest/xUnit, caching to keep runs cheap, failing the build on a groundedness/relevance regression, custom domain evaluators). Existing posts explain the API surface; few show the CI/CD discipline that makes evals actually stick on a team. Strong "why now" as teams move .NET AI apps from demo to production.
- **Prior art:**
  - https://devblogs.microsoft.com/dotnet/evaluate-the-quality-of-your-ai-applications-with-ease/
  - https://www.devleader.ca/2026/04/05/evaluating-ai-agents-with-microsoftextensionsaievaluation-in-c
  - https://learn.microsoft.com/en-us/dotnet/ai/evaluation/evaluate-with-reporting
- **Est. effort:** Medium

### Fully local, sovereign agents in C#: Microsoft Agent Framework + Foundry Local
- **Slug:** dotnet-local-agents-foundry-local-agent-framework
- **Audience:** Intermediate
- **Angle / gap:** Local-model content in .NET is dominated by Ollama tutorials. Foundry Local (OpenAI-compatible REST, ONNX runtime across CPU/GPU/NPU, models like Phi and Qwen) is newer and under-covered, and pairing it with the just-GA'd Agent Framework 1.0 for a *fully offline, data-never-leaves-the-box* agent is a fresh, timely angle with clear enterprise/privacy motivation. Include hardware/NPU acceleration notes and the trade-offs vs. cloud models — the honest comparison most local-LLM posts skip.
- **Prior art:**
  - https://dev.to/thangchung/microsoft-agent-framework-with-foundry-local-in-netc-3l63
  - https://dev.to/sreeni5018/running-ai-models-locally-on-your-mac-with-microsoft-foundry-local-fh7
  - https://dev.to/alinabi19/running-local-ai-models-in-net-with-ollama-step-by-step-guide-4die (the Ollama baseline this improves on)
- **Est. effort:** Medium

### Securing a C# MCP server for production: OAuth 2.1, Entra ID, and the On-Behalf-Of flow
- **Slug:** csharp-mcp-server-oauth-entra-production
- **Audience:** Advanced
- **Angle / gap:** "Build an MCP server in C#" is saturated, but the security layer is where teams actually get stuck. The MCP C# SDK now ships OAuth 2.1 support (RFC 9728 Protected Resource Metadata) built in; a focused piece on the resource-server role, validating tokens, caching JWKS, and the Entra ID On-Behalf-Of flow to call downstream APIs (e.g., Microsoft Graph) as the calling user — plus the MCP-specific threats (tool poisoning, data exfiltration) — fills the gap between hello-world servers and something you can actually ship.
- **Prior art:**
  - https://den.dev/blog/mcp-csharp-sdk-authorization/
  - https://devblogs.microsoft.com/ise/aca-secure-mcp-server-oauth21-azure-ad/
  - https://github.com/modelcontextprotocol/csharp-sdk
- **Est. effort:** High

### From Semantic Kernel to Agent Framework 1.0: a migration and decision guide for .NET teams
- **Slug:** semantic-kernel-to-agent-framework-migration
- **Audience:** Intermediate
- **Angle / gap:** With Agent Framework 1.0 GA (the convergence of Semantic Kernel + AutoGen), teams with existing SK code need a clear "should I migrate, and how" answer. An intro migration post exists (bspann); the gap is a decision-framework angle grounded in the 1.0 supported surface (agents, workflows, memory, middleware, orchestration) vs. early-adoption extras, with a concrete before/after code migration of a real SK agent and orchestration. Timely while the migration window is fresh and SK is entering maintenance posture.
- **Prior art:**
  - https://dev.to/bspann/migrating-from-semantic-kernel-to-microsoft-agent-framework-a-c-developers-guide-3ad5
  - https://visualstudiomagazine.com/articles/2026/04/06/microsoft-ships-production-ready-agent-framework-1-0-for-net-and-python.aspx
  - https://devblogs.microsoft.com/agent-framework/
- **Est. effort:** Medium

---

**Recommended first pick:** `dotnet-ai-agent-observability-opentelemetry` — it targets the widest, most defensible gap (near-zero .NET-native coverage despite built-in OpenTelemetry support in Microsoft.Extensions.AI and the Agent Framework), addresses a real production pain (runaway token cost) that intro content ignores, and is Medium effort. It also pairs naturally with any later agent/RAG article as a "make it production-ready" follow-up.
