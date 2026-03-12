---
name: dotnet-implementation
description: Implements a .NET C# service class from approved interface contracts and architecture decisions. Use after technical design is approved to generate a complete, compilable service class that covers all business rules.
---

# .NET Implementation

Generates a complete C# service class implementation from the interface contracts, DTOs, and architecture decisions produced by `dotnet-technical-design`. All business rules and edge cases from Phase 1 must be addressed in the output.

## Quick start

When given a technical design, run the following prompt:

> ```plaintext
> You are a senior .NET developer (.NET 9).
> Implement the service class using the interfaces, DTOs, and architecture decisions below.
>
> Requirements:
> - Implement ALL business rules from the Phase 1 analysis
> - Apply the validation strategy from the architecture decisions
> - Use the exception types and logging approach specified
> - Add XML doc comments on the class and all public methods
> - Do NOT write unit tests yet
>
> Technical Design (Phase 2 output):
> [paste interfaces, DTOs, and architecture decisions here]
>
> Business Rules (from Phase 1):
> [paste business rules and edge cases here]
> ```

## Output format

```csharp
/// <summary>Handles order placement with optional coupon discount.</summary>
public class OrderService : IOrderService
{
    private readonly IOrderRepository _repository;
    private readonly ILogger<OrderService> _logger;

    public OrderService(IOrderRepository repository, ILogger<OrderService> logger)
    {
        ArgumentNullException.ThrowIfNull(repository);
        ArgumentNullException.ThrowIfNull(logger);
        _repository = repository;
        _logger = logger;
    }

    /// <inheritdoc/>
    public async Task<OrderResult> PlaceOrderAsync(PlaceOrderRequest request, CancellationToken ct = default)
    {
        ArgumentNullException.ThrowIfNull(request);
        if (request.Quantity <= 0)
            throw new ArgumentException("Quantity must be greater than zero.", nameof(request));

        // Business rule: apply coupon discount if provided
        var price = request.UnitPrice * request.Quantity;
        bool discountApplied = false;

        if (!string.IsNullOrWhiteSpace(request.CouponCode))
        {
            var coupon = await _repository.GetCouponAsync(request.CouponCode, ct).ConfigureAwait(false);
            if (coupon is not null)
            {
                price -= price * (coupon.DiscountPercent / 100m);
                discountApplied = true;
            }
        }

        // Business rule: minimum price floor of $5
        price = Math.Max(price, 5m);

        return new OrderResult(price, discountApplied);
    }
}
```

## Human checkpoint

After generating the implementation:
- [ ] Does the code compile? Run `dotnet build` — zero warnings, zero errors
- [ ] Trace every Phase 1 **Business Rule** to a line of code
- [ ] Confirm all **Edge Cases** are handled (null, zero, boundary values)
- [ ] Confirm **architecture decisions** are followed (async, exceptions, DI)
