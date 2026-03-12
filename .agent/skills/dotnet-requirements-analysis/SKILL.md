---
name: dotnet-requirements-analysis
description: Parses a .NET feature request or user story into a structured analysis covering domain entities, business rules, input constraints, edge cases, and open questions. Use at the start of any feature to establish shared understanding before design or coding begins.
---

# .NET Requirements Analysis

Transforms a raw feature requirement or user story into a machine-readable, structured breakdown that all subsequent workflow phases use as their source of truth.

## Quick start

When given a feature request, run the following prompt:

> ```plaintext
> You are a senior .NET requirements analyst.
> Parse the requirement below and produce a structured analysis with these five sections:
>
> 1. Domain Entities     — all nouns that represent data (records, classes, DTOs)
> 2. Business Rules      — each rule as: IF [condition] THEN [outcome]
> 3. Input Constraints   — every validation that must be enforced in code
> 4. Edge Cases          — boundary conditions and special scenarios
> 5. Open Questions      — anything the requirement does not clarify
>
> Requirement:
> [paste requirement here]
> ```

## Output format

Produce the analysis in this structure:

```
## Requirements Analysis

### Domain Entities
- Order (id, customerId, items, totalPrice, couponCode)
- Coupon (code, discountPercent, minimumOrder価格)

### Business Rules
- IF coupon code is provided THEN apply discountPercent to totalPrice
- IF discounted price < $5 THEN apply $5 floor price

### Input Constraints
- quantity: integer > 0
- couponCode: optional string, max 20 chars, alphanumeric only
- price: decimal > 0

### Edge Cases
- couponCode is null or empty string (no discount applied)
- discount brings price below $5 floor
- quantity is zero or negative
- couponCode provided but not found in database

### Open Questions
- [ ] What happens if the coupon is expired?
- [ ] Is the $5 floor applied per item or per order?
```

## Human checkpoint

Before proceeding to technical design, resolve all **Open Questions** with the product manager or domain expert. No open question should remain when entering Phase 2.
