### Q1. What is the .NET Framework?

**Answer:**  
**.NET Framework** is a **software development platform** by Microsoft used to build and run Windows applications. It provides tools, libraries, and runtime to develop apps using languages like **C#, VB.NET, and F#**.

---

#### 📝 Key Features

- ✅ **Language Interoperability** — C#, VB.NET, F# work together
- ✅ **Memory Management** — Automatic via **Garbage Collector**
- ✅ **Security** — Built-in code access security
- ✅ **Platform** — Windows only *(use .NET Core for cross-platform)*

---

#### 📝 .NET Versions — Quick View

| Platform | Cross-Platform | Status |
|----------|---------------|--------|
| **.NET Framework** | ❌ Windows only | Legacy |
| **.NET Core** | ✅ Win, Mac, Linux | Modern |
| **.NET 5/6/7/8** | ✅ Unified platform | Current |

---

### 🧠 Memory Tip
> **.NET Framework** = A **toolbox** 🧰 Microsoft gave developers  
> to build Windows apps — with built-in tools for everything.

---

### Q2. What is the difference between .NET Core and .NET Framework?

**Answer:**

| Feature | .NET Framework | .NET Core |
|---------|---------------|-----------|
| **Platform** | ❌ Windows only | ✅ Windows, Mac, Linux |
| **Performance** | Slower | Faster & lightweight |
| **Open Source** | ❌ No | ✅ Yes |
| **Deployment** | System-wide install | Self-contained deploy |
| **App Types** | WinForms, WPF, ASP.NET | Web, Cloud, Microservices |
| **Future** | ❌ No new features | ✅ Actively developed |
| **Latest Version** | 4.8 (final) | .NET 8 (current) |

---

#### 📝 When to Use Which?

| Use .NET Framework | Use .NET Core / .NET 8 |
|--------------------|------------------------|
| Legacy Windows apps | New modern apps |
| WinForms / WPF projects | Cross-platform apps |
| Older enterprise systems | Microservices & Cloud |

---

### 🧠 Memory Tip
> **.NET Framework** = Old Windows-only house 🏠  
> **.NET Core** = Modern apartment 🏢 that works **everywhere**

---
### Q3. What is the Common Language Runtime (CLR)?

**Answer:**  
CLR is the **execution engine** of .NET. It runs your code and manages memory, security, and exceptions automatically.

---

#### 🔑 How It Works
```
C# Code → Compiler → IL Code → CLR (JIT) → Machine Code → Runs
```

---

#### 📝 Key Services

| Service | Purpose |
|---------|---------|
| **JIT Compiler** | Converts IL code → Machine code |
| **Garbage Collector** | Auto memory cleanup |
| **Exception Handling** | Manages try/catch |
| **Type Safety** | Ensures correct data types |
| **Security** | Code verification & access |

---

### 🧠 Memory Tip
> CLR = **Manager** 👔 of your app —  
> handles everything behind the scenes  
> so you just **focus on writing code**.

---

### Q4. What is Managed Code?

**Answer:**  
**Managed code** is code that runs **under the control of CLR**. CLR handles memory, security, and exceptions automatically.

---

#### 📝 Managed vs Unmanaged

| | Managed Code | Unmanaged Code |
|--|-------------|----------------|
| **Runs Under** | CLR | Directly on OS |
| **Memory** | Auto (GC) | Manual (developer) |
| **Language** | C#, VB.NET | C, C++ |
| **Safety** | ✅ Safe | ❌ Risk of memory leaks |

---

### 🧠 Memory Tip
> **Managed** = CLR **babysits** 👶 your code  
> **Unmanaged** = You manage **everything yourself** 😰

---
### Q5. What are Value Types and Reference Types in C#?

**Answer:**

| | Value Type | Reference Type |
|--|------------|----------------|
| **Stored In** | Stack | Heap |
| **Holds** | Actual value | Memory address |
| **Copy** | Copies the value | Copies the reference |
| **Default** | 0, false | null |
| **Examples** | int, float, bool, struct | class, string, array |

---

#### 📁 Example
```csharp
// Value Type — copy is independent
int a = 10;
int b = a;
b = 20;
Console.WriteLine(a); // Output: 10 (unchanged)

// Reference Type — both point to same object
int[] arr1 = { 1, 2, 3 };
int[] arr2 = arr1;
arr2[0] = 99;
Console.WriteLine(arr1[0]); // Output: 99 (changed!)
```

---

### 🧠 Memory Tip
> **Value Type** = Copy of a **document** 📄 — changes don't affect original  
> **Reference Type** = **Shared link** 🔗 — changes affect everyone

---

### Q6. What is an Assembly in .NET?

**Answer:**  
An **Assembly** is a **compiled output** of a .NET project. It is a `.dll` or `.exe` file that contains IL code, metadata, and resources.

---

#### 📝 Two Types

| Type | Extension | Purpose |
|------|-----------|---------|
| **Process Assembly** | `.exe` | Executable, has entry point |
| **Library Assembly** | `.dll` | Reusable library, no entry point |

---

#### 📝 Assembly Contains

| Part | Purpose |
|------|---------|
| **IL Code** | Compiled code |
| **Metadata** | Info about types, methods |
| **Manifest** | Assembly name, version, dependencies |
| **Resources** | Images, strings, files |

---

### 🧠 Memory Tip
> **Assembly** = A **packed box** 📦 of your compiled code  
> ready to be **delivered and executed** by CLR.

---
### Q8. What is the difference between String and StringBuilder?

**Answer:**

| | String | StringBuilder |
|--|--------|---------------|
| **Mutable** | ❌ Immutable | ✅ Mutable |
| **Memory** | New object each change | Modifies same object |
| **Performance** | Slow (heavy changes) | Fast (heavy changes) |
| **Namespace** | `System` | `System.Text` |
| **Best For** | Few/no changes | Loops, concatenations |

---

#### 📁 Example
```csharp
// ❌ String — creates new object every time (slow in loops)
string str = "Hello";
str += " World";  // New object created in memory

// ✅ StringBuilder — modifies same object (fast in loops)
StringBuilder sb = new StringBuilder("Hello");
sb.Append(" World");
Console.WriteLine(sb.ToString()); // Output: Hello World
```

---

#### 📁 Performance Difference
```csharp
// ❌ Bad — 1000 new string objects created
string result = "";
for (int i = 0; i < 1000; i++)
    result += i;

// ✅ Good — only one object modified
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
    sb.Append(i);
```

---

### 🧠 Memory Tip
> **String** = Written in **stone** 🪨 — every change makes a new stone  
> **StringBuilder** = Written on a **whiteboard** 🖊️ — edit freely

---

### Q9. What is Garbage Collection in .NET?

**Answer:**  
**Garbage Collection (GC)** is an **automatic memory management** feature in .NET. It finds and removes objects that are **no longer used**, freeing memory automatically.

---

#### 🔑 Three Generations

| Generation | Contains | Collected |
|------------|----------|-----------|
| **Gen 0** | New short-lived objects | Most frequently |
| **Gen 1** | Survived Gen 0 | Less frequently |
| **Gen 2** | Long-lived objects | Least frequently |

---

#### 📁 Example
```csharp
// Object becomes eligible for GC when no reference points to it
void CreateObject()
{
    Student s = new Student(); // Created on Heap
} // s goes out of scope → eligible for GC

// Force GC manually (not recommended in production)
GC.Collect();
```

---

#### 📝 IDisposable — Manual Cleanup

For **unmanaged resources** (files, DB connections), use `IDisposable`:
```csharp
using (SqlConnection con = new SqlConnection(connString))
{
    // Auto disposed after using block ✅
}
```

---

### 🧠 Memory Tip
> **GC** = A **cleaner** 🧹 that sweeps unused objects  
> from memory **automatically** — you don't need to clean yourself.

---


### Q10. What is CORS?

**Answer:**  
**CORS (Cross-Origin Resource Sharing)** is a browser security feature that **blocks requests** from a different domain by default. Backend must send specific **HTTP headers** to allow access.

---

#### 🚫 Without CORS — Request Blocked
```javascript
fetch("https://mybackend.com/api/data")
  .then(res => res.json())
  .catch(err => console.error(err));

// ❌ Browser Error:
// "Access to fetch at 'https://mybackend.com/api/data'
//  from origin 'https://myfrontend.com' has been blocked by CORS policy."
```

---

### 🧠 Memory Tip
> **Frontend** `https://myfrontend.com` → calls → **Backend** `https://mybackend.com`  
> Different domain = browser **blocks** it 🚫 → Enable CORS on backend to **allow** it ✅

---
