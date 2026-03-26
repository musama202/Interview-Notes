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