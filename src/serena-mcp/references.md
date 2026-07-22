# References — serena-mcp

## Synthesis

**Topic:** Serena, an open-source MCP server that gives AI coding agents semantic, symbol-level code tools backed by the Language Server Protocol (LSP).

### Key findings (mapped to what / why / how / pros & cons)

**What it is**
- Serena (repo `oraios/serena`) is a free, open-source **Model Context Protocol (MCP) toolkit** that turns any MCP-capable LLM into a coding agent with IDE-like abilities. Its tagline is "the IDE for your agent." [REF-1]
- It is released under the **MIT license** — no subscription, no vendor lock-in. [REF-1][REF-4]
- MCP itself is an open standard from Anthropic — a "USB-C port for AI" — that connects AI apps to external tools/data over a JSON-RPC client-server protocol (stdio or HTTP). Serena is one such server. [REF-2]

**Why use it (the value angle)**
- Most coding MCP servers (DesktopCommander, codemcp) rely on **text-based** file reading and editing. Serena is distinctive because it provides **semantic** retrieval and editing at the symbol level. [REF-3][REF-1]
- Instead of dumping whole files into the context window, Serena returns **targeted, symbol-level context** (a symbol's body, its references, a file outline), which reduces token consumption and improves accuracy on large codebases. [REF-1][REF-4][REF-3]
- Multi-step, error-prone text edits (cross-file rename, move, reference lookup) "collapse into one atomic call" — the README quotes agents calling it "the single most impactful addition" to their toolkit. [REF-1]

**How it works**
- Serena abstracts the **Language Server Protocol** and drives real, open-source language servers per language; the LSP backend is the default and is free. [REF-1]
- Core LSP-backed tools (names per current README): `find_symbol`, `symbol_overview` (file outline), `find_referencing_symbols`, `replace_symbol_body`, `insert_after_symbol`, `insert_before_symbol`, `safe_delete`, plus basic utilities `search_for_pattern`, `replace_content`, `list_dir`, `find_file`, `read_file`, `execute_shell_command`. [REF-1]
- A **memory + onboarding** system lets the agent record project knowledge for reuse across sessions/long-lived workflows, and teams can seed onboarding details so the agent learns their workflow. [REF-1][REF-4]
- **Installation** is via the Python packaging tool `uv` — `uv tool install serena-agent` (or run ad-hoc with `uvx`), then wire the launch command / HTTP URL into the client. The README explicitly warns *not* to install via MCP/plugin marketplaces (outdated commands). [REF-1][REF-4]
- **Clients:** Claude Code, Claude Desktop, Cursor, VSCode, JetBrains IDEs, Codex, Gemini-CLI, OpenCode, and Agno framework integration. [REF-1][REF-4]
- **Languages:** the README lists **40+** languages via language servers (Python, TypeScript/JavaScript, Java, C/C++, C#, Go, Rust, Ruby, PHP, Kotlin, Swift, etc.); an optional paid JetBrains backend broadens coverage. [REF-1]

**Token efficiency — what is (and isn't) quantified**
- **Anecdotal savings figure:** a DEV Community walkthrough reports "*I have seen people say Serena can save your tokens up to about 70%!*" and adds from personal use that Serena "does save tokens and extend the time before hitting Claude Code limit." This **~70% figure is community/anecdotal**, attributed to unnamed users with no stated methodology — cite it as an anecdote, not a benchmark. [REF-5]
- **Verified overhead number (a nuance / con):** Serena's tool definitions themselves cost context. Per the `serena-slim` fork's measurements, Serena exposes **29 tools** whose schemas measure **~7,348 tokens** (tiktoken `cl100k_base`), rising to an estimated **~23,878 tokens** in Claude Code (which adds ~570 tokens/tool overhead). serena-slim regroups these into 18 operations for a ~50.3% overhead reduction. This quantifies Serena's up-front context footprint — useful for the pros/cons section — but it is about tool-definition bloat, **not** retrieval savings. [REF-8]
- **Rigorous real-world benchmark (now verified, REF-9):** ManoMano "Project AEGIS" (Feb 2026) compared Vanilla Claude vs Claude Code + built-in LSP vs Claude + Serena on a production **Java payment service: 381 classes, 36,407 lines of code, 1,017 unit tests**, across four tasks culminating in a major refactor (extracting date fields into a `PaymentDates` record with full-codebase updates). Verified figures on the refactor task: [REF-9]
  - **Claude + Serena — 45 min, $27.30, 4 subagents, build passed with all 1,017 tests green** (read ~69M tokens overall while keeping API cost contained).
  - **Vanilla Claude — 60 min, $23.54, 12 subagents, build FAILED.**
  - **Claude + built-in LSP — 60 min, $28.63, build FAILED (9 tests)**; the built-in LSP hallucinated / confused same-named methods (authors recommend avoiding it for autonomous tasks).
  - **Honest counter-point:** on a *simple* read-only exploration task, "Serena cost nearly four times as much and took 60% longer" than vanilla Claude — Serena's overhead only pays off on deep modification tasks. [REF-9]
  - Takeaway for the writer: Serena is cheaper/faster **only relative to succeeding** — vanilla was nominally cheaper but *failed the build*. Frame savings as "reliability + fewer wasted subagents on large refactors," not a flat token-percentage.

**Pros & cons**
- Pros: free/MIT, semantic (not text) understanding, token-efficient retrieval, 40+ languages, broad client support, atomic refactors. [REF-1][REF-4]
- Cons / limitations: setup/config complexity (multiple layers to tune); capability depends on the quality of the underlying language server per language; LSP backend can't rename/move files or fully resolve declarations in external dependencies (JetBrains backend only); JetBrains backend excludes Rider/CLion and is paid; project is under active development with evolving docs; the tool set itself consumes sizable context (~24k tokens in Claude Code per REF-8); context-window limits still apply to very large files; always review AI edits before merging. [REF-1][REF-3][REF-4][REF-8]

### Contradictions / caveats flagged
- **Installation method conflict:** the SmartScope post claims `npm install @oraios/serena` (Node). This contradicts the official README, which is a **Python/`uv`** project (`uv tool install serena-agent` / `uvx`). Treat the official README as authoritative; the npm command appears incorrect. [REF-1 vs REF-3]
- **Language count:** README says "40+"; some third-party posts say "30+." Prefer the README figure. [REF-1 vs REF-3]
- **Exact tool set evolves:** the tool list above reflects a recent README fetch, including some JetBrains-backend-only tools (e.g. `move`, `inline`, `type_hierarchy`, debugging). Classic tool names may differ slightly (e.g. `get_symbols_overview`). Verify precise current tool names/backends against the live README before publishing — see [REF-6].
- **Token savings figure is soft; use the benchmark instead:** the "~70%" number is anecdotal (REF-5). Prefer the verified ManoMano benchmark (REF-9) for hard numbers — but note it measures **reliability, time, cost, and subagent count on a large refactor**, not a flat token-savings percentage. The one hard tool-context number (REF-8, ~24k tokens of tool schemas in Claude Code) is a *cost*, not a *saving*. Net framing for the writer: Serena's win is finishing hard tasks correctly (build + all tests green) with fewer wasted subagents, and it is *not* worth the overhead for trivial read-only queries.

### Recommended outline for the writer
1. **Hook / BLUF** — AI agents burning tokens reading whole files and botching edits; Serena fixes this with semantic, symbol-level tools. (Weave focus keyphrase in first 100 words.)
2. **What is Serena MCP?** — MIT-licensed open-source MCP server; one-line MCP background (link to MCP). [REF-1][REF-2]
3. **Why use it** — semantic vs text-based; accuracy/reliability on large codebases (lead with the ManoMano refactor benchmark, REF-9: Serena passed all 1,017 tests where vanilla and built-in-LSP builds failed); atomic refactors; token efficiency (frame ~70% as anecdotal, REF-5). [REF-1][REF-3][REF-9][REF-5]
4. **How it works** — LSP + language servers architecture; the tool set (`find_symbol`, `find_referencing_symbols`, symbol edit/insert, search); onboarding + memories; install with `uvx`; client integration. [REF-1][REF-4]
5. **Supported languages** — 40+ via LSP; optional JetBrains backend. [REF-1]
6. **Pros & cons** — table; include the tool-schema context overhead nuance (~24k tokens, REF-8). [REF-1][REF-3][REF-4][REF-8]
7. **Next Steps CTA** — install, add to Claude Code/Cursor, run onboarding.

## References
<!-- Confirmed:   [REF-N] Title — https://url -->
<!-- Placeholder: [REF-N] TODO: verify — what is still needed -->

- [REF-1] oraios/serena — A powerful MCP toolkit for coding (official GitHub repo, README, tool list, languages, install, license) — https://github.com/oraios/serena
- [REF-2] What is the Model Context Protocol (MCP)? — official MCP docs (background on MCP servers/clients) — https://modelcontextprotocol.io/introduction
- [REF-3] Serena MCP: Free AI Coding Agent with Full Codebase Understanding (2026) — SmartScope — https://smartscope.blog/en/generative-ai/claude/serena-mcp-coding-agent/
- [REF-4] How to Set Up Serena MCP Server: The Free AI Coding Agent — Apidog — https://apidog.com/blog/serena-mcp-server/
- [REF-5] How to use AI more efficiently for free (Serena MCP) — DEV Community — https://dev.to/webdeveloperhyper/how-to-use-ai-more-efficiently-for-free-serena-mcp-5gj6 — states an ANECDOTAL "save your tokens up to about 70%" (attributed to unnamed users, no methodology); use as anecdote only.
- [REF-6] Serena tool names + launch command (verified against live source/docs, 2026-07-22): symbol tools per `src/serena/tools/symbol_tools.py` are `find_symbol`, `get_symbols_overview` (NOT `symbol_overview`), `find_referencing_symbols`, `find_implementations`, `find_declaration`, `replace_symbol_body`, `insert_after_symbol`, `insert_before_symbol`, `rename_symbol`, `safe_delete_symbol`, plus diagnostics/restart tools — https://github.com/oraios/serena/blob/main/src/serena/tools/symbol_tools.py . Launch (Serena docs "Running"): install `uv tool install -p 3.13 serena-agent`; ad-hoc `uvx -p 3.13 --from git+https://github.com/oraios/serena serena start-mcp-server`; cloned `uv run serena start-mcp-server` — https://oraios.github.io/serena/02-usage/020_running.html
- [REF-7] TODO: verify — Anthropic's official MCP announcement URL (https://www.anthropic.com/news/model-context-protocol) for the "why MCP matters" background paragraph, if the writer wants a primary Anthropic citation alongside REF-2.
- [REF-8] serena-slim — serena MCP (38% less tokens) — GitHub (mcpslim/serena-slim) — https://github.com/mcpslim/serena-slim — VERIFIED numbers: Serena's 29 tool schemas = ~7,348 tokens (tiktoken cl100k_base), ~23,878 tokens in Claude Code (~570 tokens/tool overhead); serena-slim cuts to 18 ops / ~11,874 tokens (~50.3% overhead reduction). Note: measures Serena's tool-definition context footprint, not semantic-retrieval savings.
- [REF-9] Benchmarking AI Coding Agents: Claude vs Claude Code vs Serena MCP on 36K Lines of Java (Project AEGIS) — ManoMano Tech — https://medium.com/manomano-tech/project-aegis-benchmarking-ai-agents-and-why-serena-is-our-new-must-have-311673db35dd — VERIFIED (via reader proxy + corroborating search). Refactor task: Claude+Serena 45 min / $27.30 / 4 subagents / all 1,017 tests pass / ~69M tokens read; Vanilla Claude 60 min / $23.54 / 12 subagents / build failed; Claude+LSP 60 min / $28.63 / build failed (9 tests, LSP hallucinated same-named methods). Simple exploration task: Serena ~4x cost and ~60% slower than vanilla. Codebase: 381 classes, 36,407 LOC, 1,017 tests.
