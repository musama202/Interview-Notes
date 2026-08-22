# Design Patterns Questions

---
### Q1. What is Dependency Injection (DI)?

**Answer:**  
**Dependency Injection** is a design pattern where an object receives its dependencies  
from **outside** rather than creating them itself.  
ASP.NET Core has DI **built-in** via the IoC container.

---

#### ❌ Without DI — Tightly Coupled

```csharp
public class OrderService
{
    private EmailService _email = new EmailService(); // ❌ hard-coded dependency

    public void PlaceOrder()
    {
        _email.Send("Order placed!");
    }
}
```
> Problem: Can't swap `EmailService`, can't unit test, hard to maintain.

---

#### ✅ With DI — Loosely Coupled

```csharp
public class OrderService
{
    private readonly IEmailService _email;

    // ✅ dependency injected via constructor
    public OrderService(IEmailService email)
    {
        _email = email;
    }

    public void PlaceOrder()
    {
        _email.Send("Order placed!");
    }
}
```

---

#### 📌 Register in Program.cs

```csharp
builder.Services.AddScoped<IEmailService, EmailService>();
//                         ↑ interface      ↑ concrete class
```

---

#### ⏳ 3 Lifetimes — Must Know!

| Lifetime | Method | Created | Use For |
|---|---|---|---|
| Transient | `AddTransient` | Every time requested | Lightweight, stateless services |
| Scoped | `AddScoped` | Once per HTTP request | DB context, repositories |
| Singleton | `AddSingleton` | Once for app lifetime | Config, caching, logging |

```csharp
builder.Services.AddTransient<IMyService, MyService>();   // new instance every time
builder.Services.AddScoped<IMyService, MyService>();      // new instance per request
builder.Services.AddSingleton<IMyService, MyService>();   // one instance forever
```

---

#### 🧠 Memory Tip
> **Transient** = disposable cup (new every time)  
> **Scoped** = restaurant table (yours for the meal / request)  
> **Singleton** = restaurant owner (one person, always there)

---

#### 💡 Key Benefits
- ✅ Loose coupling — easy to swap implementations
- ✅ Testable — inject mocks in unit tests
- ✅ Follows **SOLID** — especially Dependency Inversion Principle (D)

### Q2. What is the Factory Design Pattern, when should we use it, and how is it different from Dependency Injection or Keyed Services in .NET? Provide a simple example.

**Answer:**  
The **Factory Design Pattern** is a **creational design pattern** that provides a way to **create objects without exposing the instantiation logic to the client**. Instead of using `new` directly, the client asks a **factory** to create the required object.

**Why we use it?**

- Encapsulates **object creation logic**
- Reduces **tight coupling** between client and concrete classes
- Supports the **Open/Closed Principle**
- Makes the system easier to **extend and maintain**

**Factory vs Dependency Injection vs Keyed Services**

| Concept | Purpose |
|------|------|
| Dependency Injection | Manages dependencies automatically |
| Keyed Services (.NET 8+) | Selects an implementation from the DI container using a key |
| Factory Pattern | Handles **custom or complex object creation logic** |

Factories are still useful when **runtime parameters or conditional logic** are required.

---

**Example:**
```csharp
// Product Interface
public interface INotification
{
    void Send();
}
// Concrete Products
public class EmailNotification : INotification
{
    public void Send() => Console.WriteLine("Email sent");
}

public class SmsNotification : INotification
{
    public void Send() => Console.WriteLine("SMS sent");
}
// Factory
public class NotificationFactory
{
    public static INotification CreateNotification(string type)
    {
        return type switch
        {
            "Email" => new EmailNotification(),
            "SMS"   => new SmsNotification(),
            _       => throw new ArgumentException("Invalid type")
        };
    }
}
// Client Code
public class Program
{
    public static void Main()
    {
        INotification email = NotificationFactory.CreateNotification("Email");
        email.Send();
    }
}
```
----------------------------

### Q3. What is Singleton Pattern?

**Answer:**  
**Singleton** ensures a class has **only one instance** throughout the app lifetime  
and provides a **global access point** to it.

---

#### ❌ Without Singleton — Multiple Instances (Problem)

```csharp
var config1 = new ConfigManager(); // new instance
var config2 = new ConfigManager(); // another new instance ❌

// config1 != config2 — different objects, inconsistent state
```

---

#### ✅ Singleton Implementation

```csharp
public class ConfigManager
{
    // 1️⃣ static instance — lives forever
    private static ConfigManager _instance;

    // 2️⃣ private constructor — no one can do "new ConfigManager()"
    private ConfigManager() { }

    // 3️⃣ global access point
    public static ConfigManager Instance
    {
        get
        {
            if (_instance == null)
                _instance = new ConfigManager();
            return _instance;
        }
    }

    public string AppName { get; set; } = "MyApp";
}
```

#### 📌 Usage

```csharp
var config = ConfigManager.Instance;
config.AppName = "MyApp v2";

// anywhere else in app
var same = ConfigManager.Instance;
Console.WriteLine(same.AppName); // ✅ "MyApp v2" — same instance
```

---

#### ⚠️ Thread-Safe Singleton (Use This in Production)

```csharp
public class ConfigManager
{
    private static readonly Lazy<ConfigManager> _instance =
        new Lazy<ConfigManager>(() => new ConfigManager());

    private ConfigManager() { }

    public static ConfigManager Instance => _instance.Value;
}
```
> `Lazy<T>` — creates instance only when first needed + thread-safe ✅

---

#### 🏆 Singleton in ASP.NET Core DI (Recommended Way)

```csharp
// Register — framework manages the single instance for you
builder.Services.AddSingleton<IConfigManager, ConfigManager>();

// Use via constructor injection
public class HomeController : ControllerBase
{
    private readonly IConfigManager _config;

    public HomeController(IConfigManager config)
    {
        _config = config; // ✅ always same instance
    }
}
```
> ✅ Prefer DI Singleton over manual implementation in ASP.NET Core apps.

---

#### 🔁 Singleton vs Other Lifetimes (Quick Recap)

| Lifetime | Instances | 
|---|---|
| Transient | New every time |
| Scoped | New per request |
| **Singleton** | **One forever** |

---

#### ⚠️ When NOT to Use Singleton
- ❌ Storing **user-specific** data (shared across all users!)
- ❌ **DB context** — use Scoped instead
- ❌ Heavy objects with **changing state**

---

#### 🧠 Memory Tip
> Singleton = **President of a country**  
> Only ONE exists at a time, everyone talks to the same person,  
> and they stay until the app (term) ends. 🏛️

### Q4. What is Repository Pattern?

**Answer:**  
**Repository Pattern** is an abstraction layer between your **business logic** and **data access logic**.  
Instead of writing DB queries directly in controllers/services, you write them in a **repository class**.

---

#### ❌ Without Repository — DB logic in Controller (Bad)

```csharp
public class OrderController : ControllerBase
{
    private readonly AppDbContext _db;

    public OrderController(AppDbContext db) { _db = db; }

    public IActionResult GetOrder(int id)
    {
        // ❌ DB query directly in controller — hard to test, duplicated everywhere
        var order = _db.Orders.FirstOrDefault(o => o.Id == id);
        return Ok(order);
    }
}
```
> Problem: DB logic scattered everywhere, can't unit test, hard to swap DB later.

---

#### ✅ Step 1 — Create the Interface

```csharp
public interface IOrderRepository
{
    Order GetById(int id);
    IEnumerable<Order> GetAll();
    void Add(Order order);
    void Delete(int id);
    void Save();
}
```

---

#### ✅ Step 2 — Implement the Repository

```csharp
public class OrderRepository : IOrderRepository
{
    private readonly AppDbContext _db;

    public OrderRepository(AppDbContext db) { _db = db; }

    public Order GetById(int id) => _db.Orders.FirstOrDefault(o => o.Id == id);

    public IEnumerable<Order> GetAll() => _db.Orders.ToList();

    public void Add(Order order) => _db.Orders.Add(order);

    public void Delete(int id)
    {
        var order = GetById(id);
        if (order != null) _db.Orders.Remove(order);
    }

    public void Save() => _db.SaveChanges();
}
```

---

#### ✅ Step 3 — Register in Program.cs

```csharp
builder.Services.AddScoped<IOrderRepository, OrderRepository>();
// Scoped ✅ — one instance per HTTP request (same as DbContext)
```

---

#### ✅ Step 4 — Use in Controller (Clean!)

```csharp
public class OrderController : ControllerBase
{
    private readonly IOrderRepository _repo;

    public OrderController(IOrderRepository repo) { _repo = repo; }

    public IActionResult GetOrder(int id)
    {
        var order = _repo.GetById(id); // ✅ no DB logic here
        return Ok(order);
    }

    public IActionResult AddOrder(Order order)
    {
        _repo.Add(order);
        _repo.Save();
        return Ok();
    }
}
```

---

#### 🏆 Generic Repository (Bonus — Advanced)

```csharp
public interface IRepository<T> where T : class
{
    T GetById(int id);
    IEnumerable<T> GetAll();
    void Add(T entity);
    void Delete(int id);
    void Save();
}

public class Repository<T> : IRepository<T> where T : class
{
    private readonly AppDbContext _db;
    private readonly DbSet<T> _dbSet;

    public Repository(AppDbContext db)
    {
        _db = db;
        _dbSet = db.Set<T>();
    }

    public T GetById(int id) => _dbSet.Find(id);
    public IEnumerable<T> GetAll() => _dbSet.ToList();
    public void Add(T entity) => _dbSet.Add(entity);
    public void Delete(int id)
    {
        var entity = GetById(id);
        if (entity != null) _dbSet.Remove(entity);
    }
    public void Save() => _db.SaveChanges();
}
```

#### 📌 Register Generic Repository

```csharp
// register once — works for ALL entities
builder.Services.AddScoped(typeof(IRepository<>), typeof(Repository<>));

// usage
public class OrderController : ControllerBase
{
    private readonly IRepository<Order> _repo;
    public OrderController(IRepository<Order> repo) { _repo = repo; }
}
```

---

#### 🔁 Repository vs Direct DbContext

| | Direct DbContext | Repository Pattern |
|---|---|---|
| DB logic location | Everywhere ❌ | One place ✅ |
| Unit testable | Hard ❌ | Easy (mock repo) ✅ |
| Swap DB later | Hard ❌ | Easy ✅ |
| Code duplication | High ❌ | None ✅ |

---

#### 🧠 Memory Tip
> Repository = **Warehouse manager** 🏭  
> Your controller (boss) says *"get me order #5"*  
> Manager handles where/how to fetch it — boss doesn't care if it's SQL, MongoDB or cache.


### Q5. What is Unit of Work Design Pattern?

**Answer:**  
**Unit of Work** groups **multiple repository operations** into a **single transaction**.  
Either **all succeed** together or **all fail** together — no partial saves.

---

#### ❌ Without Unit of Work — Partial Save Problem

```csharp
public class OrderService
{
    private readonly IOrderRepository _orderRepo;
    private readonly IInventoryRepository _inventoryRepo;

    public void PlaceOrder(Order order)
    {
        _orderRepo.Add(order);
        _orderRepo.Save();         // ✅ saved

        _inventoryRepo.Reduce(order.ProductId);
        _inventoryRepo.Save();     // ❌ crashes here!

        // Order saved but inventory NOT reduced — data inconsistent 😱
    }
}
```
> Problem: Multiple `SaveChanges()` calls = risk of **partial commits** and **inconsistent data**.

---

#### ✅ Step 1 — Define Unit of Work Interface

```csharp
public interface IUnitOfWork : IDisposable
{
    IOrderRepository Orders { get; }
    IInventoryRepository Inventory { get; }

    int Complete(); // single SaveChanges for everything
}
```

---

#### ✅ Step 2 — Implement Unit of Work

```csharp
public class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _db;

    public IOrderRepository Orders { get; }
    public IInventoryRepository Inventory { get; }

    public UnitOfWork(AppDbContext db)
    {
        _db = db;
        Orders    = new OrderRepository(db);      // ✅ share same DbContext
        Inventory = new InventoryRepository(db);  // ✅ share same DbContext
    }

    public int Complete() => _db.SaveChanges();   // one single commit

    public void Dispose() => _db.Dispose();
}
```

---

#### ✅ Step 3 — Register in Program.cs

```csharp
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
// Scoped ✅ — one instance (and one DbContext) per HTTP request
```

---

#### ✅ Step 4 — Use in Service (Clean & Safe)

```csharp
public class OrderService
{
    private readonly IUnitOfWork _uow;

    public OrderService(IUnitOfWork uow) { _uow = uow; }

    public void PlaceOrder(Order order)
    {
        _uow.Orders.Add(order);                    // stage operation 1
        _uow.Inventory.Reduce(order.ProductId);    // stage operation 2

        _uow.Complete();
        // ✅ both saved in ONE transaction
        // ❌ if anything fails — nothing is saved
    }
}
```

---

#### 🏆 With Transaction (Extra Safety)

```csharp
public async Task PlaceOrderAsync(Order order)
{
    using var transaction = await _db.Database.BeginTransactionAsync();
    try
    {
        _uow.Orders.Add(order);
        _uow.Inventory.Reduce(order.ProductId);
        await _uow.CompleteAsync();

        await transaction.CommitAsync();   // ✅ all good — commit
    }
    catch
    {
        await transaction.RollbackAsync(); // ❌ something failed — rollback all
        throw;
    }
}
```

---

#### 🔁 Repository vs Unit of Work

| | Repository | Unit of Work |
|---|---|---|
| Purpose | Abstracts DB queries | Groups repos into one transaction |
| `SaveChanges()` | Called per repo ❌ | Called once for all ✅ |
| Manages | Single entity | Multiple entities |
| Works with | Alone | Wraps repositories |

---

#### 🔁 Full Pattern Together

```
Controller
    └── Service
            └── Unit of Work
                    ├── OrderRepository      ──┐
                    ├── InventoryRepository  ──┤──► Single DbContext
                    └── Complete()  ───────────┘    One SaveChanges()
```

---

#### 🧠 Memory Tip
> Unit of Work = **Bank Transaction** 🏦  
> Transfer money: Debit account A → Credit account B  
> If **anything fails** — both operations **roll back**.  
> You never end up with money deducted but not received. 💸

### Q6. What is Strategy Design Pattern?

**Answer:**  
**Strategy Pattern** allows you to define a **family of algorithms**, put each in a **separate class**,  
and make them **interchangeable at runtime** without changing the client code.

---

#### ❌ Without Strategy — If/Else Mess (Bad)

```csharp
public class PaymentService
{
    public void Pay(string method, decimal amount)
    {
        if (method == "CreditCard")
        {
            Console.WriteLine($"Paid {amount} via Credit Card"); // ❌
        }
        else if (method == "PayPal")
        {
            Console.WriteLine($"Paid {amount} via PayPal"); // ❌
        }
        else if (method == "Crypto")
        {
            Console.WriteLine($"Paid {amount} via Crypto"); // ❌
        }
        // adding new method = modify this class forever 😓
    }
}
```
> Problem: Violates **Open/Closed Principle** — every new payment method requires modifying existing code.

---

#### ✅ Step 1 — Define the Strategy Interface

```csharp
public interface IPaymentStrategy
{
    void Pay(decimal amount);
}
```

---

#### ✅ Step 2 — Concrete Strategies (Each Algorithm in Own Class)

```csharp
public class CreditCardPayment : IPaymentStrategy
{
    public void Pay(decimal amount) =>
        Console.WriteLine($"Paid {amount} via Credit Card 💳");
}

public class PayPalPayment : IPaymentStrategy
{
    public void Pay(decimal amount) =>
        Console.WriteLine($"Paid {amount} via PayPal 🅿️");
}

public class CryptoPayment : IPaymentStrategy
{
    public void Pay(decimal amount) =>
        Console.WriteLine($"Paid {amount} via Crypto 🪙");
}
```

---

#### ✅ Step 3 — Context Class (Uses the Strategy)

```csharp
public class PaymentService
{
    private readonly IPaymentStrategy _strategy;

    // ✅ strategy injected — swap anytime
    public PaymentService(IPaymentStrategy strategy)
    {
        _strategy = strategy;
    }

    public void ProcessPayment(decimal amount)
    {
        _strategy.Pay(amount); // no if/else — just call it
    }
}
```

---

#### ✅ Step 4 — Usage (Switch Strategy at Runtime)

```csharp
// use credit card
var service = new PaymentService(new CreditCardPayment());
service.ProcessPayment(100); // Paid 100 via Credit Card 💳

// switch to PayPal — no code change in PaymentService!
var service2 = new PaymentService(new PayPalPayment());
service2.ProcessPayment(200); // Paid 200 via PayPal 🅿️
```

---

#### 🏆 With ASP.NET Core DI (Real World)

```csharp
// Register all strategies
builder.Services.AddScoped<CreditCardPayment>();
builder.Services.AddScoped<PayPalPayment>();
builder.Services.AddScoped<CryptoPayment>();

// Controller — pick strategy based on user input
public class CheckoutController : ControllerBase
{
    private readonly IServiceProvider _provider;

    public CheckoutController(IServiceProvider provider)
    {
        _provider = provider;
    }

    public IActionResult Pay(string method, decimal amount)
    {
        IPaymentStrategy strategy = method switch
        {
            "CreditCard" => _provider.GetRequiredService<CreditCardPayment>(),
            "PayPal"     => _provider.GetRequiredService<PayPalPayment>(),
            "Crypto"     => _provider.GetRequiredService<CryptoPayment>(),
            _ => throw new ArgumentException("Unknown payment method")
        };

        var service = new PaymentService(strategy);
        service.ProcessPayment(amount);
        return Ok();
    }
}
```

---

#### 🔁 Without vs With Strategy

| | Without Strategy | With Strategy |
|---|---|---|
| New payment method | Modify existing class ❌ | Add new class only ✅ |
| Open/Closed Principle | Violated ❌ | Followed ✅ |
| Unit testable | Hard ❌ | Easy (mock strategy) ✅ |
| Runtime switching | Hard ❌ | Easy ✅ |

---

#### 🧠 Memory Tip
> Strategy = **GPS Navigation** 🗺️  
> Same destination — but you can switch route strategy anytime:  
> 🚗 Fastest / 🛣️ Highway only / 🌿 Avoid tolls  
> The **car (context)** doesn't change — only the **route algorithm (strategy)** does.



### Q7. What is Saga Design Pattern?

**Answer:**  
**Saga** manages **long-running transactions** across **multiple services** in a  
**distributed/microservices** architecture — where a single database transaction is not possible.  
Each step has a **compensating action** to undo it if something fails.

---

#### ❌ The Problem — Distributed Transaction

```
Order Service   → saves order        ✅
Payment Service → charges card       ✅
Inventory Service → reduces stock    ✅
Shipping Service → creates shipment  ❌ FAILS!

// Payment already charged, stock already reduced
// No single transaction to roll back across services 😱
```
> You **cannot** use `Unit of Work` or DB transactions across separate microservices.

---

#### ✅ Saga Solution — Each Step Has a Compensating Action

```
Step 1: Create Order       ↔ Cancel Order
Step 2: Charge Payment     ↔ Refund Payment
Step 3: Reduce Inventory   ↔ Restore Inventory
Step 4: Create Shipment    ↔ Cancel Shipment

If Step 4 fails → run compensating actions in reverse:
    ← Restore Inventory
    ← Refund Payment
    ← Cancel Order
```

---

#### 🔀 Two Types of Saga

| Type | How It Works | Use When |
|---|---|---|
| **Choreography** | Services talk via events (no central brain) | Simple flows, few services |
| **Orchestration** | Central Saga Orchestrator controls the flow | Complex flows, many services |

---

#### ✅ Type 1 — Choreography (Event-Based)

```csharp
// Order Service — publishes event
public class OrderService
{
    private readonly IEventBus _bus;

    public async Task CreateOrder(Order order)
    {
        await _orderRepo.Add(order);
        await _bus.Publish(new OrderCreatedEvent(order.Id)); // 📢 fire event
    }
}

// Payment Service — listens and reacts
public class PaymentService : IEventHandler<OrderCreatedEvent>
{
    public async Task Handle(OrderCreatedEvent e)
    {
        var success = await ChargeCard(e.OrderId);

        if (success)
            await _bus.Publish(new PaymentSucceededEvent(e.OrderId));
        else
            await _bus.Publish(new PaymentFailedEvent(e.OrderId)); // ❌ trigger compensation
    }
}

// Order Service — listens for failure and compensates
public class OrderService : IEventHandler<PaymentFailedEvent>
{
    public async Task Handle(PaymentFailedEvent e)
    {
        await CancelOrder(e.OrderId); // 🔄 compensating action
    }
}
```

---

#### ✅ Type 2 — Orchestration (Central Controller)

```csharp
public class OrderSagaOrchestrator
{
    public async Task Execute(Order order)
    {
        // Step 1
        var orderCreated = await _orderService.Create(order);
        if (!orderCreated) return; // stop

        // Step 2
        var paymentDone = await _paymentService.Charge(order);
        if (!paymentDone)
        {
            await _orderService.Cancel(order.Id);  // 🔄 compensate step 1
            return;
        }

        // Step 3
        var stockReduced = await _inventoryService.Reduce(order);
        if (!stockReduced)
        {
            await _paymentService.Refund(order.Id); // 🔄 compensate step 2
            await _orderService.Cancel(order.Id);   // 🔄 compensate step 1
            return;
        }

        // Step 4
        var shipped = await _shippingService.Create(order);
        if (!shipped)
        {
            await _inventoryService.Restore(order.Id); // 🔄 compensate step 3
            await _paymentService.Refund(order.Id);    // 🔄 compensate step 2
            await _orderService.Cancel(order.Id);      // 🔄 compensate step 1
        }
    }
}
```

---

#### 🏆 Real World — Using MassTransit (Popular in .NET)

```csharp
public class OrderStateMachine : MassTransitStateMachine<OrderState>
{
    public OrderStateMachine()
    {
        Initially(
            When(OrderSubmitted)
                .Then(ctx => ctx.Saga.OrderId = ctx.Message.OrderId)
                .TransitionTo(AwaitingPayment)
                .Publish(ctx => new ProcessPaymentCommand(ctx.Saga.OrderId))
        );

        During(AwaitingPayment,
            When(PaymentSucceeded)
                .TransitionTo(AwaitingShipment)
                .Publish(ctx => new CreateShipmentCommand(ctx.Saga.OrderId)),

            When(PaymentFailed)
                .TransitionTo(Cancelled)
                .Publish(ctx => new CancelOrderCommand(ctx.Saga.OrderId)) // 🔄 compensate
        );
    }

    public State AwaitingPayment { get; private set; }
    public State AwaitingShipment { get; private set; }
    public State Cancelled { get; private set; }

    public Event<OrderSubmittedEvent> OrderSubmitted { get; private set; }
    public Event<PaymentSucceededEvent> PaymentSucceeded { get; private set; }
    public Event<PaymentFailedEvent> PaymentFailed { get; private set; }
}
```

---

#### 🔁 Saga vs Unit of Work

| | Unit of Work | Saga |
|---|---|---|
| Scope | Single service / DB | Multiple services / DBs |
| Transaction type | ACID (DB transaction) | Eventual consistency |
| Failure handling | DB rollback | Compensating actions |
| Architecture | Monolith / single DB | Microservices |

---

#### 🔁 Choreography vs Orchestration

| | Choreography | Orchestration |
|---|---|---|
| Control | Decentralized (events) | Centralized (orchestrator) |
| Coupling | Loose ✅ | Tighter |
| Visibility | Hard to track ❌ | Easy to track ✅ |
| Complexity | Simple flows ✅ | Complex flows ✅ |

---

#### 🧠 Memory Tip
> Saga = **Wedding Planning** 💍  
> Venue booked ✅ → Catering booked ✅ → Band booked ✅ → Flowers ❌ FAILS  
> You must now **unbook the band, catering, venue** in reverse — compensating actions!  
> No single contract covers everything — each vendor is a separate service. 🎊

### Q8. What is Circuit Breaker Pattern?

**Answer:**  
**Circuit Breaker** prevents an application from **repeatedly calling a failing service**.  
Instead of hammering a dead service, it **trips open** and returns a fast failure —  
giving the failing service time to recover.

---

#### ❌ Without Circuit Breaker — Cascade Failure

```
User Request → Order Service → Payment Service ❌ (down)
                                    ↓
                             timeout after 30s 😴
                                    ↓
User Request → Order Service → Payment Service ❌ (still down)
                                    ↓
                             timeout after 30s 😴
                                    ↓
// 1000 users waiting → Order Service overwhelmed → it goes down too 💥
// One failed service takes down the entire system — Cascade Failure 😱
```

---

#### ✅ Circuit Breaker — 3 States

```
                    failures < threshold
        ┌─────────────────────────────────────┐
        ▼                                     │
    [CLOSED] ──── too many failures ────► [OPEN]
    normal flow                           fast fail
                                              │
                                     after timeout
                                              │
                                              ▼
                                        [HALF-OPEN]
                                        test one request
                                        success? → CLOSED
                                        fail?    → OPEN
```

| State | Behavior |
|---|---|
| **Closed** | Requests flow normally, failures counted |
| **Open** | All requests fail immediately (no call made) |
| **Half-Open** | One test request allowed to check recovery |

---

#### ✅ Implementation with Polly (Standard in .NET)

```csharp
// Install: dotnet add package Microsoft.Extensions.Http.Polly
```

```csharp
// Program.cs
builder.Services
    .AddHttpClient<IPaymentService, PaymentService>()
    .AddPolicyHandler(GetCircuitBreakerPolicy());

static IAsyncPolicy<HttpResponseMessage> GetCircuitBreakerPolicy()
{
    return HttpPolicyExtensions
        .HandleTransientHttpError() // handles 5xx, timeouts
        .CircuitBreakerAsync(
            handledEventsAllowedBeforeBreaking: 3,  // open after 3 failures
            durationOfBreak: TimeSpan.FromSeconds(30) // stay open for 30s
        );
}
```

---

#### 🏆 Advanced — With Fallback + Retry + Circuit Breaker

```csharp
static IAsyncPolicy<HttpResponseMessage> GetResiliencePolicy()
{
    // 1️⃣ Retry — try 3 times before giving up
    var retryPolicy = HttpPolicyExtensions
        .HandleTransientHttpError()
        .RetryAsync(3, onRetry: (result, retryCount) =>
        {
            Console.WriteLine($"Retry {retryCount}...");
        });

    // 2️⃣ Circuit Breaker — open after 5 failures, wait 60s
    var circuitBreakerPolicy = HttpPolicyExtensions
        .HandleTransientHttpError()
        .CircuitBreakerAsync(
            handledEventsAllowedBeforeBreaking: 5,
            durationOfBreak: TimeSpan.FromSeconds(60),
            onBreak: (result, duration) =>
                Console.WriteLine($"⚡ Circuit OPEN for {duration.TotalSeconds}s"),
            onReset: () =>
                Console.WriteLine("✅ Circuit CLOSED — service recovered"),
            onHalfOpen: () =>
                Console.WriteLine("🔄 Circuit HALF-OPEN — testing...")
        );

    // 3️⃣ Fallback — return default if everything fails
    var fallbackPolicy = HttpPolicyExtensions
        .HandleTransientHttpError()
        .FallbackAsync(new HttpResponseMessage(HttpStatusCode.OK)
        {
            Content = new StringContent("{ 'status': 'service unavailable' }")
        });

    // wrap all together — outermost executes first
    return Policy.WrapAsync(fallbackPolicy, retryPolicy, circuitBreakerPolicy);
}
```

---

#### ✅ Usage in Service

```csharp
public class PaymentService : IPaymentService
{
    private readonly HttpClient _client;

    public PaymentService(HttpClient client) { _client = client; }

    public async Task<string> ChargeAsync(decimal amount)
    {
        // Polly policy applied automatically via HttpClient
        var response = await _client.PostAsync("/api/charge",
            new StringContent(amount.ToString()));

        // if circuit is OPEN — Polly throws BrokenCircuitException
        // fallback policy catches it and returns default response
        return await response.Content.ReadAsStringAsync();
    }
}
```

---

#### 🏆 Polly v8 — New Resilience Pipeline (.NET 8)

```csharp
// New way in .NET 8+
builder.Services.AddResiliencePipeline("payment", builder =>
{
    builder
        .AddRetry(new RetryStrategyOptions
        {
            MaxRetryAttempts = 3,
            Delay = TimeSpan.FromSeconds(1)
        })
        .AddCircuitBreaker(new CircuitBreakerStrategyOptions
        {
            FailureRatio = 0.5,               // open if 50% requests fail
            SamplingDuration = TimeSpan.FromSeconds(10),
            MinimumThroughput = 5,            // min requests before evaluating
            BreakDuration = TimeSpan.FromSeconds(30)
        })
        .AddTimeout(TimeSpan.FromSeconds(5)); // per request timeout
});
```

---

#### 🔁 Retry vs Circuit Breaker

| | Retry | Circuit Breaker |
|---|---|---|
| Purpose | Handle temporary glitch | Handle prolonged failure |
| Behavior | Tries again immediately | Stops trying for a period |
| Best for | Network blips | Service is fully down |
| Used together | ✅ Yes | ✅ Yes |

---

#### 🔁 Without vs With Circuit Breaker

| | Without | With |
|---|---|---|
| Failing service | Hammered with requests ❌ | Gets time to recover ✅ |
| User experience | Long timeouts 😴 | Fast failure response ✅ |
| Cascade failure | Likely 💥 | Prevented ✅ |
| System resilience | Low ❌ | High ✅ |

---

#### 🧠 Memory Tip
> Circuit Breaker = **Electrical Circuit Breaker** ⚡🏠  
> Too much current (failures) → breaker **trips open** → power cut to protect wiring  
> After a while → **test** if safe → if yes, **reset** (closed) → power restored  
> Protects your house (system) from burning down (cascade failure) 🔥

### Q9. What is SOLID in C#?

**Answer:**  
**SOLID** is a set of **5 design principles** that make code  
**maintainable, scalable, and testable**.  
Each letter is one principle.

---

```
S — Single Responsibility Principle  (SRP)
O — Open/Closed Principle            (OCP)
L — Liskov Substitution Principle    (LSP)
I — Interface Segregation Principle  (ISP)
D — Dependency Inversion Principle   (DIP)
```

---

#### S — Single Responsibility Principle (SRP)
> **A class should have only ONE reason to change.**

#### ❌ Bad — One class doing too much

```csharp
public class OrderService
{
    public void CreateOrder(Order order) { ... }   // business logic
    public void SendEmail(Order order) { ... }     // ❌ email logic here too
    public void LogOrder(Order order) { ... }      // ❌ logging here too
}
// If email changes → modify OrderService ❌
// If logging changes → modify OrderService ❌
```

#### ✅ Good — Each class has one job

```csharp
public class OrderService   { public void CreateOrder(Order order) { ... } }
public class EmailService   { public void SendEmail(Order order) { ... } }
public class LoggerService  { public void LogOrder(Order order) { ... } }
// each class changes for only ONE reason ✅
```

---

#### O — Open/Closed Principle (OCP)
> **Open for extension, Closed for modification.**  
> Add new behavior without changing existing code.

#### ❌ Bad — Modify existing class for every new type

```csharp
public class DiscountService
{
    public decimal GetDiscount(string customerType)
    {
        if (customerType == "Regular") return 0.1m;      // ❌
        if (customerType == "Premium") return 0.2m;      // ❌
        if (customerType == "VIP")     return 0.3m;      // ❌
        // adding new type = modify this class forever
    }
}
```

#### ✅ Good — Extend via new class, never modify existing

```csharp
public interface IDiscount { decimal GetDiscount(); }

public class RegularDiscount : IDiscount { public decimal GetDiscount() => 0.1m; }
public class PremiumDiscount : IDiscount { public decimal GetDiscount() => 0.2m; }
public class VIPDiscount     : IDiscount { public decimal GetDiscount() => 0.3m; }

// adding Gold customer = just add new class, touch nothing else ✅
public class GoldDiscount : IDiscount { public decimal GetDiscount() => 0.4m; }
```

---

#### L — Liskov Substitution Principle (LSP)
> **Subclass should be replaceable by its parent class**  
> without breaking the program.

#### ❌ Bad — Subclass breaks parent behavior

```csharp
public class Bird
{
    public virtual void Fly() => Console.WriteLine("Flying...");
}

public class Penguin : Bird
{
    public override void Fly()
    {
        throw new Exception("Penguins can't fly!"); // ❌ breaks LSP
    }
}

// using parent reference breaks with Penguin
Bird bird = new Penguin();
bird.Fly(); // 💥 Exception!
```

#### ✅ Good — Correct abstraction

```csharp
public abstract class Bird { public abstract void Move(); }

public class Eagle   : Bird { public override void Move() => Console.WriteLine("Flying ✈️"); }
public class Penguin : Bird { public override void Move() => Console.WriteLine("Swimming 🏊"); }

// ✅ substitution works — no surprises
Bird bird = new Penguin();
bird.Move(); // Swimming 🏊 — works fine
```

---

#### I — Interface Segregation Principle (ISP)
> **No class should be forced to implement methods it does not use.**  
> Split large interfaces into smaller specific ones.

#### ❌ Bad — Fat interface forces unused methods

```csharp
public interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
}

public class Robot : IWorker
{
    public void Work()  => Console.WriteLine("Working...");
    public void Eat()   => throw new NotImplementedException(); // ❌ robots don't eat
    public void Sleep() => throw new NotImplementedException(); // ❌ robots don't sleep
}
```

#### ✅ Good — Split into focused interfaces

```csharp
public interface IWorkable  { void Work(); }
public interface IEatable   { void Eat(); }
public interface ISleepable { void Sleep(); }

public class Human : IWorkable, IEatable, ISleepable
{
    public void Work()  => Console.WriteLine("Working 💼");
    public void Eat()   => Console.WriteLine("Eating 🍔");
    public void Sleep() => Console.WriteLine("Sleeping 😴");
}

public class Robot : IWorkable
{
    public void Work() => Console.WriteLine("Working ⚙️"); // ✅ only what it needs
}
```

---

#### D — Dependency Inversion Principle (DIP)
> **Depend on abstractions (interfaces), not concrete classes.**  
> High-level modules should not depend on low-level modules.

#### ❌ Bad — Depends on concrete class

```csharp
public class OrderService
{
    private SqlDatabase _db = new SqlDatabase(); // ❌ tightly coupled

    public void SaveOrder(Order order)
    {
        _db.Save(order); // can't swap to MongoDB, can't unit test
    }
}
```

#### ✅ Good — Depends on abstraction

```csharp
public interface IDatabase { void Save(Order order); }

public class SqlDatabase     : IDatabase { public void Save(Order o) => Console.WriteLine("Saved to SQL"); }
public class MongoDatabase   : IDatabase { public void Save(Order o) => Console.WriteLine("Saved to Mongo"); }

public class OrderService
{
    private readonly IDatabase _db;

    public OrderService(IDatabase db) { _db = db; } // ✅ injected

    public void SaveOrder(Order order) => _db.Save(order);
}

// swap DB without touching OrderService ✅
var service = new OrderService(new MongoDatabase());
```

---

#### 🔁 SOLID — Quick Reference Card

| Principle | Keyword | Rule |
|---|---|---|
| **S** — SRP | One job | One class, one reason to change |
| **O** — OCP | Extend, not modify | Add new class, don't touch old |
| **L** — LSP | Substitutable | Subclass must honor parent contract |
| **I** — ISP | Small interfaces | Don't force unused methods |
| **D** — DIP | Abstractions | Depend on interface, not concrete |

---

#### 🔁 Which SOLID Principle Fixes What?

| Problem | Principle |
|---|---|
| Class doing too many things | SRP |
| Adding feature breaks existing code | OCP |
| Subclass throwing `NotImplementedException` | LSP |
| Implementing empty/useless interface methods | ISP |
| Hard to swap DB / hard to unit test | DIP |

---

#### 🧠 Memory Tip
> **S**am **O**nly **L**ikes **I**ce **D**esserts 🍦  
> **S**ingle Responsibility  
> **O**pen Closed  
> **L**iskov Substitution  
> **I**nterface Segregation  
> **D**ependency Inversion

### Q10. What is Clean Architecture in .NET?

**Answer:**  
**Clean Architecture** is a software architecture pattern that organizes an application into **layers with clear separation of concerns**.

The main goal is to make the application:

- **Maintainable**
- **Testable**
- **Scalable**
- **Loosely coupled**
- **Easy to change**

The most important rule is:

> **Dependencies should point inward toward the business/domain layer.**

---

```text
Presentation
     ↓
Application
     ↓
Domain
     ↑
Infrastructure
```

---

#### 🏗️ Clean Architecture Layers

```text
┌──────────────────────────────────────────┐
│              Presentation                │
│       ASP.NET Core Controllers           │
└────────────────────┬─────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────┐
│               Application                │
│    Use Cases / Services / DTOs / CQRS    │
└────────────────────┬─────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────┐
│                 Domain                   │
│ Entities / Value Objects / Business Rules│
└──────────────────────────────────────────┘
                     ▲
                     │
┌────────────────────┴─────────────────────┐
│             Infrastructure               │
│ EF Core / SQL / Redis / APIs / RabbitMQ  │
└──────────────────────────────────────────┘
```

---

#### 1️⃣ Domain Layer

> **Contains the core business logic and rules.**

It contains:

```text
Entities
Value Objects
Business Rules
Domain Interfaces
Enums
Domain Exceptions
```

#### Example

```csharp
public class Order
{
    public Guid Id { get; private set; }

    public decimal TotalAmount { get; private set; }

    public void AddItem(decimal price)
    {
        if (price <= 0)
            throw new ArgumentException("Price must be greater than zero");

        TotalAmount += price;
    }
}
```

#### ❌ Domain should NOT depend on:

```text
ASP.NET Core
Entity Framework
SQL Server
Redis
RabbitMQ
Azure
Controllers
HTTP
```

For example:

```csharp
public class Order
{
    private readonly AppDbContext _db; // ❌ Bad
}
```

The Domain should remain **independent**.

---

#### 2️⃣ Application Layer

> **Contains application-specific use cases and orchestration logic.**

Examples:

```text
Create Order
Get Order
Cancel Order
Process Payment
Send Notification
```

#### Example

```csharp
public class CreateOrderService
{
    private readonly IOrderRepository _repository;

    public CreateOrderService(IOrderRepository repository)
    {
        _repository = repository;
    }

    public async Task CreateOrder(Order order)
    {
        await _repository.AddAsync(order);
    }
}
```

The Application layer depends on an **abstraction**:

```csharp
IOrderRepository
```

It does not care whether the implementation uses:

```text
SQL Server
MongoDB
Dataverse
External API
```

---

#### 3️⃣ Infrastructure Layer

> **Contains technical implementations and external dependencies.**

Examples:

```text
Entity Framework
SQL Server
Redis
RabbitMQ
Azure Service Bus
External APIs
Email
File Storage
Dataverse
```

#### Example

```csharp
public class OrderRepository : IOrderRepository
{
    private readonly AppDbContext _context;

    public OrderRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task AddAsync(Order order)
    {
        _context.Orders.Add(order);

        await _context.SaveChangesAsync();
    }
}
```

Here:

```text
IOrderRepository
       ▲
       │
OrderRepository
```

The **interface is the abstraction**, while Infrastructure provides the implementation.

---

#### 4️⃣ Presentation Layer

> **Handles communication with the outside world.**

In ASP.NET Core, this is usually:

```text
Controllers
Middleware
Authentication
Authorization
API Configuration
```

##### Example

```csharp
[ApiController]
[Route("api/orders")]
public class OrdersController : ControllerBase
{
    private readonly ICreateOrderService _service;

    public OrdersController(ICreateOrderService service)
    {
        _service = service;
    }

    [HttpPost]
    public async Task<IActionResult> Create(CreateOrderRequest request)
    {
        var result = await _service.CreateOrder(request);

        return Ok(result);
    }
}
```

The controller should be **thin**.

❌ Don't put complex business logic inside controllers.

---

#### 🔥 Dependency Rule

This is the **most important concept** in Clean Architecture.

#### ❌ Bad

```text
Domain → Entity Framework
Domain → SQL Server
Domain → ASP.NET Core
Domain → Redis
```

##### ✅ Good

```text
Application
     ↓
Interface
     ↑
Infrastructure
```

For example:

```csharp
// Application

public interface IOrderRepository
{
    Task AddAsync(Order order);
}
```

Infrastructure implements it:

```csharp
// Infrastructure

public class OrderRepository : IOrderRepository
{
    public async Task AddAsync(Order order)
    {
        // EF Core implementation
    }
}
```

So:

```text
             IOrderRepository
             ▲             ▲
             │             │
       Application    Infrastructure
```

This is closely related to the **Dependency Inversion Principle (DIP)**.

---

#### 🔄 Real Request Flow

Suppose the client sends:

```http
POST /api/orders
```

```json
{
    "productId": 100,
    "quantity": 2
}
```

The request flows like this:

```text
Client
  │
  ▼
Controller
  │
  ▼
Application Service / Command Handler
  │
  ▼
Domain Entity
  │
  ▼
Repository Interface
  │
  ▼
Infrastructure Repository
  │
  ▼
SQL Server
```

In simple terms:

```text
POST /orders
      ↓
OrdersController
      ↓
CreateOrderService
      ↓
Order
      ↓
IOrderRepository
      ↓
OrderRepository
      ↓
EF Core
      ↓
SQL Server
```

---

#### ❌ Bad Architecture

Everything is inside the controller:

```csharp
[HttpPost]
public async Task<IActionResult> Create(Order order)
{
    // Validation
    if (order.Amount <= 0)
        return BadRequest();

    // Business logic
    order.Total = order.Price * order.Quantity;

    // Database
    using var connection =
        new SqlConnection(connectionString);

    await connection.ExecuteAsync(
        "INSERT INTO Orders ...");

    // Email
    await _emailService.SendAsync(...);

    return Ok();
}
```

Controller is now responsible for:

```text
❌ Validation
❌ Business logic
❌ Database
❌ SQL
❌ Email
```

This creates **tight coupling**.

---

#### ✅ Good — Clean Architecture

#### Controller

```csharp
[HttpPost]
public async Task<IActionResult> Create(
    CreateOrderRequest request)
{
    var result = await _service.CreateAsync(request);

    return Ok(result);
}
```

#### Application

```csharp
public class CreateOrderService
{
    private readonly IOrderRepository _repository;

    public CreateOrderService(
        IOrderRepository repository)
    {
        _repository = repository;
    }

    public async Task<Guid> CreateAsync(
        CreateOrderRequest request)
    {
        var order = new Order(
            request.ProductId,
            request.Quantity);

        await _repository.AddAsync(order);

        return order.Id;
    }
}
```

#### Domain

```csharp
public class Order
{
    public Guid Id { get; private set; }

    public int Quantity { get; private set; }

    public Order(Guid productId, int quantity)
    {
        if (quantity <= 0)
            throw new DomainException(
                "Quantity must be greater than zero");

        Quantity = quantity;
    }
}
```

#### Infrastructure

```csharp
public class OrderRepository : IOrderRepository
{
    private readonly AppDbContext _context;

    public OrderRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task AddAsync(Order order)
    {
        _context.Orders.Add(order);

        await _context.SaveChangesAsync();
    }
}
```

Each layer now has **one clear responsibility**.

---

#### 🧪 Why is Clean Architecture easier to test?

Suppose:

```csharp
public class OrderService
{
    private readonly IOrderRepository _repository;
}
```

During unit testing:

```csharp
var repository = new Mock<IOrderRepository>();

var service =
    new OrderService(repository.Object);
```

We don't need:

```text
❌ SQL Server
❌ Database connection
❌ EF Core database
❌ External API
```

Instead:

```text
Business Logic
      ↓
Interface
      ↓
Mock
```

Therefore, business logic becomes much easier to **unit test**.

---

#### 🔁 Clean Architecture vs 3-Tier Architecture

This is a **very common interview question**.

#### Traditional 3-Tier

```text
Presentation
     ↓
Business Logic
     ↓
Data Access
     ↓
Database
```

Usually:

```text
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
```

#### Clean Architecture

```text
        Presentation
             ↓
        Application
             ↓
           Domain
             ▲
             │
       Infrastructure
```

#### Main Difference

> **Clean Architecture makes the business/domain logic independent of infrastructure such as databases and frameworks.**

---

#### 🔥 Clean Architecture + SOLID

Clean Architecture and SOLID work together.

| Clean Architecture Concept | SOLID Principle |
|---|---|
| Separate responsibilities | **SRP** |
| Extend without changing core logic | **OCP** |
| Proper abstractions | **LSP** |
| Small focused interfaces | **ISP** |
| Depend on abstractions | **DIP** |

Especially:

> **Dependency Inversion Principle (DIP) is one of the key principles behind Clean Architecture.**

---

#### 🔁 Quick Reference Card

| Layer | Responsibility | Example |
|---|---|---|
| **Domain** | Core business rules | `Order`, `Customer` |
| **Application** | Use cases / orchestration | `CreateOrderService` |
| **Infrastructure** | Technical implementations | EF Core, SQL, Redis |
| **Presentation** | HTTP/API | Controllers |

---

#### 🔁 Which Layer Does What?

| Problem / Responsibility | Layer |
|---|---|
| Business rules | Domain |
| Entities | Domain |
| Use cases | Application |
| DTOs | Application |
| Repository interfaces | Application/Domain* |
| EF Core implementation | Infrastructure |
| SQL connection | Infrastructure |
| External API implementation | Infrastructure |
| HTTP endpoints | Presentation |
| Controllers | Presentation |
| Middleware | Presentation |

> *Exact placement of repository interfaces can vary by Clean Architecture style. The important rule is that the abstraction belongs to the inner layer that needs it, while the implementation stays in Infrastructure.*

---

#### 🎯 Interview Answer — Short Version

If interviewer asks:

#### "What is Clean Architecture?"

You can answer:

> **Clean Architecture is an architectural pattern that separates an application into Presentation, Application, Domain, and Infrastructure layers. The main principle is that dependencies should point toward the core business logic. Domain and Application should not depend directly on databases, frameworks, or external services. Instead, we use abstractions such as interfaces and dependency injection. This makes the application loosely coupled, testable, maintainable, and easier to change.**

#### If they ask for an example:

> **For example, instead of my OrderService directly depending on Entity Framework, I define an IOrderRepository abstraction and let the Infrastructure layer implement it using EF Core. This allows me to change the database implementation or mock the repository during unit testing without changing the business logic.**

---

#### 🧠 Memory Tip

> **P-A-D-I**

**P**resentation → API / Controllers

**A**pplication → Use Cases

**D**omain → Business Rules

**I**nfrastructure → Database / External Services

And remember the golden rule:

> 🔥 **Business logic should not depend on the database or framework.**

#### One-Line Interview Memory

> **Clean Architecture = Separate business logic from technical details using layers and abstractions.**
