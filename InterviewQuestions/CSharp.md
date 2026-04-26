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
}
```

---

### Q6. What are the four pillars of Object-Oriented Programming (OOP)? Can you explain each with an example?

**Answer:**  
The four pillars of Object-Oriented Programming (OOP) are **Encapsulation**, **Abstraction**, **Inheritance**, and **Polymorphism**. Each pillar plays a key role in designing clean, reusable, and maintainable code.

---

#### 1. 🔒 Encapsulation
**Encapsulation** means **hiding the internal data** of a class and only exposing it through public methods (getters/setters). It protects the data from unauthorized access.

**Example:**
```csharp
using System;

class BankAccount
{
    private double balance; // Hidden from outside

    public void Deposit(double amount)
    {
        if (amount > 0)
            balance += amount;
    }

    public double GetBalance()
    {
        return balance;
    }
}

class Program
{
    static void Main()
    {
        BankAccount account = new BankAccount();
        account.Deposit(500);
        Console.WriteLine(account.GetBalance()); // Output: 500
    }
}
```

---

#### 2. 🎭 Abstraction
**Abstraction** means **hiding complex implementation details** and showing only the essential features. It is achieved using **abstract classes** or **interfaces**.

**Example:**
```csharp
using System;

abstract class Shape
{
    public abstract double CalculateArea(); // Only declaration, no implementation
}

class Circle : Shape
{
    private double radius;

    public Circle(double radius)
    {
        this.radius = radius;
    }

    public override double CalculateArea()
    {
        return Math.PI * radius * radius;
    }
}

class Program
{
    static void Main()
    {
        Shape shape = new Circle(5);
        Console.WriteLine(shape.CalculateArea()); // Output: 78.539...
    }
}
```

---

#### 3. 🧬 Inheritance
**Inheritance** allows a class (**child**) to **acquire the properties and methods** of another class (**parent**), promoting code reusability.

**Example:**
```csharp
using System;

class Animal
{
    public void Eat()
    {
        Console.WriteLine("Animal is eating...");
    }
}

class Dog : Animal // Dog inherits Animal
{
    public void Bark()
    {
        Console.WriteLine("Dog is barking...");
    }
}

class Program
{
    static void Main()
    {
        Dog dog = new Dog();
        dog.Eat();  // Inherited from Animal → Animal is eating...
        dog.Bark(); // Dog's own method → Dog is barking...
    }
}
```

---

#### 4. 🔄 Polymorphism
**Polymorphism** means **one method behaves differently** based on the object that calls it. It can be achieved via **method overriding** (runtime) or **method overloading** (compile-time).

**Example:**
```csharp
using System;

class Animal
{
    public virtual void MakeSound()
    {
        Console.WriteLine("Some generic animal sound...");
    }
}

class Cat : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("Cat says: Meow!");
    }
}

class Dog : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("Dog says: Woof!");
    }
}

class Program
{
    static void Main()
    {
        Animal myAnimal;

        myAnimal = new Cat();
        myAnimal.MakeSound(); // Output: Cat says: Meow!

        myAnimal = new Dog();
        myAnimal.MakeSound(); // Output: Dog says: Woof!
    }
}
```

---

### 📝 Summary Table

| Pillar          | Key Concept                          | Achieved By                        |
|-----------------|--------------------------------------|------------------------------------|
| Encapsulation   | Hide data, expose via methods        | Private fields + public methods    |
| Abstraction     | Hide complexity, show essentials     | Abstract classes / Interfaces      |
| Inheritance     | Reuse parent class features          | `: ParentClass` syntax             |
| Polymorphism    | Same method, different behavior      | Method overriding / overloading    |

---

### Q7. What is the difference between Association, Aggregation, and Composition in C#? Can you provide real-world examples?

**Answer:**  
These three concepts define **how classes relate to each other** in Object-Oriented Programming. They differ in terms of **ownership**, **dependency**, and **lifecycle**.

---

#### 1. 🔗 Association
**Association** is a relationship where **two classes are related but independent** of each other. Neither class owns the other, and both can exist independently.

> **Real-World Example:** A **Teacher** and a **Student** — A teacher can have multiple students, and a student can have multiple teachers. Both exist independently.

**Example:**
```csharp
using System;

class Teacher
{
    public string Name { get; set; }

    public Teacher(string name)
    {
        Name = name;
    }

    public void Teach()
    {
        Console.WriteLine($"{Name} is teaching...");
    }
}

class Student
{
    public string Name { get; set; }

    public Student(string name)
    {
        Name = name;
    }

    public void LearnFrom(Teacher teacher)
    {
        Console.WriteLine($"{Name} is learning from {teacher.Name}");
    }
}

class Program
{
    static void Main()
    {
        Teacher teacher = new Teacher("Sir Ahmed");
        Student student = new Student("Ali");

        student.LearnFrom(teacher);
        // Output: Ali is learning from Sir Ahmed
    }
}
```

---

#### 2. 🧩 Aggregation *(Has-A | Weak Ownership)*
**Aggregation** is a **"Has-A"** relationship where one class **contains a reference** to another, but the contained object **can exist independently** even if the parent is destroyed.

> **Real-World Example:** A **Department** and a **Professor** — A department has professors, but if the department is closed, the professors still exist.

**Example:**
```csharp
using System;
using System.Collections.Generic;

class Professor
{
    public string Name { get; set; }

    public Professor(string name)
    {
        Name = name;
    }

    public void Display()
    {
        Console.WriteLine($"Professor: {Name}");
    }
}

class Department
{
    public string DepartmentName { get; set; }
    private List<Professor> professors; // Aggregation (Professor exists independently)

    public Department(string name, List<Professor> professors)
    {
        DepartmentName = name;
        this.professors = professors;
    }

    public void ShowProfessors()
    {
        Console.WriteLine($"\nDepartment: {DepartmentName}");
        foreach (var prof in professors)
            prof.Display();
    }
}

class Program
{
    static void Main()
    {
        // Professors exist independently
        List<Professor> profs = new List<Professor>
        {
            new Professor("Dr. Khan"),
            new Professor("Dr. Aisha")
        };

        Department dept = new Department("Computer Science", profs);
        dept.ShowProfessors();
        // Output:
        // Department: Computer Science
        // Professor: Dr. Khan
        // Professor: Dr. Aisha
    }
}
```

---

#### 3. 🏗️ Composition *(Has-A | Strong Ownership)*
**Composition** is a **strong "Has-A"** relationship where one class **owns** another, and the **child object cannot exist without the parent**. If the parent is destroyed, the child is also destroyed.

> **Real-World Example:** A **House** and its **Rooms** — Rooms cannot exist without the house. If the house is demolished, the rooms are gone too.

**Example:**
```csharp
using System;
using System.Collections.Generic;

class Room
{
    public string RoomName { get; set; }

    public Room(string name)
    {
        RoomName = name;
    }

    public void Display()
    {
        Console.WriteLine($"  Room: {RoomName}");
    }
}

class House
{
    public string Address { get; set; }
    private List<Room> rooms; // Composition (Rooms created inside House)

    public House(string address)
    {
        Address = address;
        // Rooms are created BY the House (strong ownership)
        rooms = new List<Room>
        {
            new Room("Living Room"),
            new Room("Bedroom"),
            new Room("Kitchen")
        };
    }

    public void ShowRooms()
    {
        Console.WriteLine($"House at: {Address}");
        foreach (var room in rooms)
            room.Display();
    }
}

class Program
{
    static void Main()
    {
        House house = new House("123 Main Street");
        house.ShowRooms();
        // Output:
        // House at: 123 Main Street
        //   Room: Living Room
        //   Room: Bedroom
        //   Room: Kitchen
    }
}
```

---


---

### 🧠 Quick Memory Tip

> - **Association** → *"I know you"* (no ownership)  
> - **Aggregation** → *"I have you, but you can live without me"* (weak ownership)  
> - **Composition** → *"You are part of me, you cannot exist without me"* (strong ownership)

---

### Q8. What is a partial class in C#, and why would you use it?

**Answer:**  
A **partial class** in C# allows you to **split the definition of a class across multiple files**. All parts are combined into a single class at **compile time**. It is defined using the `partial` keyword.

---

#### 🔑 Syntax
```csharp
// File 1
partial class MyClass
{
    // Part 1 of the class
}

// File 2
partial class MyClass
{
    // Part 2 of the class
}
```

> At compile time, both files are **merged into one single class** automatically.

---

### Q9. What is a Static Class, Static Method, and Static Property in C#?

**Answer:**  
The `static` keyword means the member **belongs to the class itself**, not to any object. You **do not need to create an object** to access it.

---

#### 1. 🏛️ Static Class
- Cannot be **instantiated** or **inherited**
- Contains **only static members**
- Used for **utility/helper** methods
```csharp
static class MathHelper
{
    public static double Square(double n) => n * n;
}

// Usage — no object needed
Console.WriteLine(MathHelper.Square(4)); // Output: 16
```

---

#### 2. ⚙️ Static Method
- Called using **class name**, not an object
- **Cannot access** instance members directly
```csharp
class Converter
{
    public static double CelsiusToFahrenheit(double c) => (c * 9 / 5) + 32;
}

// Usage
Console.WriteLine(Converter.CelsiusToFahrenheit(100)); // Output: 212
```

---

#### 3. 📦 Static Property
- **Shared across all instances** of the class
- Retains value throughout the application's lifetime
```csharp
class Student
{
    public static int TotalStudents { get; private set; } = 0;
    public string Name { get; set; }

    public Student(string name) { Name = name; TotalStudents++; }
}

// Usage
Student s1 = new Student("Ali");
Student s2 = new Student("Sara");
Console.WriteLine(Student.TotalStudents); // Output: 2
```

---

### 📝 Summary Table

| Feature          | Static Class       | Static Method         | Static Property          |
|------------------|--------------------|-----------------------|--------------------------|
| **Belongs To**   | Class              | Class                 | Class                    |
| **Need Object?** | ❌ No             | ❌ No                 | ❌ No                    |
| **Common Use**   | Helper/Utility     | Conversions, Math     | Counters, Global Config  |
| **Example**      | `Math`, `Console`  | `Math.Sqrt()`         | `Student.TotalStudents`  |

---

### 🧠 Memory Tip
> **Static** = belongs to the **class**, not the **object**.  
> Like a **shared whiteboard** in a classroom — everyone reads it, but it belongs to the **room**, not any single student.

---
### Q10. What is a sealed class in C#? When and why would you use one?

**Answer:**  
A **sealed class** is a class that **cannot be inherited**. Once a class is marked `sealed`, no other class can derive from it.

---

#### 🔑 Syntax
```csharp
sealed class MyClass
{
    // Cannot be inherited
}
```

---

#### 📁 Example
```csharp
using System;

sealed class PaymentGateway
{
    public void ProcessPayment(double amount)
    {
        Console.WriteLine($"Payment of {amount:C} processed securely.");
    }
}

// ❌ This will cause a COMPILE ERROR
// class FakeGateway : PaymentGateway { }  // ERROR: cannot inherit from sealed class

class Program
{
    static void Main()
    {
        PaymentGateway gateway = new PaymentGateway();
        gateway.ProcessPayment(5000);
        // Output: Payment of $5,000.00 processed securely.
    }
}
```

---

#### 🔒 Sealed Method (Bonus)
You can also seal a **single method** in a derived class to stop further overriding.
```csharp
class Animal
{
    public virtual void Speak() => Console.WriteLine("Animal speaks");
}

class Dog : Animal
{
    public sealed override void Speak() => Console.WriteLine("Dog barks");
}

// ❌ Cannot override Speak() anymore
// class Puppy : Dog
// {
//     public override void Speak() { } // ERROR!
// }
```

---

### 📝 When & Why to Use Sealed Class?

| Reason               | Explanation                                              |
|----------------------|----------------------------------------------------------|
| 🔐 **Security**      | Prevent others from modifying critical class behavior    |
| ⚡ **Performance**   | Compiler optimizes sealed classes (no virtual dispatch)  |
| 🎯 **Design Intent** | Signals this class is **complete** and not meant to be extended |
| 🏦 **Example Use**   | Payment systems, encryption classes, authentication      |

---

### 🧠 Memory Tip
> **Sealed class** = a **dead-end road** 🚧  
> You can use it, but you **cannot extend it further**.

---
### Q11. What are extension methods in C#?

**Answer:**  
Extension methods let you **add new methods to an existing class** without modifying or inheriting it. Defined in a **static class** using the `this` keyword.

---

#### 🔑 Syntax
```csharp
static class MyExtensions
{
    public static ReturnType MethodName(this TargetType obj) { }
}
```

---

#### 📁 Example
```csharp
using System;

static class StringExtensions
{
    public static bool IsValidEmail(this string email) 
        => email.Contains("@") && email.Contains(".");

    public static string Reverse(this string text)
    {
        char[] chars = text.ToCharArray();
        Array.Reverse(chars);
        return new string(chars);
    }
}

class Program
{
    static void Main()
    {
        Console.WriteLine("ali@gmail.com".IsValidEmail()); // True
        Console.WriteLine("Hello".Reverse());              // olleH
    }
}
```

---

### ⚠️ Key Rules

| Rule                              | Detail                              |
|-----------------------------------|-------------------------------------|
| ✅ Must be in a **static class**  | `static class MyExtensions`         |
| ✅ Method must be **static**      | `public static void Method(...)`    |
| ✅ First param needs **`this`**   | `this string s`                     |
| ❌ No private member access       | Cannot access private fields        |
| ❌ Cannot override existing methods | Only adds new methods             |

---

### 📝 When to Use?

- Extend classes you **don't own** (`string`, `int`, `List`)
- **LINQ** is built entirely on extension methods (`.Where()`, `.Select()`)
- Cleaner, reusable helper methods across the project

---

### 🧠 Memory Tip
> Like adding a **new pocket** 👖 to a jacket you didn't make —  
> enhance it **without changing the original**.

---
### Q12. How do different access modifiers behave in C#?

**Answer:**  
Access modifiers control **who can access** a class, method, or property.

---

#### 📝 Quick Reference Table

| Modifier              | Same Class | Same Assembly | Subclass | Everywhere |
|-----------------------|------------|---------------|----------|------------|
| `public`              | ✅        | ✅            | ✅      | ✅         |
| `private`             | ✅        | ❌            | ❌      | ❌         |
| `protected`           | ✅        | ❌            | ✅      | ❌         |
| `internal`            | ✅        | ✅            | ❌      | ❌         |
| `protected internal`  | ✅        | ✅            | ✅      | ❌         |
| `private protected`   | ✅        | ❌            | ✅*     | ❌         |

> *`private protected` — subclass access only **within the same assembly**

---

#### 📁 Example
```csharp
class Person
{
    public    string Name     = "Ali";      // Accessible everywhere
    private   int    age      = 25;         // Only inside this class
    protected string Country  = "Pakistan"; // This class + subclasses
    internal  string City     = "Karachi";  // Same project only
}

class Employee : Person
{
    public void Show()
    {
        Console.WriteLine(Name);     // ✅ public
        Console.WriteLine(Country);  // ✅ protected
        // Console.WriteLine(age);   // ❌ private — ERROR
    }
}
```

---

### 🧠 Memory Tip

> - `public`    → **Everyone** can see  
> - `private`   → **Only me** can see  
> - `protected` → **Me + my children** can see  
> - `internal`  → **My team** (same project) can see

---

### Q13. In multi-level inheritance, how are constructors and destructors called?

**Answer:**  
In multi-level inheritance, constructors are called **top to bottom** (Parent → Child) and destructors are called **bottom to top** (Child → Parent).

---

#### 📁 Example
```csharp
using System;

class GrandParent
{
    public GrandParent() => Console.WriteLine("GrandParent Constructor");
    ~GrandParent()       => Console.WriteLine("GrandParent Destructor");
}

class Parent : GrandParent
{
    public Parent() => Console.WriteLine("Parent Constructor");
    ~Parent()       => Console.WriteLine("Parent Destructor");
}

class Child : Parent
{
    public Child() => Console.WriteLine("Child Constructor");
    ~Child()       => Console.WriteLine("Child Destructor");
}

class Program
{
    static void Main()
    {
        Child obj = new Child();
    }
}

// Output:
// GrandParent Constructor  ← Top to Bottom
// Parent Constructor
// Child Constructor
// Child Destructor         ← Bottom to Top
// Parent Destructor
// GrandParent Destructor
```

---

### 🧠 Memory Tip
> **Constructor** → Parent gives birth first 👶 (Top → Bottom)  
> **Destructor**  → Child leaves first 🚪 (Bottom → Top)

---
### Q14. What is a Thread in C#?

**Answer:**  
A **Thread** is the **smallest unit of execution** in a program. By default, every C# program runs on a **single main thread**. We can create additional threads to run tasks **simultaneously**.

---

#### 📁 Basic Example
```csharp
using System;
using System.Threading;

class Program
{
    static void PrintNumbers()
    {
        for (int i = 1; i <= 5; i++)
        {
            Console.WriteLine($"Thread: {i}");
            Thread.Sleep(500); // Pause 500ms
        }
    }

    static void Main()
    {
        Thread t = new Thread(PrintNumbers);
        t.Start(); // Start new thread

        for (int i = 1; i <= 5; i++)
        {
            Console.WriteLine($"Main: {i}");
            Thread.Sleep(500);
        }
    }
}

// Output (runs simultaneously):
// Main: 1
// Thread: 1
// Main: 2
// Thread: 2 ...
```

---

#### 📝 Thread Key Properties & Methods

| Method / Property   | Purpose                                      |
|---------------------|----------------------------------------------|
| `t.Start()`         | Starts the thread                            |
| `t.Join()`          | Main thread waits until this thread finishes |
| `t.Sleep(ms)`       | Pauses thread for given milliseconds         |
| `t.IsAlive`         | Checks if thread is still running            |
| `t.Name`            | Set/get thread name                          |
| `t.Priority`        | Set priority (Low, Normal, High)             |

---

#### 📁 Thread vs Task
```csharp
// Thread — manual, low level
Thread t = new Thread(() => Console.WriteLine("Thread"));
t.Start();

// Task — modern, recommended way (uses thread pool)
Task.Run(() => Console.WriteLine("Task"));
```

---

### 📝 Types of Threads

| Type                | Detail                                          |
|---------------------|-------------------------------------------------|
| **Foreground**      | App waits for it to finish (default)            |
| **Background**      | App exits even if it's still running            |
```csharp
Thread t  = new Thread(DoWork);
t.IsBackground = true; // Set as background thread
t.Start();
```

---

### 🧠 Memory Tip
> A **Thread** is like a **worker** 👷 in a factory.  
> One worker (main thread) does all tasks by default.  
> Add more workers (threads) to do **multiple tasks at the same time**.

---

### Q15. What is the purpose of `async` and `await` in C#?

**Answer:**  
`async` and `await` allow you to write **asynchronous code** that runs **without blocking the main thread**. The app keeps responding while waiting for long tasks (API calls, file reads, DB queries).

---

#### 🔑 Syntax
```csharp
public async Task MethodName()
{
    await SomeLongRunningTask();
}
```

> - `async`  → marks the method as asynchronous  
> - `await`  → pauses execution **until the task completes** without blocking the thread

---

#### 📁 Example — Without vs With async/await
```csharp
using System;
using System.Threading.Tasks;

class Program
{
    // ❌ Synchronous — blocks the thread while waiting
    static void GetDataSync()
    {
        Task.Delay(3000).Wait();          // Blocks for 3 seconds
        Console.WriteLine("Data loaded!");
    }

    // ✅ Asynchronous — thread is free while waiting
    static async Task GetDataAsync()
    {
        await Task.Delay(3000);           // Waits without blocking
        Console.WriteLine("Data loaded!");
    }

    static async Task Main()
    {
        Console.WriteLine("Fetching data...");
        await GetDataAsync();
        Console.WriteLine("Done!");
    }
}

// Output:
// Fetching data...
// (waits 3 seconds — thread is FREE)
// Data loaded!
// Done!
```

---

#### 📁 Real-World Example — API Call
```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

class ApiService
{
    static async Task FetchData()
    {
        HttpClient client = new HttpClient();
        string result = await client.GetStringAsync("https://api.example.com/data");
        Console.WriteLine($"Response: {result}");
    }

    static async Task Main()
    {
        Console.WriteLine("App is running...");
        await FetchData();   // Non-blocking API call
        Console.WriteLine("App continues...");
    }
}
```

---

### 📝 When to Use?

- 🌐 **API / HTTP calls**
- 🗄️ **Database queries**
- 📁 **File read/write operations**
- ⏳ Any **long-running task** that would freeze the UI

---

### 🧠 Memory Tip
> `async/await` is like **ordering food** 🍕 at a restaurant.  
> You place your order (`await`) and do other things —  
> you are **not standing at the counter** waiting (blocking).  
> When food is ready, you get notified and continue.

---
### Q16. What is the difference between Parallelism and Concurrency in C#?

**Answer:**

| | Concurrency | Parallelism |
|--|-------------|-------------|
| **Definition** | Multiple tasks **in progress** at the same time | Multiple tasks **executing** at the same time |
| **Execution** | Tasks switch rapidly (take turns) | Tasks run truly simultaneously |
| **Requires** | Single or multi-core CPU | Multi-core CPU |
| **Best For** | I/O-bound tasks | CPU-bound tasks |
| **Example** | `async/await`, Threads | `Parallel.For`, `Task.WhenAll` |

---

#### 📁 Concurrency Example — `async/await`
```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task FetchData(string source)
    {
        Console.WriteLine($"Fetching from {source}...");
        await Task.Delay(2000);   // Simulates I/O wait
        Console.WriteLine($"Done: {source}");
    }

    static async Task Main()
    {
        // Tasks START together but take turns (concurrent)
        await Task.WhenAll(
            FetchData("API"),
            FetchData("Database"),
            FetchData("Cache")
        );
        Console.WriteLine("All fetched!");
    }
}

// Output:
// Fetching from API...
// Fetching from Database...
// Fetching from Cache...
// Done: API
// Done: Database
// Done: Cache
// All fetched!
```

---

#### 📁 Parallelism Example — `Parallel.For`
```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        // Tasks run TRULY at the same time on multiple CPU cores
        Parallel.For(1, 5, i =>
        {
            Console.WriteLine($"Processing Task {i} on Thread {
                              System.Threading.Thread.CurrentThread.ManagedThreadId}");
        });

        Console.WriteLine("All tasks complete!");
    }
}

// Output (order may vary — truly parallel):
// Processing Task 1 on Thread 3
// Processing Task 3 on Thread 5
// Processing Task 2 on Thread 4
// Processing Task 4 on Thread 6
// All tasks complete!
```

---

### 🧠 Memory Tip
> **Concurrency** = One chef 👨‍🍳 **switching** between multiple dishes  
> **Parallelism** = Multiple chefs 👨‍🍳👨‍🍳 each cooking a **different dish at the same time**

---

### Q17. How does Task differ from Thread in C#?

**Answer:**  
`Thread` is **low-level** manual control. `Task` is **high-level** and uses the thread pool automatically — recommended for most scenarios.

---

#### 📁 Thread Example
```csharp
using System;
using System.Threading;

class Program
{
    static void Main()
    {
        Thread t = new Thread(() =>
        {
            Console.WriteLine("Thread is running...");
            Thread.Sleep(1000);
            Console.WriteLine("Thread done!");
        });

        t.Start();
        t.Join(); // Wait for thread to finish
    }
}
```

---

#### 📁 Task Example
```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        await Task.Run(() =>
        {
            Console.WriteLine("Task is running...");
            Task.Delay(1000).Wait();
            Console.WriteLine("Task done!");
        });
    }
}
```

---

#### 📁 Task Returning a Value
```csharp
static async Task Main()
{
    // Task<T> can return a result — Thread cannot
    int result = await Task.Run(() => 10 * 5);
    Console.WriteLine($"Result: {result}"); // Output: 50
}
```

---

### 🧠 Memory Tip
> **Thread** = Hiring a **new employee** 👷 every time (costly)  
> **Task** = Assigning work to an **existing employee** from a pool (efficient) ✅

---

### Q 18. How do you efficiently handle CPU-bound operations in C#?

**Answer:**  
Use `Parallel.For` / `Parallel.ForEach` for loops, `Task.Run` to offload work, and `async`/`await` to keep the main thread free.

---

#### 📁 Parallel.For / ForEach
```csharp
Parallel.For(0, 10, i => DoWork(i));

Parallel.ForEach(items, item => Process(item));
```

---

#### 📁 Task.Run (CPU-bound)
```csharp
int result = await Task.Run(() => HeavyCalculation());
```

---

#### 📁 PLINQ
```csharp
var results = data.AsParallel().Where(x => x % 2 == 0).ToList();
```

---

### 🧠 Memory Tip
> **CPU-bound** → `Parallel` / `Task.Run` → use multiple cores 🖥️  
> **I/O-bound** → `async`/`await` → free the thread while waiting ⏳

---

