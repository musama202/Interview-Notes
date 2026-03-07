# C# Interview Questions

---

### Q1. What is the difference between `ref` and `out` parameters in C#? Can you provide a real-world example?

**Answer:**  
- `ref`: Pass by reference; the value is both input and output.  
- `out`: Pass by reference; the value is only output (must be assigned inside).  
- `in`: Pass by reference; the value is read-only and cannot be modified inside the method.

**Example:**
```csharp
// C# Example: ref, out, and in

using System;

class Program
{
    static void Main()
    {
        // ref example
        int a = 5;
        RefExample(ref a);  // must be initialized before passing
        Console.WriteLine($"ref example result: {a}"); // Output: 15

        // out example
        int b;  // no need to initialize
        OutExample(out b);
        Console.WriteLine($"out example result: {b}"); // Output: 20

        // in example
        int c = 10;
        InExample(in c);    // read-only inside method
        Console.WriteLine($"in example result: {c}"); // Output: 10
    }

    static void RefExample(ref int x)
    {
        x += 10;  // modifies original value
    }

    static void OutExample(out int y)
    {
        y = 20;   // must assign before method ends
    }

    static void InExample(in int z)
    {
        // z += 5; // ❌ Error: cannot modify because it's read-only
        Console.WriteLine($"Inside in method: {z}");
    }
}
```

---

### Q2. How does method overloading (early binding) differ from method overriding (late binding) in C#?

**Answer:**  
- *Method overloading* is compile-time polymorphism (early binding). Multiple methods share a name in the same class but differ in signature.  
- *Method overriding* is runtime polymorphism (late binding). A derived class provides a new implementation for a virtual/abstract method from the base class.

**Notes:**  
Overriding enables runtime polymorphism and adheres to the Open/Closed Principle by allowing derived classes to extend behavior without modifying the base.


**Example:**

```csharp
using System;

class Program
{
    static void Main()
    {
        // Method Overloading (Early Binding)
        Calculator calc = new Calculator();
        Console.WriteLine(calc.Add(5, 10));      // Output: 15
        Console.WriteLine(calc.Add(5.5, 10.3));  // Output: 15.8

        // Method Overriding (Late Binding)
        Animal myAnimal = new Animal();
        Animal myDog = new Dog();

        myAnimal.MakeSound();  // Output: Animal sound
        myDog.MakeSound();     // Output: Dog barks
    }
}

// Method Overloading Example
class Calculator
{
    public int Add(int a, int b)
    {
        return a + b;
    }

    public double Add(double a, double b)
    {
        return a + b;
    }
}

// Method Overriding Example
class Animal
{
    public virtual void MakeSound()
    {
        Console.WriteLine("Animal sound");
    }
}

class Dog : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("Dog barks");
    }
}
```

---
### Q3. What are the differences between an abstract class and an interface? When would you choose one over the other?

### Q3. Differences between an Abstract Class and an Interface in C#

**Answer:**

| Feature                     | Abstract Class                                  | Interface                                  |
|-------------------------------|-----------------------------------------------|-------------------------------------------|
| Inheritance                   | Can inherit only one abstract class          | Can implement multiple interfaces         |
| Members                        | Can have fields, constructors, methods (abstract & non-abstract) | Can only have method signatures (C# 8+ allows default implementations) |
| Access Modifiers               | Supports `public`, `protected`, `private`    | Members are `public` by default           |
| Use Case                       | Shared base functionality among related classes | Define a contract for unrelated classes  |
| Instantiation                  | Cannot be instantiated                        | Cannot be instantiated                     |

**Scenario:**  
- Use an **abstract class** when you have a **common base class** with shared code and default behavior.  
- Use an **interface** when you want to enforce a **contract** across **unrelated classes**, or support **multiple inheritance**.

---

**Example:**  

```csharp
using System;

// Abstract class example
abstract class Vehicle
{
    public string Brand { get; set; }

    public Vehicle(string brand)
    {
        Brand = brand;
    }

    public abstract void Start(); // Must be implemented by derived classes

    public void Honk() // Shared functionality
    {
        Console.WriteLine($"{Brand} says: Beep Beep!");
    }
}

class Car : Vehicle
{
    public Car(string brand) : base(brand) { }

    public override void Start()
    {
        Console.WriteLine($"{Brand} car is starting!");
    }
}

// Interface example
interface IFlyable
{
    void Fly(); // Only the contract
}

class Airplane : IFlyable
{
    public void Fly()
    {
        Console.WriteLine("Airplane is flying!");
    }
}

class Drone : IFlyable
{
    public void Fly()
    {
        Console.WriteLine("Drone is flying!");
    }
}

// Usage
class Program
{
    static void Main()
    {
        Vehicle myCar = new Car("Toyota");
        myCar.Start(); // Toyota car is starting!
        myCar.Honk();  // Toyota says: Beep Beep!

        IFlyable myDrone = new Drone();
        IFlyable myPlane = new Airplane();
        myDrone.Fly(); // Drone is flying!
        myPlane.Fly(); // Airplane is flying!
    }
}

```

---
### Q4. Since C# does not support multiple inheritance directly, how can you implement it?

**Answer:**  
C# does not allow a class to inherit from more than one class. However, **multiple inheritance** can be achieved by using **interfaces**. A class can implement multiple interfaces, which allows it to inherit multiple **contracts**.  

**Example:**  

```csharp
using System;

// Interface 1
interface IWorker
{
    void Work();
}

// Interface 2
interface IManager
{
    void Manage();
}

// Class implementing multiple interfaces
class TeamLead : IWorker, IManager
{
    public void Work()
    {
        Console.WriteLine("TeamLead is working...");
    }

    public void Manage()
    {
        Console.WriteLine("TeamLead is managing the team...");
    }
}

// Usage
class Program
{
    static void Main()
    {
        TeamLead lead = new TeamLead();
        lead.Work();   // TeamLead is working...
        lead.Manage(); // TeamLead is managing the team...
    }
}
```

---

### Q5. What is the Diamond Problem in OOP (C#)?

**Answer:**  
The **diamond problem** occurs in multiple inheritance when a class inherits from two classes that both inherit from a common base class, creating **ambiguity**.  

Diagram:
  A
 / \
B   C
 \ /
  D

  
**Explanation:**  
- Class **B** and **C** both inherit from **A**.  
- Class **D** inherits from both **B** and **C**.  
- If **D** tries to access a method/property defined in **A**, the compiler cannot determine whether it should come from **B** or **C**, creating ambiguity.  

**C# Context:**  
- C# **does not support multiple class inheritance**, so the classic diamond problem does **not occur** with classes.  
- C# **supports multiple interface inheritance**, and a **similar ambiguity** can occur if:  
  - Multiple interfaces have the **same method signature**.  
  - A class implements those interfaces and doesn’t clearly resolve the method.  

---

**C# Example Using Interfaces:**  

```csharp
using System;

// Interface 1
interface IA
{
    void Show();
}

// Interface 2
interface IB : IA
{
}

// Interface 3
interface IC : IA
{
}

// Class D implements both IB and IC
class D : IB, IC
{
    // Must implement Show() once to resolve ambiguity
    public void Show()
    {
        Console.WriteLine("Implementation of Show() in D");
    }
}

class Program
{
    static void Main()
    {
        D obj = new D();
        obj.Show(); // Output: Implementation of Show() in D
    }
}```
---
