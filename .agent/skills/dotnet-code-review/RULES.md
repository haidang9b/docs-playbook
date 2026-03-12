# .NET Code Review – Full Rules Reference

Loaded by the `dotnet-code-review` skill when detailed rule examples are required.

---

## N — Naming & Style

### N1 · PascalCase for types and members *(Minor)*
Classes, interfaces, enums, methods, and properties must use PascalCase.
```csharp
// ❌
public class orderService { }
public void placeorder() { }

// ✅
public class OrderService { }
public void PlaceOrder() { }
```

### N2 · Private fields use `_camelCase` *(Minor)*
```csharp
// ❌
private IOrderRepository repository;
private IOrderRepository Repository;

// ✅
private readonly IOrderRepository _repository;
```

### N3 · Interfaces start with `I` *(Major)*
```csharp
// ❌
public interface OrderRepository { }

// ✅
public interface IOrderRepository { }
```

### N4 · Async methods end with `Async` *(Major)*
```csharp
// ❌
public async Task<Order> PlaceOrder() { }

// ✅
public async Task<Order> PlaceOrderAsync() { }
```

---

## S — SOLID Principles

### S1 · Single Responsibility *(Major)*
A class has one reason to change. Signs of violation: class name contains "And", "Manager", "Helper", or "Util" with broad responsibilities; more than ~200 lines; multiple unrelated dependency injections.

### S2 · Depend on abstractions *(Blocker)*
Constructor parameters and field types must be interfaces, not concrete classes.
```csharp
// ❌ — depends on concrete class
public class OrderService
{
    private readonly SqlOrderRepository _repo;  // ❌ concrete
    public OrderService(SqlOrderRepository repo) { _repo = repo; }
}

// ✅ — depends on abstraction
public class OrderService
{
    private readonly IOrderRepository _repo;    // ✅ interface
    public OrderService(IOrderRepository repo) { _repo = repo; }
}
```

### S3 · Method length ≤ 30 lines *(Minor)*
Extract private helper methods or separate classes when a method exceeds 30 lines.

---

## A — Async/Await

### A1 · No `async void` *(Blocker)*
`async void` cannot be awaited and swallows exceptions. Only valid in UI event handlers.
```csharp
// ❌
public async void ProcessOrderAsync() { }

// ✅
public async Task ProcessOrderAsync() { }
```

### A2 · No `.Result` or `.Wait()` *(Blocker)*
Synchronously blocking on async code causes deadlocks in ASP.NET contexts.
```csharp
// ❌
var order = _repo.GetOrderAsync(id).Result;
_repo.SaveAsync(order).Wait();

// ✅
var order = await _repo.GetOrderAsync(id);
await _repo.SaveAsync(order);
```

### A3 · `CancellationToken` on I/O-bound methods *(Major)*
All public async methods that perform I/O must accept a `CancellationToken`.
```csharp
// ❌
public async Task<Order> GetOrderAsync(int id) { }

// ✅
public async Task<Order> GetOrderAsync(int id, CancellationToken ct = default) { }
```

### A4 · `ConfigureAwait(false)` in libraries *(Minor)*
Use `ConfigureAwait(false)` in library/infrastructure code to avoid context switching overhead. Not required in ASP.NET Core application code.

---

## E — Exception Handling

### E1 · Never catch `Exception` silently *(Blocker)*
```csharp
// ❌
try { ... }
catch (Exception) { }  // swallows everything

// ✅
try { ... }
catch (Exception ex)
{
    _logger.LogError(ex, "Order placement failed");
    throw;  // rethrow to preserve stack trace
}
```

### E2 · Validate inputs with typed exceptions *(Major)*
```csharp
// ❌
if (quantity <= 0) throw new Exception("bad quantity");

// ✅
if (quantity <= 0)
    throw new ArgumentException("Quantity must be > 0.", nameof(quantity));
```

### E3 · No empty catch blocks *(Blocker)*
An empty `catch` block hides bugs. At minimum, log and rethrow.

### E4 · Use `ArgumentNullException.ThrowIfNull()` *(Minor)*
```csharp
// ❌
if (order == null) throw new ArgumentNullException(nameof(order));

// ✅ (.NET 6+)
ArgumentNullException.ThrowIfNull(order);
```

---

## P — Performance & Security

### P1 · No string concatenation in loops *(Major)*
```csharp
// ❌
string result = "";
foreach (var item in items) result += item;

// ✅
var sb = new StringBuilder();
foreach (var item in items) sb.Append(item);
var result = sb.ToString();
```

### P2 · Prefer `IEnumerable<T>` in public APIs *(Minor)*
Exposing `List<T>` allows callers to mutate your internal collection. Prefer `IReadOnlyList<T>` or `IEnumerable<T>`.

### P3 · No synchronous I/O in async methods *(Blocker)*
```csharp
// ❌
public async Task SaveReportAsync(string path, string content)
{
    File.WriteAllText(path, content);   // synchronous — blocks thread
}

// ✅
public async Task SaveReportAsync(string path, string content, CancellationToken ct = default)
{
    await File.WriteAllTextAsync(path, content, ct);
}
```

### SEC1 · No hardcoded secrets *(Blocker)*
Connection strings, API keys, passwords, and tokens must come from environment variables, `IConfiguration`, or a secrets manager — never hardcoded in source.

### SEC2 · Sanitize external inputs *(Major)*
All data from HTTP requests, message queues, or file uploads must be validated before use. Reject or sanitize before passing to business logic.

### SEC3 · Parameterized queries only *(Blocker)*
```csharp
// ❌
var sql = $"SELECT * FROM Orders WHERE Id = {id}";

// ✅
var order = await _db.Orders.FindAsync(id, ct);
// or with Dapper:
var order = await conn.QueryFirstAsync<Order>(
    "SELECT * FROM Orders WHERE Id = @id", new { id });
```
