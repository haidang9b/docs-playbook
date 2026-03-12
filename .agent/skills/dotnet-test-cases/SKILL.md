---
name: dotnet-test-cases
description: Generates and verifies a plain-language test case matrix for a .NET service from requirements analysis and implementation. Use before writing xUnit code to establish and review what must be tested. Outputs a structured table developers can approve as acceptance criteria.
---

# .NET Test Case Generator & Verifier

Produces a human-readable **test case matrix** from a requirements analysis and service implementation. This separates *what to test* from *how to implement tests* — the matrix is reviewed and approved before any xUnit code is written.

## Phase A: Generate the Matrix

Run this prompt against the Phase 1 analysis and Phase 3 implementation:

> ```plaintext
> You are a senior .NET QA engineer.
> From the requirements analysis and service implementation below, produce a test case matrix.
>
> Use these columns:
> | # | Method | Scenario | Input | Expected Output | Type |
>
> Type values:
> - Happy Path   — valid inputs, successful outcome
> - Edge Case    — boundary conditions, optional fields
> - Validation   — invalid input that triggers ArgumentException
> - Exception    — infrastructure failure, external dependency error
>
> Rules:
> - One row per distinct scenario
> - Do NOT write any C# code
> - Cover every Business Rule and Edge Case from the requirements analysis
>
> Requirements Analysis: [paste Phase 1 output]
> Implementation:        [paste Phase 3 service class]
> ```

## Phase B: Verify the Matrix

Before proceeding to xUnit code generation, verify the matrix against the Phase 1 analysis:

| Check | Criterion |
|---|---|
| ✅ Business Rules | Every IF/THEN rule has at least one Happy Path row |
| ✅ Edge Cases | Every Phase 1 edge case has a corresponding row |
| ✅ Input Constraints | Every constraint has a Validation or Exception row |
| ✅ No duplicates | No two rows share the same inputs AND expected output |

If a row is missing, add it:

> ```plaintext
> Add a row: Method=X, Scenario=[description], Input=Y, Expected=Z, Type=Edge Case
> ```

**Approve the matrix** before proceeding. The final row count becomes the target for `dotnet test`.

## Example output

| # | Method | Scenario | Input | Expected Output | Type |
|---|---|---|---|---|---|
| 1 | `PlaceOrderAsync` | No coupon | qty=2, price=$10 | $20 returned | Happy Path |
| 2 | `PlaceOrderAsync` | Valid coupon SAVE10 | qty=1, price=$30 | $27 returned | Happy Path |
| 3 | `PlaceOrderAsync` | Discount below floor | price=$6, SAVE10 | $5 floor applied | Edge Case |
| 4 | `PlaceOrderAsync` | Zero quantity | qty=0 | `ArgumentException` | Validation |
| 5 | `PlaceOrderAsync` | Null coupon code | code=null | no discount, no exception | Edge Case |
| 6 | `PlaceOrderAsync` | Repository throws | DB unavailable | exception propagated | Exception |
