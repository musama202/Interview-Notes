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

