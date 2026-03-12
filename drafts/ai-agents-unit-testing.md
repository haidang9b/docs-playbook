---
title: "AI Agent Skill for .NET Unit Testing: Build It Step by Step"
description: "Learn how to create a custom Claude AI Agent Skill for .NET unit testing. Step-by-step guide to writing SKILL.md files that generate xUnit and NSubstitute tests automatically."
focus_keyphrase: "AI agent skill .NET unit testing"
keywords: ["AI agent skill .NET unit testing", "Claude Agent Skills", "SKILL.md tutorial", "xUnit test generation", "NSubstitute mocking"]
author: ""
date: "2025-03-12"
---

# Build Your Own AI Agent Skill for .NET Unit Testing

Every team eventually has the same conversation: *"We all know we should write more unit tests, but it takes so long."* What if your AI assistant already knew your team's framework preferences, naming conventions, mocking library, and coverage targets—and would apply them automatically every time you asked it to write a test? That is exactly what a **Claude Agent Skill** enables.

In this post you will learn:

1. What Agent Skills are and how they work
2. The directory structure and file format required
3. How to write an effective `SKILL.md` that Claude discovers automatically
4. A complete, working `dotnet-unit-testing` skill—built step by step

---

## What Is a Claude Agent Skill?

An **Agent Skill** is a reusable, filesystem-based package that gives Claude domain-specific expertise. Think of it as an onboarding guide you write once and your AI colleague reads automatically whenever the task matches.

According to the [official Claude documentation](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview):

> *Skills are reusable, filesystem-based resources that provide Claude with domain-specific expertise: workflows, context, and best practices that transform general-purpose agents into specialists.*

Unlike a one-off prompt you paste into a chat window, a Skill:

- **Loads on demand** – Claude only reads it when your request triggers a match.
- **Persists across conversations** – install once, available everywhere.
- **Composes** – combine multiple Skills to handle complex, multi-step workflows.

---

## How the 3-Level Loading Architecture Works

The key insight behind Agent Skills is **progressive disclosure**: Claude loads only what is needed for each task, keeping the context window lean.

### Level 1 – Metadata (always loaded at startup)

The YAML frontmatter at the top of `SKILL.md` is the only thing Claude loads continuously. It is injected into the system prompt so Claude knows the Skill exists and when to trigger it—without consuming tokens for the full instructions.

```yaml
---
name: dotnet-unit-testing
description: Generates xUnit/NUnit/MSTest unit tests for .NET C# projects. Use when
  the user asks to write unit tests, generate test cases, or improve test coverage.
---
```

The `name` field must be lowercase letters, numbers, and hyphens only (max 64 chars). The `description` field is critical—write it in third person, include both *what* the Skill does and *when* to trigger it.

### Level 2 – Instructions (loaded when triggered)

When your request matches the Skill's description, Claude reads the body of `SKILL.md` via a bash `cat` command. This is where you put your step-by-step workflow, templates, and conventions—the procedural knowledge Claude will follow.

```markdown
# .NET Unit Testing

## Quick start
1. Detect the framework (xUnit / NUnit / MSTest)
2. Detect the mocking library (NSubstitute / Moq / FakeItEasy)
3. Analyze the target class signatures
4. Generate the test class using the canonical template below
...
```

Keep this body under 500 lines for optimal performance.

### Level 3 – Resources and scripts (loaded as needed)

Additional markdown files, reference docs, or executable scripts sit alongside `SKILL.md` in the Skill directory. Claude reads them only when the Level 2 instructions reference them—zero token cost otherwise.

```
dotnet-unit-testing/
├── SKILL.md          ← always loaded when triggered
├── PATTERNS.md       ← loaded only when advanced patterns are needed
└── scripts/
    └── validate.sh   ← executed when Claude needs to run checks
```

Here is how it flows end-to-end:

```
User: "Write unit tests for my OrderService class"
         │
         ▼
Claude reads SKILL.md  ←── Level 2: instructions enter context
         │
         ▼
Follows Quick Start → detects xUnit + NSubstitute
         │
         ▼
Generates test class using canonical template
         │
User: "Are there any anti-patterns in these tests?"
         │
         ▼
Claude reads PATTERNS.md  ←── Level 3: loaded only now
```

---

## Step 1 – Create the Skill Directory

Agent Skills live inside a designated skills folder in your repository. In this project the path is:

```
.agent/skills/dotnet-unit-testing/
```

Create the directory:

```bash
mkdir -p .agent/skills/dotnet-unit-testing
```

---

## Step 2 – Write `SKILL.md`

The `SKILL.md` file is the core of every Skill. It has two parts: the YAML frontmatter (Level 1 metadata) and the markdown body (Level 2 instructions).

### 2a. Write the frontmatter

Follow the official field rules:

| Field | Rule |
|---|---|
| `name` | Lowercase, hyphens only, max 64 chars, no reserved words |
| `description` | Third person, max 1024 chars, include *what* + *when* |

```yaml
---
name: dotnet-unit-testing
description: Generates, analyzes, and improves xUnit/NUnit/MSTest unit tests for
  .NET C# projects. Use when the user asks to write unit tests, generate test cases,
  improve test coverage, mock dependencies, or review existing tests in a .NET or
  C# codebase.
---
```

### 2b. Write the instructions body

Structure the body as a step-by-step workflow Claude will execute. Always include:

- A **Quick start** ordered list that describes the decision sequence
- A **canonical template** Claude can adapt for any class
- A **naming convention** section (this is where most teams diverge)
- A reference to any Level 3 files for advanced content

Here is the full body for the `dotnet-unit-testing` Skill:

```markdown
# .NET Unit Testing

## Quick start

1. **Detect the framework** – Check `<PackageReference>` entries in `.csproj`.
   Prefer the framework already in use. Fallback: xUnit → NUnit → MSTest.
2. **Detect the mocking library** – Look for NSubstitute, Moq, FakeItEasy.
   Default to NSubstitute if nothing is configured.
3. **Analyze the target class** – Read constructor dependencies, public method
   signatures, XML doc comments, and nullable annotations.
4. **Generate the test class** – Follow the template below.
5. **Cover all branches** – Every `if`, `switch`, `throw`, `return` needs a test.
6. **Review for zombie tests** – See [PATTERNS.md](PATTERNS.md).

## Test class template

Use this structure for every new xUnit test class:

​```csharp
public class <ClassName>Tests
{
    private readonly I<Dependency> _dep = Substitute.For<I<Dependency>>();
    private readonly <ClassName> _sut;   // sut = System Under Test

    public <ClassName>Tests() => _sut = new <ClassName>(_dep);

    [Fact]
    public async Task <MethodName>_<Scenario>_<ExpectedOutcome>()
    {
        // Arrange
        _dep.<Method>(<args>).Returns(<value>);

        // Act
        var result = await _sut.<MethodName>(<args>);

        // Assert
        Assert.Equal(<expected>, result.<Property>);
    }
}
​```

## Naming convention

Always use `MethodName_Scenario_ExpectedOutcome`.

## Coverage targets

Every generated suite must cover:
- Happy path – valid inputs, expected return value
- Boundary conditions – zero, negative, null, empty
- Exception paths – every `throw` in the method body
- Async correctness – every `async Task` tested with `await`

## Advanced patterns

For mocking patterns, edge-case categories, and anti-patterns,
see [PATTERNS.md](PATTERNS.md).
```

---

## Step 3 – Add a `PATTERNS.md` for Progressive Disclosure

Level 3 files are loaded only when needed, keeping your core `SKILL.md` focused. Create `PATTERNS.md` alongside `SKILL.md`:

```
.agent/skills/dotnet-unit-testing/
├── SKILL.md      ← core instructions
└── PATTERNS.md   ← advanced reference, loaded on demand
```

`PATTERNS.md` should contain content that is often-useful but not always needed—in this case, mocking patterns and anti-patterns:

````markdown
# .NET Unit Testing – Patterns Reference

## Mocking with NSubstitute

```csharp
// Return a value
_repo.GetProductPriceAsync(1).Returns(99.99m);

// Verify a call was made exactly once
await _repo.Received(1).CreateOrderAsync(Arg.Any<int>(), Arg.Any<int>(), Arg.Any<decimal>());

// Throw an exception from a dependency
_repo.GetProductPriceAsync(Arg.Any<int>()).Throws(new TimeoutException());
```

## Anti-pattern: Zombie tests

A zombie test passes but asserts nothing meaningful:

```csharp
// BAD – tests NSubstitute, not your code
[Fact]
public async Task GetPrice_ReturnsPrice()
{
    _repo.GetProductPriceAsync(1).Returns(100m);
    var result = await _sut.GetProductPriceAsync(1);
    Assert.Equal(100m, result); // trivially true
}
```

Fix: assert on computed business behavior, not raw mock return values.
````

---

## Step 4 – Verify the Final Directory Structure

After creating both files, your Skill directory should look like this:

```
.agent/skills/dotnet-unit-testing/
├── SKILL.md      ← frontmatter + step-by-step workflow
└── PATTERNS.md   ← advanced patterns (Level 3, loaded on demand)
```

This matches the canonical structure from the Claude Agent Skills documentation.

---

## Step 5 – Use the Skill in Claude Code

With the Skill installed, open Claude Code and ask:

```
Write unit tests for my OrderService class.
```

Claude will automatically:
1. Detect the `dotnet-unit-testing` Skill is relevant.
2. Read `SKILL.md` to load the instructions.
3. Inspect your project to detect xUnit and NSubstitute.
4. Generate a test class following your naming conventions and AAA pattern.

If you follow up with:

```
Are there any anti-patterns in the tests you just wrote?
```

Claude reads `PATTERNS.md`—and only then—to review the suite against your defined anti-patterns.

---

## Design Principles Behind This Skill

This Skill was authored following the best practices from the [official Claude docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices):

| Principle | How it is applied |
|---|---|
| **Concise is key** | SKILL.md body stays under 200 lines |
| **Third-person description** | "Generates..." not "I can help you..." |
| **What + When in description** | States capability and trigger conditions |
| **Progressive disclosure** | Anti-patterns deferred to PATTERNS.md |
| **Decision-first Quick start** | Ordered list guides Claude's sequence |
| **Canonical template** | Removes ambiguity with a concrete code scaffold |

---

## What to Do Next

You now have a fully structured Agent Skill and understand how the 3-level architecture keeps your AI assistant efficient. Here are three concrete next steps:

1. **Install the Skill** – The `.agent/skills/` directory in this repository is already discovered automatically by Claude Code. No additional configuration is needed.
2. **Test it** – Pick the most under-tested class in your codebase, open Claude Code, and ask it to write unit tests. Verify the output matches your naming convention and framework.
3. **Extend it** – Add more Level 3 reference files (e.g., `INTEGRATION-TESTS.md`, `SNAPSHOTS.md`) as your team's needs grow. Each file is loaded only when referenced, so there is no performance penalty.

---

*Sources Consulted:*
- [Claude Agent Skills Overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [Skill Authoring Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [xUnit.net Documentation](https://xunit.net/)
- [NSubstitute Documentation](https://nsubstitute.github.io/)
