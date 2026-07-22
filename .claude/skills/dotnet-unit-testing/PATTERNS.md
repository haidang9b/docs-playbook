# .NET Unit Testing – Patterns Reference

This file is loaded by the `dotnet-unit-testing` skill when the user asks for advanced mocking patterns or anti-pattern review.

---

## Mocking patterns

### NSubstitute (preferred)

```csharp
// Return a value
_repo.GetProductPriceAsync(1).Returns(99.99m);

// Return based on argument matcher
_repo.GetProductPriceAsync(Arg.Any<int>()).Returns(50m);

// Verify a call was made
await _repo.Received(1).CreateOrderAsync(Arg.Any<int>(), Arg.Any<int>(), Arg.Any<decimal>());

// Throw an exception
_repo.GetProductPriceAsync(Arg.Any<int>()).Throws(new DbException("timeout"));
```

### Moq (alternative)

```csharp
var mock = new Mock<IOrderRepository>();
mock.Setup(r => r.GetProductPriceAsync(It.IsAny<int>())).ReturnsAsync(99.99m);
mock.Verify(r => r.CreateOrderAsync(It.IsAny<int>(), It.IsAny<int>(), It.IsAny<decimal>()), Times.Once);
```

---

## Edge-case prompting patterns

When generating edge cases, iterate through these categories:

| Category | Example inputs |
|---|---|
| Null / empty | `null`, `""`, `string.Empty` |
| Boundary integers | `0`, `-1`, `int.MaxValue`, `int.MinValue` |
| Boundary decimals | `0m`, `-0.01m`, `decimal.MaxValue` |
| Empty collections | `[]`, `new List<T>()` |
| Concurrency | Run the same method twice in parallel and assert idempotency |

---

## Anti-patterns to avoid

### ❌ Zombie tests

A zombie test passes but asserts something trivially true because it only reflects the mock setup:

```csharp
// BAD – just proves NSubstitute returns what you told it to
[Fact]
public async Task GetPrice_ReturnsPrice()
{
    _repo.GetProductPriceAsync(1).Returns(100m);
    var result = await _sut.GetProductPriceAsync(1);
    Assert.Equal(100m, result); // This tests NSubstitute, not your code
}
```

**Fix:** Assert on business behavior, not mock return values.

### ❌ Testing private implementation

Do not use reflection to access private methods. Test behavior through the public API only.

### ❌ Multiple behaviors per Fact

Each `[Fact]` must test exactly one behavior. Split compound assertions into separate facts.

```csharp
// BAD
[Fact]
public async Task PlaceOrder_Works()
{
    // ... tests 3 different things in one fact
    Assert.Equal(80m, result.FinalPrice);
    Assert.Equal(42, result.OrderId);
    Assert.True(result.IsConfirmed);
}

// GOOD – split into three focused facts
```

### ❌ Arrange duplication

Extract shared setup into the constructor or a private helper method, not repeated across every `[Fact]`.

---

## xUnit-specific idioms

| Use case | Attribute |
|---|---|
| Single scenario | `[Fact]` |
| Multiple data sets | `[Theory]` + `[InlineData]` |
| Data from a class | `[Theory]` + `[MemberData]` |
| Skip a test | `[Fact(Skip = "reason")]` |
| Custom display name | `[Fact(DisplayName = "Readable name")]` |
