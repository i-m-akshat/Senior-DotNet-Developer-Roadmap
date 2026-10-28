# THE COMPLETE .NET DEVELOPER ENCYCLOPEDIA + INTERVIEW MASTERY GUIDE
# Volume 4: OOP + Design Principles

> **Audience:** .NET developer ~3 years experience. Knows surface-level OOP. Needs deep "why" understanding for senior interviews.
> **Goal:** Master OOP not as definitions, but as *design decisions* made under real engineering constraints.

---

# TABLE OF CONTENTS

1. [The Four Pillars of OOP](#the-four-pillars)
   - [Encapsulation](#encapsulation)
   - [Inheritance](#inheritance)
   - [Polymorphism](#polymorphism)
   - [Abstraction](#abstraction)
2. [Interface vs Abstract Class — Complete Treatment](#interface-vs-abstract-class)
3. [Composition vs Inheritance](#composition-vs-inheritance)
4. [SOLID Principles — Full Treatment](#solid-principles)
5. [Dependency Injection — Complete](#dependency-injection)

---

# THE FOUR PILLARS OF OOP

## Encapsulation

### 1. Definition

Encapsulation is **bundling data (state) and behavior (methods) together** inside a class, while **controlling external access** to that data. It is NOT simply "making fields private" — that is *information hiding*, a related but distinct concept.

**Encapsulation = Data + Behavior bound together**
**Information Hiding = Deciding what to expose and what to protect**

Both work together. You encapsulate by putting balance and Deposit() in the same class. You hide by making balance private.

### 2. Historical Reason

Before OOP, programs were written procedurally: data structures (`struct`) held data and separate functions operated on them. Any function could modify any data. This led to:
- Impossible-to-trace bugs (who changed this value?)
- No invariants enforced (balance went negative)
- Global state disasters

OOP was invented specifically to give data *owners* — to say "this data belongs to this object, and only this object manages it."

### 3. Problem It Solves

```csharp
// BAD: Procedural approach — anyone can corrupt the balance
public class BankAccount
{
    public decimal Balance; // public field — no protection
    public string AccountNumber;
}

// Somewhere in the codebase...
account.Balance = -99999; // Perfectly legal. Catastrophic.
account.Balance += depositAmount; // No validation, no audit trail
```

With encapsulation, the `BankAccount` class *owns* its balance and enforces business rules.

### 4. Internal Working

In C#, encapsulation is enforced via:
- **Access modifiers**: `private`, `protected`, `internal`, `public`
- **Properties**: Expose state through controlled get/set with backing fields
- **Methods**: The only way to transition state, enforcing invariants

The CLR enforces access modifiers at IL (Intermediate Language) level — attempting to access a private member via reflection still works but requires `BindingFlags.NonPublic`, making it explicit and intentional.

### 5. Production Example: BankAccount

```csharp
// GOOD: Encapsulated BankAccount
public class BankAccount
{
    // Private backing field — external code CANNOT set this directly
    private decimal _balance;
    private readonly List<Transaction> _transactions = new();
    
    // Read-only property — external code can READ but not WRITE
    public decimal Balance => _balance;
    public string AccountNumber { get; }
    public IReadOnlyList<Transaction> Transactions => _transactions.AsReadOnly();
    
    public BankAccount(string accountNumber, decimal initialDeposit)
    {
        if (string.IsNullOrWhiteSpace(accountNumber))
            throw new ArgumentException("Account number required", nameof(accountNumber));
        if (initialDeposit < 0)
            throw new ArgumentException("Initial deposit cannot be negative");
            
        AccountNumber = accountNumber;
        _balance = initialDeposit;
    }
    
    // Behavior and data are together — the class OWNS its state transitions
    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Deposit amount must be positive");
        
        _balance += amount;
        _transactions.Add(new Transaction(TransactionType.Deposit, amount, _balance));
    }
    
    public void Withdraw(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Withdrawal amount must be positive");
        if (amount > _balance)
            throw new InvalidOperationException($"Insufficient funds. Balance: {_balance}, Requested: {amount}");
        
        _balance -= amount;
        _transactions.Add(new Transaction(TransactionType.Withdrawal, amount, _balance));
    }
    
    // Behavior specific to this domain — belongs HERE, not in a service
    public bool CanWithdraw(decimal amount) => amount > 0 && amount <= _balance;
}
```

### 6. Property vs Public Field — Why Properties Win

```csharp
// Public field — breaks encapsulation
public decimal Balance;

// Property — controlled access with future flexibility
public decimal Balance { get; private set; }

// You can later add:
public decimal Balance 
{ 
    get => _balance;
    private set
    {
        if (value < 0) throw new InvalidOperationException("Balance cannot be negative");
        _balance = value;
        OnBalanceChanged?.Invoke(value); // Event notification
    }
}
```

**Why properties win:**
1. You can add validation without breaking API consumers
2. You can add logging/notification logic
3. You can make get and set have different access levels
4. Properties work with data binding, serialization, reflection — fields don't

### 7. Execution Flow

1. Client calls `account.Deposit(500)`
2. `Deposit` validates amount > 0
3. Updates `_balance` (private backing field)
4. Appends to `_transactions` list
5. Client reads `account.Balance` → gets computed property → returns `_balance`
6. Client **cannot** do `account._balance = 0` → compile error

### 8. Design Reasoning

The key insight: **Encapsulation is about protecting invariants.** A BankAccount invariant is "balance cannot go negative without overdraft." By encapsulating, we guarantee this is ALWAYS true, regardless of what 47 different services try to do.

### 9. Performance

- Properties with simple get/set are **inlined by JIT** — zero overhead vs public fields
- `get => _balance` (expression-bodied) compiles identically to a field access
- Only expensive if getter has complex logic (database call, computation)

### 10. Common Mistakes

```csharp
// MISTAKE 1: Exposing mutable collections
public List<Transaction> Transactions { get; } = new(); // External code can Add/Remove!

// FIX: Return IReadOnlyList
public IReadOnlyList<Transaction> Transactions => _transactions.AsReadOnly();

// MISTAKE 2: Anemic domain model — data without behavior
public class Order
{
    public decimal Total { get; set; } // No validation, no logic
    public OrderStatus Status { get; set; } // Anyone can set to anything
}
// Then OrderService has 500 lines of business logic that SHOULD be in Order

// MISTAKE 3: Getter-only properties still exposing mutable objects
public Address ShippingAddress { get; private set; } // Address is a class, mutable!
// FIX: Either make Address a record (immutable) or clone on return
```

### 11. Production Problems

**Problem:** In a payment processing system, a `PaymentRequest` object was fully public. A logging middleware accidentally mutated `Amount` to zero before it reached the payment gateway. Transactions went through as $0.

**Fix:** Encapsulation + immutable records.
```csharp
public sealed record PaymentRequest(
    string OrderId,
    decimal Amount,
    string Currency,
    string CustomerId
); // Immutable by default — records are copy-on-modify
```

### 12. Interview Prep

**Basic:** "What is encapsulation?"
> Encapsulation bundles data and behavior together in a class and controls external access through access modifiers and properties. It's not just making fields private — it's about the class *owning* its state and enforcing business invariants.

**Intermediate:** "What's the difference between encapsulation and information hiding?"
> Encapsulation is about grouping data and behavior together. Information hiding is about what you expose. They work together: you encapsulate a `BankAccount` (data + methods in one class), and you hide the `_balance` field (information hiding) so external code can't corrupt it.

**Senior:** "How does encapsulation relate to the design of domain models?"
> A well-encapsulated domain model is a Rich Domain Model where invariants are enforced by the object itself. The alternative is an Anemic Domain Model — objects are just data bags, and all logic lives in services. Anemic models violate encapsulation because the data and its behavior are separated. In DDD, entities and aggregates should encapsulate their state transitions.

**Tricky:** "Can you have encapsulation without OOP?"
> Yes. Closures in functional languages encapsulate state. Modules in non-OOP languages can encapsulate. OOP formalizes it with classes, but the concept is architectural, not language-specific.

---

## Inheritance

### 1. Definition

Inheritance is a mechanism where a **derived class (child)** acquires the properties and behaviors of a **base class (parent)**. It creates an **IS-A relationship**.

### 2. Historical Reason

In the 1980s–90s, inheritance was seen as the primary code reuse mechanism. "Don't rewrite PaymentProcessor logic — inherit from it." This led to massive, deep hierarchies. By the 2000s, the industry recognized the problems and began favoring composition. Inheritance still has a place but requires careful judgment.

### 3. Problem It Solves (and Creates)

**Legitimate use:** Polymorphism — treating different subtypes uniformly through a base type reference.

```csharp
// PaymentProcessor hierarchy
public abstract class PaymentProcessor
{
    protected readonly ILogger _logger;
    
    protected PaymentProcessor(ILogger logger) => _logger = logger;
    
    // Template method — skeleton defined here
    public async Task<PaymentResult> ProcessAsync(PaymentRequest request)
    {
        _logger.LogInformation("Processing payment {Amount} via {Gateway}", 
            request.Amount, GetType().Name);
        
        var validationResult = Validate(request);
        if (!validationResult.IsValid)
            return PaymentResult.Failed(validationResult.Errors);
        
        var result = await ChargeAsync(request);
        
        await PostProcessAsync(request, result);
        return result;
    }
    
    protected abstract Task<PaymentResult> ChargeAsync(PaymentRequest request);
    
    protected virtual ValidationResult Validate(PaymentRequest request)
    {
        // Default validation — subclasses can extend
        if (request.Amount <= 0)
            return ValidationResult.Failure("Amount must be positive");
        return ValidationResult.Success();
    }
    
    protected virtual Task PostProcessAsync(PaymentRequest request, PaymentResult result)
        => Task.CompletedTask; // Default: do nothing
}

public class StripePaymentProcessor : PaymentProcessor
{
    private readonly IStripeClient _stripeClient;
    
    public StripePaymentProcessor(IStripeClient stripeClient, ILogger<StripePaymentProcessor> logger)
        : base(logger)
    {
        _stripeClient = stripeClient;
    }
    
    protected override async Task<PaymentResult> ChargeAsync(PaymentRequest request)
    {
        var charge = await _stripeClient.Charges.CreateAsync(new ChargeCreateOptions
        {
            Amount = (long)(request.Amount * 100), // Stripe uses cents
            Currency = request.Currency,
            Customer = request.CustomerId
        });
        
        return charge.Status == "succeeded"
            ? PaymentResult.Success(charge.Id)
            : PaymentResult.Failed(charge.FailureMessage);
    }
}
```

### 4. The Fragile Base Class Problem

```csharp
public class OrderCollection
{
    protected List<Order> _orders = new();
    
    public virtual void Add(Order order) => _orders.Add(order);
    
    public virtual void AddRange(IEnumerable<Order> orders)
    {
        foreach (var order in orders)
            Add(order); // Calls virtual Add
    }
}

public class AuditedOrderCollection : OrderCollection
{
    private int _addCount;
    
    public override void Add(Order order)
    {
        _addCount++;
        base.Add(order);
    }
}

// BUG: AddRange calls virtual Add → _addCount is incremented per item ✓
// If base class CHANGES AddRange to _orders.AddRange(orders) directly,
// _addCount stops incrementing → derived class breaks silently
// This is the FRAGILE BASE CLASS PROBLEM
```

### 5. Calling Virtual Methods from Constructors — The Dangerous Trap

```csharp
public class BaseProcessor
{
    public BaseProcessor()
    {
        Initialize(); // DANGEROUS! Calls derived class method
    }
    
    protected virtual void Initialize()
    {
        Console.WriteLine("Base initialize");
    }
}

public class DerivedProcessor : BaseProcessor
{
    private readonly string _connectionString; // Not yet initialized!
    
    public DerivedProcessor(string connectionString)
        : base() // base() runs FIRST
    {
        _connectionString = connectionString;
    }
    
    protected override void Initialize()
    {
        // _connectionString is NULL here! Base constructor runs first.
        Console.WriteLine($"Connecting to {_connectionString}"); // NullReferenceException!
    }
}
```

**Rule:** Never call virtual methods in constructors. The derived class hasn't been initialized yet.

### 6. `sealed` Classes for Protection

```csharp
// Sealed: cannot be inherited — guards against fragile base class
public sealed class RazorpayPaymentProcessor : PaymentProcessor
{
    protected override async Task<PaymentResult> ChargeAsync(PaymentRequest request)
    {
        // Razorpay-specific implementation
        // sealed because Razorpay API is very specific — don't extend this
    }
}
```

### 7. When Inheritance Becomes a Trap

- Deep hierarchies (>3 levels) become impossible to understand
- When you inherit for code reuse but there's no IS-A relationship
- When you can't explain the substitutability (LSP violation)
- When you change base class and 12 subclasses break

### 8. Interview Prep

**Basic:** "What is inheritance?"
> Inheritance allows a derived class to reuse and extend behavior from a base class. The child class IS-A type of the parent.

**Intermediate:** "What is the fragile base class problem?"
> When a base class changes its internal implementation — not its public API — derived classes can silently break because they depend on the internal behavior. For example, if a base class `AddRange` calls `Add` internally and a subclass overrides `Add` to count items, a base class change to not call `Add` breaks the count without any compiler warning.

**Senior:** "When would you prefer inheritance over composition?"
> Inheritance is appropriate when: (1) there's a true IS-A relationship, (2) you need polymorphism through a base type, (3) you're implementing Template Method pattern, (4) you own the full hierarchy and can control changes. Composition is better when you want behavior reuse without IS-A semantics or when you need runtime flexibility.

**Tricky:** "Why is calling virtual methods from constructors dangerous?"
> Because object construction in C# is bottom-up: base constructor runs before the derived class constructor. When the base constructor calls a virtual method, the derived class's override runs — but the derived class fields haven't been initialized yet. This leads to NullReferenceExceptions or corrupted state.

---

## Polymorphism

### 1. Definition

Polymorphism means "many forms" — the ability of a single interface/method to work with objects of different types. In C#: compile-time (overloading) and runtime (overriding).

### 2. Compile-Time Polymorphism (Overloading)

```csharp
public class PaymentCalculator
{
    // Same method name, different signatures — resolved at COMPILE TIME
    public decimal Calculate(Order order) => order.Total;
    
    public decimal Calculate(Order order, DiscountCode discount)
        => order.Total * (1 - discount.DiscountPercent / 100m);
    
    public decimal Calculate(Order order, DiscountCode discount, bool includeTax)
        => includeTax 
            ? Calculate(order, discount) * 1.18m 
            : Calculate(order, discount);
}
```

### 3. Runtime Polymorphism — `virtual` / `override` / `new`

```csharp
public class PaymentGateway
{
    public virtual string GetGatewayName() => "Generic Gateway";
    public virtual async Task<PaymentResult> ProcessAsync(PaymentRequest req) 
        => throw new NotImplementedException();
}

public class StripeGateway : PaymentGateway
{
    public override string GetGatewayName() => "Stripe"; // OVERRIDES — polymorphic
    public override async Task<PaymentResult> ProcessAsync(PaymentRequest req)
    {
        // Runtime dispatch — this is called when variable is PaymentGateway
        // but actual object is StripeGateway
    }
}

public class PayPalGateway : PaymentGateway
{
    // THE INTERVIEW TRAP: `new` keyword HIDES instead of OVERRIDES
    public new string GetGatewayName() => "PayPal"; // NOT polymorphic!
}

// Demonstration:
PaymentGateway gateway = new PayPalGateway();
Console.WriteLine(gateway.GetGatewayName()); // Prints "Generic Gateway" ← SURPRISE!
// Because `new` breaks the virtual dispatch chain

PayPalGateway concrete = new PayPalGateway();
Console.WriteLine(concrete.GetGatewayName()); // Prints "PayPal" — variable type matters
```

### 4. How vtable Works in CLR

Every class with virtual methods gets a **Virtual Method Table (vtable)** — a data structure in memory that maps virtual method slots to actual method addresses.

```
PaymentGateway vtable:
  [0] → PaymentGateway.GetGatewayName  (0x0A0A)
  [1] → PaymentGateway.ProcessAsync     (0x0B0B)

StripeGateway vtable (inherits + overrides):
  [0] → StripeGateway.GetGatewayName   (0x1A1A) ← replaced
  [1] → StripeGateway.ProcessAsync      (0x1B1B) ← replaced

PayPalGateway vtable (hides with `new`):
  [0] → PaymentGateway.GetGatewayName  (0x0A0A) ← NOT replaced!
  [1] → PaymentGateway.ProcessAsync     (0x0B0B) ← NOT replaced!
  + PayPalGateway has its own separate slot for the `new` method
```

When you call through a `PaymentGateway` reference, CLR looks up slot [0] in the vtable. For `PayPalGateway`, slot [0] still points to `PaymentGateway.GetGatewayName` because `new` didn't replace it.

### 5. Abstract Methods

```csharp
public abstract class NotificationSender
{
    // Abstract: no implementation, MUST be overridden
    public abstract Task SendAsync(Notification notification);
    
    // Not abstract: has default implementation, CAN be overridden
    public virtual bool CanSend(Notification notification) => true;
    
    // Not virtual: cannot be overridden
    public void LogNotification(Notification notification)
        => Console.WriteLine($"Sending notification {notification.Id}");
}

public class EmailNotificationSender : NotificationSender
{
    public override async Task SendAsync(Notification notification)
    {
        // Must implement — compiler error if missing
    }
}
```

### 6. Production: Multiple Payment Gateway Implementations

```csharp
// Interface-based polymorphism (preferred over class inheritance for this case)
public interface IPaymentGateway
{
    string GatewayName { get; }
    bool Supports(string currency);
    Task<PaymentResult> ProcessAsync(PaymentRequest request, CancellationToken ct = default);
    Task<RefundResult> RefundAsync(string transactionId, decimal amount, CancellationToken ct = default);
}

public class StripeGateway : IPaymentGateway
{
    public string GatewayName => "Stripe";
    public bool Supports(string currency) => new[] { "USD", "EUR", "GBP", "INR" }.Contains(currency);
    
    public async Task<PaymentResult> ProcessAsync(PaymentRequest request, CancellationToken ct = default)
    {
        // Stripe API call
    }
    
    public async Task<RefundResult> RefundAsync(string transactionId, decimal amount, CancellationToken ct = default)
    {
        // Stripe refund API
    }
}

// Runtime polymorphism in action:
public class PaymentService
{
    private readonly IEnumerable<IPaymentGateway> _gateways;
    
    public PaymentService(IEnumerable<IPaymentGateway> gateways) => _gateways = gateways;
    
    public async Task<PaymentResult> ProcessAsync(PaymentRequest request)
    {
        // Runtime dispatch — correct gateway selected based on currency
        var gateway = _gateways.FirstOrDefault(g => g.Supports(request.Currency))
            ?? throw new InvalidOperationException($"No gateway supports {request.Currency}");
        
        return await gateway.ProcessAsync(request); // Polymorphic call
    }
}
```

### 7. Interview Prep

**Basic:** "What is polymorphism?"
> Polymorphism allows objects of different types to be treated as the same base type, with each type providing its own implementation. At runtime, the correct method is called based on the actual object type.

**Intermediate:** "What's the difference between `override` and `new` in C#?"
> `override` participates in virtual dispatch — when called through a base class reference, the derived class's method runs. `new` hides the base method without replacing it in the vtable, so calling through a base class reference still calls the base method. `override` is almost always what you want.

**Senior:** "How does the CLR resolve virtual method calls?"
> The CLR maintains a vtable per type. Each virtual method slot maps to a method address. When a virtual call is made, CLR looks up the object's actual type, finds its vtable, and dispatches to the address in the relevant slot. `override` replaces the slot; `new` adds a new slot without replacing. This happens at IL level via `callvirt` instruction.

**Tricky:** "Can you have polymorphism without inheritance?"
> Yes — through interfaces. Interface-based polymorphism is preferred over class inheritance for behavioral substitution (SOLID: DIP). Also, C# generics with constraints provide compile-time polymorphism. Duck typing (via dynamic) is another form.

---

## Abstraction

### 1. Definition

Abstraction means exposing **what** an object does (its interface/capabilities) while hiding **how** it does it (its implementation). It's about managing complexity by showing only relevant details.

### 2. How Interfaces Provide Abstraction

```csharp
// The abstraction: clients know WHAT but not HOW
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(Guid orderId);
    Task<IReadOnlyList<Order>> GetByCustomerAsync(string customerId);
    Task SaveAsync(Order order);
    Task DeleteAsync(Guid orderId);
}

// The reality hidden behind abstraction:
public class SqlOrderRepository : IOrderRepository
{
    private readonly OrderDbContext _db;
    
    public async Task<Order?> GetByIdAsync(Guid orderId)
        => await _db.Orders
            .Include(o => o.Items)
            .Include(o => o.ShippingAddress)
            .FirstOrDefaultAsync(o => o.Id == orderId);
    
    // SQL complexity hidden — consumer doesn't know or care
}

public class CachedOrderRepository : IOrderRepository
{
    private readonly IOrderRepository _inner;
    private readonly IDistributedCache _cache;
    
    public async Task<Order?> GetByIdAsync(Guid orderId)
    {
        var cacheKey = $"order:{orderId}";
        var cached = await _cache.GetAsync(cacheKey);
        if (cached != null) return Deserialize<Order>(cached);
        
        var order = await _inner.GetByIdAsync(orderId);
        if (order != null) await _cache.SetAsync(cacheKey, Serialize(order), _cacheOptions);
        return order;
    }
}
```

### 3. How Abstract Classes Provide Abstraction

```csharp
public abstract class ReportGenerator
{
    // What the report generator DOES (abstraction)
    public async Task<Report> GenerateAsync(ReportRequest request)
    {
        var data = await FetchDataAsync(request);
        var processed = ProcessData(data);
        return FormatReport(processed);
    }
    
    // HOW is abstracted — subclass provides the details
    protected abstract Task<RawData> FetchDataAsync(ReportRequest request);
    protected abstract ProcessedData ProcessData(RawData data);
    protected abstract Report FormatReport(ProcessedData data);
}

public class SalesReportGenerator : ReportGenerator
{
    protected override async Task<RawData> FetchDataAsync(ReportRequest request)
    {
        // Specific data fetching for sales report
    }
    // etc.
}
```

### 4. Interface vs Abstract Class for Abstraction

| When to use Interface | When to use Abstract Class |
|---|---|
| Multiple implementations possible | Shared implementation to inherit |
| Multiple "contracts" (roles) needed | Single inheritance hierarchy |
| DI/testability boundaries | Template method pattern |
| No shared state needed | Protected shared state needed |
| You don't own all implementors | You control the hierarchy |

### 5. Interview Prep

**Tricky:** "Is abstraction about hiding implementation or reducing complexity?"
> Both. Abstraction reduces cognitive load (you interact with `IPaymentGateway.ProcessAsync`, not Stripe's REST API internals). And it hides implementation so it can change without affecting clients. These are two sides of the same coin.

---

# INTERFACE vs ABSTRACT CLASS — COMPLETE TREATMENT

> This is the **most asked OOP interview question**. Know this cold.

## The Full Comparison Table

| Feature | Interface | Abstract Class |
|---|---|---|
| Inheritance | Multiple interfaces allowed | Single class only |
| State | No instance state (until C#8 static) | Can have fields, properties |
| Constructor | None | Yes (called via base()) |
| Access modifiers | All public (by default pre-C#8) | Any modifier |
| Implementation | Default methods only (C#8+) | Any methods |
| Versioning | Adding member = breaking change* | Can add non-abstract members |
| Purpose | Contract/role | Partial implementation + extension |

*Before default interface methods in C# 8.

## When EXACTLY to Use Interface

### 1. Multiple Implementations of Same Contract

```csharp
// Many payment gateways, many notification channels, many repositories
// All follow the same CONTRACT but have completely different implementations
public interface INotificationChannel
{
    bool CanHandle(NotificationType type);
    Task SendAsync(Notification notification, CancellationToken ct = default);
}

// Each implementation has different dependencies, different logic
public class EmailChannel : INotificationChannel { ... }
public class SmsChannel : INotificationChannel { ... }
public class PushChannel : INotificationChannel { ... }
public class SlackChannel : INotificationChannel { ... }
```

### 2. Dependency Injection Boundaries

```csharp
// NEVER inject concrete classes — inject interfaces
// Allows swapping implementations without changing consumers

// In Program.cs / Startup.cs:
builder.Services.AddScoped<IOrderRepository, SqlOrderRepository>();
// Swap to: builder.Services.AddScoped<IOrderRepository, MongoOrderRepository>();
// Zero changes to OrderService!
```

### 3. Testing (Mock Boundaries)

```csharp
// Interface makes mocking trivial
public class OrderServiceTests
{
    [Fact]
    public async Task PlaceOrder_SufficientStock_ReturnsSuccess()
    {
        // Arrange
        var mockRepo = new Mock<IOrderRepository>();
        var mockPayment = new Mock<IPaymentGateway>();
        var mockInventory = new Mock<IInventoryService>();
        
        mockInventory.Setup(i => i.IsAvailableAsync(It.IsAny<string>(), It.IsAny<int>()))
            .ReturnsAsync(true);
        mockPayment.Setup(p => p.ProcessAsync(It.IsAny<PaymentRequest>(), default))
            .ReturnsAsync(PaymentResult.Success("txn_123"));
        
        var service = new OrderService(mockRepo.Object, mockPayment.Object, mockInventory.Object);
        
        // Act & Assert
        var result = await service.PlaceOrderAsync(new PlaceOrderRequest { ... });
        Assert.True(result.IsSuccess);
    }
}
```

### 4. When a Type Plays Multiple Roles

```csharp
// One class can implement multiple interfaces — multiple roles
public class OrderProcessor : IOrderProcessor, IOrderStatusUpdater, IHealthCheck
{
    // Implements all three contracts
}
// You can't do this with abstract classes!
```

## When EXACTLY to Use Abstract Class

### 1. Template Method Pattern (Most Common)

```csharp
// When you have an algorithm skeleton with customizable steps
public abstract class DataImporter
{
    // Template method — the algorithm
    public async Task ImportAsync(Stream dataStream)
    {
        using var reader = CreateReader(dataStream);
        
        await BeforeImportAsync();
        
        int rowsImported = 0;
        await foreach (var record in reader.ReadAllAsync())
        {
            if (Validate(record))
            {
                var entity = Transform(record);
                await PersistAsync(entity);
                rowsImported++;
            }
        }
        
        await AfterImportAsync(rowsImported);
    }
    
    protected abstract IDataReader CreateReader(Stream stream);
    protected abstract bool Validate(RawRecord record);
    protected abstract T Transform(RawRecord record);
    protected abstract Task PersistAsync(T entity);
    
    // Optional hooks with defaults
    protected virtual Task BeforeImportAsync() => Task.CompletedTask;
    protected virtual Task AfterImportAsync(int rowsImported) => Task.CompletedTask;
}
```

### 2. Shared Implementation with Protected State

```csharp
public abstract class CachedRepository<T>
{
    private readonly IMemoryCache _cache; // Shared state — impossible with interface
    protected readonly ILogger Logger;
    
    protected CachedRepository(IMemoryCache cache, ILogger logger)
    {
        _cache = cache;
        Logger = logger;
    }
    
    protected async Task<T?> GetCachedAsync(string key, Func<Task<T?>> fetch)
    {
        if (_cache.TryGetValue(key, out T? cached)) return cached;
        
        var value = await fetch();
        if (value != null) _cache.Set(key, value, TimeSpan.FromMinutes(5));
        return value;
    }
    
    // Subclasses use shared cache infrastructure
    protected abstract Task<T?> GetFromDatabaseAsync(Guid id);
}
```

## Default Interface Methods (C# 8+) — Blurring the Line

```csharp
public interface IPaymentGateway
{
    Task<PaymentResult> ProcessAsync(PaymentRequest request);
    
    // Default implementation — C# 8+
    // Existing implementors don't need to implement this
    async Task<PaymentResult> ProcessWithRetryAsync(PaymentRequest request, int maxRetries = 3)
    {
        for (int attempt = 1; attempt <= maxRetries; attempt++)
        {
            try
            {
                return await ProcessAsync(request);
            }
            catch (TransientException) when (attempt < maxRetries)
            {
                await Task.Delay(TimeSpan.FromSeconds(Math.Pow(2, attempt)));
            }
        }
        return await ProcessAsync(request); // Last attempt, let exception propagate
    }
}
```

**Why this blurs the line:** You can now add methods to interfaces without breaking implementors. But: default interface methods are NOT inherited by implementing classes — you must cast to the interface to call them. They're primarily for API evolution, not design.

## The Interface Trap: Fat Interfaces (ISP Violation)

```csharp
// FAT INTERFACE — violates Interface Segregation
public interface IOrderService
{
    Task<Order> GetOrderAsync(Guid id);
    Task<PagedResult<Order>> GetAllOrdersAsync(PagingParams paging);
    Task<Order> CreateOrderAsync(CreateOrderRequest request);
    Task UpdateOrderAsync(Guid id, UpdateOrderRequest request);
    Task CancelOrderAsync(Guid id, string reason);
    Task<ShippingInfo> GetShippingInfoAsync(Guid id);
    Task UpdateShippingAsync(Guid id, ShippingInfo info);
    Task<Invoice> GenerateInvoiceAsync(Guid id);
    Task<PaymentStatus> GetPaymentStatusAsync(Guid id);
    Task ProcessPaymentAsync(Guid id, PaymentRequest payment);
    // ... 10 more methods
}
// Classes implementing this MUST implement ALL methods, even irrelevant ones
```

## The Abstract Class Trap: Forcing Inheritance

```csharp
// BAD: Using inheritance just for code reuse
public abstract class BaseService
{
    protected ILogger Logger { get; }
    protected IConfiguration Config { get; }
    
    protected BaseService(ILogger logger, IConfiguration config) { ... }
    
    // Everyone inherits just for Logger and Config
}

// This forces all services into an inheritance chain
// Better: inject Logger and Config via constructor
// Or: use composition with a shared ServiceContext
```

## Interview Prep — Interface vs Abstract Class

**Basic:** "When do you use interface vs abstract class?"
> Interface when defining a contract that multiple unrelated classes will implement, especially for DI and testability. Abstract class when you have shared implementation and want to define an algorithm skeleton that subclasses customize.

**Intermediate:** "Can you implement multiple interfaces? Multiple abstract classes?"
> Multiple interfaces: yes. Multiple abstract classes: no — C# supports single class inheritance. This is why interfaces are preferred for defining roles/contracts.

**Senior:** "How do default interface methods in C# 8 change the interface vs abstract class decision?"
> Default interface methods allow interfaces to evolve without breaking implementors, similar to abstract classes with non-abstract methods. But they don't support state or constructors. The line blurs for API evolution, but for template method pattern requiring protected state, abstract classes are still the right choice.

**Tricky:** "If interfaces are so great, why does C# have abstract classes?"
> Because some patterns genuinely need protected state, constructors, and the template method pattern. Also, single inheritance of classes (vs. multiple interfaces) can be a feature — it means there's one authoritative place for shared implementation, making inheritance trees easier to understand. Abstract classes say "these things ARE the same kind of thing." Interfaces say "these things CAN DO the same things."

---

# COMPOSITION vs INHERITANCE

## "Favor Composition Over Inheritance" — Why

**Gang of Four, Design Patterns (1994):** *"Favor object composition over class inheritance."* This was a reaction to abuse of deep inheritance hierarchies.

**Composition:** Build complex behavior by combining simpler objects.
**Inheritance:** Build complex behavior by extending existing objects.

### When Composition Wins

```csharp
// INHERITANCE APPROACH: Problems
public class Order { }
public class DiscountedOrder : Order { } // IS-A? Sometimes...
public class InternationalOrder : Order { } // IS-A? Sometimes...
public class DiscountedInternationalOrder : DiscountedOrder { } // Explosion!
// What if you have 5 variations? 2^5 = 32 subclasses!

// COMPOSITION APPROACH: Flexible, combinable
public class Order
{
    private readonly IDiscountStrategy _discountStrategy;
    private readonly IShippingCalculator _shippingCalculator;
    private readonly ITaxCalculator _taxCalculator;
    
    public Order(
        IDiscountStrategy discountStrategy,
        IShippingCalculator shippingCalculator,
        ITaxCalculator taxCalculator)
    {
        _discountStrategy = discountStrategy;
        _shippingCalculator = shippingCalculator;
        _taxCalculator = taxCalculator;
    }
    
    public decimal CalculateTotal(IEnumerable<OrderItem> items)
    {
        var subtotal = items.Sum(i => i.Price * i.Quantity);
        var discount = _discountStrategy.Calculate(subtotal);
        var shipping = _shippingCalculator.Calculate(items, subtotal - discount);
        var tax = _taxCalculator.Calculate(subtotal - discount + shipping);
        return subtotal - discount + shipping + tax;
    }
}

// At runtime, compose any combination:
var order = new Order(
    new VolumeDiscountStrategy(threshold: 1000),
    new InternationalShippingCalculator("IN"),
    new GSTaxCalculator(rate: 0.18m)
);
```

### When Inheritance Is Legitimate

1. **True IS-A relationships** that won't change (`Animal → Dog`, `Shape → Circle`)
2. **Template Method Pattern** where algorithm skeleton is shared
3. **Framework extension points** (`Controller`, `BackgroundService`, `DbContext`)
4. **When you control the entire hierarchy**

### The Decorator Pattern as Composition

```csharp
// Adding behavior without inheritance
public interface IOrderRepository { Task<Order?> GetByIdAsync(Guid id); }

public class SqlOrderRepository : IOrderRepository { ... }

// Adds caching WITHOUT inheriting from SqlOrderRepository
public class CachingOrderRepository : IOrderRepository
{
    private readonly IOrderRepository _inner; // COMPOSITION
    private readonly IDistributedCache _cache;
    
    public CachingOrderRepository(IOrderRepository inner, IDistributedCache cache)
    {
        _inner = inner;
        _cache = cache;
    }
    
    public async Task<Order?> GetByIdAsync(Guid id)
    {
        var key = $"order:{id}";
        // Try cache first, then delegate to inner
        ...
    }
}
```

---

# SOLID PRINCIPLES

## S — Single Responsibility Principle

### 1. Definition

**"A class should have only one reason to change."** — Robert C. Martin

One *reason to change* means one *actor* (stakeholder) who can demand the class changes.

### 2. The Bad Code

```csharp
// VIOLATION: OrderService does EVERYTHING
public class OrderService
{
    private readonly SqlConnection _db;
    private readonly SmtpClient _smtp;
    private readonly ILogger _logger;
    
    public async Task<bool> PlaceOrderAsync(PlaceOrderRequest request)
    {
        // 1. VALIDATION — Reason to change: Business rules change
        if (request.Items == null || !request.Items.Any())
            return false;
        if (request.TotalAmount > 100000)
            return false; // Hardcoded business rule!
        
        // 2. DATABASE ACCESS — Reason to change: DB schema changes
        var order = new Order { ... };
        await _db.ExecuteAsync(
            "INSERT INTO Orders (Id, CustomerId, Total) VALUES (@Id, @CustomerId, @Total)",
            order);
        
        // 3. EMAIL NOTIFICATION — Reason to change: Email template changes
        var message = new MailMessage();
        message.To.Add(request.CustomerEmail);
        message.Subject = "Order Confirmed";
        message.Body = $"Your order {order.Id} has been confirmed.";
        await _smtp.SendMailAsync(message);
        
        // 4. LOGGING — Reason to change: Log format changes
        _logger.LogInformation($"[{DateTime.Now}] Order placed: {order.Id} for {request.CustomerEmail}");
        
        // 5. REPORTING — Reason to change: Report format changes
        File.AppendAllText("orders.csv", $"{order.Id},{request.TotalAmount}\n");
        
        return true;
    }
}
// This class has 5+ reasons to change. One change ripples through everything.
```

### 3. The Good Code

```csharp
// Each class: ONE responsibility, ONE reason to change

// Responsibility 1: Order validation
public class OrderValidator
{
    private readonly IEnumerable<IOrderValidationRule> _rules;
    
    public OrderValidator(IEnumerable<IOrderValidationRule> rules) => _rules = rules;
    
    public ValidationResult Validate(PlaceOrderRequest request)
    {
        var errors = _rules
            .Where(r => !r.IsSatisfiedBy(request))
            .Select(r => r.ErrorMessage)
            .ToList();
        
        return errors.Any() 
            ? ValidationResult.Failure(errors) 
            : ValidationResult.Success();
    }
}

// Responsibility 2: Order persistence
public class OrderRepository : IOrderRepository
{
    private readonly OrderDbContext _db;
    
    public async Task SaveAsync(Order order)
        => await _db.Orders.AddAsync(order);
}

// Responsibility 3: Order notification
public class OrderNotificationService : IOrderNotificationService
{
    private readonly IEnumerable<INotificationChannel> _channels;
    
    public async Task NotifyOrderPlacedAsync(Order order, string customerEmail)
    {
        foreach (var channel in _channels)
            await channel.SendAsync(new OrderPlacedNotification(order, customerEmail));
    }
}

// Responsibility 4: Orchestration (the only job of OrderService now)
public class OrderService : IOrderService
{
    private readonly IOrderValidator _validator;
    private readonly IOrderRepository _repository;
    private readonly IOrderNotificationService _notifications;
    private readonly ILogger<OrderService> _logger;
    
    public OrderService(
        IOrderValidator validator,
        IOrderRepository repository,
        IOrderNotificationService notifications,
        ILogger<OrderService> logger)
    {
        _validator = validator;
        _repository = repository;
        _notifications = notifications;
        _logger = logger;
    }
    
    public async Task<OrderResult> PlaceOrderAsync(PlaceOrderRequest request)
    {
        var validation = _validator.Validate(request);
        if (!validation.IsValid)
            return OrderResult.Failed(validation.Errors);
        
        var order = Order.Create(request);
        await _repository.SaveAsync(order);
        
        await _notifications.NotifyOrderPlacedAsync(order, request.CustomerEmail);
        
        _logger.LogInformation("Order {OrderId} placed successfully", order.Id);
        
        return OrderResult.Success(order.Id);
    }
}
```

### 4. When to Pragmatically Violate SRP

- In small scripts/utilities where complexity isn't worth the abstraction
- In CRUD controllers with simple validation (splitting is over-engineering)
- When performance matters more than purity (object creation has cost)

### 5. Interview Prep

**Tricky Q:** "Is SRP 'one reason to change' or 'one responsibility'?"
> These sound similar but are different. "One responsibility" is ambiguous — what counts as one? "One reason to change" is precise — it means one *actor* (business stakeholder) who can demand changes. A `ReportGenerator` has one reason to change: the reporting requirements. When you hear "one responsibility," think: who would ask you to change this? If the answer is "my DBA AND my marketing team," SRP is violated.

---

## O — Open/Closed Principle

### 1. Definition

**"Software entities should be open for extension, closed for modification."** — Bertrand Meyer, Robert C. Martin

Open for extension: add new behavior. Closed for modification: don't change existing tested code.

### 2. The Bad Code

```csharp
// VIOLATION: Every new payment type requires modifying this code
public class PaymentProcessor
{
    public PaymentResult Process(PaymentRequest request)
    {
        switch (request.PaymentType)
        {
            case "CreditCard":
                // Credit card processing logic
                return ProcessCreditCard(request);
            
            case "PayPal":
                // PayPal logic
                return ProcessPayPal(request);
            
            case "Stripe":
                // Stripe logic
                return ProcessStripe(request);
            
            // Adding Razorpay? MODIFY this switch. All tests rerun. Risk of regression.
            default:
                throw new NotSupportedException($"Payment type {request.PaymentType} not supported");
        }
    }
}
```

### 3. The Good Code

```csharp
// GOOD: Strategy pattern — extend by adding new class, no modification
public interface IPaymentStrategy
{
    bool CanHandle(string paymentType);
    Task<PaymentResult> ProcessAsync(PaymentRequest request);
}

public class CreditCardPaymentStrategy : IPaymentStrategy
{
    public bool CanHandle(string paymentType) => paymentType == "CreditCard";
    
    public async Task<PaymentResult> ProcessAsync(PaymentRequest request)
    {
        // Credit card logic — isolated, tested, stable
    }
}

public class StripePaymentStrategy : IPaymentStrategy
{
    public bool CanHandle(string paymentType) => paymentType == "Stripe";
    
    public async Task<PaymentResult> ProcessAsync(PaymentRequest request)
    {
        // Stripe logic
    }
}

// Adding Razorpay: Create RazorpayPaymentStrategy, register in DI. DONE.
// Zero changes to existing code.

public class PaymentProcessor
{
    private readonly IEnumerable<IPaymentStrategy> _strategies;
    
    public PaymentProcessor(IEnumerable<IPaymentStrategy> strategies) 
        => _strategies = strategies;
    
    public async Task<PaymentResult> ProcessAsync(PaymentRequest request)
    {
        var strategy = _strategies.FirstOrDefault(s => s.CanHandle(request.PaymentType))
            ?? throw new NotSupportedException($"Payment type {request.PaymentType} not supported");
        
        return await strategy.ProcessAsync(request);
    }
}

// DI registration:
builder.Services.AddScoped<IPaymentStrategy, CreditCardPaymentStrategy>();
builder.Services.AddScoped<IPaymentStrategy, StripePaymentStrategy>();
builder.Services.AddScoped<IPaymentStrategy, RazorpayPaymentStrategy>(); // NEW — no old code changed
builder.Services.AddScoped<PaymentProcessor>();
```

### 4. When OCP Is NOT Worth It (Tricky Interview Q)

```csharp
// When you have 2 known, stable cases that will NEVER change
// OCP creates unnecessary abstraction
public decimal CalculateShipping(string method) => method switch
{
    "Standard" => 5.99m,
    "Express" => 14.99m,
    _ => throw new ArgumentException()
};
// If this will NEVER need a third case, OCP abstraction is over-engineering
```

**Rule:** Apply OCP at points of **predicted variability**. Don't abstract everything speculatively.

### 5. Interview Prep

**Tricky Q:** "Can you give an example where OCP is NOT worth it?"
> When the variation is already complete and bounded. If I have a `TaxCalculator` that handles exactly two known tax zones and business has confirmed there will never be a third, adding a strategy pattern for OCP adds indirection without benefit. OCP is most valuable at architectural seams — payment gateways, notification channels, report formats — where extensions are expected and frequent.

---

## L — Liskov Substitution Principle

### 1. Definition

**"Subtypes must be substitutable for their base types without altering the correctness of the program."** — Barbara Liskov, 1987

If you have `PaymentProcessor processor = new StripeProcessor()`, the program should work correctly treating it as `PaymentProcessor`.

### 2. The Classic Violation (Rectangle/Square)

```csharp
public class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }
    public int Area() => Width * Height;
}

public class Square : Rectangle
{
    public override int Width
    {
        get => base.Width;
        set { base.Width = value; base.Height = value; } // Enforces square rule
    }
    public override int Height
    {
        get => base.Height;
        set { base.Width = value; base.Height = value; }
    }
}

// This breaks LSP:
Rectangle rect = new Square();
rect.Width = 4;
rect.Height = 5;
Console.WriteLine(rect.Area()); // Expected: 20, Actual: 25 ← LSP violated!
// Substituting Square for Rectangle breaks the program
```

### 3. Enterprise Equivalent: Employee Hierarchy

```csharp
// BAD: LSP violation in HR system
public class Employee
{
    public virtual decimal CalculateBonus() => Salary * 0.10m;
    public virtual int GetMaxVacationDays() => 20;
    public decimal Salary { get; set; }
}

public class ContractEmployee : Employee
{
    public override decimal CalculateBonus()
        => throw new InvalidOperationException("Contract employees don't get bonuses!");
    
    public override int GetMaxVacationDays()
        => throw new InvalidOperationException("Contract employees don't get vacation!");
}

// Client code BREAKS when it gets a ContractEmployee instead of Employee
void ProcessBonuses(IEnumerable<Employee> employees)
{
    foreach (var emp in employees)
        ProcessBonus(emp.CalculateBonus()); // THROWS for contract employees!
}
```

### 4. The Good Redesign

```csharp
// GOOD: Redesign the hierarchy to honor LSP
public abstract class Employee
{
    public string Id { get; }
    public string Name { get; }
}

public abstract class PermanentEmployee : Employee
{
    public decimal Salary { get; }
    public virtual decimal CalculateBonus() => Salary * 0.10m;
    public virtual int GetMaxVacationDays() => 20;
}

public class ContractEmployee : Employee
{
    public decimal HourlyRate { get; }
    public int HoursWorked { get; }
    public decimal CalculateEarnings() => HourlyRate * HoursWorked; // Different concept entirely
}

// Now ContractEmployee doesn't inherit vacation/bonus — no LSP violation
// Client code is explicit about what type it needs
void ProcessBonuses(IEnumerable<PermanentEmployee> employees)
{
    foreach (var emp in employees)
        ProcessBonus(emp.CalculateBonus()); // Safe!
}
```

### 5. LSP Rules Summary

- **Preconditions** cannot be strengthened in a subtype
- **Postconditions** cannot be weakened in a subtype
- **Invariants** of the supertype must be preserved
- Methods should not throw unexpected exceptions

### 6. Interview Prep

**Tricky Q:** "When have you VIOLATED LSP in production?"
> Great answer: "We had a `DataExporter` base class with a `Export(DataSet data)` method. A `LargeDataExporter` subclass added a restriction that DataSet must have fewer than 100,000 rows — throwing an exception otherwise. This strengthened the precondition and violated LSP. Client code that worked fine with `DataExporter` broke when given `LargeDataExporter`. We fixed it by having `LargeDataExporter` implement a different interface `IBatchedExporter` with explicit batching semantics."

---

## I — Interface Segregation Principle

### 1. Definition

**"No client should be forced to depend on methods it does not use."** — Robert C. Martin

### 2. The Bad Code

```csharp
// FAT INTERFACE — forces implementing classes to support irrelevant operations
public interface IOrderService
{
    Task<Order> GetOrderAsync(Guid id);
    Task<PagedResult<Order>> SearchOrdersAsync(OrderSearchParams search);
    Task<Order> CreateOrderAsync(CreateOrderRequest request);
    Task<Order> UpdateOrderAsync(Guid id, UpdateOrderRequest request);
    Task CancelOrderAsync(Guid id, string reason);
    Task<ShippingInfo> GetShippingAsync(Guid id);
    Task UpdateShippingAsync(Guid id, ShippingInfo info);
    Task<Invoice> GenerateInvoiceAsync(Guid id);
    Task<PaymentStatus> GetPaymentStatusAsync(Guid id);
    Task RetryPaymentAsync(Guid id);
    Task<OrderStats> GetStatisticsAsync(DateRange range);
    Task ExportOrdersAsync(ExportFormat format, Stream output);
}

// Warehouse system only needs GetOrder + UpdateShipping:
public class WarehouseOrderService : IOrderService
{
    public Task<Order> GetOrderAsync(Guid id) => ...; // IMPLEMENTS
    public Task UpdateShippingAsync(Guid id, ShippingInfo info) => ...; // IMPLEMENTS
    
    // Forced to implement irrelevant methods:
    public Task<Order> CreateOrderAsync(CreateOrderRequest request) 
        => throw new NotImplementedException(); // This IS an LSP violation too!
    // ... 10 more throw NotImplementedException
}
```

### 3. The Good Code — Role-Based Interfaces

```csharp
// Segregated interfaces by CLIENT ROLE
public interface IOrderReader
{
    Task<Order?> GetOrderAsync(Guid id);
    Task<PagedResult<Order>> SearchOrdersAsync(OrderSearchParams search);
}

public interface IOrderWriter
{
    Task<Order> CreateOrderAsync(CreateOrderRequest request);
    Task<Order> UpdateOrderAsync(Guid id, UpdateOrderRequest request);
}

public interface IOrderCancellation
{
    Task CancelOrderAsync(Guid id, string reason);
}

public interface IOrderShipping
{
    Task<ShippingInfo> GetShippingAsync(Guid id);
    Task UpdateShippingAsync(Guid id, ShippingInfo info);
}

public interface IOrderPayment
{
    Task<PaymentStatus> GetPaymentStatusAsync(Guid id);
    Task RetryPaymentAsync(Guid id);
}

public interface IOrderReporting
{
    Task<OrderStats> GetStatisticsAsync(DateRange range);
    Task ExportOrdersAsync(ExportFormat format, Stream output);
}

// Full implementation (one class can implement multiple interfaces):
public class OrderService : IOrderReader, IOrderWriter, IOrderCancellation, 
    IOrderShipping, IOrderPayment, IOrderReporting
{
    // Implements everything — but organized by role
}

// Warehouse service only needs its interfaces:
public class WarehouseIntegrationService
{
    private readonly IOrderReader _orderReader;
    private readonly IOrderShipping _orderShipping;
    
    // Constructor only accepts what it NEEDS — ISP satisfied
    public WarehouseIntegrationService(IOrderReader orderReader, IOrderShipping orderShipping)
    {
        _orderReader = orderReader;
        _orderShipping = orderShipping;
    }
}
```

### 4. Interview Prep

**Tricky Q:** "How many methods should an interface have?"
> There's no fixed number. The guidance is: as many methods as a single CLIENT ROLE needs, no more. A `IOrderReader` might have 5 methods if all 5 are needed by every read client. The rule isn't "one method per interface" (that leads to interface explosion). It's "don't force clients to depend on methods they don't use." In practice, I design interfaces from the consumer's perspective: what does this client actually need?

---

## D — Dependency Inversion Principle

### 1. Definition

**"High-level modules should not depend on low-level modules. Both should depend on abstractions."** — Robert C. Martin

High-level = business logic. Low-level = database, email, HTTP clients. Both should depend on *interfaces*.

### 2. The Bad Code

```csharp
// BAD: High-level OrderService directly depends on low-level SqlOrderRepository
public class OrderService
{
    // Direct instantiation — concrete coupling
    private readonly SqlOrderRepository _repository = new SqlOrderRepository(
        "Server=prod-db;Database=Orders;...");
    
    private readonly SmtpEmailSender _emailSender = new SmtpEmailSender(
        "smtp.company.com", 587);
    
    public async Task PlaceOrderAsync(PlaceOrderRequest request)
    {
        // OrderService must change if:
        // - You switch from SQL Server to PostgreSQL
        // - You switch from SMTP to SendGrid
        // - You want to unit test (can't mock concrete classes)
        var order = Order.Create(request);
        await _repository.SaveAsync(order);        // Coupled to SQL
        await _emailSender.SendAsync(request.Email, "Order confirmed"); // Coupled to SMTP
    }
}
```

### 3. The Good Code

```csharp
// GOOD: All dependencies are abstractions
public class OrderService : IOrderService
{
    private readonly IOrderRepository _repository;   // Abstraction
    private readonly IEmailSender _emailSender;      // Abstraction
    private readonly ILogger<OrderService> _logger;  // Abstraction
    
    // Constructor injection — dependencies are PUSHED IN
    public OrderService(
        IOrderRepository repository,
        IEmailSender emailSender,
        ILogger<OrderService> logger)
    {
        _repository = repository;
        _emailSender = emailSender;
        _logger = logger;
    }
    
    public async Task<OrderResult> PlaceOrderAsync(PlaceOrderRequest request)
    {
        var order = Order.Create(request);
        await _repository.SaveAsync(order);
        await _emailSender.SendAsync(request.CustomerEmail, "Order confirmed", $"Order {order.Id} confirmed");
        _logger.LogInformation("Order {OrderId} placed", order.Id);
        return OrderResult.Success(order.Id);
    }
}

// Registrations in DI container:
builder.Services.AddScoped<IOrderRepository, SqlOrderRepository>();    // Low-level
builder.Services.AddScoped<IEmailSender, SendGridEmailSender>();       // Low-level
builder.Services.AddScoped<IOrderService, OrderService>();             // High-level
```

---

# DEPENDENCY INJECTION — COMPLETE

## What DI Really Is

Dependency Injection is the technique of providing an object's dependencies from *outside* rather than letting the object create them. It implements the Dependency Inversion Principle and the Inversion of Control principle.

**IoC (Inversion of Control):** Control of object creation is inverted — the framework controls it, not your code.

## Constructor vs Property vs Method Injection

```csharp
// 1. CONSTRUCTOR INJECTION (PREFERRED)
public class OrderService
{
    private readonly IOrderRepository _repo;
    
    public OrderService(IOrderRepository repo)
    {
        _repo = repo ?? throw new ArgumentNullException(nameof(repo));
    }
    // Advantages: dependencies required (no null state), clear, testable
    // Disadvantage: many dependencies = messy constructor (SRP violation signal!)
}

// 2. PROPERTY INJECTION (rare, use sparingly)
public class OrderService
{
    public IOrderRepository Repository { get; set; } // Optional dependency
    // Disadvantage: nullable state possible, harder to enforce required deps
    // Use for: optional dependencies, circular dependencies (very rare)
}

// 3. METHOD INJECTION (contextual injection)
public class OrderProcessor
{
    public async Task ProcessAsync(Order order, IPaymentGateway gateway)
    {
        // Gateway injected per-call — not stored
        await gateway.ProcessAsync(new PaymentRequest { Amount = order.Total });
    }
    // Use for: dependencies that vary per operation, or are externally provided
}
```

## Why Constructor Injection Is Preferred

1. Dependencies are **required** — no null checks needed
2. Class is **never in invalid state** — dependencies ready at construction
3. **Testability** — easy to pass mocks in constructors
4. **Visibility** — large constructor signals SRP violation (too many responsibilities)

## Service Lifetimes

```csharp
// 1. SINGLETON: One instance per application lifetime
builder.Services.AddSingleton<IConnectionPool, SqlConnectionPool>();
// Use for: stateless services, caches, configuration, expensive-to-create objects
// DANGER: Singleton holds mutable state shared across all requests → thread safety issues

// 2. SCOPED: One instance per HTTP request (or scope)
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddScoped<OrderDbContext>(); // DbContext is always Scoped!
// Use for: database contexts, services that span one request, units of work

// 3. TRANSIENT: New instance every time it's requested
builder.Services.AddTransient<IPasswordHasher, BCryptPasswordHasher>();
// Use for: lightweight, stateless services, factory-like scenarios
```

## The Captive Dependency Problem

```csharp
// BUG: Singleton capturing a Scoped dependency
public class OrderCacheService // Registered as SINGLETON
{
    private readonly IOrderRepository _repo; // Registered as SCOPED
    
    public OrderCacheService(IOrderRepository repo) // SINGLETON captures SCOPED
    {
        _repo = repo; // This scoped instance is now HELD FOREVER by the singleton!
    }
    // Problem: The scoped repository (with its DbContext) lives for the entire app lifetime
    // DbContext accumulates changes, connections are leaked, data is stale
}

// FIX 1: Use IServiceScopeFactory to create scopes explicitly
public class OrderCacheService // SINGLETON
{
    private readonly IServiceScopeFactory _scopeFactory;
    
    public OrderCacheService(IServiceScopeFactory scopeFactory)
        => _scopeFactory = scopeFactory;
    
    public async Task<Order?> GetOrderAsync(Guid id)
    {
        // Create a new scope per operation — correct lifecycle management
        using var scope = _scopeFactory.CreateScope();
        var repo = scope.ServiceProvider.GetRequiredService<IOrderRepository>();
        return await repo.GetByIdAsync(id);
    }
}

// FIX 2: Make your cache service Scoped (if it doesn't need singleton lifetime)
// FIX 3: Make the dependency Singleton too (if it's truly stateless)
```

## .NET DI Container Internals (How It Resolves)

```csharp
// Registration builds a ServiceDescriptor graph
builder.Services.AddScoped<IOrderService, OrderService>();
// Creates: ServiceDescriptor { ServiceType=IOrderService, ImplementationType=OrderService, Lifetime=Scoped }

// At resolution time (when controller/handler is instantiated):
// 1. IServiceProvider.GetRequiredService<IOrderService>()
// 2. Container checks ServiceDescriptor for IOrderService → OrderService
// 3. Inspects OrderService constructor: (IOrderRepository, IEmailSender, ILogger<OrderService>)
// 4. Recursively resolves each parameter
// 5. Creates OrderService with resolved dependencies
// 6. For Scoped: stores in current scope dictionary → same instance returned within scope
// 7. For Singleton: stores in root container → same instance for app lifetime

// IServiceProvider vs IServiceScopeFactory:
IServiceProvider provider = ...; // Current scope's provider
// Use: GetService<T>() within current scope

IServiceScopeFactory factory = ...; // Creates NEW scopes
// Use: When you're in a singleton and need scoped services
using var scope = factory.CreateScope();
var service = scope.ServiceProvider.GetRequiredService<IOrderService>();
```

## Interview Prep — DI

**Basic:** "What is Dependency Injection?"
> DI is a technique where a class receives its dependencies from outside rather than creating them itself. This makes classes testable, configurable, and loosely coupled. In .NET, the built-in DI container manages object creation and lifetime.

**Intermediate:** "What is the Captive Dependency Problem?"
> It occurs when a longer-lived service (singleton) holds a reference to a shorter-lived service (scoped/transient). The short-lived service is "captured" and lives as long as the singleton — defeating its intended lifecycle. Fix: use `IServiceScopeFactory` to create explicit scopes in singletons.

**Senior:** "How does the .NET DI container resolve dependencies?"
> During registration, `ServiceDescriptor` objects are added to a `ServiceCollection`. At startup, `ServiceCollection.BuildServiceProvider()` compiles these into a `ServiceProvider`. When `GetRequiredService<T>()` is called, the provider finds the descriptor for T, uses reflection to examine the implementation type's constructor, recursively resolves each parameter, creates the instance, and manages its lifetime (scoped in a dictionary, singleton in root scope).

**Tricky:** "Is Dependency Injection the same as Inversion of Control?"
> DI is one implementation of IoC. IoC is the broader principle: control of something (object creation, program flow) is inverted from your code to the framework. DI specifically inverts control of dependency creation. Other IoC examples: event systems (framework calls your handlers), template method (framework calls your override), iterator pattern.

---

# SUMMARY: SOLID IN ONE TABLE

| Principle | Core Question | Symptom of Violation | Key Solution |
|---|---|---|---|
| SRP | "Who asks you to change this?" | God class, 500-line service | Separate concerns by actor |
| OCP | "Can I extend without modifying?" | Switch statements, if-else chains | Strategy + Polymorphism |
| LSP | "Is substitution safe?" | `throw NotImplementedException()` in subclass | Redesign hierarchy |
| ISP | "Do clients use all methods?" | Fat interfaces, forced implementations | Role-based interfaces |
| DIP | "Who controls dependencies?" | `new` in business logic, concrete coupling | Interfaces + DI |

---

*Volume 4 Complete — Proceed to Volume 5: Design Patterns*
