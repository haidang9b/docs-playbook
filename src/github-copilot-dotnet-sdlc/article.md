---
title: "GitHub Copilot for .NET: Applying AI Across the SDLC"
description: "GitHub Copilot for .NET: a phase-by-phase guide to using AI across planning, coding, testing, review, docs, and maintenance on your team."
focus_keyphrase: "GitHub Copilot for .NET"
keywords: ["GitHub Copilot for .NET", "Copilot agent mode", "Copilot Chat", "Copilot testing for .NET", "Copilot code review", "custom instructions", "AI SDLC"]
author: ""
date: "2026-07-23"
slug: "github-copilot-dotnet-sdlc"
---

# GitHub Copilot for .NET: Applying AI Across the SDLC

Most teams switch on **GitHub Copilot for .NET**, watch it finish a few lines of C#, and quietly file it under "fancy autocomplete." That framing leaves most of the value on the table. Copilot is not a single feature — it is a family of assistive and agentic capabilities, and each one maps to a different phase of the software development lifecycle [REF-1][REF-2].

This guide walks through the **SDLC phase by phase** — planning, coding, testing, code review, documentation, and maintenance — and shows which Copilot feature fits each step, with concrete C# and .NET examples. It is written for intermediate .NET developers who are rolling Copilot out to a team and want to move past autocomplete without over-trusting the output.

---

## Why apply Copilot across the SDLC (not just autocomplete)

The official feature set splits cleanly into two categories, and understanding the split is what unlocks whole-lifecycle use [REF-1][REF-2]:

- **Assistive features** keep you in the driver's seat: inline **code completion**, next edit suggestions, **Copilot Chat**, pull request summaries, and commit-message generation [REF-1].
- **Agentic features** take initiative: **agent mode** in the IDE, the **cloud agent** on GitHub.com, Copilot CLI, and **Copilot code review** [REF-1][REF-2].

The mental model that follows is simple: use **assistive** features for the tight, second-by-second loop of writing code, and reach for **agentic** features when a task spans multiple files, multiple steps, or an entire issue.

| SDLC phase | Primary Copilot feature | Where it runs |
|---|---|---|
| Planning | Ask mode, Plan agent (`/plan`), cloud agent | IDE + GitHub.com [REF-28][REF-6] |
| Coding | Completions, Ask/Edit/Agent modes | IDE [REF-17][REF-19][REF-27] |
| Testing | `/tests`, Copilot testing for .NET (`@Test`), Generate Tests | IDE [REF-15][REF-39] |
| Code review | Review Selection, PR summaries, Copilot code review | IDE + GitHub.com [REF-20][REF-9][REF-10] |
| Documentation | `/doc`, `///` auto-docs, Generate Docs | IDE [REF-3][REF-25][REF-43] |
| Maintenance | Agent mode, app modernization, cloud agent | IDE + GitHub.com [REF-27][REF-22][REF-7] |

One terminology note up front: **agent mode** is the autonomous, in-editor mode in Visual Studio and VS Code, while the **cloud agent** (older docs say "coding agent") runs on GitHub.com, is driven by issues, and is powered by GitHub Actions [REF-5][REF-18][REF-27]. They are related but distinct — keep them separate in your head and in your team's docs.

---

## Setup for .NET teams

**Copilot ships in both Visual Studio and VS Code.** In Visual Studio 2022 and 2026, the Copilot experience is built in, covering completions, chat, and agent mode [REF-17][REF-18]. In VS Code, install Copilot alongside the **C# Dev Kit** for the full .NET experience, including agent mode and smart actions [REF-27][REF-43].

### Pick a plan

Copilot has a **Free** tier (2,000 completions per month, limited chat and agent usage) and paid tiers — **Pro** ($10/mo), **Pro+** ($39/mo), **Business** ($19/user/mo), and **Enterprise** ($39/user/mo) [REF-34][REF-35]. On paid plans, completions and next edit suggestions are unlimited; other features draw on AI credits [REF-35]. This matters for the testing phase below, where the .NET-specific experience is paid-only [REF-15].

### Establish team conventions with custom instructions

Before anyone writes a prompt, set up **custom instructions**. A repository-level `.github/copilot-instructions.md` file lets Copilot honor your project's conventions on every request [REF-11], and Visual Studio reads the same custom-instruction mechanism [REF-23]. You can also add path-specific `*.instructions.md` files to target rules at particular folders [REF-11].

```markdown
<!-- .github/copilot-instructions.md -->
# Project conventions for Copilot

- Target .NET 8 with nullable reference types enabled.
- Use file-scoped namespaces and primary constructors where practical.
- Prefer `async`/`await` end-to-end; never block on `.Result` or `.Wait()`.
- Follow the existing repository pattern in `src/Data`; do not add new ORMs.
- Write xUnit tests with Arrange/Act/Assert comments and FluentAssertions.
```

Treat this file as the single most valuable investment in Copilot quality. It steers every downstream phase, from scaffolding to tests to docs.

---

## Phase 1 — Planning & requirements

Copilot is genuinely useful *before* the first line of code. Start in **Ask mode**, which answers questions and proposes approaches without editing your files unless you explicitly apply a suggestion [REF-20].

Use it to break a feature into steps:

```text
Ask: I need to add rate limiting to our ASP.NET Core minimal API.
Walk me through the options in .NET 8 and outline an implementation plan.
```

### Plan agent in VS Code

VS Code goes a step further with a dedicated **Plan agent** (`/plan`) that produces a structured implementation plan and a todo list *before* coding, saving it to a session memory file that agent mode can then follow [REF-28]. This keeps large tasks from turning into an unreviewable wall of edits.

```text
/plan Add a new "Orders" bounded context: EF Core entities,
a repository, a service, and minimal API endpoints. Produce a todo list.
```

### Turning GitHub issues into plans

On GitHub.com, you can hand a whole issue to the **cloud agent**: assign the issue to Copilot and it researches the repository, works on a branch, and opens a pull request for review [REF-6][REF-7]. This is issue-driven planning-plus-execution, and it is distinct from the in-IDE agent mode discussed next [REF-5].

---

## Phase 2 — Coding

This is where most of the day-to-day value lives, and it spans all three chat modes plus inline completion.

### Inline completions and next edit suggestions

As you type C#, Copilot offers **inline completions** you accept with Tab; **next edit suggestions** then predict the *following* edit based on the change you just made [REF-1][REF-17]. Write a descriptive signature and let the body follow:

```csharp
// Type this signature and pause — Copilot completes the body:
public static decimal CalculateDiscount(decimal subtotal, CustomerTier tier) =>
    tier switch
    {
        CustomerTier.Gold => subtotal * 0.85m,
        CustomerTier.Silver => subtotal * 0.92m,
        _ => subtotal
    };
```

### Ask, Edit, and Agent modes

Copilot Chat has **three modes**, and choosing the right one is half the skill [REF-18][REF-19][REF-20]:

- **Ask** — answers and explains; makes no edits unless you apply them [REF-20].
- **Edit** — makes iterative, multi-file edits that you review inline before accepting [REF-19].
- **Agent** — works autonomously: it plans steps, edits files, runs terminal commands and tests, and iterates on the build/test results [REF-18][REF-27].

Visual Studio also ships specialized built-in agents you invoke by name, including `@debug`, `@profiler`, `@test`, and `@vs` [REF-18].

Scaffold a class with an **Ask** or **Edit** prompt, using context variables like `#file`, `#selection`, and `#sym` to ground the request, or `@workspace` to reference the whole project [REF-3][REF-24]:

```text
Edit: Create an OrderService class in #file:Services/ that depends on
IOrderRepository and ILogger<OrderService>. Add a PlaceOrderAsync method
that validates the cart, persists the order, and returns a Result<OrderDto>.
```

### EF Core with Copilot (illustrative)

A common question is whether Copilot can scaffold an **EF Core** `DbContext` and entities. There is **no dedicated, documented "scaffold EF Core with Copilot" workflow** — treat this as an illustrative use of chat or agent mode, not a first-class feature. The closest official source with real Entity Framework prompts is the MSSQL extension's quickstart, which uses the `@mssql` chat participant against a connected database [REF-38]:

```text
@mssql Generate an Entity Framework model in C# based on the
existing SalesLT.Product table.
```

If you want deterministic, schema-accurate scaffolding instead, that is a job for classic EF Core tooling (`dotnet ef dbcontext scaffold`), which is EF Core functionality rather than a Copilot feature [REF-42].

---

## Phase 3 — Testing

Testing is where .NET gets a genuinely differentiated experience — but you have to know which path you are on, because there are three.

### Generic `/tests`

The slash command **`/tests`** generates tests for the current selection in any Copilot-enabled IDE [REF-3]. It is quick and broadly available, but it is not specialized to .NET test frameworks.

```text
/tests Generate xUnit tests for the selected CalculateDiscount method,
covering Gold, Silver, and default tiers plus a zero-subtotal edge case.
```

### Copilot testing for .NET (Visual Studio, paid only)

**"GitHub Copilot testing for .NET"** is a distinct, .NET-specific capability — and it is scoped: it requires **Visual Studio 2026 (18.3+)** and a **paid** Copilot subscription (the Free plan is not supported) [REF-15]. It is a guided, long-running experience in Copilot Chat that generates, builds, and *runs* C# tests, matching your solution's existing framework (**xUnit, NUnit, or MSTest**) or defaulting to MSTest [REF-15][REF-16].

It is grounded in the C# compiler for deterministic results, creates a separate test project, and is invoked with the `@Test #<target>` syntax [REF-15][REF-16]:

```text
@Test #OrderService
```

Because it runs LLM-generated commands, it asks for explicit consent and recommends a sandboxed environment [REF-15]. Do not confuse this with the generic `/tests` command or the VS Code path below — they are separate features.

### Generate Tests in VS Code

In VS Code, the equivalent is the **Generate Tests** smart action, complemented by `/setupTests`, `/fixTestFailure`, and agent-mode auto-fixing of failing tests [REF-39][REF-43]. When a test fails, `/fixTestFailure` targets the specific failure rather than regenerating the suite [REF-39].

---

## Phase 4 — Code review

Copilot supports review both locally and on the pull request — with one important limitation to communicate to your team.

### In-IDE pre-PR review

Visual Studio offers **Review Selection**, which reviews highlighted code in the editor before you ever push [REF-20]. It is a fast way to catch obvious issues while the change is still fresh.

### PR summaries

On GitHub.com, Copilot can generate a **pull request summary** — a prose overview plus a bulleted, line-linked list of changes [REF-10]. Note it is generated on demand and is **not** auto-updated as you push more commits, so regenerate it before merge [REF-10].

### Copilot code review — and its limits

You can request **Copilot as a reviewer** on a PR; it typically responds in under 30 seconds and is available on all paid plans [REF-8][REF-9]. The crucial caveat: **Copilot code review leaves inline "Comment" reviews only — it never Approves or Requests changes** [REF-8][REF-9]. That means it **does not satisfy a required-approval branch protection rule**. Treat it as an extra reviewer that surfaces issues, not as a gate you can rely on for sign-off.

---

## Phase 5 — Documentation

Documentation is a natural fit for Copilot because the source of truth — your code — is right there.

### `/doc` and `///` auto-generation

The **`/doc`** slash command generates a documentation comment for a symbol [REF-3]. In Visual Studio, simply typing **`///`** above a member triggers automatic XML doc-comment generation that you accept with Tab [REF-25]:

```csharp
/// <summary>
/// Calculates the discounted subtotal for a customer based on their tier.
/// </summary>
/// <param name="subtotal">The pre-discount order subtotal.</param>
/// <param name="tier">The customer's loyalty tier.</param>
/// <returns>The subtotal after the tier discount is applied.</returns>
public static decimal CalculateDiscount(decimal subtotal, CustomerTier tier) => /* ... */;
```

You can also drive generation from comments — writing a descriptive comment and letting Copilot fill in the implementation or docs below it [REF-26].

### Generate Docs and broader docs

In VS Code, the **Generate Docs** smart action covers the same need for a selected symbol [REF-43]. For larger artifacts like a README or API docs, use chat to draft them from your code, following Microsoft's guided training on generating documentation with Copilot tools [REF-36].

---

## Phase 6 — Maintenance, refactoring & modernization

Long-lived .NET codebases are where agentic features pay off most, because the work spans many files and iterations.

### Multi-file refactoring with agent mode

**Agent mode** can refactor across files, migrate legacy code, and generate supporting docs — planning the steps and iterating until the build and tests pass [REF-27]. This is where an autonomous mode beats line-by-line edits.

```text
Agent: Extract the inline validation in OrderService into a dedicated
OrderValidator class, update all call sites, and keep the existing tests green.
```

### Debugging assistance

Visual Studio documents **debugging with Copilot** as a separate capability, helping you interpret exceptions and reason about failing code during a debug session [REF-21].

### App modernization and Azure migration

Microsoft ships **GitHub Copilot app modernization** for .NET — a Copilot agent that works across Visual Studio, VS Code, the CLI, and GitHub.com to upgrade .NET projects and migrate applications to Azure [REF-22]. This is purpose-built for the "we're stuck on an old framework" problem.

### Issue-driven fixes with the cloud agent

For routine maintenance tickets, assign the issue to the **cloud agent** on GitHub.com; it works on a branch and opens a PR you then review [REF-7]. This is the same issue-driven flow introduced in the planning phase, applied to bug fixes and small enhancements [REF-5].

---

## Customizing & extending Copilot for your .NET repo

Out of the box, Copilot knows C# in general but nothing about *your* repo. The fix is to commit configuration alongside your code, so every teammate — and every Copilot session — inherits the same conventions, workflows, and tools. Four features cover this, from broadest support to most powerful.

| Feature | What it's for | Where it lives |
|---|---|---|
| **Custom instructions** | Team conventions applied to every request | `.github/copilot-instructions.md`, `.github/instructions/*.instructions.md` [REF-11][REF-46] |
| **Prompt files** | Reusable, invokable workflows for repeat tasks | `.github/prompts/*.prompt.md` [REF-44] |
| **Custom agents** | Scoped personas with fixed tools + instructions | `.github/agents/*.agent.md` [REF-45] |
| **MCP servers** | External tools/context for agent mode | `.vscode/mcp.json` / `mcp.json` [REF-49][REF-50] |

Custom instructions are the most broadly supported (GitHub-wide and across IDEs) [REF-11][REF-23][REF-46]. Prompt files and custom agents are documented as **VS Code** features but travel with the repo, so any teammate on VS Code inherits them [REF-44][REF-45]. MCP is cross-IDE but has the tightest availability rules — covered below.

### Path-specific custom instructions

We introduced `.github/copilot-instructions.md` in the setup section as the repo-wide conventions file [REF-11][REF-23]. Go one level deeper with **path-specific instructions**: `*.instructions.md` files in `.github/instructions/` that carry an `applyTo` glob, so a rule set attaches only to matching files [REF-46]. This lets you keep C# style rules off your YAML and Markdown, and apply EF Core conventions only to your data layer.

```markdown
---
applyTo: "src/Data/**/*.cs"
---
# EF Core conventions (data layer only)

- Configure entities with `IEntityTypeConfiguration<T>`, not data annotations.
- Never call `SaveChanges` inside a repository; the unit of work owns commits.
- Use `AsNoTracking()` for read-only queries.
```

### Prompt files: reusable .NET workflows

**Prompt files** turn a repeatable task into a slash command. Drop a `*.prompt.md` file in `.github/prompts/`, then invoke it in chat with `/name` (or the "Chat: Run Prompt" command) [REF-44]. Prompt files support front matter (`description`, `model`, `tools`), can reference other workspace files with relative links, and take variables like `${selection}` and `${input:name}` [REF-44].

A `/newendpoint` prompt can scaffold a minimal API endpoint and its test the team's way, every time:

```markdown
---
description: Scaffold a minimal API endpoint plus an xUnit test, our way.
---
Create a new minimal API endpoint for ${input:resource}.

- Register it via a `MapGroup` extension in `src/Api/Endpoints/`.
- Validate the request with FluentValidation and return `Results.Problem` on failure.
- Add an xUnit integration test using our `WebApplicationFactory` fixture.

Follow the conventions in ${file:.github/copilot-instructions.md}.
```

Invoke it with `/newendpoint`, supply the `resource` value, and you get consistent scaffolding without re-typing the house rules.

### Custom agents: scoped personas

**Custom agents** are named personas that appear in the chat mode picker alongside Ask, Edit, and Agent [REF-45]. Each is a Markdown file with front matter defining its `description`, `name`, allowed `tools`, and `model`, plus a body of instructions that constrains how it behaves — for example, a "Test Author" agent limited to test-related tools, or a read-only "Architecture Reviewer" [REF-45].

One terminology note: VS Code **renamed "custom chat modes" to "custom agents"**, moving the file format from `.chatmode.md` (in `.github/chatmodes/`) to **`.agent.md`** (in `.github/agents/`) [REF-45]. Legacy `.chatmode.md` files still work, but new agents should use `.agent.md` [REF-45]. Visual Studio documents its own custom-agents experience as well [REF-51].

```markdown
---
description: Reviews C# for our async and nullability rules; suggests only.
name: DotnetReviewer
tools: ['search', 'usages']
---
You are a senior .NET reviewer. Flag blocking `.Result`/`.Wait()` calls,
missing `ConfigureAwait` in library code, and nullable-annotation gaps.
Explain each issue; do not edit files.
```

### MCP servers: connect Copilot to your tools

**Model Context Protocol (MCP)** is an open standard that extends Copilot **agent mode** with external tools and context — live databases, cloud APIs, and services [REF-47][REF-48]. For .NET teams the ecosystem is rich: the **MSSQL/SQL Server MCP** exposes live database schema and queries, the **Azure MCP** reaches Azure resources, the **Azure DevOps MCP** manages work items, and the **GitHub MCP** manages issues and PRs — all usable from agent mode [REF-50].

In VS Code, configure servers in `.vscode/mcp.json`; Visual Studio reads `.mcp.json` (or `mcp.json`) [REF-49][REF-50]:

```json
{
  "servers": {
    "mssql": {
      "command": "npx",
      "args": ["-y", "@azure/mssql-mcp"]
    }
  }
}
```

With that connected, an agent-mode request like "generate an EF Core entity that matches the live `SalesLT.Product` schema" can read the real schema instead of guessing.

**Honest availability caveats** — state these to your team before you rely on MCP [REF-47][REF-48][REF-50]:

- MCP is **GA and cross-IDE**, but in **Visual Studio it requires VS 2022 17.14+ or VS 2026** [REF-50].
- It works in **agent mode only**, and **tools are disabled by default** — each requires per-tool approval [REF-50].
- On **Business and Enterprise** orgs, an admin must enable the **"MCP servers in Copilot"** policy (and can set an allowlist) before anyone can use MCP [REF-48][REF-50].

---

## Best practices, limits & guardrails

Copilot accelerates work; it does not absolve you of responsibility. Microsoft and GitHub are explicit that generated code **may be insecure or inaccurate**, and that human oversight and verification are required [REF-30][REF-31].

**Verify everything.** Responsible-use guidance stresses that suggestions can "hallucinate" and must be reviewed before acceptance — especially for security-sensitive code [REF-30][REF-31]. PR summaries and cloud-agent output carry the same caveat [REF-32][REF-33].

**Control matching public code.** Copilot's **code referencing** filter checks suggestions (roughly 150 characters of surrounding context) against an index of public GitHub code [REF-37]. Teams set the **"Suggestions matching public code"** policy to **Block** or **Allow**; choosing Allow surfaces the matching file's URL and license, and matches occur in fewer than 1% of suggestions [REF-37][REF-40][REF-41].

**Sandbox agentic runs.** Because Copilot testing for .NET and agent mode execute generated commands, require explicit consent and run them in a sandboxed environment [REF-15].

**Steer quality with custom instructions.** The `copilot-instructions.md` file you set up earlier is your main lever for consistent, convention-following output — invest in it as your codebase and team grow [REF-11][REF-23].

---

## Next Steps

Start small and layer on capability:

1. **Turn on the basics.** Enable completions and next edit suggestions for the whole team [REF-17].
2. **Write your `copilot-instructions.md`.** Encode your .NET conventions before scaling up usage [REF-11].
3. **Add one agentic workflow per phase.** Try `/plan` for planning [REF-28], agent mode for a real refactor [REF-27], `@Test` (VS 2026, paid) or Generate Tests for testing [REF-15][REF-39], and Copilot as a PR reviewer for review [REF-9].
4. **Set your matching-public-code policy** before rollout, and brief the team that Copilot code review comments do not count as approvals [REF-40][REF-8].

Adopt Copilot as a lifecycle assistant, keep a human in the loop at every gate, and you will get far more than faster typing.

---

*Sources Consulted:*

- [REF-1] GitHub Copilot features — https://docs.github.com/en/copilot/get-started/features
- [REF-2] Copilot feature matrix — https://docs.github.com/en/copilot/reference/copilot-feature-matrix
- [REF-3] GitHub Copilot Chat cheat sheet (slash commands, participants, context variables) — https://docs.github.com/en/copilot/reference/chat-cheat-sheet
- [REF-5] About GitHub Copilot cloud agent — https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent
- [REF-6] Using Copilot cloud agent on GitHub — https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/use-cloud-agent-on-github
- [REF-7] Asking GitHub Copilot to create a pull request (assign an issue to Copilot) — https://docs.github.com/copilot/how-tos/use-copilot-agents/coding-agent/assign-copilot-to-an-issue
- [REF-8] About GitHub Copilot code review — https://docs.github.com/en/copilot/concepts/agents/code-review
- [REF-9] Using GitHub Copilot code review — https://docs.github.com/copilot/using-github-copilot/code-review/using-copilot-code-review
- [REF-10] Creating a pull request summary with GitHub Copilot — https://docs.github.com/copilot/using-github-copilot/creating-a-pull-request-summary-with-github-copilot
- [REF-11] Adding repository custom instructions for GitHub Copilot — https://docs.github.com/copilot/customizing-copilot/adding-custom-instructions-for-github-copilot
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
- [REF-27] Introducing GitHub Copilot agent mode (VS Code blog) — https://code.visualstudio.com/blogs/2025/02/24/introducing-copilot-agent-mode
- [REF-28] Planning with agents in VS Code (Plan agent, /plan) — https://code.visualstudio.com/docs/agents/planning
- [REF-30] Responsible use of GitHub Copilot features — https://docs.github.com/en/copilot/responsible-use
- [REF-31] Responsible use of GitHub Copilot Chat in your IDE — https://docs.github.com/en/copilot/responsible-use/chat-in-your-ide
- [REF-32] Responsible use of GitHub Copilot pull request summaries — https://docs.github.com/en/copilot/responsible-use-of-github-copilot-features/responsible-use-of-github-copilot-pull-request-summaries
- [REF-33] Responsible use of GitHub Copilot cloud agent on GitHub.com — https://docs.github.com/en/copilot/responsible-use/copilot-coding-agent
- [REF-34] Plans for GitHub Copilot — https://docs.github.com/en/copilot/get-started/plans
- [REF-35] Models and pricing for GitHub Copilot — https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing
- [REF-36] Generate Documentation Using GitHub Copilot Tools (Microsoft Learn Training) — https://learn.microsoft.com/en-us/training/modules/generate-documentation-using-github-copilot-tools/
- [REF-37] GitHub Copilot code referencing (duplication detection; matching-public-code filter; license/URL logging) — https://docs.github.com/en/copilot/concepts/completions/code-referencing
- [REF-38] Quickstart: Code Generation with GitHub Copilot for SQL — Entity Framework model prompts (MSSQL extension for VS Code) — https://learn.microsoft.com/en-us/sql/tools/visual-studio-code-extensions/github-copilot/code-generation?view=sql-server-ver17
- [REF-39] Test with GitHub Copilot (VS Code — Generate Tests smart action, /setupTests, /fixTestFailure, agent-mode auto-fix) — https://code.visualstudio.com/docs/agents/guides/test-with-copilot
- [REF-40] Managing GitHub Copilot policies as an individual subscriber ("Suggestions matching public code" Allow/Block setting) — https://docs.github.com/en/copilot/managing-copilot/managing-copilot-as-an-individual-subscriber/managing-copilot-policies-as-an-individual-subscriber
- [REF-41] Finding public code that matches GitHub Copilot suggestions — https://docs.github.com/copilot/using-github-copilot/finding-public-code-that-matches-github-copilot-suggestions
- [REF-42] Reverse Engineering — EF Core scaffolding (Scaffold-DbContext / dotnet ef dbcontext scaffold) — https://learn.microsoft.com/en-us/ef/core/managing-schemas/scaffolding/
- [REF-43] AI smart actions in Visual Studio Code (Generate Tests / Generate Docs) — https://code.visualstudio.com/docs/editing/copilot-smart-actions
- [REF-44] Use prompt files in VS Code (`.prompt.md` in `.github/prompts`, invoke with `/name`, variables `${selection}`/`${input:...}`) — https://code.visualstudio.com/docs/copilot/customization/prompt-files
- [REF-45] Custom agents in VS Code (formerly "custom chat modes"; `.agent.md` in `.github/agents/`; legacy `.chatmode.md` still supported) — https://code.visualstudio.com/docs/copilot/customization/custom-chat-modes
- [REF-46] Use custom instructions in VS Code (path-specific `*.instructions.md`, `applyTo` globs, `.github/instructions/`) — https://code.visualstudio.com/docs/agent-customization/custom-instructions
- [REF-47] About Model Context Protocol (MCP) — GitHub Copilot (concept; IDE support matrix) — https://docs.github.com/en/copilot/concepts/context/mcp
- [REF-48] Extending GitHub Copilot Chat with Model Context Protocol (MCP) servers (GitHub Docs how-to) — https://docs.github.com/en/copilot/how-tos/provide-context/use-mcp-in-your-ide/extend-copilot-chat-with-mcp
- [REF-49] Add and manage MCP servers in VS Code (`.vscode/mcp.json`, MCP: Add Server) — https://code.visualstudio.com/docs/copilot/customization/mcp-servers
- [REF-50] Use MCP Servers to Extend GitHub Copilot — Visual Studio (VS 2022 17.14+/VS 2026, agent mode, `.mcp.json`, allowlist, Azure/SQL/GitHub MCP examples) — https://learn.microsoft.com/en-us/visualstudio/ide/mcp-servers?view=visualstudio
- [REF-51] Use custom agents in GitHub Copilot — Visual Studio — https://learn.microsoft.com/en-us/visualstudio/ide/copilot-specialized-agents?view=visualstudio
