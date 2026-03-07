# Design Patterns Questions

---

### Q1. What is the Factory Design Pattern, when should we use it, and how is it different from Dependency Injection or Keyed Services in .NET? Provide a simple example.

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