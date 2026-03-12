---
title: "Agentic AI Workflow for .NET Developers: From Requirements to Tested Code"
description: "Use an agentic AI workflow for .NET developers to move from raw requirements to production C# service code and xUnit tests — in four structured phases: analyze, design, implement, and test."
focus_keyphrase: "agentic AI workflow .NET developer"
keywords: ["agentic AI workflow .NET", "AI requirements analysis C#", "Claude Code .NET", "xUnit test generation AI", "Semantic Kernel .NET agent"]
slug: "agentic-ai-workflow-dotnet-developer"
author: ""
date: "2025-03-12"
---

# Agentic AI Workflow for .NET Developers: From Requirements to Tested Code

A new feature request arrives in your backlog. Traditionally, a developer spends several hours reading through it, sketching a class diagram, writing the service layer, and authoring unit tests. The entire activity consumes significant time before the first PR is opened.

**Agentic AI workflows** compress that cycle dramatically. Instead of a single AI prompt, an agentic workflow chains specialized steps — each building on the previous output — to carry a requirement all the way from raw text to tested, production-ready code.

In this post, you will follow a single feature request through a four-phase agentic AI workflow purpose-built for .NET developers:

| Phase | What the AI does | What you review |
|---|---|---|
| **1. Analyze** | Reads requirements → extracts entities, rules, edge cases | Structured requirement breakdown |
| **2. Technical Notes** | Designs interfaces, DTOs, and architecture decisions | C# interface contracts |
| **3. Implement Service** | Writes the full service class | Production C# code |
| **4. Write Unit Tests** | Generates xUnit tests with NSubstitute | Full test suite |

---

## What Is an Agentic AI Workflow?

An **agentic AI workflow** is a sequence of coordinated AI tasks where each step's output feeds the next. Unlike a one-shot prompt ("write me a service"), an agentic workflow gives the AI a *goal*, a *plan*, and a *feedback loop*—so it can observe, decide, act, and adapt at each stage.

For .NET developers, this means:

- **Structured context** flows between phases — the analysis informs the design, the design informs the code, the code informs the tests.
- **Human checkpoints** sit between each phase — you review and approve before the AI proceeds.
- **Specialized agents** handle each phase — the AI adopts a different role (Analyst → Architect → Developer → QA Engineer) for maximum output quality.

The tools that make this practical today: **Claude Code**, **GitHub Copilot Agent Mode**, and **Cursor** with multi-step agent instructions.

---

## The Scenario: Implementing a Product Discount Service

Throughout this post, you will use the following requirement as a running example:

> **Feature Request: Discount Engine**
> The system must apply promotional discount codes to product orders. A code of `SAVE10` deducts a flat $10. A code of `HALF` applies a 50% discount. No discount applies below a $5 floor price. Invalid or expired codes return the original price. Quantity must be greater than zero or the order is rejected.

This is intentionally simple enough to follow but complex enough to have real edge cases.

---

## Phase 1: Analyze Requirements

### What the AI agent does

The Analyzer agent reads the raw requirement text and extracts a structured breakdown: domain entities, business rules, input constraints, and edge cases. This eliminates the ambiguity that causes bugs later.

### Your prompt

```plaintext
You are a senior .NET requirements analyst.

Read the following feature request and produce a structured analysis:
1. Domain Entities — list all nouns that represent data objects.
2. Business Rules — list each conditional rule as an "IF … THEN …" statement.
3. Input Constraints — list all validations that must be enforced.
4. Edge Cases — list scenarios that could cause unexpected behavior.
5. Open Questions — list anything the requirement does not clarify.

Feature Request:
[Paste the requirement here]
```

### AI output — structured analysis

```markdown
## Domain Entities
- **Order** — productId, quantity, couponCode (optional), finalPrice
- **DiscountCode** — code (string), type (Flat | Percentage), value (decimal)
- **OrderResult** — orderId, finalPrice

## Business Rules
- IF couponCode = "SAVE10" THEN discount = $10 (flat)
- IF couponCode = "HALF" THEN discount = 50% of basePrice
- IF (basePrice − discount) < $5 THEN finalPrice = $5 (floor)
- IF couponCode is null OR unrecognized THEN discount = $0

## Input Constraints
- quantity MUST be > 0; otherwise reject with ArgumentException
- basePrice MUST be > 0 (enforced by repository)

## Edge Cases
- SAVE10 on a $6 item → result would be −$4; floor rule kicks in → $5
- HALF on a $9 item → $4.50 which is below floor → $5
- Null couponCode must not throw NullReferenceException

## Open Questions
- Should expired/invalid codes silently fall back to $0 or return an error?
- Is the $5 floor applied before or after tax?
```

**Your checkpoint:** Review this analysis before proceeding. Clarify open questions with the product manager, then approve.

---

## Phase 2: Technical Notes

### What the AI agent does

The Architect agent takes the structured analysis and produces concrete C# interface contracts, DTOs, and key architecture decisions. This phase locks in the *shape* of the code before any implementation begins.

### Your prompt

```plaintext
You are a senior .NET architect.

Using the requirement analysis below, design the .NET service layer:
1. Define C# interface(s) with XML doc comments for all public members.
2. Define C# record or class DTOs for inputs and outputs.
3. State key architecture decisions (e.g., where validation lives, 
   exception strategy, async/await usage).
4. Do NOT write the service implementation yet.

[Paste Phase 1 output here]
```

### AI output — technical notes

```csharp
// === Interfaces ===

/// <summary>Retrieves product pricing from the data store.</summary>
public interface IProductRepository
{
    /// <summary>Returns the base price for a product.</summary>
    Task<decimal> GetProductPriceAsync(int productId);
}

/// <summary>Calculates discount amounts for coupon codes.</summary>
public interface IDiscountCalculator
{
    /// <summary>
    /// Returns the discount amount for a given base price and code.
    /// Returns 0 if the code is invalid or null.
    /// </summary>
    Task<decimal> CalculateAsync(decimal basePrice, string? couponCode);
}

/// <summary>Orchestrates order placement with discount logic.</summary>
public interface IOrderService
{
    Task<OrderResult> PlaceOrderAsync(int productId, int quantity, string? couponCode);
}

// === DTOs ===

/// <summary>Result returned after a successful order placement.</summary>
public record OrderResult(int OrderId, decimal FinalPrice);

// === Architecture Decisions ===
// 1. Validation (quantity > 0) lives in OrderService, not the controller.
// 2. Floor price logic ($5 minimum) lives in OrderService after discount calc.
// 3. IDiscountCalculator is responsible for interpreting coupon codes;
//    OrderService does not contain any coupon-specific string literals.
// 4. All methods are async to support I/O-bound repository operations.
// 5. Exceptions: ArgumentException for invalid inputs, propagate repo exceptions.
```

**Your checkpoint:** Review interface contracts and architecture decisions. Adjust naming or responsibilities before the AI writes implementation code.

---

## Phase 3: Implement Service

### What the AI agent does

The Developer agent receives the interfaces and architecture decisions from Phase 2 and writes the full, compilable C# service implementation — no gaps, no TODOs.

### Your prompt

```plaintext
You are a senior .NET developer.

Implement the following .NET service class in C# (.NET 9):
- Implement IOrderService using the interfaces and DTOs defined below.
- Apply all business rules from the analysis (including the $5 floor).
- Follow the architecture decisions: validation in service, async throughout.
- Include XML doc comments on the class and PlaceOrderAsync method.
- Do NOT write unit tests yet.

[Paste Phase 2 output here]
```

### AI output — service implementation

```csharp
/// <summary>
/// Orchestrates product order placement, applying discount codes
/// and enforcing a minimum floor price of $5.
/// </summary>
public class OrderService : IOrderService
{
    private const decimal FloorPrice = 5m;

    private readonly IProductRepository _productRepository;
    private readonly IDiscountCalculator _discountCalculator;
    private readonly IOrderRepository _orderRepository;

    public OrderService(
        IProductRepository productRepository,
        IDiscountCalculator discountCalculator,
        IOrderRepository orderRepository)
    {
        _productRepository = productRepository;
        _discountCalculator = discountCalculator;
        _orderRepository = orderRepository;
    }

    /// <summary>
    /// Places an order for a product, applying any eligible discount
    /// and enforcing a $5 floor on the final price.
    /// </summary>
    /// <param name="productId">The product to order.</param>
    /// <param name="quantity">Must be greater than zero.</param>
    /// <param name="couponCode">Optional promotional code.</param>
    /// <returns>The placed order details, including final price.</returns>
    /// <exception cref="ArgumentException">
    /// Thrown when <paramref name="quantity"/> is zero or negative.
    /// </exception>
    public async Task<OrderResult> PlaceOrderAsync(
        int productId, int quantity, string? couponCode)
    {
        if (quantity <= 0)
            throw new ArgumentException(
                "Quantity must be greater than zero.", nameof(quantity));

        var basePrice = await _productRepository.GetProductPriceAsync(productId);
        var discount  = await _discountCalculator.CalculateAsync(basePrice, couponCode);

        var finalPrice = Math.Max(basePrice - discount, FloorPrice);

        var order = await _orderRepository.CreateOrderAsync(
            productId, quantity, finalPrice);

        return new OrderResult(order.Id, finalPrice);
    }
}
```

**Your checkpoint:** Review the implementation against Phase 1 business rules. Verify that `Math.Max(basePrice - discount, FloorPrice)` correctly handles all edge cases from the analysis.

---

## Phase 4: Write Unit Tests

### What the AI agent does

The QA Engineer agent takes the full service implementation and the Phase 1 edge cases, then generates a complete xUnit + NSubstitute test suite — covering happy paths, boundaries, and exception cases.

### Your prompt

```plaintext
You are a senior .NET QA engineer.

Write a complete xUnit test suite for OrderService.PlaceOrderAsync.
- Use NSubstitute to mock all dependencies.
- Follow the naming convention: MethodName_Scenario_ExpectedOutcome.
- Use [Theory] + [InlineData] for boundary/parametric cases.
- Cover every business rule and edge case from the analysis below.
- Apply the Arrange/Act/Assert pattern with explicit section comments.

[Paste Phase 1 analysis + Phase 3 implementation here]
```

### AI output — test suite

```csharp
using NSubstitute;
using Xunit;

public class OrderServiceTests
{
    // ── Dependencies ──────────────────────────────────────────────────────────
    private readonly IProductRepository   _products   = Substitute.For<IProductRepository>();
    private readonly IDiscountCalculator  _discounts  = Substitute.For<IDiscountCalculator>();
    private readonly IOrderRepository     _orders     = Substitute.For<IOrderRepository>();
    private readonly OrderService         _sut;

    public OrderServiceTests()
    {
        _sut = new OrderService(_products, _discounts, _orders);
    }

    // ── Happy path ────────────────────────────────────────────────────────────

    [Fact]
    public async Task PlaceOrderAsync_NoCoupon_ReturnsFullBasePrice()
    {
        // Arrange
        _products.GetProductPriceAsync(1).Returns(50m);
        _discounts.CalculateAsync(50m, null).Returns(0m);
        _orders.CreateOrderAsync(1, 2, 50m).Returns(new Order { Id = 10 });

        // Act
        var result = await _sut.PlaceOrderAsync(1, 2, null);

        // Assert
        Assert.Equal(10,  result.OrderId);
        Assert.Equal(50m, result.FinalPrice);
    }

    [Fact]
    public async Task PlaceOrderAsync_Save10Coupon_DeductsTenDollars()
    {
        // Arrange
        _products.GetProductPriceAsync(1).Returns(30m);
        _discounts.CalculateAsync(30m, "SAVE10").Returns(10m);
        _orders.CreateOrderAsync(1, 1, 20m).Returns(new Order { Id = 11 });

        // Act
        var result = await _sut.PlaceOrderAsync(1, 1, "SAVE10");

        // Assert
        Assert.Equal(20m, result.FinalPrice);
    }

    [Fact]
    public async Task PlaceOrderAsync_HalfCoupon_AppliesFiftyPercentDiscount()
    {
        // Arrange
        _products.GetProductPriceAsync(1).Returns(40m);
        _discounts.CalculateAsync(40m, "HALF").Returns(20m);
        _orders.CreateOrderAsync(1, 1, 20m).Returns(new Order { Id = 12 });

        // Act
        var result = await _sut.PlaceOrderAsync(1, 1, "HALF");

        // Assert
        Assert.Equal(20m, result.FinalPrice);
    }

    // ── Floor price rule ──────────────────────────────────────────────────────

    [Fact]
    public async Task PlaceOrderAsync_DiscountBelowFloor_ReturnsFiveFloor()
    {
        // Arrange — SAVE10 on $6 item → $-4 → floor at $5
        _products.GetProductPriceAsync(1).Returns(6m);
        _discounts.CalculateAsync(6m, "SAVE10").Returns(10m);
        _orders.CreateOrderAsync(1, 1, 5m).Returns(new Order { Id = 13 });

        // Act
        var result = await _sut.PlaceOrderAsync(1, 1, "SAVE10");

        // Assert
        Assert.Equal(5m, result.FinalPrice);
    }

    [Fact]
    public async Task PlaceOrderAsync_HalfCouponNearFloor_ReturnsFiveFloor()
    {
        // Arrange — HALF on $9 item → $4.50 → floor at $5
        _products.GetProductPriceAsync(1).Returns(9m);
        _discounts.CalculateAsync(9m, "HALF").Returns(4.5m);
        _orders.CreateOrderAsync(1, 1, 5m).Returns(new Order { Id = 14 });

        // Act
        var result = await _sut.PlaceOrderAsync(1, 1, "HALF");

        // Assert
        Assert.Equal(5m, result.FinalPrice);
    }

    // ── Input validation ──────────────────────────────────────────────────────

    [Theory]
    [InlineData(0)]
    [InlineData(-1)]
    [InlineData(-100)]
    public async Task PlaceOrderAsync_ZeroOrNegativeQuantity_ThrowsArgumentException(
        int invalidQuantity)
    {
        // Act & Assert
        var ex = await Assert.ThrowsAsync<ArgumentException>(() =>
            _sut.PlaceOrderAsync(1, invalidQuantity, null));

        Assert.Equal("quantity", ex.ParamName);
    }

    [Fact]
    public async Task PlaceOrderAsync_NullCouponCode_DoesNotThrow()
    {
        // Arrange
        _products.GetProductPriceAsync(1).Returns(20m);
        _discounts.CalculateAsync(20m, null).Returns(0m);
        _orders.CreateOrderAsync(1, 1, 20m).Returns(new Order { Id = 15 });

        // Act — should not throw NullReferenceException
        var result = await _sut.PlaceOrderAsync(1, 1, null);

        // Assert
        Assert.Equal(20m, result.FinalPrice);
    }
}
```

**Your checkpoint:** Run the tests (`dotnet test`), review assertion logic against Phase 1 edge cases, and add any gaps the AI missed.

---

## The Human-in-the-Loop Model

The workflow above is not fully automated — and that is intentional. Industry adoption patterns show that agentic AI performs best as a **collaborative partner, not an autonomous replacement**.

```
┌──────────────┐    ✅ review    ┌──────────────┐    ✅ review    ┌──────────────┐    ✅ review    ┌──────────────┐
│  Phase 1     │ ──────────────▶ │  Phase 2     │ ──────────────▶ │  Phase 3     │ ──────────────▶ │  Phase 4     │
│  Analyze     │                 │  Tech Notes  │                 │  Implement   │                 │  Unit Tests  │
└──────────────┘                 └──────────────┘                 └──────────────┘                 └──────────────┘
     AI role:                         AI role:                         AI role:                         AI role:
     Analyst                          Architect                        Developer                        QA Engineer
```

At each checkpoint you:
1. **Verify** the output matches the original intent.
2. **Correct** anything wrong before the next phase consumes it as input.
3. **Approve** to proceed — or send the AI back with feedback.

This feedback loop is what prevents compounding errors. A bad analysis produces a bad design produces broken code produces misleading tests — catching problems early at Phase 1 is far cheaper than fixing them at Phase 4.

---

## Key Takeaways

- **Chain your prompts deliberately.** Each phase's output becomes the next phase's input. Structure wins over raw intelligence.
- **Assign roles explicitly.** Telling the AI it is a "senior .NET architect" or "QA engineer" measurably improves output quality and specialization.
- **Keep phases short.** A phase that does too much loses focus. Four tight phases beat one sprawling prompt every time.
- **Always run the tests.** AI-generated tests can be syntactically correct but semantically wrong. `dotnet test` is your ground truth.
- **Save your prompts as templates.** Once this workflow produces good results for your codebase, save each phase prompt as a reusable `.cursorrules` entry or Claude Agent Skill.

---

## Ready to Implement Your First Agentic Workflow?

The four-phase workflow above can be applied to any .NET feature — not just discount engines. Start with a small, well-scoped requirement from your current backlog.

**Next Steps:**
1. Copy the four phase prompts above into your Claude Code, Cursor, or Copilot Chat session.
2. Run **Phase 1** on a real requirement from your backlog and share the structured output with your team.
3. Save the prompts as a team-wide template — or package them as an [Agent Skill](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) in your repository for automatic discovery.

---

*Sources Consulted:*
- [Microsoft Semantic Kernel – Official Docs](https://learn.microsoft.com/en-us/semantic-kernel/overview/)
- [Microsoft Agent Framework – GitHub](https://github.com/microsoft/agents)
- [Agentic AI in Software Development – IBM](https://www.ibm.com/think/topics/agentic-ai)
- [xUnit.net Documentation](https://xunit.net/)
- [NSubstitute Documentation](https://nsubstitute.github.io/)
