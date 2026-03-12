---
title: "How to Create an Agentic AI Workflow for .NET Development"
description: "A step-by-step guide to creating a reusable .agent/workflows file that automates the full .NET development cycle — from requirements analysis to tested code — using AI agent skills."
focus_keyphrase: "create agentic AI workflow .NET"
keywords: ["create agentic AI workflow .NET", "workflow md file", "dotnet agent workflow", "AI dev workflow requirements to tests", "Claude Code workflow"]
slug: "create-agentic-ai-workflow-dotnet"
author: ""
date: "2025-03-12"
---

# How to Create an Agentic AI Workflow for .NET Development

Most developers use AI assistants reactively — paste code in, get an answer, repeat. But the real productivity breakthrough comes when AI works *with you proactively*, following a shared playbook for every feature request. That playbook is a **workflow file**.

In this guide, you will create a workflow file from scratch: a structured `.md` document your AI agent reads automatically to execute a full .NET development cycle — analysis, technical design, implementation, **code review**, and unit tests — every time you start a new feature.

By the end, you will have:
- A working `.agent/workflows/dotnet-feature-workflow.md` file
- A clear understanding of each section and why it matters
- A ready-to-run workflow for your next feature

---

## What Is a Workflow File?

A **workflow file** is a Markdown document stored in `.agent/workflows/` that gives your AI agent a repeatable, step-by-step playbook. Unlike a single prompt, a workflow:

- **Sequences multiple phases** — each building on the previous output
- **Integrates AI Skills** — calling specialised skills (`/dotnet-code-review`, `/dotnet-unit-testing`) at the right moment
- **Includes human checkpoints** — so you review and guide at key decision points
- **Persists in your repository** — every team member gets the same playbook automatically

The format used in this project is a Markdown file with a YAML frontmatter block and numbered phases:

```
.agent/
└── workflows/
    └── dotnet-feature-workflow.md  ← what you will build today
```

---

## Understanding the `.agent/` Project Structure

Before writing a single line, let's look at the complete directory structure your repository uses. Everything lives inside `.agent/` at the repository root:

```
.agent/
├── workflows/
│   └── dotnet-feature-workflow.md   ← orchestrates all phases end-to-end
└── skills/
    ├── dotnet-code-review/          ← reviews C# code against 18 standard rules
    │   ├── SKILL.md                 ← skill entry point: rules summary + report format
    │   └── RULES.md                 ← reference: full rule definitions with C# examples
    └── dotnet-unit-testing/         ← generates xUnit tests with NSubstitute
        ├── SKILL.md                 ← skill entry point: test template + naming conventions
        └── PATTERNS.md              ← reference: advanced mocking patterns + anti-patterns
```

### How a skill is invoked

When a workflow (or you directly) types `/dotnet-code-review`, the AI agent:
1. Looks up `.agent/skills/dotnet-code-review/`
2. Reads `SKILL.md` — the main instructions enter the context window
3. Reads additional files (`RULES.md`, `PATTERNS.md`) **only when referenced** — zero token cost otherwise

This is the **progressive disclosure** model: SKILL.md is a lightweight entry point, and heavier reference files are paged in on demand.

### The two skills used in this workflow

#### `/dotnet-code-review`

```
dotnet-code-review/
├── SKILL.md      ← Level 1+2: 18-rule summary, report format, verdict system
└── RULES.md      ← Level 3: full rule definitions with before/after C# examples
```

The skill checks code against **18 rules** in five categories and outputs a structured report:

```
## Code Review Report
**File:** OrderService.cs
**Verdict:** ✅ PASS | ⚠️ PASS WITH NOTES | ❌ FAIL

### 🔴 Blockers  (must fix before merge)
- [S2] Constructor takes SqlOrderRepository (concrete) — use IOrderRepository instead — Line 12

### 🟡 Majors  (should fix before merge)
### 🔵 Minors  (suggested improvements)
### ✅ Passed rules
```

#### `/dotnet-unit-testing`

```
dotnet-unit-testing/
├── SKILL.md      ← Level 1+2: framework detection, canonical test template, naming rules
└── PATTERNS.md   ← Level 3: NSubstitute/Moq patterns, edge-case table, anti-patterns
```

The skill auto-detects the test framework and mocking library already in your project and generates a complete xUnit test class following `MethodName_Scenario_ExpectedOutcome` naming.

---

## Step 1: Create the File and Add Frontmatter

Create a new file at `.agent/workflows/dotnet-feature-workflow.md`.

Every workflow file starts with a YAML frontmatter block containing exactly one required field: `description`. This is what the AI agent reads to decide *when* to trigger this workflow automatically.

```markdown
---
description: .NET Feature Development Workflow — analyze requirements, produce technical notes, implement a service class, and write xUnit unit tests.
---
```

**Writing an effective description:**
- State the workflow's trigger context ("when given a feature request or user story")
- List the phases in order so the AI knows the complete scope
- Be specific — a vague description causes the wrong workflow to trigger

---

## Step 2: Add the Overview Section

After the frontmatter, add a human-readable overview. This explains the workflow's purpose to teammates reading the file directly.

```markdown
# .NET Feature Development Workflow

## Overview
This workflow guides an AI agent through a complete .NET feature implementation cycle. It transforms a raw requirement or user story into production-ready code with unit tests, using structured phases and human review checkpoints throughout.

**Skills used:** `dotnet-code-review`, `dotnet-unit-testing`
**Output:** C# service class + Code Review Report + xUnit test suite
**Target audience:** Intermediate to senior .NET developers
```

---

## Step 3: Add Phase 1 — Analyze Requirements

The first phase is the most important. A structured requirements analysis locks in shared understanding before any code is written. Errors caught here cost minutes to fix; errors found in Phase 4 cost hours.

Add the following to your file:

```markdown
---

## Phase 1: Analyze Requirements
Parse the feature request and produce a structured breakdown that all subsequent phases will use as their source of truth.

* **1. Define Core Elements:**
  * **Domain Entities:** Identify all nouns that represent data (records, classes, DTOs).
  * **Business Rules:** Restate each rule as an IF … THEN … statement.
  * **Input Constraints:** List every validation that must be enforced in code.
  * **Edge Cases:** List all boundary conditions and special scenarios.
  * **Open Questions:** Flag anything the requirement does not clarify.

* **2. Execute Requirements Capture:**
  Paste the raw requirement text and run the following prompt.
  > **🤖 AI Workflow Prompt:**
  > ```plaintext
  > You are a senior .NET requirements analyst.
  > Produce a structured analysis with these sections:
  > Domain Entities | Business Rules | Input Constraints | Edge Cases | Open Questions.
  > Requirement: [paste requirement here]
  > ```

* **3. Human Checkpoint ✅**
  Review the structured analysis. Resolve all Open Questions with the product manager before proceeding to Phase 2.
```

**Why this structure works:** By forcing the AI to express rules as IF/THEN statements, you get machine-readable logic that the implementation phase can verify against directly.

---

## Step 4: Add Phase 2 — Technical Notes

The Architect phase converts the analysis into concrete C# contracts — interfaces, DTOs, and explicit architecture decisions. This is the "design before build" gate that prevents costly refactors.

```markdown
---

## Phase 2: Technical Notes
Translate the Phase 1 analysis into C# interface contracts and architecture decisions. No implementation code is written in this phase.

* **1. Design the Service Layer:**
  > **🤖 AI Workflow Prompt:**
  > ```plaintext
  > You are a senior .NET architect.
  > Using the Phase 1 analysis below, produce:
  > 1. C# interfaces with XML doc comments for all public members
  > 2. C# records or classes for all DTOs
  > 3. Architecture decisions (validation strategy, exception handling, async usage)
  > Do NOT write implementation code yet.
  >
  > [Paste Phase 1 output here]
  > ```

* **2. Human Checkpoint ✅**
  Review the interfaces and architecture decisions.
  Confirm: Does the interface contract cover all Phase 1 business rules?
  Adjust naming or method signatures before proceeding.
```

**Key rule:** Nothing in Phase 2 should surprise you when you see the implementation in Phase 3. If the interface contracts look wrong, fix them now.

---

## Step 5: Add Phase 3 — Implement Service

The Developer phase takes the contracts from Phase 2 and produces a complete, compilable service class. The AI has no ambiguity — interfaces, DTOs, and architecture decisions are all provided explicitly.

```markdown
---

## Phase 3: Implement Service
Write the full C# service implementation using the contracts and decisions from Phase 2.

* **1. Generate the Service Class:**
  > **🤖 AI Workflow Prompt:**
  > ```plaintext
  > You are a senior .NET developer (.NET 9).
  > Implement the service class using the interfaces and DTOs below.
  > Apply all business rules and architecture decisions from the analysis.
  > Include XML doc comments on the class and all public methods.
  > Do NOT write unit tests yet.
  >
  > [Paste Phase 2 output here]
  > ```

* **2. Human Checkpoint ✅**
  Compile the generated code. Verify each branch of the implementation
  traces back to a specific business rule from Phase 1.
  Common issues to check:
  * [ ] Does validation throw the right exception type?
  * [ ] Are all async calls awaited?
  * [ ] Are edge cases (null, zero, boundary values) handled?
```

---

## Step 6: Add Phase 4 — Code Review

Before generating unit tests, the code must pass a structured review. The `dotnet-code-review` skill checks the implementation against **18 standard rules** across five categories and produces a Blocker / Major / Minor report with a clear PASS / FAIL verdict. Unit tests written against buggy code only entrench the bugs — the review gate prevents that.

```markdown
---

## Phase 4: Code Review
Run the `dotnet-code-review` skill against the Phase 3 implementation.
No unit tests are written until the implementation achieves at least PASS WITH NOTES.

* **1. Run the Code Review Skill:**
  > **🤖 AI Workflow (dotnet-code-review skill):**
  > ```plaintext
  > /dotnet-code-review
  > Review the C# service class below against all standard rules.
  > Produce a structured Blocker / Major / Minor report and a PASS verdict.
  >
  > [Paste Phase 3 service class here]
  > ```

* **2. Review categories checked automatically:**
  | Category | Example rules |
  |---|---|
  | Naming & Style | PascalCase, `_camelCase` fields, `IPrefix`, `Async` suffix |
  | SOLID Principles | Single responsibility, interfaces over concrete types |
  | Async/Await | No `async void`, no `.Result`/`.Wait()`, `CancellationToken` |
  | Exception Handling | No silent catches, typed exceptions, no empty catch blocks |
  | Security & Performance | No hardcoded secrets, parameterized queries, no sync I/O |

* **3. Human Checkpoint ✅**
  * `❌ FAIL` → fix all Blockers, re-run skill.
  * `⚠️ PASS WITH NOTES` → fix Majors or document accepted exceptions.
  * `✅ PASS` → proceed to Phase 5.
```

**Why code review before tests:** Tests written against a class with a dependency-inversion violation or an `async void` bug will pass green and still ship a defect. The review gate ensures the implementation is structurally sound first.

---

## Step 7a: Add Phase 5a — Generate Test Cases

Before writing a single line of xUnit code, ask the AI to produce a **test case matrix** — a plain-language table listing every scenario that must be tested. This separates *what* to test from *how* to implement it.

```markdown
---

## Phase 5a: Generate Test Cases
Before writing any xUnit code, produce a test case matrix from the Phase 1 analysis.

* **1. Generate the Test Case Matrix:**
  > **🤖 AI Workflow Prompt:**
  > ```plaintext
  > You are a senior .NET QA engineer.
  > From the Phase 1 analysis and Phase 3 implementation below, produce a
  > test case matrix with these columns:
  > | # | Method | Scenario | Input | Expected Output | Type |
  > Type = Happy Path | Edge Case | Validation | Exception
  > Do NOT write any C# code yet.
  >
  > [Paste Phase 1 analysis + Phase 3 implementation here]
  > ```
```

**Why generate before implementing:** The matrix makes coverage gaps visible in seconds. Spotting a missing edge case in a table row costs nothing to fix. Spotting it after the test suite is written costs a rewrite.

---

## Step 7b: Add Phase 5b — Verify Test Cases

This is the developer’s review gate. Read every row in the matrix against the Phase 1 analysis. This is the cheapest point in the entire workflow to fix missing coverage.

```markdown
---

## Phase 5b: Verify Test Cases
Review the test case matrix against Phase 1 before any C# code is generated.

* **1. Human Checkpoint ✅**
  * [ ] Every Business Rule has at least one Happy Path row
  * [ ] Every Edge Case has a corresponding row
  * [ ] Every Input Constraint has a Validation or Exception row
  * [ ] No duplicate scenarios

* **2. Fill gaps:**
  > ```plaintext
  > Add a row: method=X, scenario=[desc], input=Y, expected=Z, type=Edge Case.
  > ```

* **3. Approve matrix ✅ — this becomes the acceptance criteria for the test suite.**
```

**Key principle:** The approved matrix row count is your target. `dotnet test` must ultimately report the same number of passing tests as there are approved rows.

---

## Step 7c: Add Phase 5c — Implement & Verify Tests

With an approved matrix, generate xUnit code — one test per row — then run the suite.

```markdown
---

## Phase 5c: Implement & Verify Tests
Generate xUnit test code from the approved matrix, then run and measure.

* **1. Generate xUnit Tests:**
  > **🤖 AI Workflow (dotnet-unit-testing skill):**
  > ```plaintext
  > /dotnet-unit-testing
  > Write xUnit tests for every row in the approved test case matrix below.
  > - Use NSubstitute for all dependency mocks
  > - Naming: MethodName_Scenario_ExpectedOutcome
  > - Use [Theory]+[InlineData] for rows with the same assertion logic
  > - Apply Arrange/Act/Assert with section comments
  > - One test method per matrix row
  >
  > Test Case Matrix: [Paste Phase 5b approved matrix]
  > Implementation:   [Paste Phase 3 service class]
  > ```

* **2. Run and Measure:**
  > ```bash
  >   dotnet test
  >   dotnet-coverage collect "dotnet test" --output coverage.xml --output-format cobertura
  >   reportgenerator -reports:coverage.xml -targetdir:coverage-report -reporttypes:Html
  > ```

* **3. Final Human Checkpoint ✅**
  * Test count equals the number of approved rows in the Phase 5b matrix
  * Remove zombie tests — tests that only assert mock-configured return values without verifying real behaviour
  * Branch coverage ≥ 80% (check `coverage-report/index.html`)

```
---



## Step 8: Add the Publishing Checklist

Close the workflow with a pre-commit checklist. This ensures completeness before the PR is opened.

```markdown
---

## Pre-Commit Checklist
Before opening a pull request, verify:

* [ ] All Phase 1 business rules have a corresponding unit test
* [ ] `dotnet build` passes with zero warnings
* [ ] `dotnet test` passes — all tests green
* [ ] Code coverage is ≥ 80% branch coverage
* [ ] Code review verdict is `✅ PASS` or `⚠️ PASS WITH NOTES` with documented exceptions
* [ ] XML doc comments exist on all public interfaces and service class
* [ ] No zombie tests committed (tests that only assert mock return values)
```

---

## The Complete File

After completing all steps, your workflow file looks like this:

```
.agent/workflows/dotnet-feature-workflow.md
```

```markdown
---
description: .NET Feature Development Workflow — analyze requirements, produce
  technical notes, implement a service class, run a structured code review,
  and write xUnit unit tests.
---

# .NET Feature Development Workflow

## Overview
[overview text]

---

## Phase 1: Analyze Requirements
[phase 1 content]

---

## Phase 2: Technical Notes
[phase 2 content]

---

## Phase 3: Implement Service
[phase 3 content]

---

## Phase 4: Code Review
[phase 4 content — dotnet-code-review skill]

---

## Phase 5a: Generate Test Cases
[phase 5a content — AI test case matrix]

---

## Phase 5b: Verify Test Cases
[phase 5b content — developer review & approval]

---

## Phase 5c: Implement & Verify Tests
[phase 5c content — dotnet-unit-testing skill + dotnet test]

---

## Pre-Commit Checklist
[checklist]
```

Every phase in this workflow follows the same simple format: a heading, a description, numbered steps, an AI callout block, and a human checkpoint. This consistency means any developer on your team can read, run, or extend any phase without needing extra context.

### Workflow file format at a glance

```markdown
## Phase N: [Name]
[One-sentence description of what this phase achieves]

* **1. [Action]**
  > **🤖 AI Workflow Prompt:**
  > [prompt here]

* **2. Human Checkpoint ✅**
  [review criteria]
```

---

## Why Workflow Files Beat One-Shot Prompts

| One-Shot Prompt | Workflow File |
|---|---|
| Result varies by how you phrase the request | Reproducible output every time |
| Context is lost between sessions | Phases pass context forward explicitly |
| One developer benefits | Entire team uses the same process |
| No human review gates | Mandatory checkpoints prevent compounding errors |
| Stored in chat history, easily lost | Versioned in git alongside your code |

---

## Next Steps

1. **Create the file now** — copy the complete sections from this post into `.agent/workflows/dotnet-feature-workflow.md` in your repository.
2. **Run your first feature** — pick a small, well-scoped user story and run Phase 1. Share the structured output with your team as a proof of concept.
3. **Extend with more skills** — as you build more Agent Skills (like `dotnet-unit-testing`), add references to them in the workflow's AI callout blocks.

---

*Sources Consulted:*
- [Antigravity Workflow Documentation](https://antigravity.google/docs/rules-workflows)
- [Claude Agent Skills Overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [xUnit.net Documentation](https://xunit.net/)
- [NSubstitute Documentation](https://nsubstitute.github.io/)
