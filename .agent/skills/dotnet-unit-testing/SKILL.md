---
name: dotnet-unit-testing
description: Generates, analyzes, and improves xUnit/NUnit/MSTest unit tests for .NET C# projects. Use when the user asks to write unit tests, generate test cases, improve test coverage, mock dependencies, or review existing tests in a .NET or C# codebase.
---

# .NET Unit Testing

Generates production-quality unit tests for .NET C# code using xUnit, NUnit, or MSTest alongside mocking libraries such as NSubstitute, Moq, and FakeItEasy.

## Quick start

When asked to write unit tests, follow this sequence:

1. **Detect the framework** – Check existing test project files for `<PackageReference>` entries. Prefer the framework already in use. Fallback order: xUnit → NUnit → MSTest.
2. **Detect the mocking library** – Look for NSubstitute, Moq, or FakeItEasy references. Default to NSubstitute if nothing is configured.
3. **Analyze the target class** – Read its constructor dependencies, public method signatures, XML doc comments, and nullable annotations.
4. **Generate the test class** – Follow the structure in the [Test class template](#test-class-template) section below.
5. **Cover all branches** – Confirm each `if`, `switch`, `throw`, and `return` path has at least one test.
6. **Review for zombie tests** – See [PATTERNS.md](PATTERNS.md) for anti-patterns to remove.

## Test class template

Use this canonical structure for every new xUnit test class:

```csharp
// <ClassName>Tests.cs
using NSubstitute;
using Xunit;

public class <ClassName>Tests
{
    // Arrange shared substitutes once per test class
    private readonly I<Dependency> _dep = Substitute.For<I<Dependency>>();
    private readonly <ClassName> _sut;   // sut = System Under Test

    public <ClassName>Tests()
    {
        _sut = new <ClassName>(_dep);
    }

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

    [Theory]
    [InlineData(<value1>)]
    [InlineData(<value2>)]
    public async Task <MethodName>_<Scenario>_<ExpectedOutcome>(int input)
    {
        // Act & Assert
        await Assert.ThrowsAsync<ArgumentException>(() =>
            _sut.<MethodName>(input));
    }
}
```

## Naming convention

Always use `MethodName_Scenario_ExpectedOutcome`:

| ✅ Good | ❌ Bad |
|---|---|
| `PlaceOrder_NegativeQuantity_ThrowsArgumentException` | `TestPlaceOrder1` |
| `GetUser_UserNotFound_ReturnsNull` | `TestGetUser` |
| `Calculate_ValidDiscount_ReturnsReducedPrice` | `Should_work` |

## Coverage targets

Ensure every generated suite covers:

- **Happy path** – valid inputs, expected return value
- **Boundary conditions** – zero, negative, max-value, empty string, null
- **Exception paths** – every `throw` statement in the method body
- **Async correctness** – all `async Task` methods tested with `await` and `Assert.ThrowsAsync`

## Running coverage

After generating tests, measure coverage with:

```bash
dotnet tool install --global dotnet-coverage
dotnet-coverage collect "dotnet test" --output coverage.xml --output-format cobertura
dotnet tool install --global dotnet-reportgenerator-globaltool
reportgenerator -reports:coverage.xml -targetdir:coverage-report -reporttypes:Html
```

Open `coverage-report/index.html` to find uncovered branches.

## Advanced patterns

For mocking patterns, edge-case generators, and anti-patterns, see [PATTERNS.md](PATTERNS.md).
