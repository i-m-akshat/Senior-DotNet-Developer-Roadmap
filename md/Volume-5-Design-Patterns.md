# THE COMPLETE .NET DEVELOPER ENCYCLOPEDIA + INTERVIEW MASTERY GUIDE
# Volume 5: Design Patterns

> **Audience:** .NET developer ~3 years experience. Knows pattern names, needs deep "why" and production context.
> **Goal:** Know not just WHAT each pattern is, but WHEN to use it and WHEN NOT TO.

---

# TABLE OF CONTENTS

1. [Creational Patterns](#creational-patterns)
   - [Singleton](#singleton-pattern)
   - [Factory Method](#factory-method-pattern)
   - [Abstract Factory](#abstract-factory-pattern)
   - [Builder](#builder-pattern)
2. [Structural Patterns](#structural-patterns)
   - [Repository](#repository-pattern)
   - [Decorator](#decorator-pattern)
   - [Adapter](#adapter-pattern)
   - [Facade](#facade-pattern)
3. [Behavioral Patterns](#behavioral-patterns)
   - [Strategy](#strategy-pattern)
   - [Observer](#observer-pattern)
   - [Command](#command-pattern)
   - [Mediator](#mediator-pattern)
   - [Chain of Responsibility](#chain-of-responsibility)
   - [Template Method](#template-method-pattern)
   - [State](#state-pattern)
4. [Observer vs Mediator vs Events — Comparison](#observer-vs-mediator-vs-events)

---

# CREATIONAL PATTERNS

> Creational patterns abstract the process of object creation, making systems independent of how objects are created, composed, and represented.

---

## Singleton Pattern

### 1. Definition

Ensures a class has **exactly one instance** and provides a **global point of access** to it.

### 2. Historical Reason

Originally invented for shared resources: connection pools, configuration stores, hardware device proxies. The idea: if only one of something should exist, enforce it structurally.

### 3. Problem It Solves

```csharp
// BAD: Static class — not testable, not DI-friendly, not mockable
public static class ConfigurationManager
{
    private static readonly Dictionary<string, string> _config = new();
    
    static ConfigurationManager()
    {
        // Loads from file at app start — no way to inject test config
        _config = File.ReadAllLines("appsettings.ini")
            .ToDictionary(l => l.Split('=')[0], l => l.Split('=')[1]);
    }
    
    public static string Get(string key) => _config.GetValueOrDefault(key, string.Empty);
}

// Cannot mock in tests. Cannot swap implementation. Global mutable state.
```

### 4. Thread-Safe Singleton — Double-Checked Locking with `Lazy<T>`

```csharp
// APPROACH 1: Lazy<T> — simplest, thread-safe, preferred
public sealed class ConnectionPool
{
    // Lazy<T> uses double-checked locking internally
    // Thread-safe, created only when first accessed, value never changes after
    private static readonly Lazy<ConnectionPool> _instance = 
        new(() => new ConnectionPool(), LazyThreadSafetyMode.ExecutionAndPublication);
    
    public static ConnectionPool Instance => _instance.Value;
    
    private ConnectionPool()
    {
        // Private constructor — external code cannot instantiate
        _maxConnections = 20;
        InitializePool();
    }
    
    private int _maxConnections;
    private readonly SemaphoreSlim _semaphore = new(20, 20);
    private readonly Queue<SqlConnection> _pool = new();
    
    public async Task<SqlConnection> AcquireAsync()
    {
        await _semaphore.WaitAsync();
        return _pool.TryDequeue(out var conn) ? conn : CreateConnection();
    }
    
    public void Release(SqlConnection conn)
    {
        _pool.Enqueue(conn);
        _semaphore.Release();
    }
    
    private void InitializePool()
    {
        for (int i = 0; i < _maxConnections; i++)
            _pool.Enqueue(CreateConnection());
    }
    
    private SqlConnection CreateConnection() => 
        new SqlConnection("Server=prod-db;Database=Orders;Integrated Security=true");
}

// APPROACH 2: DI-managed Singleton (PREFERRED in modern .NET)
// Let the DI container manage singleton lifetime — much better!
builder.Services.AddSingleton<IConnectionPool, SqlConnectionPool>();

// The DI container:
// - Creates ONE instance for the app lifetime
// - Handles thread safety during creation
// - Allows interface abstraction (mockable!)
// - No global state anti-pattern
```

### 5. Production Example: Distributed Cache

```csharp
// In .NET, singletons via DI are standard for:
// 1. IMemoryCache — one cache per process
// 2. HttpClient (via IHttpClientFactory) — manages connection pools
// 3. IConfiguration — app config loaded once
// 4. Logging providers

builder.Services.AddSingleton<IMemoryCache, MemoryCache>(); // One cache
builder.Services.AddSingleton<IConfiguration>(config);      // One config
builder.Services.AddSingleton<IApiKeyStore, InMemoryApiKeyStore>(); // One key store

// InMemoryApiKeyStore (SaaS: valid API keys cached in memory)
public class InMemoryApiKeyStore : IApiKeyStore
{
    private readonly ConcurrentDictionary<string, ApiKeyInfo> _keys = new();
    private readonly Timer _refreshTimer;
    
    public InMemoryApiKeyStore(IApiKeyRepository repository)
    {
        // Load all keys on startup, refresh every 5 minutes
        RefreshAsync(repository).Wait();
        _refreshTimer = new Timer(
            async _ => await RefreshAsync(repository), 
            null, 
            TimeSpan.FromMinutes(5), 
            TimeSpan.FromMinutes(5));
    }
    
    public bool IsValid(string apiKey) => 
        _keys.TryGetValue(apiKey, out var info) && !info.IsExpired;
    
    private async Task RefreshAsync(IApiKeyRepository repository)
    {
        var keys = await repository.GetAllActiveAsync();
        _keys.Clear();
        foreach (var key in keys)
            _keys[key.Key] = key;
    }
}
```

### 6. Execution Flow

1. App starts → `builder.Services.AddSingleton<IApiKeyStore, InMemoryApiKeyStore>()`
2. First request to resolve `IApiKeyStore` → DI container creates `InMemoryApiKeyStore`
3. Constructor called once — loads all API keys from DB
4. All subsequent requests → same instance returned (no DB call)
5. Timer fires every 5 min → refreshes keys in background

### 7. When NOT to Use Singleton

- When it holds **request-specific state** (use Scoped instead)
- When it depends on Scoped services (captive dependency problem)
- When you need **multiple instances** in testing (global state makes parallel tests fail)
- When the object is cheap to create (over-engineering)

### 8. Interview Prep

**Basic:** "What is the Singleton pattern?"
> A pattern that ensures a class has only one instance throughout the application lifetime and provides a global access point.

**Intermediate:** "How do you make a Singleton thread-safe in C#?"
> Using `Lazy<T>` with `LazyThreadSafetyMode.ExecutionAndPublication` — it handles double-checked locking internally. Or better, use the DI container with `AddSingleton<>`, which handles thread safety and also allows interface abstraction.

**Senior:** "Is Singleton an anti-pattern?"
> It can be. The classic static-class Singleton is an anti-pattern because it introduces hidden global state, makes testing difficult, and couples code to a concrete class. BUT a DI-managed singleton (registered with `AddSingleton`) solves all those problems — it's a single instance, but resolved through an interface, mockable in tests, and with no static state. The pattern itself isn't wrong; the static implementation is.

**Tricky:** "What's the difference between Singleton and static class?"
> A static class cannot implement interfaces, cannot be inherited, cannot be instantiated, cannot be passed as a parameter, and cannot be mocked. A singleton class CAN do all of these — it's just constrained to one instance. DI singletons are the cleanest: interface-based, mockable, lifecycle-managed by the framework.

---

## Factory Method Pattern

### 1. Definition

Defines an interface for creating an object, but **lets subclasses decide which class to instantiate**. Factory Method defers instantiation to subclasses.

### 2. Problem It Solves

```csharp
// BAD: Client code directly creates payment gateways — coupled to concrete types
public class CheckoutService
{
    public async Task<PaymentResult> ProcessPaymentAsync(Order order, string gateway)
    {
        IPaymentGateway paymentGateway;
        
        // Client must know about all concrete classes
        if (gateway == "Stripe")
            paymentGateway = new StripeGateway(Environment.GetEnvironmentVariable("STRIPE_KEY"));
        else if (gateway == "PayPal")
            paymentGateway = new PayPalGateway(Environment.GetEnvironmentVariable("PAYPAL_CLIENT_ID"),
                Environment.GetEnvironmentVariable("PAYPAL_SECRET"));
        else if (gateway == "Razorpay")
            paymentGateway = new RazorpayGateway(Environment.GetEnvironmentVariable("RAZORPAY_KEY"));
        else
            throw new ArgumentException($"Unknown gateway: {gateway}");
        
        return await paymentGateway.ProcessAsync(new PaymentRequest { Amount = order.Total });
    }
}
```

### 3. Pattern Solution

```csharp
// Factory Method: encapsulates creation logic
public interface IPaymentGatewayFactory
{
    IPaymentGateway Create(string gatewayName);
}

public class PaymentGatewayFactory : IPaymentGatewayFactory
{
    private readonly IConfiguration _config;
    private readonly ILogger<PaymentGatewayFactory> _logger;
    private readonly IHttpClientFactory _httpClientFactory;
    
    public PaymentGatewayFactory(
        IConfiguration config,
        ILogger<PaymentGatewayFactory> logger,
        IHttpClientFactory httpClientFactory)
    {
        _config = config;
        _logger = logger;
        _httpClientFactory = httpClientFactory;
    }
    
    public IPaymentGateway Create(string gatewayName) => gatewayName.ToUpperInvariant() switch
    {
        "STRIPE" => new StripeGateway(
            _config["PaymentGateways:Stripe:SecretKey"],
            _httpClientFactory.CreateClient("Stripe"),
            _logger.CreateLogger<StripeGateway>()),
        
        "PAYPAL" => new PayPalGateway(
            _config["PaymentGateways:PayPal:ClientId"],
            _config["PaymentGateways:PayPal:Secret"],
            _httpClientFactory.CreateClient("PayPal")),
        
        "RAZORPAY" => new RazorpayGateway(
            _config["PaymentGateways:Razorpay:KeyId"],
            _config["PaymentGateways:Razorpay:Secret"]),
        
        _ => throw new NotSupportedException($"Payment gateway '{gatewayName}' is not supported")
    };
}

// DI-based alternative (Open/Closed — preferred):
public class DependencyResolvingGatewayFactory : IPaymentGatewayFactory
{
    private readonly IServiceProvider _serviceProvider;
    
    public DependencyResolvingGatewayFactory(IServiceProvider serviceProvider)
        => _serviceProvider = serviceProvider;
    
    public IPaymentGateway Create(string gatewayName)
    {
        // Resolves from DI — adding a gateway = register in DI, no factory change
        var gateways = _serviceProvider.GetServices<IPaymentGateway>();
        return gateways.FirstOrDefault(g => g.GatewayName.Equals(gatewayName, StringComparison.OrdinalIgnoreCase))
            ?? throw new NotSupportedException($"Gateway '{gatewayName}' not registered");
    }
}

// Usage:
public class CheckoutService
{
    private readonly IPaymentGatewayFactory _gatewayFactory;
    
    public async Task<PaymentResult> ProcessPaymentAsync(Order order, string preferredGateway)
    {
        var gateway = _gatewayFactory.Create(preferredGateway);
        return await gateway.ProcessAsync(new PaymentRequest { Amount = order.Total });
    }
}
```

### 4. When NOT to Use Factory Method

- When you only ever have ONE concrete implementation — overkill
- When object creation is trivially simple (`new Order()`)
- When the DI container already handles creation (it often makes Factory Method redundant)

### 5. Interview Prep

**Tricky:** "What's the difference between Factory Method and Abstract Factory?"
> Factory Method creates ONE product via a method (often in a class hierarchy). Abstract Factory creates FAMILIES of related products. Factory Method: `CreatePaymentGateway()`. Abstract Factory: `CreatePaymentGateway() + CreateRefundProcessor() + CreateWebhookHandler()` — all related objects for one payment provider.

---

## Abstract Factory Pattern

### 1. Definition

Provides an interface for creating **families of related or dependent objects** without specifying their concrete classes.

### 2. Problem It Solves — Database Provider Families

```csharp
// Problem: You need to switch between SQL Server and PostgreSQL
// Each database has different connection, command, and parameter types

// Abstract Factory: creates a family of related objects
public interface IDatabaseProviderFactory
{
    IDbConnection CreateConnection(string connectionString);
    IDbCommand CreateCommand(string sql, IDbConnection connection);
    IDbDataParameter CreateParameter(string name, object value);
    IDataAdapter CreateDataAdapter(IDbCommand command);
}

public class SqlServerProviderFactory : IDatabaseProviderFactory
{
    public IDbConnection CreateConnection(string connectionString)
        => new SqlConnection(connectionString);
    
    public IDbCommand CreateCommand(string sql, IDbConnection connection)
        => new SqlCommand(sql, (SqlConnection)connection);
    
    public IDbDataParameter CreateParameter(string name, object value)
        => new SqlParameter(name, value);
    
    public IDataAdapter CreateDataAdapter(IDbCommand command)
        => new SqlDataAdapter((SqlCommand)command);
}

public class PostgreSqlProviderFactory : IDatabaseProviderFactory
{
    public IDbConnection CreateConnection(string connectionString)
        => new NpgsqlConnection(connectionString);
    
    public IDbCommand CreateCommand(string sql, IDbConnection connection)
        => new NpgsqlCommand(sql, (NpgsqlConnection)connection);
    
    public IDbDataParameter CreateParameter(string name, object value)
        => new NpgsqlParameter(name, value);
    
    public IDataAdapter CreateDataAdapter(IDbCommand command)
        => new NpgsqlDataAdapter((NpgsqlCommand)command);
}

public class InMemoryProviderFactory : IDatabaseProviderFactory
{
    // For unit testing — fake all DB objects
    public IDbConnection CreateConnection(string connectionString) => new FakeConnection();
    public IDbCommand CreateCommand(string sql, IDbConnection connection) => new FakeCommand();
    // etc.
}

// Usage — switches entire family by swapping factory:
public class DataAccessService
{
    private readonly IDatabaseProviderFactory _factory;
    private readonly string _connectionString;
    
    public DataAccessService(IDatabaseProviderFactory factory, IConfiguration config)
    {
        _factory = factory;
        _connectionString = config.GetConnectionString("DefaultConnection");
    }
    
    public async Task<IEnumerable<Order>> GetOrdersAsync()
    {
        using var connection = _factory.CreateConnection(_connectionString);
        await connection.OpenAsync();
        using var command = _factory.CreateCommand("SELECT * FROM Orders", connection);
        // Works with ANY database — factory determines which
    }
}

// DI: swap entire database family with one line
builder.Services.AddSingleton<IDatabaseProviderFactory, SqlServerProviderFactory>();
// Or: builder.Services.AddSingleton<IDatabaseProviderFactory, PostgreSqlProviderFactory>();
```

### 3. Real Production Use — SaaS Multi-Tenant Storage

```csharp
// Each tenant can have different storage back-end
public interface IStorageFactory
{
    IBlobStorage CreateBlobStorage();
    IQueueStorage CreateQueueStorage();
    ITableStorage CreateTableStorage();
}

public class AzureStorageFactory : IStorageFactory
{
    private readonly string _connectionString;
    public AzureStorageFactory(string connectionString) => _connectionString = connectionString;
    
    public IBlobStorage CreateBlobStorage() => new AzureBlobStorage(_connectionString);
    public IQueueStorage CreateQueueStorage() => new AzureQueueStorage(_connectionString);
    public ITableStorage CreateTableStorage() => new AzureTableStorage(_connectionString);
}

public class AwsStorageFactory : IStorageFactory
{
    private readonly AWSCredentials _credentials;
    public AWSCredentials Credentials => _credentials;
    
    public IBlobStorage CreateBlobStorage() => new S3BlobStorage(_credentials);
    public IQueueStorage CreateQueueStorage() => new SqsQueueStorage(_credentials);
    public ITableStorage CreateTableStorage() => new DynamoTableStorage(_credentials);
}
```

### 4. When NOT to Use Abstract Factory

- When you only have one concrete family (adds unnecessary abstraction)
- When the families are unlikely to change (YAGNI)
- When families grow — each new product type requires changes to ALL factories

---

## Builder Pattern

### 1. Definition

Separates the **construction** of a complex object from its **representation**, letting the same construction process create different representations.

### 2. Problem: Constructor Explosion (Telescoping Constructors)

```csharp
// BAD: Telescoping constructors — unreadable, error-prone
public class Order
{
    public Order(string customerId, string currency) { ... }
    public Order(string customerId, string currency, string couponCode) { ... }
    public Order(string customerId, string currency, string couponCode, Address shippingAddress) { ... }
    public Order(string customerId, string currency, string couponCode, Address shippingAddress, 
        ShippingMethod shippingMethod) { ... }
    public Order(string customerId, string currency, string couponCode, Address shippingAddress, 
        ShippingMethod shippingMethod, bool isGiftOrder) { ... }
    // ... and so on
    
    // At call site — which argument is which?!
    var order = new Order("cust_123", "USD", null, address, ShippingMethod.Express, true, false, null);
    // Is that 'true' isGiftOrder or isPriority? Impossible to know without IDE.
}
```

### 3. Pattern Solution: Fluent Builder

```csharp
// The complex object
public sealed class Order
{
    public Guid Id { get; private init; }
    public string CustomerId { get; private init; }
    public string Currency { get; private init; }
    public IReadOnlyList<OrderItem> Items { get; private init; }
    public Address? ShippingAddress { get; private init; }
    public ShippingMethod ShippingMethod { get; private init; }
    public string? CouponCode { get; private init; }
    public bool IsGiftOrder { get; private init; }
    public string? GiftMessage { get; private init; }
    public decimal? InsuranceAmount { get; private init; }
    
    // Private constructor — only Builder can create
    private Order() { }
    
    // Builder lives as nested class for access to private constructor
    public sealed class Builder
    {
        private readonly string _customerId;
        private readonly string _currency;
        private readonly List<OrderItem> _items = new();
        private Address? _shippingAddress;
        private ShippingMethod _shippingMethod = ShippingMethod.Standard;
        private string? _couponCode;
        private bool _isGiftOrder;
        private string? _giftMessage;
        private decimal? _insuranceAmount;
        
        // Required parameters in constructor — can't build without them
        public Builder(string customerId, string currency)
        {
            if (string.IsNullOrWhiteSpace(customerId))
                throw new ArgumentException("Customer ID required", nameof(customerId));
            if (string.IsNullOrWhiteSpace(currency))
                throw new ArgumentException("Currency required", nameof(currency));
            
            _customerId = customerId;
            _currency = currency;
        }
        
        // Fluent methods — each returns 'this' for chaining
        public Builder AddItem(string productId, int quantity, decimal unitPrice)
        {
            if (quantity <= 0) throw new ArgumentException("Quantity must be positive");
            if (unitPrice <= 0) throw new ArgumentException("Price must be positive");
            
            _items.Add(new OrderItem(productId, quantity, unitPrice));
            return this;
        }
        
        public Builder ShipTo(Address address)
        {
            _shippingAddress = address ?? throw new ArgumentNullException(nameof(address));
            return this;
        }
        
        public Builder WithShippingMethod(ShippingMethod method)
        {
            _shippingMethod = method;
            return this;
        }
        
        public Builder WithCoupon(string couponCode)
        {
            _couponCode = couponCode;
            return this;
        }
        
        public Builder AsGiftOrder(string giftMessage)
        {
            _isGiftOrder = true;
            _giftMessage = giftMessage ?? throw new ArgumentNullException(nameof(giftMessage));
            return this;
        }
        
        public Builder WithInsurance(decimal amount)
        {
            if (amount <= 0) throw new ArgumentException("Insurance amount must be positive");
            _insuranceAmount = amount;
            return this;
        }
        
        // Build: validate and create the immutable Order
        public Order Build()
        {
            if (!_items.Any())
                throw new InvalidOperationException("Order must have at least one item");
            
            if (_shippingAddress == null)
                throw new InvalidOperationException("Shipping address is required");
            
            return new Order
            {
                Id = Guid.NewGuid(),
                CustomerId = _customerId,
                Currency = _currency,
                Items = _items.AsReadOnly(),
                ShippingAddress = _shippingAddress,
                ShippingMethod = _shippingMethod,
                CouponCode = _couponCode,
                IsGiftOrder = _isGiftOrder,
                GiftMessage = _giftMessage,
                InsuranceAmount = _insuranceAmount
            };
        }
    }
}

// Usage — crystal clear and readable:
var order = new Order.Builder("cust_123", "USD")
    .AddItem("SKU-001", quantity: 2, unitPrice: 29.99m)
    .AddItem("SKU-002", quantity: 1, unitPrice: 49.99m)
    .ShipTo(new Address("123 Main St", "New York", "NY", "10001", "US"))
    .WithShippingMethod(ShippingMethod.Express)
    .WithCoupon("SAVE10")
    .AsGiftOrder("Happy Birthday!")
    .WithInsurance(20m)
    .Build();
```

### 4. Execution Flow

1. `new Order.Builder("cust_123", "USD")` → validates required params, sets state
2. Each `.AddItem(...)` → validates, adds to `_items`, returns `this`
3. `.ShipTo(...)`, `.WithCoupon(...)` etc. → sets optional state
4. `.Build()` → validates completeness, creates immutable `Order` with `private init`
5. If any invariant fails → exception at build time, not at use time

### 5. When NOT to Use Builder

- Simple objects with ≤3 constructor parameters
- Objects created by ORMs (EF Core handles construction)
- When `record` types with `with` expressions suffice

### 6. Interview Prep

**Tricky:** "How is Builder different from Factory?"
> Factory encapsulates WHICH type to create (the selection). Builder encapsulates HOW to construct a KNOWN type step-by-step. Factory answers "which object?" Builder answers "how to assemble THIS object?"

---

# STRUCTURAL PATTERNS

> Structural patterns deal with object composition — how objects and classes are combined to form larger structures.

---

## Repository Pattern

### 1. Definition

Mediates between the domain and data mapping layers, providing a collection-like interface for accessing domain objects.

### 2. Problem It Solves

```csharp
// BAD: Data access logic scattered in business services
public class OrderService
{
    private readonly OrderDbContext _db;
    
    public async Task<Order?> GetOrderAsync(Guid id)
    {
        // EF query in service layer — business logic mixes with data access
        return await _db.Orders
            .Include(o => o.Items)
            .Include(o => o.Customer)
            .Where(o => o.Id == id && !o.IsDeleted)
            .FirstOrDefaultAsync();
    }
    
    public async Task<IEnumerable<Order>> GetPendingOrdersAsync()
    {
        // Query duplicated in multiple services
        return await _db.Orders
            .Where(o => o.Status == OrderStatus.Pending && !o.IsDeleted)
            .OrderBy(o => o.CreatedAt)
            .ToListAsync();
    }
}
// Problem: Test OrderService → need real DB. Change schema → find all LINQ queries.
```

### 3. Pattern Solution

```csharp
// Repository Interface — the abstraction
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(Guid id, CancellationToken ct = default);
    Task<IReadOnlyList<Order>> GetByCustomerAsync(string customerId, CancellationToken ct = default);
    Task<IReadOnlyList<Order>> GetPendingAsync(CancellationToken ct = default);
    Task<IReadOnlyList<Order>> GetByDateRangeAsync(DateTimeOffset from, DateTimeOffset to, CancellationToken ct = default);
    Task AddAsync(Order order, CancellationToken ct = default);
    Task UpdateAsync(Order order, CancellationToken ct = default);
    Task<bool> ExistsAsync(Guid id, CancellationToken ct = default);
}

// Concrete implementation — data access details here only
public class EfOrderRepository : IOrderRepository
{
    private readonly OrderDbContext _db;
    
    public EfOrderRepository(OrderDbContext db) => _db = db;
    
    public async Task<Order?> GetByIdAsync(Guid id, CancellationToken ct = default)
        => await _db.Orders
            .AsNoTracking()
            .Include(o => o.Items).ThenInclude(i => i.Product)
            .Include(o => o.ShippingAddress)
            .FirstOrDefaultAsync(o => o.Id == id && !o.IsDeleted, ct);
    
    public async Task<IReadOnlyList<Order>> GetPendingAsync(CancellationToken ct = default)
        => await _db.Orders
            .Where(o => o.Status == OrderStatus.Pending && !o.IsDeleted)
            .OrderBy(o => o.CreatedAt)
            .Take(1000) // Safeguard
            .ToListAsync(ct);
    
    public async Task AddAsync(Order order, CancellationToken ct = default)
    {
        await _db.Orders.AddAsync(order, ct);
        // Note: SaveChanges handled by Unit of Work, not here
    }
    
    // ... other implementations
}

// Unit of Work — coordinates multiple repositories in one transaction
public interface IUnitOfWork : IDisposable
{
    IOrderRepository Orders { get; }
    IPaymentRepository Payments { get; }
    IInventoryRepository Inventory { get; }
    Task<int> SaveChangesAsync(CancellationToken ct = default);
}

public class EfUnitOfWork : IUnitOfWork
{
    private readonly OrderDbContext _db;
    
    public EfUnitOfWork(OrderDbContext db)
    {
        _db = db;
        Orders = new EfOrderRepository(db);
        Payments = new EfPaymentRepository(db);
        Inventory = new EfInventoryRepository(db);
    }
    
    public IOrderRepository Orders { get; }
    public IPaymentRepository Payments { get; }
    public IInventoryRepository Inventory { get; }
    
    public async Task<int> SaveChangesAsync(CancellationToken ct = default)
        => await _db.SaveChangesAsync(ct);
    
    public void Dispose() => _db.Dispose();
}

// Service uses repository abstraction:
public class OrderService
{
    private readonly IUnitOfWork _uow;
    
    public async Task<OrderResult> PlaceOrderAsync(PlaceOrderRequest request, CancellationToken ct)
    {
        // Check inventory
        var isAvailable = await _uow.Inventory.IsAvailableAsync(request.Items, ct);
        if (!isAvailable) return OrderResult.Failed("Items not available");
        
        var order = Order.Create(request);
        await _uow.Orders.AddAsync(order, ct);
        
        // Reserve inventory
        await _uow.Inventory.ReserveAsync(order.Items, ct);
        
        // Both saved in ONE transaction
        await _uow.SaveChangesAsync(ct);
        
        return OrderResult.Success(order.Id);
    }
}
```

### 4. Generic Repository — Pros and Cons

```csharp
// Generic Repository
public interface IRepository<T> where T : class
{
    Task<T?> GetByIdAsync(Guid id);
    Task<IReadOnlyList<T>> GetAllAsync();
    Task AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(Guid id);
}

// PROS: Reduces boilerplate for simple CRUD
// CONS: 
// - Exposes GetAll() for all entities (dangerous for large tables!)
// - Can't express domain-specific queries (GetPendingOrders, GetByCustomer)
// - Leads to "GenericRepository<Order>.GetAll().Where(...)" — leaks into service layer
// - You end up adding domain-specific methods anyway

// VERDICT: Use SPECIFIC repositories. Generic can be a base class but NOT the API.
public abstract class RepositoryBase<T> : IRepository<T> where T : class
{
    protected readonly DbContext Context;
    
    protected RepositoryBase(DbContext context) => Context = context;
    
    public virtual async Task<T?> GetByIdAsync(Guid id) 
        => await Context.Set<T>().FindAsync(id);
    
    // Subclasses ADD domain-specific methods on top
}

public class OrderRepository : RepositoryBase<Order>, IOrderRepository
{
    public OrderRepository(OrderDbContext context) : base(context) { }
    
    // Domain-specific — these are what make repository valuable
    public async Task<IReadOnlyList<Order>> GetPendingAsync()
        => await Context.Set<Order>()
            .Where(o => o.Status == OrderStatus.Pending)
            .ToListAsync();
}
```

### 5. When Repository Is an Anti-Pattern (Over EF Core)

```csharp
// If you're using EF Core and NOT testing with mocks:
// - DbContext IS a unit of work
// - DbSet<T> IS a repository
// - Adding a repository layer on top = redundant abstraction

// ACCEPTABLE to skip repository if:
// 1. Team won't switch databases
// 2. Tests use real DB (integration tests)
// 3. Application is simple CRUD
// 4. You're using DbContext directly in handlers (CQRS/Vertical Slice)

// In CQRS/Minimal API approach (common today):
public class GetOrderQueryHandler
{
    private readonly OrderDbContext _db; // Direct DbContext — no repository
    
    public async Task<OrderDto?> Handle(GetOrderQuery query, CancellationToken ct)
        => await _db.Orders
            .Where(o => o.Id == query.OrderId)
            .Select(o => new OrderDto { Id = o.Id, Total = o.Total })
            .FirstOrDefaultAsync(ct);
    // Query is the abstraction; repository adds nothing here
}
```

### 6. Interview Prep

**Tricky:** "Is the Repository pattern an anti-pattern over Entity Framework?"
> It depends on goals. If your goals are: testability via mocks, database-independence, and centralized query logic — Repository adds value. If you use integration tests with real DB, EF Core IS your repository (DbSet is a repository, DbContext is a unit of work). Adding another layer creates redundant abstraction. In modern CQRS/Vertical Slice architectures, it's common to skip Repository and query DbContext directly in handlers.

---

## Decorator Pattern

### 1. Definition

Attaches additional responsibilities to an object **dynamically** by wrapping it. Provides a flexible alternative to subclassing.

### 2. Problem It Solves

```csharp
// Need to add: caching, logging, retry — to an existing service
// WITHOUT modifying the original class (OCP)
// WITHOUT inheritance chains (composition over inheritance)
```

### 3. Pattern Solution

```csharp
// Base interface
public interface IOrderService
{
    Task<Order?> GetOrderAsync(Guid id);
    Task<OrderResult> PlaceOrderAsync(PlaceOrderRequest request);
}

// Core implementation
public class OrderService : IOrderService
{
    private readonly IOrderRepository _repo;
    
    public async Task<Order?> GetOrderAsync(Guid id) 
        => await _repo.GetByIdAsync(id);
    
    public async Task<OrderResult> PlaceOrderAsync(PlaceOrderRequest request)
    {
        // Core business logic
    }
}

// Decorator 1: Adds caching
public class CachingOrderService : IOrderService
{
    private readonly IOrderService _inner;    // Wraps IOrderService (not OrderService!)
    private readonly IDistributedCache _cache;
    private static readonly TimeSpan CacheDuration = TimeSpan.FromMinutes(5);
    
    public CachingOrderService(IOrderService inner, IDistributedCache cache)
    {
        _inner = inner;
        _cache = cache;
    }
    
    public async Task<Order?> GetOrderAsync(Guid id)
    {
        var key = $"order:{id}";
        var cached = await _cache.GetStringAsync(key);
        
        if (cached != null)
            return JsonSerializer.Deserialize<Order>(cached);
        
        var order = await _inner.GetOrderAsync(id); // Delegate to wrapped service
        
        if (order != null)
            await _cache.SetStringAsync(key, JsonSerializer.Serialize(order),
                new DistributedCacheEntryOptions { AbsoluteExpirationRelativeToNow = CacheDuration });
        
        return order;
    }
    
    public Task<OrderResult> PlaceOrderAsync(PlaceOrderRequest request)
        => _inner.PlaceOrderAsync(request); // Pass through — no caching for writes
}

// Decorator 2: Adds logging
public class LoggingOrderService : IOrderService
{
    private readonly IOrderService _inner;
    private readonly ILogger<LoggingOrderService> _logger;
    
    public LoggingOrderService(IOrderService inner, ILogger<LoggingOrderService> logger)
    {
        _inner = inner;
        _logger = logger;
    }
    
    public async Task<Order?> GetOrderAsync(Guid id)
    {
        _logger.LogInformation("Getting order {OrderId}", id);
        var sw = Stopwatch.StartNew();
        
        try
        {
            var result = await _inner.GetOrderAsync(id);
            _logger.LogInformation("Got order {OrderId} in {Ms}ms. Found: {Found}", 
                id, sw.ElapsedMilliseconds, result != null);
            return result;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to get order {OrderId}", id);
            throw;
        }
    }
    
    public async Task<OrderResult> PlaceOrderAsync(PlaceOrderRequest request)
    {
        _logger.LogInformation("Placing order for customer {CustomerId}", request.CustomerId);
        var result = await _inner.PlaceOrderAsync(request);
        _logger.LogInformation("Order placement result: {Success}", result.IsSuccess);
        return result;
    }
}

// DI registration with Scrutor (popular NuGet for decorator registration):
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.Decorate<IOrderService, CachingOrderService>();  // Wraps OrderService
builder.Services.Decorate<IOrderService, LoggingOrderService>();  // Wraps CachingOrderService

// Resolution order: LoggingOrderService → CachingOrderService → OrderService
// Call chain: Logging.GetOrder → Caching.GetOrder → OrderService.GetOrder

// Manual registration (without Scrutor):
builder.Services.AddScoped<OrderService>();
builder.Services.AddScoped<IOrderService>(sp =>
{
    var core = sp.GetRequiredService<OrderService>();
    var cached = new CachingOrderService(core, sp.GetRequiredService<IDistributedCache>());
    var logged = new LoggingOrderService(cached, sp.GetRequiredService<ILogger<LoggingOrderService>>());
    return logged;
});
```

### 4. Execution Flow

```
Client.GetOrderAsync(id)
  → LoggingOrderService.GetOrderAsync(id)          [logs start, starts timer]
    → CachingOrderService.GetOrderAsync(id)        [checks Redis cache]
      → OrderService.GetOrderAsync(id)             [queries SQL — if cache miss]
    ← CachingOrderService [stores in Redis, returns order]
  ← LoggingOrderService [logs completion, elapsed time, returns order]
← Client receives Order
```

### 5. When NOT to Use Decorator

- When cross-cutting concerns are better handled by middleware (ASP.NET pipeline)
- When AOP (Aspect-Oriented Programming) via `DispatchProxy` or `Castle DynamicProxy` is cleaner
- When only ONE decorator is ever needed (just modify the class)
- When the interface has many methods — every decorator must implement them all

---

## Adapter Pattern

### 1. Definition

Converts the interface of a class into another interface that clients expect. Lets classes work together that otherwise couldn't due to incompatible interfaces.

### 2. Problem: Incompatible Third-Party SDK

```csharp
// Your system uses this contract:
public interface IPaymentGateway
{
    Task<PaymentResult> ProcessAsync(PaymentRequest request);
    Task<RefundResult> RefundAsync(string transactionId, decimal amount);
}

// Legacy SDK you must integrate — completely different interface:
public class LegacyPaymentSdk
{
    public bool MakePayment(string cardNumber, string cvv, decimal amount, string currency)
    { /* Returns bool - success/failure */ return true; }
    
    public string GetLastTransactionId() { return "TXN-123"; }
    
    public bool ReverseTransaction(string transactionId, double amount)
    { return true; }
    
    public string GetLastError() { return string.Empty; }
}
```

### 3. Pattern Solution

```csharp
// Adapter: translates YOUR interface to THE LEGACY SDK
public class LegacyPaymentGatewayAdapter : IPaymentGateway
{
    private readonly LegacyPaymentSdk _legacySdk;
    private readonly ILogger<LegacyPaymentGatewayAdapter> _logger;
    
    public LegacyPaymentGatewayAdapter(LegacyPaymentSdk legacySdk, 
        ILogger<LegacyPaymentGatewayAdapter> logger)
    {
        _legacySdk = legacySdk;
        _logger = logger;
    }
    
    public async Task<PaymentResult> ProcessAsync(PaymentRequest request)
    {
        // Translate YOUR model to LEGACY SDK's model
        var cardNumber = request.PaymentMethod.CardNumber;
        var cvv = request.PaymentMethod.CVV;
        var amount = request.Amount;
        var currency = request.Currency;
        
        // Legacy SDK is synchronous — wrap in Task.Run
        var success = await Task.Run(() => 
            _legacySdk.MakePayment(cardNumber, cvv, amount, currency));
        
        if (success)
        {
            var transactionId = _legacySdk.GetLastTransactionId();
            _logger.LogInformation("Legacy payment succeeded: {TransactionId}", transactionId);
            return PaymentResult.Success(transactionId);
        }
        else
        {
            var error = _legacySdk.GetLastError();
            _logger.LogWarning("Legacy payment failed: {Error}", error);
            return PaymentResult.Failed(error);
        }
    }
    
    public async Task<RefundResult> RefundAsync(string transactionId, decimal amount)
    {
        // Legacy SDK uses double, we use decimal — convert
        var success = await Task.Run(() => 
            _legacySdk.ReverseTransaction(transactionId, (double)amount));
        
        return success 
            ? RefundResult.Success(transactionId) 
            : RefundResult.Failed(_legacySdk.GetLastError());
    }
}

// Usage in DI — fully transparent to CheckoutService:
builder.Services.AddSingleton<LegacyPaymentSdk>();
builder.Services.AddScoped<IPaymentGateway, LegacyPaymentGatewayAdapter>();
// CheckoutService doesn't know it's talking to a legacy system
```

### 4. When NOT to Use Adapter

- When you OWN the class you're adapting (just change its interface)
- When the mismatch is too large (might need a full integration layer)
- When you can use inheritance directly (`class NewClass : INewInterface, OldClass`) — class adapter

---

## Facade Pattern

### 1. Definition

Provides a **simplified interface** to a complex subsystem. Doesn't hide the subsystem, but makes it easier to use.

### 2. Problem: Complex Checkout Subsystem

```csharp
// Checkout involves many systems — complex for controller to orchestrate
public class CheckoutController : ControllerBase
{
    // BAD: Controller knows about all subsystems
    private readonly IInventoryService _inventory;
    private readonly IPaymentGateway _payment;
    private readonly IShippingService _shipping;
    private readonly INotificationService _notifications;
    private readonly ILoyaltyService _loyalty;
    private readonly IOrderRepository _orders;
    private readonly IFraudDetectionService _fraud;
    private readonly ITaxCalculationService _tax;
    
    public async Task<IActionResult> Checkout(CheckoutRequest request)
    {
        // Controller doing too much — should not know all these steps
        if (!await _inventory.CheckAvailabilityAsync(request.Items)) 
            return BadRequest("Out of stock");
        
        var fraudScore = await _fraud.CheckAsync(request.CustomerId, request.PaymentMethod);
        if (fraudScore > 0.8) return BadRequest("Payment declined");
        
        var taxAmount = await _tax.CalculateAsync(request.Items, request.ShippingAddress);
        
        var paymentResult = await _payment.ProcessAsync(new PaymentRequest { 
            Amount = request.Total + taxAmount });
        if (!paymentResult.IsSuccess) return BadRequest("Payment failed");
        
        var order = Order.Create(request);
        await _orders.SaveAsync(order);
        
        await _inventory.ReserveAsync(request.Items, order.Id);
        
        var shippingLabel = await _shipping.CreateLabelAsync(order);
        
        await _loyalty.AwardPointsAsync(request.CustomerId, request.Total);
        
        await _notifications.SendOrderConfirmationAsync(request.CustomerEmail, order);
        
        return Ok(new CheckoutResponse { OrderId = order.Id, ShippingLabel = shippingLabel });
    }
}
```

### 3. Pattern Solution — Facade

```csharp
// Facade: simplified entry point for complex checkout subsystem
public interface ICheckoutFacade
{
    Task<CheckoutResult> CheckoutAsync(CheckoutRequest request, CancellationToken ct = default);
}

public class CheckoutFacade : ICheckoutFacade
{
    private readonly IInventoryService _inventory;
    private readonly IPaymentGateway _payment;
    private readonly IShippingService _shipping;
    private readonly INotificationService _notifications;
    private readonly ILoyaltyService _loyalty;
    private readonly IOrderRepository _orders;
    private readonly IFraudDetectionService _fraud;
    private readonly ITaxCalculationService _tax;
    private readonly ILogger<CheckoutFacade> _logger;
    
    public CheckoutFacade(/* all dependencies */) { /* inject */ }
    
    public async Task<CheckoutResult> CheckoutAsync(CheckoutRequest request, CancellationToken ct)
    {
        _logger.LogInformation("Starting checkout for customer {CustomerId}", request.CustomerId);
        
        // Orchestrates the entire flow — hidden from caller
        if (!await _inventory.CheckAvailabilityAsync(request.Items, ct))
            return CheckoutResult.Failed("One or more items are out of stock");
        
        var fraudScore = await _fraud.CheckAsync(request.CustomerId, request.PaymentMethod, ct);
        if (fraudScore > 0.8)
            return CheckoutResult.Failed("Payment declined by security system");
        
        var taxAmount = await _tax.CalculateAsync(request.Items, request.ShippingAddress, ct);
        var totalWithTax = request.Total + taxAmount;
        
        var paymentResult = await _payment.ProcessAsync(
            new PaymentRequest { Amount = totalWithTax, Currency = request.Currency }, ct);
        
        if (!paymentResult.IsSuccess)
            return CheckoutResult.Failed($"Payment failed: {paymentResult.FailureReason}");
        
        var order = Order.Create(request, paymentResult.TransactionId, taxAmount);
        await _orders.SaveAsync(order, ct);
        
        // These can run in parallel — no interdependency
        await Task.WhenAll(
            _inventory.ReserveAsync(request.Items, order.Id, ct),
            _loyalty.AwardPointsAsync(request.CustomerId, request.Total, ct),
            _shipping.CreateLabelAsync(order, ct)
                .ContinueWith(async t => await _notifications.SendOrderConfirmationAsync(
                    request.CustomerEmail, order, t.Result, ct)));
        
        _logger.LogInformation("Checkout completed. OrderId: {OrderId}", order.Id);
        return CheckoutResult.Success(order.Id, taxAmount);
    }
}

// Controller is now clean — just uses facade:
public class CheckoutController : ControllerBase
{
    private readonly ICheckoutFacade _checkout;
    
    [HttpPost("checkout")]
    public async Task<IActionResult> Checkout(
        [FromBody] CheckoutRequest request, 
        CancellationToken ct)
    {
        var result = await _checkout.CheckoutAsync(request, ct);
        return result.IsSuccess ? Ok(result) : BadRequest(result.ErrorMessage);
    }
}
```

### 4. When NOT to Use Facade

- When the subsystem is already simple
- When you need clients to access subsystem details (Facade hides too much)
- When it becomes a "God Object" — too many responsibilities (violates SRP)

---

# BEHAVIORAL PATTERNS

> Behavioral patterns deal with algorithms and assignment of responsibilities between objects.

---

## Strategy Pattern

### 1. Definition

Defines a family of algorithms, encapsulates each one, and makes them interchangeable. Lets the algorithm vary independently from clients that use it.

### 2. Problem and Solution

```csharp
// BAD: If/else chains for discount calculation
public decimal CalculateDiscount(Order order, string customerType)
{
    if (customerType == "Premium") return order.Total * 0.20m;
    else if (customerType == "VIP") return order.Total * 0.30m;
    else if (customerType == "Seasonal") return order.Total > 1000 ? order.Total * 0.15m : 0;
    else return 0;
}

// GOOD: Strategy Pattern
public interface IDiscountStrategy
{
    string StrategyName { get; }
    decimal CalculateDiscount(Order order);
}

public class PremiumCustomerDiscount : IDiscountStrategy
{
    public string StrategyName => "Premium";
    public decimal CalculateDiscount(Order order) => order.Total * 0.20m;
}

public class VIPCustomerDiscount : IDiscountStrategy
{
    public string StrategyName => "VIP";
    public decimal CalculateDiscount(Order order) => order.Total * 0.30m;
}

public class SeasonalDiscount : IDiscountStrategy
{
    private readonly decimal _threshold;
    private readonly decimal _discountPercent;
    
    public SeasonalDiscount(decimal threshold, decimal discountPercent)
    {
        _threshold = threshold;
        _discountPercent = discountPercent;
    }
    
    public string StrategyName => "Seasonal";
    public decimal CalculateDiscount(Order order) 
        => order.Total >= _threshold ? order.Total * _discountPercent : 0;
}

public class NoDiscount : IDiscountStrategy
{
    public string StrategyName => "None";
    public decimal CalculateDiscount(Order order) => 0;
}

// Context that uses strategy:
public class OrderPricingService
{
    private readonly IEnumerable<IDiscountStrategy> _strategies;
    
    public OrderPricingService(IEnumerable<IDiscountStrategy> strategies)
        => _strategies = strategies;
    
    public PricingResult CalculatePrice(Order order, string customerType)
    {
        var strategy = _strategies.FirstOrDefault(s => 
            s.StrategyName.Equals(customerType, StringComparison.OrdinalIgnoreCase))
            ?? new NoDiscount();
        
        var discount = strategy.CalculateDiscount(order);
        return new PricingResult(order.Total, discount, order.Total - discount, strategy.StrategyName);
    }
}

// DI Registration:
builder.Services.AddScoped<IDiscountStrategy, PremiumCustomerDiscount>();
builder.Services.AddScoped<IDiscountStrategy, VIPCustomerDiscount>();
builder.Services.AddScoped<IDiscountStrategy, SeasonalDiscount>(sp =>
    new SeasonalDiscount(threshold: 1000, discountPercent: 0.15m));
```

### 3. When NOT to Use Strategy

- When you have 2 fixed, never-changing algorithms (over-engineering)
- When the algorithm differences are trivial (lambda is enough)

---

## Observer Pattern (Event-Driven)

### 1. Definition

Defines a one-to-many dependency between objects so when one object changes state, all dependents are notified and updated automatically.

### 2. Production Example: Order Placed → Multiple Reactions

```csharp
// C# Events as Observer Pattern:
public class Order
{
    // The SUBJECT exposes an event (observable)
    public event EventHandler<OrderPlacedEventArgs>? OrderPlaced;
    
    private OrderStatus _status;
    
    public void Place()
    {
        _status = OrderStatus.Placed;
        // Notify all observers
        OrderPlaced?.Invoke(this, new OrderPlacedEventArgs(Id, CustomerId, Total));
    }
}

// OBSERVERS register and react:
public class WarehouseNotifier
{
    public WarehouseNotifier(Order order)
    {
        order.OrderPlaced += OnOrderPlaced; // Subscribe
    }
    
    private async void OnOrderPlaced(object? sender, OrderPlacedEventArgs e)
    {
        await NotifyWarehouseAsync(e.OrderId);
    }
}

// Domain Events approach (better for production):
public record OrderPlacedEvent(Guid OrderId, string CustomerId, decimal Total, DateTimeOffset OccurredAt)
    : IDomainEvent;

public class Order
{
    private readonly List<IDomainEvent> _domainEvents = new();
    public IReadOnlyList<IDomainEvent> DomainEvents => _domainEvents.AsReadOnly();
    
    public void Place()
    {
        Status = OrderStatus.Placed;
        _domainEvents.Add(new OrderPlacedEvent(Id, CustomerId, Total, DateTimeOffset.UtcNow));
    }
    
    public void ClearDomainEvents() => _domainEvents.Clear();
}

// After SaveChanges, publish domain events:
public class DomainEventDispatcher
{
    private readonly IMediator _mediator;
    
    public async Task DispatchAsync(IEnumerable<IDomainEvent> events)
    {
        foreach (var @event in events)
            await _mediator.Publish(@event);
    }
}

// Handlers — multiple observers for one event:
public class OrderPlacedEmailHandler : INotificationHandler<OrderPlacedEvent>
{
    public async Task Handle(OrderPlacedEvent notification, CancellationToken ct)
        => await SendOrderConfirmationEmailAsync(notification);
}

public class OrderPlacedInventoryHandler : INotificationHandler<OrderPlacedEvent>
{
    public async Task Handle(OrderPlacedEvent notification, CancellationToken ct)
        => await ReserveInventoryAsync(notification.OrderId);
}

public class OrderPlacedAnalyticsHandler : INotificationHandler<OrderPlacedEvent>
{
    public async Task Handle(OrderPlacedEvent notification, CancellationToken ct)
        => await TrackConversionAsync(notification);
}
// All three react to the same event — decoupled!
```

### 3. Observer vs Events vs Kafka

| | C# Events | MediatR Notifications | Kafka Messages |
|---|---|---|---|
| Scope | In-process | In-process | Cross-service |
| Coupling | Tight (direct reference) | Loose (interface) | Fully decoupled |
| Persistence | None | None | Durable, replayable |
| Error handling | Complex | Per-handler | Consumer groups, retry |
| Use when | Simple callbacks | In-process domain events | Cross-service, reliability needed |

---

## Command Pattern

### 1. Definition

Encapsulates a request as an object, letting you parameterize clients with different requests, queue or log requests, and support undo/redo.

### 2. Production Example: Banking Transactions with Undo

```csharp
// Command interface
public interface IBankingCommand
{
    string CommandId { get; }
    string Description { get; }
    Task ExecuteAsync();
    Task UndoAsync();
}

// Concrete commands
public class DepositCommand : IBankingCommand
{
    private readonly BankAccount _account;
    private readonly decimal _amount;
    private string? _transactionId;
    
    public string CommandId { get; } = Guid.NewGuid().ToString();
    public string Description => $"Deposit {_amount:C}";
    
    public DepositCommand(BankAccount account, decimal amount)
    {
        _account = account;
        _amount = amount;
    }
    
    public async Task ExecuteAsync()
    {
        _transactionId = await _account.DepositAsync(_amount);
    }
    
    public async Task UndoAsync()
    {
        if (_transactionId == null)
            throw new InvalidOperationException("Command not yet executed");
        await _account.WithdrawAsync(_amount, reason: $"Undo of deposit {_transactionId}");
    }
}

public class TransferCommand : IBankingCommand
{
    private readonly BankAccount _source;
    private readonly BankAccount _destination;
    private readonly decimal _amount;
    
    public string CommandId { get; } = Guid.NewGuid().ToString();
    public string Description => $"Transfer {_amount:C}";
    
    public async Task ExecuteAsync()
    {
        await _source.WithdrawAsync(_amount, reason: "Transfer out");
        await _destination.DepositAsync(_amount);
    }
    
    public async Task UndoAsync()
    {
        // Reverse: withdraw from destination, deposit back to source
        await _destination.WithdrawAsync(_amount, reason: "Undo transfer");
        await _source.DepositAsync(_amount);
    }
}

// Command Processor with undo stack and audit log
public class BankingCommandProcessor
{
    private readonly Stack<IBankingCommand> _undoStack = new();
    private readonly List<CommandAuditLog> _auditLog = new();
    private readonly ILogger<BankingCommandProcessor> _logger;
    
    public async Task ExecuteAsync(IBankingCommand command)
    {
        _logger.LogInformation("Executing command: {CommandId} - {Description}", 
            command.CommandId, command.Description);
        
        await command.ExecuteAsync();
        
        _undoStack.Push(command);
        _auditLog.Add(new CommandAuditLog(command.CommandId, command.Description, 
            DateTimeOffset.UtcNow, "EXECUTED"));
        
        _logger.LogInformation("Command executed. Stack depth: {Depth}", _undoStack.Count);
    }
    
    public async Task UndoAsync()
    {
        if (!_undoStack.TryPop(out var command))
            throw new InvalidOperationException("No commands to undo");
        
        _logger.LogInformation("Undoing command: {CommandId}", command.CommandId);
        await command.UndoAsync();
        
        _auditLog.Add(new CommandAuditLog(command.CommandId, command.Description,
            DateTimeOffset.UtcNow, "UNDONE"));
    }
    
    public IReadOnlyList<CommandAuditLog> GetAuditLog() => _auditLog.AsReadOnly();
}
```

### 3. When NOT to Use Command

- Simple CRUD without undo/redo requirement
- When MediatR already handles the request/response pattern
- When overhead of encapsulating commands is more than the benefit

---

## Mediator Pattern

### 1. Definition

Defines an object that encapsulates how a set of objects interact. Promotes loose coupling by preventing objects from referring to each other explicitly.

### 2. MediatR in ASP.NET Core — Complete Example

```csharp
// Install: dotnet add package MediatR.Extensions.Microsoft.DependencyInjection

// 1. REQUEST (Query or Command):
public record GetOrderByIdQuery(Guid OrderId) : IRequest<OrderDto?>;

public record PlaceOrderCommand(
    string CustomerId,
    string Currency,
    IReadOnlyList<OrderItemRequest> Items,
    Address ShippingAddress
) : IRequest<PlaceOrderResult>;

// 2. HANDLERS:
public class GetOrderByIdHandler : IRequestHandler<GetOrderByIdQuery, OrderDto?>
{
    private readonly IOrderRepository _repo;
    private readonly IMapper _mapper;
    
    public GetOrderByIdHandler(IOrderRepository repo, IMapper mapper)
    {
        _repo = repo;
        _mapper = mapper;
    }
    
    public async Task<OrderDto?> Handle(GetOrderByIdQuery request, CancellationToken ct)
    {
        var order = await _repo.GetByIdAsync(request.OrderId, ct);
        return order == null ? null : _mapper.Map<OrderDto>(order);
    }
}

public class PlaceOrderHandler : IRequestHandler<PlaceOrderCommand, PlaceOrderResult>
{
    private readonly IOrderService _orderService;
    private readonly IMediator _mediator;
    
    public async Task<PlaceOrderResult> Handle(PlaceOrderCommand request, CancellationToken ct)
    {
        var result = await _orderService.PlaceOrderAsync(request, ct);
        
        if (result.IsSuccess)
        {
            // Publish notification — mediator dispatches to all handlers
            await _mediator.Publish(new OrderPlacedEvent(result.OrderId, request.CustomerId), ct);
        }
        
        return result;
    }
}

// 3. NOTIFICATIONS (Observer through Mediator):
public record OrderPlacedEvent(Guid OrderId, string CustomerId) : INotification;

public class SendOrderConfirmationHandler : INotificationHandler<OrderPlacedEvent>
{
    public async Task Handle(OrderPlacedEvent notification, CancellationToken ct)
        => await SendEmailAsync(notification);
}

public class UpdateInventoryHandler : INotificationHandler<OrderPlacedEvent>
{
    public async Task Handle(OrderPlacedEvent notification, CancellationToken ct)
        => await UpdateInventoryAsync(notification.OrderId, ct);
}

// 4. PIPELINE BEHAVIORS (Cross-cutting concerns):
public class LoggingBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    private readonly ILogger<LoggingBehavior<TRequest, TResponse>> _logger;
    
    public LoggingBehavior(ILogger<LoggingBehavior<TRequest, TResponse>> logger)
        => _logger = logger;
    
    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken ct)
    {
        var requestName = typeof(TRequest).Name;
        _logger.LogInformation("Handling {RequestName}: {@Request}", requestName, request);
        var sw = Stopwatch.StartNew();
        
        var response = await next(); // Call next behavior or handler
        
        _logger.LogInformation("Handled {RequestName} in {Ms}ms", requestName, sw.ElapsedMilliseconds);
        return response;
    }
}

public class ValidationBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    private readonly IEnumerable<IValidator<TRequest>> _validators;
    
    public ValidationBehavior(IEnumerable<IValidator<TRequest>> validators)
        => _validators = validators;
    
    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken ct)
    {
        if (!_validators.Any()) return await next();
        
        var context = new ValidationContext<TRequest>(request);
        var failures = _validators
            .Select(v => v.Validate(context))
            .SelectMany(r => r.Errors)
            .Where(f => f != null)
            .ToList();
        
        if (failures.Any())
            throw new ValidationException(failures);
        
        return await next();
    }
}

// 5. REGISTRATION:
builder.Services.AddMediatR(cfg => 
    cfg.RegisterServicesFromAssembly(typeof(Program).Assembly));
builder.Services.AddTransient(typeof(IPipelineBehavior<,>), typeof(LoggingBehavior<,>));
builder.Services.AddTransient(typeof(IPipelineBehavior<,>), typeof(ValidationBehavior<,>));

// 6. CONTROLLER — thin, uses mediator:
[ApiController]
[Route("api/orders")]
public class OrdersController : ControllerBase
{
    private readonly IMediator _mediator;
    
    public OrdersController(IMediator mediator) => _mediator = mediator;
    
    [HttpGet("{id}")]
    public async Task<ActionResult<OrderDto>> GetById(Guid id, CancellationToken ct)
    {
        var order = await _mediator.Send(new GetOrderByIdQuery(id), ct);
        return order == null ? NotFound() : Ok(order);
    }
    
    [HttpPost]
    public async Task<ActionResult<PlaceOrderResult>> PlaceOrder(
        [FromBody] PlaceOrderCommand command, CancellationToken ct)
    {
        var result = await _mediator.Send(command, ct);
        return result.IsSuccess 
            ? CreatedAtAction(nameof(GetById), new { id = result.OrderId }, result)
            : BadRequest(result.ErrorMessage);
    }
}
```

### 4. Pipeline Execution Flow

```
Controller → _mediator.Send(PlaceOrderCommand)
  → ValidationBehavior.Handle() [validates request]
    → LoggingBehavior.Handle() [logs request, starts timer]
      → PlaceOrderHandler.Handle() [business logic]
        → _mediator.Publish(OrderPlacedEvent)
          → SendOrderConfirmationHandler.Handle()
          → UpdateInventoryHandler.Handle()
      ← PlaceOrderHandler returns PlaceOrderResult
    ← LoggingBehavior [logs completion time]
  ← ValidationBehavior [passes through]
← Controller returns 201 Created
```

### 5. When NOT to Use MediatR/Mediator

- When it adds indirection for simple CRUD with no cross-cutting concerns
- When request/handler mapping is 1:1 and never changes (just call the service directly)
- When debugging becomes hard because the "chain" is implicit
- Over-used in small apps — becomes "enterprise-itis"

---

## Chain of Responsibility

### 1. Definition

Passes requests along a chain of handlers. Each handler decides either to process the request or pass it to the next handler.

### 2. Middleware IS Chain of Responsibility

```csharp
// ASP.NET Core Middleware = Chain of Responsibility
app.UseAuthentication();    // Handler 1: Check JWT
app.UseAuthorization();     // Handler 2: Check permissions  
app.UseRateLimiting();      // Handler 3: Check rate limits
app.UseResponseCaching();   // Handler 4: Return cached response
app.MapControllers();       // Handler 5: Process actual request

// Each middleware: process → pass to next → or short-circuit
```

### 3. Custom Validation Pipeline for Payment Orders

```csharp
public abstract class PaymentValidationHandler
{
    private PaymentValidationHandler? _next;
    
    public PaymentValidationHandler SetNext(PaymentValidationHandler next)
    {
        _next = next;
        return next; // Enables chaining: handler1.SetNext(handler2).SetNext(handler3)
    }
    
    public async Task<ValidationResult> ValidateAsync(PaymentRequest request)
    {
        var result = await HandleAsync(request);
        
        // If this handler passes (or chooses to continue), pass to next
        if (result.IsValid && _next != null)
            return await _next.ValidateAsync(request);
        
        return result;
    }
    
    protected abstract Task<ValidationResult> HandleAsync(PaymentRequest request);
}

public class AmountValidationHandler : PaymentValidationHandler
{
    protected override Task<ValidationResult> HandleAsync(PaymentRequest request)
    {
        if (request.Amount <= 0)
            return Task.FromResult(ValidationResult.Failure("Amount must be positive"));
        if (request.Amount > 1_000_000)
            return Task.FromResult(ValidationResult.Failure("Amount exceeds maximum limit"));
        return Task.FromResult(ValidationResult.Success());
    }
}

public class FraudDetectionHandler : PaymentValidationHandler
{
    private readonly IFraudDetectionService _fraudService;
    
    public FraudDetectionHandler(IFraudDetectionService fraudService)
        => _fraudService = fraudService;
    
    protected override async Task<ValidationResult> HandleAsync(PaymentRequest request)
    {
        var score = await _fraudService.GetFraudScoreAsync(request.CustomerId, request.Amount);
        return score > 0.85 
            ? ValidationResult.Failure("Transaction flagged as potentially fraudulent")
            : ValidationResult.Success();
    }
}

public class DailyLimitHandler : PaymentValidationHandler
{
    private readonly ITransactionRepository _repo;
    private const decimal DailyLimit = 50_000m;
    
    protected override async Task<ValidationResult> HandleAsync(PaymentRequest request)
    {
        var todayTotal = await _repo.GetDailyTotalAsync(request.CustomerId, DateOnly.FromDateTime(DateTime.Today));
        return todayTotal + request.Amount > DailyLimit
            ? ValidationResult.Failure($"Daily limit of {DailyLimit:C} exceeded")
            : ValidationResult.Success();
    }
}

// Build the chain:
public class PaymentValidationPipeline
{
    private readonly PaymentValidationHandler _chain;
    
    public PaymentValidationPipeline(IFraudDetectionService fraud, ITransactionRepository repo)
    {
        var amountHandler = new AmountValidationHandler();
        var fraudHandler = new FraudDetectionHandler(fraud);
        var limitHandler = new DailyLimitHandler(repo);
        
        // Chain: Amount → Fraud → DailyLimit
        amountHandler.SetNext(fraudHandler).SetNext(limitHandler);
        _chain = amountHandler;
    }
    
    public Task<ValidationResult> ValidateAsync(PaymentRequest request)
        => _chain.ValidateAsync(request);
}
```

### 4. When NOT to Use Chain of Responsibility

- When ALL handlers should process (use Observer instead)
- When order doesn't matter (use a collection of validators)
- When a single handler always handles (no chain needed)

---

## Template Method Pattern

### 1. Definition

Defines the **skeleton of an algorithm** in a base class, deferring some steps to subclasses. Subclasses override specific steps without changing the algorithm's structure.

### 2. Production Example: Payment Processing Pipeline

```csharp
public abstract class PaymentProcessor
{
    private readonly ILogger _logger;
    
    protected PaymentProcessor(ILogger logger) => _logger = logger;
    
    // TEMPLATE METHOD — skeleton defined here, FINAL (sealed would prevent override)
    public async Task<PaymentResult> ProcessAsync(PaymentRequest request)
    {
        _logger.LogInformation("[{Gateway}] Processing payment {Amount}", 
            GatewayName, request.Amount);
        
        // Step 1: Validate (can be overridden for gateway-specific validation)
        var validation = await ValidateAsync(request);
        if (!validation.IsValid)
            return PaymentResult.Failed(validation.Errors);
        
        // Step 2: Prepare request (gateway-specific transformation)
        var gatewayRequest = await PrepareRequestAsync(request);
        
        // Step 3: Execute charge (MUST be implemented by subclass)
        var response = await ExecuteChargeAsync(gatewayRequest);
        
        // Step 4: Process response (gateway-specific parsing)
        var result = ParseResponse(response);
        
        // Step 5: Post-process (optional hook — default does nothing)
        await PostProcessAsync(request, result);
        
        _logger.LogInformation("[{Gateway}] Payment result: {Success}", GatewayName, result.IsSuccess);
        return result;
    }
    
    protected abstract string GatewayName { get; }
    
    // Concrete — same for all gateways
    protected virtual Task<ValidationResult> ValidateAsync(PaymentRequest request)
    {
        if (request.Amount <= 0)
            return Task.FromResult(ValidationResult.Failure("Amount must be positive"));
        return Task.FromResult(ValidationResult.Success());
    }
    
    // Abstract — MUST implement
    protected abstract Task<object> PrepareRequestAsync(PaymentRequest request);
    protected abstract Task<object> ExecuteChargeAsync(object gatewayRequest);
    protected abstract PaymentResult ParseResponse(object response);
    
    // Hook — optional override
    protected virtual Task PostProcessAsync(PaymentRequest request, PaymentResult result)
        => Task.CompletedTask;
}

public class StripePaymentProcessor : PaymentProcessor
{
    private readonly IStripeClient _stripe;
    
    protected override string GatewayName => "Stripe";
    
    protected override async Task<object> PrepareRequestAsync(PaymentRequest request)
        => new ChargeCreateOptions 
        { 
            Amount = (long)(request.Amount * 100), // Stripe uses cents
            Currency = request.Currency,
            Customer = request.StripeCustomerId
        };
    
    protected override async Task<object> ExecuteChargeAsync(object gatewayRequest)
        => await _stripe.Charges.CreateAsync((ChargeCreateOptions)gatewayRequest);
    
    protected override PaymentResult ParseResponse(object response)
    {
        var charge = (Charge)response;
        return charge.Status == "succeeded"
            ? PaymentResult.Success(charge.Id)
            : PaymentResult.Failed(charge.FailureMessage);
    }
    
    protected override async Task PostProcessAsync(PaymentRequest request, PaymentResult result)
    {
        if (result.IsSuccess)
            await UpdateStripeMetadataAsync(result.TransactionId, request.OrderId);
    }
}
```

---

## State Pattern

### 1. Definition

Allows an object to alter its behavior when its internal state changes. The object will appear to change its class.

### 2. Production Example: Order State Machine

```csharp
// State interface
public interface IOrderState
{
    string StateName { get; }
    Task ConfirmAsync(Order order);
    Task ProcessAsync(Order order);
    Task ShipAsync(Order order, string trackingNumber);
    Task DeliverAsync(Order order);
    Task CancelAsync(Order order, string reason);
}

// Base state with default behavior (throw for invalid transitions)
public abstract class OrderStateBase : IOrderState
{
    public abstract string StateName { get; }
    
    protected void ThrowInvalidTransition(string action)
        => throw new InvalidOperationException(
            $"Cannot perform '{action}' when order is in '{StateName}' state");
    
    public virtual Task ConfirmAsync(Order order) { ThrowInvalidTransition("Confirm"); return Task.CompletedTask; }
    public virtual Task ProcessAsync(Order order) { ThrowInvalidTransition("Process"); return Task.CompletedTask; }
    public virtual Task ShipAsync(Order order, string trackingNumber) { ThrowInvalidTransition("Ship"); return Task.CompletedTask; }
    public virtual Task DeliverAsync(Order order) { ThrowInvalidTransition("Deliver"); return Task.CompletedTask; }
    public virtual Task CancelAsync(Order order, string reason) { ThrowInvalidTransition("Cancel"); return Task.CompletedTask; }
}

// Concrete states
public class PendingState : OrderStateBase
{
    public override string StateName => "Pending";
    
    public override Task ConfirmAsync(Order order)
    {
        order.TransitionTo(new ConfirmedState());
        order.ConfirmedAt = DateTimeOffset.UtcNow;
        return Task.CompletedTask;
    }
    
    public override Task CancelAsync(Order order, string reason)
    {
        order.TransitionTo(new CancelledState());
        order.CancellationReason = reason;
        order.CancelledAt = DateTimeOffset.UtcNow;
        return Task.CompletedTask;
    }
}

public class ConfirmedState : OrderStateBase
{
    public override string StateName => "Confirmed";
    
    public override async Task ProcessAsync(Order order)
    {
        // Process payment
        var paymentResult = await order.PaymentGateway.ProcessAsync(
            new PaymentRequest { Amount = order.Total });
        
        if (!paymentResult.IsSuccess)
            throw new PaymentFailedException(paymentResult.FailureReason);
        
        order.TransactionId = paymentResult.TransactionId;
        order.TransitionTo(new ProcessingState());
    }
    
    public override Task CancelAsync(Order order, string reason)
    {
        order.TransitionTo(new CancelledState());
        order.CancellationReason = reason;
        return Task.CompletedTask;
    }
}

public class ProcessingState : OrderStateBase
{
    public override string StateName => "Processing";
    
    public override Task ShipAsync(Order order, string trackingNumber)
    {
        order.TrackingNumber = trackingNumber;
        order.ShippedAt = DateTimeOffset.UtcNow;
        order.TransitionTo(new ShippedState());
        return Task.CompletedTask;
    }
    // Cannot cancel once processing started (business rule enforced by state)
}

public class ShippedState : OrderStateBase
{
    public override string StateName => "Shipped";
    
    public override Task DeliverAsync(Order order)
    {
        order.DeliveredAt = DateTimeOffset.UtcNow;
        order.TransitionTo(new DeliveredState());
        return Task.CompletedTask;
    }
}

public class DeliveredState : OrderStateBase
{
    public override string StateName => "Delivered";
    // Terminal state — no more transitions (all methods throw via base)
}

public class CancelledState : OrderStateBase
{
    public override string StateName => "Cancelled";
    // Terminal state
}

// Order delegates to current state:
public class Order
{
    private IOrderState _state;
    
    public Order() => _state = new PendingState();
    
    public string CurrentState => _state.StateName;
    
    internal void TransitionTo(IOrderState newState)
    {
        _logger.LogInformation("Order {Id} transitioning from {From} to {To}",
            Id, _state.StateName, newState.StateName);
        _state = newState;
        Status = Enum.Parse<OrderStatus>(newState.StateName);
    }
    
    // Delegates to state — ORDER doesn't know how to transition, STATE does
    public Task ConfirmAsync() => _state.ConfirmAsync(this);
    public Task ProcessAsync() => _state.ProcessAsync(this);
    public Task ShipAsync(string trackingNumber) => _state.ShipAsync(this, trackingNumber);
    public Task DeliverAsync() => _state.DeliverAsync(this);
    public Task CancelAsync(string reason) => _state.CancelAsync(this, reason);
}

// Usage — state machine in action:
var order = new Order();
await order.ConfirmAsync();        // Pending → Confirmed ✓
await order.ProcessAsync();        // Confirmed → Processing ✓
await order.ShipAsync("TRK-123"); // Processing → Shipped ✓
await order.CancelAsync("...");    // InvalidOperationException! Cannot cancel shipped order ✓
```

### 3. When NOT to Use State Pattern

- When states are few (2-3) and transitions are simple (enum + switch is fine)
- When states don't have behavior differences (just data differences)
- When a state machine library (Stateless, MassTransit state machine) already handles it

---

# OBSERVER VS MEDIATOR VS EVENTS — COMPARISON

| Dimension | C# Events | Domain Events + Mediator | Kafka/Message Bus |
|---|---|---|---|
| **Coupling** | Tight (subscriber holds reference to publisher) | Loose (through IMediator) | Fully decoupled (broker in between) |
| **Scope** | In-process only | In-process only | Cross-process, cross-service |
| **Persistence** | None | None | Durable, replayable |
| **Error handling** | Exceptions propagate to publisher | Per-handler, configurable | Consumer groups, DLQ, retry |
| **Ordering** | Guaranteed (synchronous) | Configurable | Partition-ordered |
| **Transaction** | Same transaction | Same transaction | Separate (outbox pattern needed) |
| **Use when** | Simple UI callbacks, tight coupling OK | In-process domain events, CQRS | Cross-service, reliability critical |
| **Example** | Button click handlers | OrderPlaced → Email + Inventory | Payment processed → Fulfillment service |

## When to Use Each

```
C# Events:
  ✓ UI components (PropertyChanged)
  ✓ Simple pub-sub within one object graph
  ✗ Cross-service communication
  ✗ When handlers need DI services

Domain Events + MediatR:
  ✓ Rich domain model events (Order.Place() → OrderPlacedEvent)
  ✓ CQRS command/query separation
  ✓ Pipeline behaviors (logging, validation, auth)
  ✗ Cross-service communication
  ✗ When durability is required

Kafka/Service Bus:
  ✓ Cross-microservice communication
  ✓ When event history/replay is needed
  ✓ When consumers are separate services
  ✓ High-throughput event streaming
  ✗ Simple in-process notifications (overkill)
  ✗ When transaction atomicity is required without outbox pattern
```

---

# PATTERNS QUICK REFERENCE TABLE

| Pattern | Category | Problem | Key Participants | .NET Example |
|---|---|---|---|---|
| Singleton | Creational | One shared instance | Singleton class, DI container | `AddSingleton<>`, `IMemoryCache` |
| Factory Method | Creational | Who creates the object? | Creator, Product | `IPaymentGatewayFactory` |
| Abstract Factory | Creational | Families of objects | Abstract Factory, Products | `IDatabaseProviderFactory` |
| Builder | Creational | Complex construction | Builder, Director, Product | `Order.Builder`, Fluent builders |
| Repository | Structural | Separate data access | Repository, UoW, DbContext | `IOrderRepository` + EF Core |
| Decorator | Structural | Add behavior dynamically | Component, Decorator | Caching/Logging wrappers |
| Adapter | Structural | Incompatible interfaces | Adapter, Adaptee, Target | Legacy SDK adapters |
| Facade | Structural | Simplify complex subsystem | Facade, Subsystems | `CheckoutFacade` |
| Strategy | Behavioral | Algorithm variations | Strategy, Context | `IDiscountStrategy` |
| Observer | Behavioral | Reactive notifications | Subject, Observers | `event`, MediatR INotification |
| Command | Behavioral | Encapsulate operations | Command, Invoker, Receiver | `IBankingCommand`, undo stack |
| Mediator | Behavioral | Reduce coupling | Mediator, Colleagues | MediatR, `IMediator` |
| Chain of Resp. | Behavioral | Sequential processing | Handler chain | Middleware, validation pipeline |
| Template Method | Behavioral | Algorithm skeleton | Abstract class | `PaymentProcessor` base |
| State | Behavioral | State-dependent behavior | Context, States | `OrderState` machine |

---

# ANTI-PATTERNS TO KNOW FOR INTERVIEWS

| Anti-Pattern | What It Is | Better Alternative |
|---|---|---|
| God Object | One class does everything | SRP, decompose |
| Magic Numbers | Hardcoded values | Named constants, config |
| Spaghetti Code | No structure, flow jumps everywhere | Structured patterns, layers |
| Anemic Domain Model | Data objects, logic in services | Rich domain model |
| Primitive Obsession | Using strings/ints for domain concepts | Value objects, strong types |
| Leaky Abstraction | Implementation details bleed through interface | Better encapsulation |
| Big Ball of Mud | No architecture, all mixed | Layered/clean architecture |
| Cargo Cult Programming | Copying patterns without understanding | Learn the WHY |

---

# SENIOR INTERVIEW: "WHICH PATTERN WOULD YOU USE AND WHY?"

**Scenario: Order Management System — Add logging + caching to OrderService without changing it.**
> Use **Decorator Pattern**. Implement `LoggingOrderService : IOrderService` and `CachingOrderService : IOrderService`, each wrapping the previous. Register with Scrutor's `.Decorate<>()`. No modification to existing OrderService — Open/Closed Principle satisfied.

**Scenario: Payment gateway — support Stripe, PayPal, Razorpay. New ones will be added.**
> Use **Strategy Pattern** + **Factory Method**. Each gateway implements `IPaymentStrategy`. `PaymentGatewayFactory` resolves the correct strategy from DI. Adding a new gateway = add a class + register in DI. Zero modification to existing code.

**Scenario: Order status — Pending, Confirmed, Processing, Shipped, Delivered. Invalid transitions must be rejected.**
> Use **State Pattern**. Each status is a state class. Order delegates all state-transition calls to the current state. Invalid transitions throw `InvalidOperationException`. State machines are also a solution (Stateless library).

**Scenario: Order placed → notify warehouse, send email, update analytics, award loyalty points.**
> Use **Observer/Mediator Pattern**. Publish `OrderPlacedEvent` via MediatR after saving. Multiple `INotificationHandler<OrderPlacedEvent>` implementations handle each concern independently. Decoupled, extensible, independently testable.

**Scenario: Build complex Order object with 10+ optional properties, validation at creation time.**
> Use **Builder Pattern** with fluent API. Required params in constructor, optional via fluent methods, validation in `Build()`. Prevents invalid objects from ever being created.

---

*Volume 5 Complete — All Creational, Structural, and Behavioral patterns covered with production-quality C# examples.*
