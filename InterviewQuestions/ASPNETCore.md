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


----------------

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

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  

### Q2. Please Introduce yourself?
**Answer:**  