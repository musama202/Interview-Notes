# C# Interview Questions

---

### Q1. What is TPL (Task Parallel Library) in C# / .NET Core?

**Answer:**  
The **Task Parallel Library (TPL)** in .NET Core is used to execute multiple operations concurrently using **tasks** instead of manually managing threads.  
It simplifies parallel programming by automatically handling **thread management**, **scheduling**, and **load balancing**, resulting in better performance for **CPU-bound** and **I/O-bound** operations.  
It also integrates seamlessly with **async/await** for asynchronous programming.

**Example:**
```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        // Start multiple tasks in parallel
        Task task1 = Task.Run(() => DoWork("Task 1"));
        Task task2 = Task.Run(() => DoWork("Task 2"));

        // Wait for both tasks to complete
        Task.WaitAll(task1, task2);

        Console.WriteLine("All tasks completed.");
    }

    static void DoWork(string taskName)
    {
        Console.WriteLine($"{taskName} started on thread {Task.CurrentId}");
        Task.Delay(1000).Wait(); // simulate work
        Console.WriteLine($"{taskName} completed.");
    }
}
```
----------------------------
### Q2. Before the Task Parallel Library (TPL), how did we handle multithreading or parallel work in .NET?
**Answer:**  
Before TPL, developers had to manually create and manage threads using the Thread class or the ThreadPool.
**Example:**
```csharp
using System;
using System.Threading;

class Program
{
    static void Main()
    {
        Thread t1 = new Thread(() => DoWork("Thread 1"));
        Thread t2 = new Thread(() => DoWork("Thread 2"));

        t1.Start();
        t2.Start();

        t1.Join(); // Wait for t1 to finish
        t2.Join(); // Wait for t2 to finish

        Console.WriteLine("All threads completed.");
    }

    static void DoWork(string name)
    {
        Console.WriteLine($"{name} started...");
        Thread.Sleep(1000); // Simulate work
        Console.WriteLine($"{name} completed.");
    }
}
```
---------------------------
### Q2. How TPL is different beteen async/await?

**Answer:**  
async/await is for asynchronous programming, mainly for I/O-bound tasks, allowing threads to be freed while waiting for operations to complete.
TPL is for parallel programming, mainly CPU-bound tasks, where multiple operations run concurrently on multiple threads.
While TPL focuses on parallelism, async/await focuses on asynchronous non-blocking execution.

---------------------------
### Q2. What is different between I/O bound and Cpu bound?
**Answer:**  
A CPU-bound task is limited by the CPU’s processing power and spends most of its time computing, while an I/O-bound task spends most of its time waiting for external operations like disk or network access.
CPU-bound tasks benefit from parallel execution to utilize multiple cores, whereas I/O-bound tasks benefit from asynchronous programming (async/await) to free threads while waiting, improving scalability.”

---------------------------
### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  