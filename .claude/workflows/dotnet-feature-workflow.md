---
description: .NET Feature Development Workflow — analyze a feature requirement, produce technical notes and C# interface contracts, implement a service class, run a structured code review, and write a full xUnit unit test suite. Use when given a feature request, user story, or requirement to implement in a .NET C# project.
---

# .NET Feature Development Workflow

## Overview
This workflow guides an AI agent through a complete .NET feature implementation cycle. It transforms a raw requirement or user story into production-ready C# code with a full unit test suite, using structured phases and mandatory human review checkpoints.

**Skills used:** `dotnet-code-review`, `dotnet-unit-testing`
**Output:** C# service class + Code Review Report + xUnit/NSubstitute test suite
**Target audience:** Intermediate to senior .NET developers

---

## Phase 1: Analyze Requirements
Parse the feature request and produce a structured breakdown that all subsequent phases will use as their shared source of truth.

* **1. Define Core Elements:**
  * **Domain Entities:** Identify all nouns that represent data (records, classes, DTOs).
  * **Business Rules:** Restate each rule as an IF … THEN … statement.
  * **Input Constraints:** List every validation that must be enforced in code.
  * **Edge Cases:** List all boundary conditions and special scenarios.
  * **Open Questions:** Flag anything the requirement does not clarify.

* **2. Execute Requirements Capture:**
  > **🤖 AI Workflow Prompt:**
  > ```plaintext
  > You are a senior .NET requirements analyst.
  > Produce a structured analysis with five sections:
  > Domain Entities | Business Rules | Input Constraints | Edge Cases | Open Questions.
  > Requirement: [paste requirement or user story here]
  > ```

* **3. Human Checkpoint ✅**
  Review the structured analysis. Resolve all Open Questions with the product owner before proceeding to Phase 2.

---

## Phase 2: Technical Notes
Translate the Phase 1 analysis into C# interface contracts and architecture decisions. No implementation code is produced in this phase.

* **1. Design the Service Layer:**
  > **🤖 AI Workflow Prompt:**
  > ```plaintext
  > You are a senior .NET architect (.NET 9).
  > Using the Phase 1 analysis below, produce:
  > 1. C# interfaces with XML doc comments for all public members
  > 2. C# records or classes for all DTOs
  > 3. Architecture decisions: where validation lives, exception strategy, async usage
  > Do NOT write implementation code yet.
  >
  > [Paste Phase 1 output here]
  > ```

* **2. Human Checkpoint ✅**
  Review interface contracts and architecture decisions.
  * Does each interface method correspond to a business rule?
  * Is the exception strategy (ArgumentException, custom exceptions) consistent?
  * Adjust naming or signatures before proceeding.

---

## Phase 3: Implement Service
Write the full, compilable C# service class using the contracts and decisions from Phase 2.

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
  Compile and review:
  * [ ] `dotnet build` succeeds with zero warnings
  * [ ] Each `if`/`switch` branch traces to a Phase 1 business rule
  * [ ] Null, zero, and boundary values handled per Phase 1 edge cases
  * [ ] All async dependencies are properly awaited

---

## Phase 4: Code Review
Run the `dotnet-code-review` skill against the Phase 3 implementation. The AI acts as a senior reviewer, checking the code against 18 standard rules across five categories. No test code is written until the implementation passes review.

* **1. Run the Code Review Skill:**
  > **🤖 AI Workflow (dotnet-code-review skill):**
  > ```plaintext
  > /dotnet-code-review
  > Review the following C# service class against all standard rules.
  > Produce a structured report with Blocker, Major, and Minor findings.
  > Provide a PASS / PASS WITH NOTES / FAIL verdict.
  >
  > [Paste Phase 3 service class here]
  > ```

* **2. Review categories checked:**
  | Category | Rules checked |
  |---|---|
  | **Naming & Style** | PascalCase, `_camelCase` fields, `I` prefix, `Async` suffix |
  | **SOLID Principles** | Single responsibility, depend on abstractions, method length |
  | **Async/Await** | No `async void`, no `.Result`/`.Wait()`, `CancellationToken` usage |
  | **Exception Handling** | No silent catches, typed exceptions, no empty catch blocks |
  | **Security & Performance** | No hardcoded secrets, parameterized queries, no sync I/O in async |

* **3. Human Checkpoint ✅**
  * If verdict is `❌ FAIL` → fix all Blockers, then re-run the skill.
  * If verdict is `⚠️ PASS WITH NOTES` → fix Majors or document accepted exceptions.
  * If verdict is `✅ PASS` → proceed to Phase 5.

---

## Phase 5a: Generate Test Cases
Before writing any xUnit code, produce a **test case matrix** — a plain-language list of every scenario that must be covered. This gives the developer a human-readable checklist to verify against Phase 1.

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

* **2. Example output:**
  | # | Method | Scenario | Input | Expected Output | Type |
  |---|---|---|---|---|---|
  | 1 | `PlaceOrderAsync` | No coupon | qty=2, code=null | basePrice returned | Happy Path |
  | 2 | `PlaceOrderAsync` | SAVE10 coupon | qty=1, price=$30 | $20 returned | Happy Path |
  | 3 | `PlaceOrderAsync` | Discount below floor | price=$6, SAVE10 | $5 floor returned | Edge Case |
  | 4 | `PlaceOrderAsync` | Zero quantity | qty=0 | `ArgumentException` | Validation |
  | 5 | `PlaceOrderAsync` | Null coupon code | code=null | no exception thrown | Edge Case |

---

## Phase 5b: Verify Test Cases
The developer reviews the matrix against Phase 1 before any test code is written. This is the cheapest possible point to catch missing coverage.

* **1. Human Checkpoint ✅**
  For each row, confirm:
  * [ ] Every Phase 1 **Business Rule** has at least one Happy Path row
  * [ ] Every Phase 1 **Edge Case** has a corresponding row
  * [ ] Every Phase 1 **Input Constraint** has a Validation or Exception row
  * [ ] No duplicate scenarios (same inputs, same expected result)

* **2. Fill gaps:**
  If rows are missing, add them manually or prompt the AI:
  > ```plaintext
  > Add a row for: method=X, scenario=[description], input=Y, expected=Z, type=Edge Case.
  > ```

* **3. Approve matrix ✅ — this becomes the acceptance criteria for the test suite.**

---

## Phase 5c: Implement & Verify Tests
With an approved matrix, generate xUnit code — one test method per matrix row.

* **1. Generate xUnit Tests:**
  > **🤖 AI Workflow (dotnet-unit-testing skill):**
  > ```plaintext
  > /dotnet-unit-testing
  > Write xUnit tests for every row in the approved test case matrix below.
  > - Use NSubstitute for all dependency mocks
  > - Naming: MethodName_Scenario_ExpectedOutcome
  > - Use [Theory]+[InlineData] for rows that share the same assertion
  > - Apply Arrange/Act/Assert with section comments
  > - One test method per matrix row
  >
  > Test Case Matrix: [Paste Phase 5b approved matrix]
  > Implementation:   [Paste Phase 3 service class]
  > ```

* **2. Run and Measure:**
  ```bash
  dotnet test
  dotnet-coverage collect "dotnet test" --output coverage.xml --output-format cobertura
  reportgenerator -reports:coverage.xml -targetdir:coverage-report -reporttypes:Html
  ```
  Open `coverage-report/index.html`. Every matrix row must map to a green test.

* **3. Final Human Checkpoint ✅**
  * Test count equals approved matrix row count
  * Remove zombie tests (tests asserting only mock return values)
  * Branch coverage ≥ 80%

---

## Pre-Commit Checklist
Before opening a pull request, verify all items are complete:

* [ ] All Phase 1 business rules have a corresponding `[Fact]` or `[Theory]`
* [ ] `dotnet build` passes with zero warnings
* [ ] `dotnet test` passes — all tests green
* [ ] Branch coverage is ≥ 80%
* [ ] Code review verdict is `✅ PASS` or `⚠️ PASS WITH NOTES` with documented exceptions
* [ ] XML doc comments on all public interfaces and the service class
* [ ] No zombie tests (tests asserting only mock-configured return values)
* [ ] Open Questions from Phase 1 are resolved and documented

---

**Next Steps:** Once merged, monitor the feature through production metrics and schedule a review in 3 months to verify business rules have not evolved. If they have, re-run this workflow from Phase 1.
