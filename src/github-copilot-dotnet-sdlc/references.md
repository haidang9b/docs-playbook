# References — github-copilot-dotnet-sdlc

## Synthesis

**Topic:** GitHub Copilot for .NET — applying AI across the full SDLC (plan → code → test → review → docs → maintain), for intermediate .NET developers.

### Key findings

1. **Copilot is not one feature but a family of features, each fitting a different SDLC phase.** The official feature list [REF-1] and feature matrix [REF-2] group them as *assistive* (inline code completion, next edit suggestions, Copilot Chat, PR summaries, commit-message generation) and *agentic* (agent mode in the IDE, the cloud/coding agent on GitHub.com, Copilot CLI, code review). This is the backbone of the phase-by-phase mapping.

2. **Chat modes matter for .NET readers.** In Visual Studio and VS Code, Copilot Chat has three distinct modes — **Ask** (no edits unless you apply), **Edit** (multi-file iterative edits with inline review), and **Agent** (autonomous: plans steps, edits files, runs terminal commands/tests, iterates on build/test results) [REF-18][REF-19][REF-20]. VS also ships built-in specialized agents like `@debug`, `@profiler`, `@test`, and `@vs` [REF-18].

3. **Slash commands are the quickest per-phase entry points.** `/explain`, `/fix`, `/tests`, `/doc`, `/optimize`, `/fixTestFailure`, plus chat participants (`@workspace`, `@vscode`, `@github`, `@terminal`) and context variables (`#file`, `#selection`, `#sym`) [REF-3][REF-24]. These map cleanly to code, test, docs, and maintenance phases.

4. **There is a dedicated, .NET-specific test experience.** "GitHub Copilot testing for .NET" in Visual Studio 2026 (18.3+) is a guided, long-running capability in Copilot Chat that generates, builds, and runs C# tests for xUnit, NUnit, or MSTest — matching the solution's existing framework, or defaulting to MSTest. It is deterministic, grounded in the C# compiler, uses the `@Test #<target>` syntax, creates a separate test project, and requires a **paid** Copilot subscription (Free not supported) [REF-15][REF-16]. In VS Code, the equivalent is the **Generate Tests** smart action plus `/setupTests`, `/fixTestFailure`, and agent-mode auto-fix of failing tests [REF-39][REF-43]. This is a standout .NET example.

5. **Planning phase is now first-class.** VS Code has a **Plan agent** (`/plan`) that produces an implementation plan and todo list before coding, saved to a session memory file [REF-28]. On GitHub.com, the **cloud/coding agent** can be handed a GitHub issue (assign the issue to Copilot), research the repo, make changes on a branch, and open a PR for review [REF-5][REF-6][REF-7]. Good material for the plan and maintenance phases.

6. **Review phase has two Copilot features:** **Copilot code review** (request Copilot as a reviewer on a PR; leaves inline "Comment" reviews only — never Approve/Request-changes, so it doesn't satisfy required approvals; usually <30s; all paid plans) [REF-8][REF-9], and **PR summaries** (prose overview + bulleted, line-linked change list; manual, not auto-updated) [REF-10]. In-IDE, VS also offers "Review Selection" for local pre-PR review [REF-20].

7. **Maintenance/refactoring & modernization:** Agent mode can refactor across files, migrate legacy code, and generate docs [REF-27]. Microsoft ships **GitHub Copilot app modernization** — a Copilot agent across VS, VS Code, CLI, and GitHub.com that upgrades .NET projects and migrates apps to Azure [REF-22]. Debugging assistance is documented separately [REF-21].

8. **Documentation phase:** `/doc` generates a doc comment for a symbol [REF-3]; in VS, typing `///` triggers automatic XML doc-comment generation you accept with Tab [REF-25]. Comment-driven prompting is also documented [REF-26]. In VS Code, the **Generate Docs** smart action covers the same need [REF-43].

9. **Guardrails / honest-balance material is well documented.** Responsible-use pages stress human oversight, that generated code may be insecure or inaccurate ("hallucination"), and the need to verify [REF-30][REF-31][REF-32][REF-33]. Copilot's **code referencing / duplication-detection** filter checks suggestions (~150 chars of surrounding context) against an index of public GitHub code; teams set **"Suggestions matching public code"** to **Block** or **Allow** (Allow surfaces the matching file URLs and license) — matches occur in <1% of suggestions [REF-37][REF-40][REF-41]. Copilot testing for .NET requires explicit consent to run LLM-generated commands and recommends a sandboxed environment [REF-15]. Custom instructions (`.github/copilot-instructions.md`, path-specific `*.instructions.md`) let teams enforce project conventions [REF-11][REF-23].

10. **Plans/pricing (brief):** Free (2,000 completions/mo, limited chat/agent), Pro ($10/mo), Pro+ ($39/mo), Business ($19/user/mo), Enterprise ($39/user/mo); completions and next-edit suggestions are unlimited on paid plans; other features use AI credits [REF-34][REF-35].

11. **EF Core with Copilot is illustrative, not a dedicated documented workflow.** There is **no official "scaffold EF Core DbContext with Copilot" walkthrough**. The closest confirmed source is the MSSQL-extension quickstart, which shows real C# Entity Framework prompts (e.g. "Generate an Entity Framework model in C# based on the existing `SalesLT.Product` table") via the `@mssql` chat participant against a connected DB [REF-38]. Present EF Core scaffolding as an illustrative chat/agent use case ([REF-38], plus [REF-22] for modernization); classic reverse-engineering (`dotnet ef dbcontext scaffold`) is EF Core tooling, not a Copilot feature [REF-42].

12. **Customization & extensibility — how a team tailors Copilot to a .NET repo (new H2).** Four layers, each mapping to a role:
    - **Custom instructions = conventions.** Repo-wide `.github/copilot-instructions.md` (always applied) plus path-specific `*.instructions.md` in `.github/instructions/` with an `applyTo` glob (e.g. apply C#/EF conventions only to `**/*.cs`) [REF-11][REF-23][REF-46]. These steer *every* Copilot response toward the team's C#/.NET style.
    - **Prompt files = reusable workflows.** Standalone `*.prompt.md` files in `.github/prompts/`, invoked manually with `/promptname` (or Command Palette "Chat: Run Prompt"). They support frontmatter (`description`, `model`, `tools`, `agent`), can reference other workspace files with relative links, and take variables like `${selection}` and `${input:name}` — ideal for repeatable .NET tasks ("scaffold a minimal API controller with our logging + validation") [REF-44].
    - **Custom agents (formerly "custom chat modes") = scoped personas.** Markdown files with a fixed set of instructions + allowed tools that appear in the chat mode dropdown alongside Ask/Edit/Agent. Front matter includes `description`, `name`, `tools`, `model`, `mcp-servers`, etc. **Terminology/format changed:** VS Code now calls these **custom agents** using the **`.agent.md`** extension in **`.github/agents/`**; legacy **`.chatmode.md`** files (in `.github/chatmodes/`) still work but should be renamed to `.agent.md` [REF-45]. Visual Studio also has a "custom agents" experience [REF-51].
    - **MCP servers = external tools/context.** Model Context Protocol is an open standard that connects Copilot **agent mode** to external tools/data (databases, APIs, GitHub itself). Configured via `mcp.json`/`.mcp.json`; tools are opt-in and require per-tool approval [REF-47][REF-48][REF-49][REF-50]. **.NET-relevant angle:** the **Azure MCP server** and the **MSSQL/SQL Server MCP** expose live database schema/queries to Copilot; the **Azure DevOps MCP** manages work items; the **GitHub MCP** manages issues/PRs — all directly citeable [REF-50].

### Contradictions / cautions for the writer
- Terminology drift (agents): GitHub docs now say **"cloud agent"** where older material said **"coding agent"**; VS/VS Code call the in-editor autonomous mode **"agent mode."** Distinguish the *IDE agent* (local) from the *cloud/coding agent* (GitHub.com, GitHub Actions-powered). [REF-5][REF-18][REF-27]
- Terminology drift (customization): **"custom chat modes" were renamed to "custom agents"** in VS Code, and the file extension moved from **`.chatmode.md` → `.agent.md`** (folder `.github/chatmodes/` → `.github/agents/`). Use the current terms; mention the legacy names once so readers on older docs aren't confused. [REF-45]
- Copilot testing for .NET is **VS 2026 (18.3+) and paid-only** — don't present it as universally available. [REF-15]
- **MCP availability:** GA and broadly supported (VS Code, Visual Studio, JetBrains, Xcode, Eclipse, Cursor, Windsurf). In **Visual Studio, MCP requires VS 2022 17.14+ or VS 2026** (latest servicing recommended), only works in **agent mode**, and tools are disabled by default (per-tool approval; MCP server *trust* dialog added in VS 2026 18.7). Business/Enterprise orgs must enable the "MCP servers in Copilot" policy; admins can set an allowlist. [REF-47][REF-48][REF-50]
- Prompt files and custom agents/chat modes are documented in the **VS Code** docs; frame `.github/prompts` and `.agent.md`/`.chatmode.md` as VS Code features (portable via the repo) rather than universal across every IDE. Custom *instructions* are the most broadly supported customization (GitHub-wide + IDEs). [REF-44][REF-45][REF-11]
- Feature availability varies by IDE and plan; cite the feature matrix rather than over-generalizing. [REF-2]
- EF Core + Copilot: don't claim a dedicated official walkthrough; frame as illustrative and cite [REF-38]. [REF-38]

### Recommended outline (H2/H3)
- **H2: Why apply Copilot across the SDLC (not just autocomplete)** — reframe from "faster typing" to whole-lifecycle assistant; intro the assistive-vs-agentic split. [REF-1][REF-2]
- **H2: Setup for .NET teams** — VS 2022/2026 vs VS Code + C# Dev Kit; plans in brief; custom instructions (`copilot-instructions.md`) as the team-conventions foundation. [REF-11][REF-23][REF-34]
- **H2: Phase 1 — Planning & requirements** — Ask/Chat to break down a feature; VS Code Plan agent (`/plan`); turning GitHub issues into plans. [REF-28][REF-6]
- **H2: Phase 2 — Coding** — completions & next edit suggestions; Ask/Edit/Agent modes; `@workspace`, context variables; .NET example (scaffold a class, EF Core DbContext/entities — illustrative). [REF-17][REF-19][REF-3][REF-27][REF-38]
- **H2: Phase 3 — Testing** — generic `/tests` vs the dedicated **Copilot testing for .NET** (`@Test`, xUnit/NUnit/MSTest) in VS; the **Generate Tests** smart action in VS Code. [REF-15][REF-16][REF-39]
- **H2: Phase 4 — Code review** — in-IDE "Review Selection"; PR summaries; Copilot as PR reviewer and its limits. [REF-20][REF-9][REF-10]
- **H2: Phase 5 — Documentation** — `/doc`, `///` XML doc auto-generation, Generate Docs (VS Code), README/API docs via chat. [REF-3][REF-25][REF-43][REF-36]
- **H2: Phase 6 — Maintenance, refactoring & modernization** — agent-mode multi-file refactors; debugging with Copilot; app modernization / Azure migration; cloud agent for issue-driven fixes. [REF-27][REF-21][REF-22][REF-7]
- **H2: Customizing & extending Copilot for your .NET repo** — custom instructions (conventions) → prompt files (reusable workflows) → custom agents / chat modes (scoped personas) → MCP servers (external tools, e.g. Azure/SQL/GitHub); note VS vs VS Code availability. [REF-11][REF-46][REF-44][REF-45][REF-47][REF-48][REF-49][REF-50][REF-51]
- **H2: Best practices, limits & guardrails** — verify everything; security review; matching-public-code / IP filter (Block vs Allow) and code referencing; data handling; consent/sandboxing for agentic + MCP runs; custom instructions to steer quality. [REF-30][REF-31][REF-32][REF-33][REF-37][REF-40][REF-15]
- **H2: Next Steps** — CTA: start with completions + custom instructions, add one agentic workflow per phase.

---

## References
<!-- Confirmed:   [REF-N] Title — https://url -->
<!-- Placeholder: [REF-N] TODO: verify — what is still needed -->

### Core Copilot capabilities (GitHub Docs)
- [REF-1] GitHub Copilot features — https://docs.github.com/en/copilot/get-started/features
- [REF-2] Copilot feature matrix — https://docs.github.com/en/copilot/reference/copilot-feature-matrix
- [REF-3] GitHub Copilot Chat cheat sheet (slash commands, participants, context variables) — https://docs.github.com/en/copilot/reference/chat-cheat-sheet
- [REF-4] Asking GitHub Copilot questions in your IDE — https://docs.github.com/en/copilot/how-tos/chat-with-copilot/chat-in-ide
- [REF-5] About GitHub Copilot cloud agent — https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent
- [REF-6] Using Copilot cloud agent on GitHub — https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/use-cloud-agent-on-github
- [REF-7] Asking GitHub Copilot to create a pull request (assign an issue to Copilot) — https://docs.github.com/copilot/how-tos/use-copilot-agents/coding-agent/assign-copilot-to-an-issue
- [REF-8] About GitHub Copilot code review — https://docs.github.com/en/copilot/concepts/agents/code-review
- [REF-9] Using GitHub Copilot code review — https://docs.github.com/copilot/using-github-copilot/code-review/using-copilot-code-review
- [REF-10] Creating a pull request summary with GitHub Copilot — https://docs.github.com/copilot/using-github-copilot/creating-a-pull-request-summary-with-github-copilot
- [REF-11] Adding repository custom instructions for GitHub Copilot — https://docs.github.com/copilot/customizing-copilot/adding-custom-instructions-for-github-copilot
- [REF-12] Using GitHub Copilot CLI (overview) — https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/overview
- [REF-13] About GitHub Copilot CLI — https://docs.github.com/copilot/concepts/agents/about-copilot-cli
- [REF-14] GitHub Copilot CLI command reference — https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference

### .NET / Visual Studio integration (Microsoft Learn & VS Blog)
- [REF-15] Overview of GitHub Copilot testing for .NET (Visual Studio) — https://learn.microsoft.com/en-us/visualstudio/test/github-copilot-test-dotnet-overview?view=visualstudio
- [REF-16] Generate and run unit tests using GitHub Copilot testing (Visual Studio) — https://learn.microsoft.com/en-us/visualstudio/test/unit-testing-with-github-copilot-test-dotnet?view=visualstudio
- [REF-17] About GitHub Copilot Completions in Visual Studio — https://learn.microsoft.com/en-us/visualstudio/ide/visual-studio-github-copilot-extension?view=vs-2022
- [REF-18] Use Agent Mode in Visual Studio — https://learn.microsoft.com/en-us/visualstudio/ide/copilot-agent-mode?view=visualstudio
- [REF-19] GitHub Copilot Edits in Visual Studio — https://learn.microsoft.com/en-us/visualstudio/ide/copilot-edits?view=vs-2022
- [REF-20] About GitHub Copilot Chat in Visual Studio (Ask mode, Review Selection) — https://learn.microsoft.com/en-us/visualstudio/ide/visual-studio-github-copilot-chat?view=visualstudio
- [REF-21] Debug with GitHub Copilot (Visual Studio) — https://learn.microsoft.com/en-us/visualstudio/debugger/debug-with-copilot?view=visualstudio
- [REF-22] GitHub Copilot app modernization overview (.NET) — https://learn.microsoft.com/en-us/dotnet/core/porting/github-copilot-app-modernization/overview
- [REF-23] Customize chat responses / custom instructions in Visual Studio — https://learn.microsoft.com/en-us/visualstudio/ide/copilot-chat-context?view=visualstudio
- [REF-24] Mastering Slash Commands with GitHub Copilot in Visual Studio (VS Blog) — https://devblogs.microsoft.com/visualstudio/mastering-slash-commands-with-github-copilot-in-visual-studio/
- [REF-25] Introducing automatic documentation comment generation in Visual Studio (VS Blog) — https://devblogs.microsoft.com/visualstudio/introducing-automatic-documentation-comment-generation-in-visual-studio/
- [REF-26] How to use Comments to Prompt GitHub Copilot for Visual Studio (VS Blog) — https://devblogs.microsoft.com/visualstudio/how-to-use-comments-to-prompt-github-copilot-visual-studio/
- [REF-36] Generate Documentation Using GitHub Copilot Tools (Microsoft Learn Training) — https://learn.microsoft.com/en-us/training/modules/generate-documentation-using-github-copilot-tools/
- [REF-38] Quickstart: Code Generation with GitHub Copilot for SQL — Entity Framework model prompts (MSSQL extension for VS Code) — https://learn.microsoft.com/en-us/sql/tools/visual-studio-code-extensions/github-copilot/code-generation?view=sql-server-ver17
- [REF-42] Reverse Engineering — EF Core scaffolding (Scaffold-DbContext / dotnet ef dbcontext scaffold) — https://learn.microsoft.com/en-us/ef/core/managing-schemas/scaffolding/

### VS Code agent, planning & testing
- [REF-27] Introducing GitHub Copilot agent mode (VS Code blog) — https://code.visualstudio.com/blogs/2025/02/24/introducing-copilot-agent-mode
- [REF-28] Planning with agents in VS Code (Plan agent, /plan) — https://code.visualstudio.com/docs/agents/planning
- [REF-29] Best practices for using AI in VS Code — https://code.visualstudio.com/docs/agents/best-practices
- [REF-39] Test with GitHub Copilot (VS Code — Generate Tests smart action, /setupTests, /fixTestFailure, agent-mode auto-fix) — https://code.visualstudio.com/docs/agents/guides/test-with-copilot
- [REF-43] AI smart actions in Visual Studio Code (Generate Tests / Generate Docs) — https://code.visualstudio.com/docs/editing/copilot-smart-actions

### Customization & extensibility
- [REF-44] Use prompt files in VS Code (`.prompt.md` in `.github/prompts`, invoke with `/name`, variables `${selection}`/`${input:...}`) — https://code.visualstudio.com/docs/copilot/customization/prompt-files
- [REF-45] Custom agents in VS Code (formerly "custom chat modes"; `.agent.md` in `.github/agents/`; legacy `.chatmode.md` still supported) — https://code.visualstudio.com/docs/copilot/customization/custom-chat-modes
- [REF-46] Use custom instructions in VS Code (path-specific `*.instructions.md`, `applyTo` globs, `.github/instructions/`) — https://code.visualstudio.com/docs/agent-customization/custom-instructions
- [REF-47] About Model Context Protocol (MCP) — GitHub Copilot (concept; IDE support matrix) — https://docs.github.com/en/copilot/concepts/context/mcp
- [REF-48] Extending GitHub Copilot Chat with Model Context Protocol (MCP) servers (GitHub Docs how-to) — https://docs.github.com/en/copilot/how-tos/provide-context/use-mcp-in-your-ide/extend-copilot-chat-with-mcp
- [REF-49] Add and manage MCP servers in VS Code (`.vscode/mcp.json`, MCP: Add Server) — https://code.visualstudio.com/docs/copilot/customization/mcp-servers
- [REF-50] Use MCP Servers to Extend GitHub Copilot — Visual Studio (VS 2022 17.14+/VS 2026, agent mode, `.mcp.json`, allowlist, Azure/SQL/GitHub MCP examples) — https://learn.microsoft.com/en-us/visualstudio/ide/mcp-servers?view=visualstudio
- [REF-51] Use custom agents in GitHub Copilot — Visual Studio — https://learn.microsoft.com/en-us/visualstudio/ide/copilot-specialized-agents?view=visualstudio

### Best practices, limitations & guardrails
- [REF-30] Responsible use of GitHub Copilot features — https://docs.github.com/en/copilot/responsible-use
- [REF-31] Responsible use of GitHub Copilot Chat in your IDE — https://docs.github.com/en/copilot/responsible-use/chat-in-your-ide
- [REF-32] Responsible use of GitHub Copilot pull request summaries — https://docs.github.com/en/copilot/responsible-use-of-github-copilot-features/responsible-use-of-github-copilot-pull-request-summaries
- [REF-33] Responsible use of GitHub Copilot cloud agent on GitHub.com — https://docs.github.com/en/copilot/responsible-use/copilot-coding-agent
- [REF-37] GitHub Copilot code referencing (duplication detection; matching-public-code filter; license/URL logging) — https://docs.github.com/en/copilot/concepts/completions/code-referencing
- [REF-40] Managing GitHub Copilot policies as an individual subscriber ("Suggestions matching public code" Allow/Block setting) — https://docs.github.com/en/copilot/managing-copilot/managing-copilot-as-an-individual-subscriber/managing-copilot-policies-as-an-individual-subscriber
- [REF-41] Finding public code that matches GitHub Copilot suggestions — https://docs.github.com/copilot/using-github-copilot/finding-public-code-that-matches-github-copilot-suggestions

### Plans / pricing
- [REF-34] Plans for GitHub Copilot — https://docs.github.com/en/copilot/get-started/plans
- [REF-35] Models and pricing for GitHub Copilot — https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing

### Notes on resolved gaps
- **[REF-37] (was TODO):** Resolved. The matching-public-code / IP filter is documented as **code referencing** [REF-37], with the Allow/Block control on the policy pages [REF-40] and the matched-code log at [REF-41]. All verified.
- **[REF-38] (was TODO):** Partially resolved. No dedicated "scaffold EF Core with Copilot" page exists; the closest verified source with real EF/C# Copilot prompts is the MSSQL-extension quickstart [REF-38]. EF Core scaffolding should be framed as illustrative; classic EF tooling is [REF-42].
- **[REF-39] (was TODO):** Resolved. VS Code equivalent of the VS ".NET testing" page is "Test with GitHub Copilot" [REF-39], complemented by the smart-actions page [REF-43]. All verified.

### Notes on customization & extensibility (REF-44–REF-51)
- **Terminology change (must honor):** VS Code renamed **"custom chat modes" → "custom agents"** and the file format **`.chatmode.md` → `.agent.md`** (`.github/chatmodes/` → `.github/agents/`). Legacy `.chatmode.md` files still work but should be renamed. [REF-45]
- **VS Code vs Visual Studio:** Prompt files [REF-44] and custom agents/chat modes [REF-45] are VS Code docs (portable via the repo). Custom *instructions* are broadly supported (GitHub-wide + IDEs) [REF-11][REF-23][REF-46]. Visual Studio has its own custom-agents page [REF-51].
- **MCP availability caveat:** GA and cross-IDE [REF-47]. In **Visual Studio, MCP needs VS 2022 17.14+ or VS 2026** (agent mode only; tools disabled by default; per-tool approval; server-trust dialog in VS 2026 18.7); Business/Enterprise must enable the "MCP servers in Copilot" policy; admins can set an allowlist [REF-50][REF-48]. Config lives in `mcp.json`/`.mcp.json` (VS) or `.vscode/mcp.json` (VS Code) [REF-49][REF-50].
- **.NET angle for MCP:** Azure MCP, MSSQL/SQL Server MCP (live DB schema/queries), Azure DevOps MCP (work items), and GitHub MCP (issues/PRs) are documented examples usable from Copilot agent mode [REF-50].
