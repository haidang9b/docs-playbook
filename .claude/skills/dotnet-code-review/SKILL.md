---
name: dotnet-code-review
description: Reviews .NET C# code against standard quality rules covering naming conventions, SOLID principles, async/await correctness, exception handling, security, and performance. Use when asked to review, audit, or check C# code quality before a pull request or merge.
---

# .NET Code Review

Reviews C# code against a standard ruleset. Produces a structured report with categorised findings — Blocker, Major, Minor — and an overall pass/fail verdict.

## Quick start

When asked to review code, run through ALL five rule categories below in order. Do not skip categories even if the code looks clean — each category catches a different class of problem.

1. **Naming & Style** — enforce conventions
2. **SOLID & Design** — check architectural correctness
3. **Async/Await** — catch async bugs
4. **Exception Handling** — verify safety patterns
5. **Security & Performance** — flag risks

Output one structured report. See [RULES.md](RULES.md) for the full rule definitions and examples.

## Report format

Always produce the review in this exact format:

```
## Code Review Report
**File:** <ClassName.cs>
**Verdict:** ✅ PASS | ⚠️ PASS WITH NOTES | ❌ FAIL

### 🔴 Blockers  (must fix before merge)
- [RULE-ID] <description of violation> — Line <N>

### 🟡 Majors  (should fix before merge)
- [RULE-ID] <description of violation> — Line <N>

### 🔵 Minors  (suggested improvements)
- [RULE-ID] <description> — Line <N>

### ✅ Passed rules
- List all rule categories with zero violations
```

**Verdict logic:**
- `❌ FAIL` — any Blocker finding
- `⚠️ PASS WITH NOTES` — no Blockers, one or more Majors
- `✅ PASS` — no Blockers or Majors (Minors allowed)

## Rule categories (summary)

Full definitions and code examples are in [RULES.md](RULES.md).

| ID | Category | Severity |
|---|---|---|
| N1 | Class, interface, method names use PascalCase | Minor |
| N2 | Private fields use `_camelCase` prefix | Minor |
| N3 | Interfaces start with `I` | Major |
| N4 | Async methods end with `Async` | Major |
| S1 | Single Responsibility: one class, one reason to change | Major |
| S2 | Depend on abstractions (interfaces), not concrete classes | Blocker |
| S3 | Methods ≤ 30 lines; extract if larger | Minor |
| A1 | Never use `async void` (except event handlers) | Blocker |
| A2 | Never `Task.Result` or `.Wait()` — use `await` | Blocker |
| A3 | Use `CancellationToken` on all I/O-bound async methods | Major |
| A4 | `ConfigureAwait(false)` in library code | Minor |
| E1 | Never catch `Exception` without rethrowing or logging | Blocker |
| E2 | Throw `ArgumentException`/`ArgumentNullException` for invalid inputs | Major |
| E3 | No empty catch blocks | Blocker |
| E4 | Use `ArgumentNullException.ThrowIfNull()` (.NET 6+) | Minor |
| P1 | No `string` concatenation in loops — use `StringBuilder` | Major |
| P2 | Prefer `IEnumerable<T>` over `List<T>` in public APIs | Minor |
| P3 | No synchronous file/DB calls in async methods | Blocker |
| SEC1 | No hardcoded secrets, connection strings, or API keys | Blocker |
| SEC2 | Validate and sanitize all external inputs | Major |
| SEC3 | Use parameterized queries — no raw SQL string concatenation | Blocker |
