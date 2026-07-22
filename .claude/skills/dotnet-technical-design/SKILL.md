---
name: dotnet-technical-design
description: Converts a .NET requirements analysis into C# interface contracts, DTOs, and architecture decisions. Use after requirements are approved to design the service layer before any implementation code is written.
---

# .NET Technical Design

Translates a structured requirements analysis (from `dotnet-requirements-analysis`) into concrete C# contracts — interfaces, DTOs, and explicit architecture decisions. No implementation code is written in this phase.

## Quick start

When given a requirements analysis, run the following prompt:

> ```plaintext
> You are a senior .NET architect (.NET 9).
> Using the requirements analysis below, produce:
>
> 1. C# interface(s) with XML doc comments on all public members
> 2. C# record(s) or class(es) for all DTOs identified in Domain Entities
> 3. Architecture decisions document covering:
>    - Validation strategy (where and how validation errors are thrown)
>    - Exception handling approach (exception types, logging, rethrowing)
>    - Async usage (CancellationToken, ConfigureAwait)
>    - Dependency injection (what gets injected via constructor)
>
> Do NOT write any implementation code yet.
>
> Requirements Analysis:
> [paste Phase 1 output here]
> ```

## Output format

```csharp
// Interfaces
/// <summary>Handles order placement with coupon support.</summary>
public interface IOrderService
{
    /// <summary>Places an order, applying a coupon discount if provided.</summary>
    Task<OrderResult> PlaceOrderAsync(PlaceOrderRequest request, CancellationToken ct = default);
}

// DTOs
public record PlaceOrderRequest(int Quantity, decimal UnitPrice, string? CouponCode);
public record OrderResult(decimal FinalPrice, bool DiscountApplied);
```

```markdown
## Architecture Decisions
- Validation: throw ArgumentException for invalid inputs at service entry point
- Exceptions: catch and log infrastructure exceptions; rethrow domain exceptions
- Async: all public methods accept CancellationToken; use ConfigureAwait(false)
- DI: inject IOrderRepository and ILogger<OrderService> via constructor
```

## Human checkpoint

Review the interface contracts before proceeding to implementation:
- Does every Business Rule from Phase 1 traceable to a method signature?
- Are all DTOs complete with the fields identified in Domain Entities?
- Are architecture decisions clear enough to implement without ambiguity?
