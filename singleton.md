# Singleton Pattern (Creational Design Pattern)

The **Singleton Pattern** is one of the most commonly used *Creational Design Patterns*.
Its main goal is to **ensure that only one object (instance)** of a class is created and that it is **accessible globally** throughout the program.

You can think of it like a **single manager** in a company — only one manager exists, and everyone uses the same manager for communication.

---

##  Why Use Singleton?

We use a Singleton when:

* Only **one instance** of a class should exist.
* You need **a global access point** to that instance.
* Useful for things like:

  * Logging
  * Database connection
  * Configuration manager
  * Thread pool
  * Printer spooler

---

## Key Features

1. **Private constructor** → Prevents direct object creation using `new`.
2. **Static instance pointer** → Holds the single object of the class.
3. **Public static method (`getInstance`)** → Gives access to that one instance.

---

## C++ Example (Simple and Clear)

```cpp
#include <iostream>
using namespace std;

class Singleton {
private:
    // Step 1: Create a static pointer to hold one instance
    static Singleton* instance;

    // Step 2: Make constructor private so no one can create object directly
    Singleton() {
        cout << "Singleton instance created!" << endl;
    }

public:
    // Step 3: Delete copy constructor and assignment operator
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    // Step 4: Public method to access the single instance
    static Singleton* getInstance() {
        if (instance == nullptr) {
            instance = new Singleton(); // create only once
        }
        return instance;
    }

    void showMessage() {
        cout << "Hello from Singleton object!" << endl;
    }
};

// Step 5: Initialize static instance to NULL
Singleton* Singleton::instance = nullptr;

// Step 6: Main function
int main() {
    Singleton* s1 = Singleton::getInstance();
    Singleton* s2 = Singleton::getInstance();

    s1->showMessage();

    if (s1 == s2) {
        cout << "Both pointers point to the same instance!" << endl;
    } else {
        cout << "Different instances!" << endl;
    }

    return 0;
}
```

---

## Output

```
Singleton instance created!
Hello from Singleton object!
Both pointers point to the same instance!
```

---

## Explanation (In Simple Words)

* When `getInstance()` is called the first time → it creates one object.
* The next time you call `getInstance()` → it returns the **same old object**.
* This way, **only one object** ever exists in the entire program.

---

## Thread-Safe Singleton (Modern C++11 version)

```cpp
#include <iostream>
#include <mutex>
using namespace std;

class Singleton {
private:
    static Singleton* instance;
    static mutex mtx;

    Singleton() { cout << "Thread-safe Singleton created!" << endl; }

public:
    static Singleton* getInstance() {
        lock_guard<mutex> lock(mtx);
        if (instance == nullptr)
            instance = new Singleton();
        return instance;
    }
};

Singleton* Singleton::instance = nullptr;
mutex Singleton::mtx;

int main() {
    Singleton* s1 = Singleton::getInstance();
    Singleton* s2 = Singleton::getInstance();
    return 0;
}
```

---

## Summary

| **Concept** | **Description**                                     |
| ----------- | --------------------------------------------------- |
| Purpose     | Ensure only one instance of a class exists.         |
| Constructor | Private (no direct creation).                       |
| Access      | Through a static method (`getInstance()`).          |
| Common Uses | Database connection, logger, configuration manager. |
| Benefit     | Saves memory, ensures consistency, central control. |

---
---
## Singleton Pattern** in **C# (.NET)** 


## 🧱 Basic (Naïve) Implementation

```csharp
public sealed class Singleton
{
    private static Singleton _instance = null;

    // Private constructor ensures external classes cannot instantiate it.
    private Singleton()
    {
        // Initialize expensive resources here if necessary
    }

    public static Singleton Instance
    {
        get
        {
            if (_instance == null)
                _instance = new Singleton();
            return _instance;
        }
    }

    public void DoWork()
    {
        Console.WriteLine("Singleton instance is working...");
    }
}
```

### ❌ Problems:

* **Not thread-safe.** Two threads could create two instances simultaneously.
* **No lazy initialization guarantees** in concurrent scenarios.

---

## ⚙️ Thread-Safe Implementation (Locking)

```csharp
public sealed class Singleton
{
    private static Singleton _instance = null;
    private static readonly object _lock = new object();

    private Singleton() { }

    public static Singleton Instance
    {
        get
        {
            lock (_lock)
            {
                if (_instance == null)
                    _instance = new Singleton();
                return _instance;
            }
        }
    }
}
```

### ✅ Pros:

* Thread-safe.
* Simple and explicit.

### ❌ Cons:

* Locking every time can reduce performance.

---

## ⚡ Double-Checked Locking

Improves performance by locking **only when necessary**.

```csharp
public sealed class Singleton
{
    private static Singleton _instance;
    private static readonly object _lock = new object();

    private Singleton() { }

    public static Singleton Instance
    {
        get
        {
            if (_instance == null)
            {
                lock (_lock)
                {
                    if (_instance == null)
                        _instance = new Singleton();
                }
            }
            return _instance;
        }
    }
}
```

### ✅ Pros:

* Thread-safe.
* Avoids unnecessary locking after initialization.

### ⚠️ Note:

* Works correctly in .NET due to memory model guarantees (since .NET 2.0).

---

## 🧵 Using `Lazy<T>` (Best Practice in Modern .NET)

C#’s `Lazy<T>` provides **built-in thread-safety and lazy initialization**.

```csharp
public sealed class Singleton
{
    private static readonly Lazy<Singleton> _instance =
        new Lazy<Singleton>(() => new Singleton());

    private Singleton() { }

    public static Singleton Instance => _instance.Value;
}
```

### ✅ Pros:

* Thread-safe by default.
* Lazy-loaded.
* Clean and modern.
* Supports custom thread-safety modes (`LazyThreadSafetyMode`).

---

## 🧰 Example Usage

```csharp
class Program
{
    static void Main()
    {
        Singleton.Instance.DoWork();
    }
}
```

Output:

```
Singleton instance is working...
```

---

## 🧩 Real-World Use Cases

| Use Case               | Example                                    |
| ---------------------- | ------------------------------------------ |
| **Logging**            | `Logger.Instance.Log("Message")`           |
| **Configuration**      | Load app settings only once                |
| **Caching**            | Maintain a single cache store              |
| **Connection Pooling** | Manage shared DB connections               |
| **Service Locators**   | Provide global access to service providers |

---

## 🧩 Singleton + Dependency Injection (Modern .NET Core)

In ASP.NET Core or .NET 8+, use **DI container** instead of manual singletons:

```csharp
// Program.cs
builder.Services.AddSingleton<IMyService, MyService>();

// Usage via constructor injection
public class MyController
{
    private readonly IMyService _service;
    public MyController(IMyService service)
    {
        _service = service;
    }
}
```

✅ **Better testability**, **lifecycle control**, and **cleaner design**.

---

## ⚖️ When *Not* to Use Singleton

Avoid if:

* The class maintains **mutable shared state** (risk of data races).
* You can achieve the same effect via **dependency injection**.
* It complicates **testing or scalability**.

---
---

## Singleton Pattern (Python version)

## ⚙️ 1. Basic (Naïve) Singleton in Python

```python
class Singleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super(Singleton, cls).__new__(cls)
            print("Creating new instance")
        return cls._instance

    def do_work(self):
        print("Singleton instance is working...")


# Usage
s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # True
```

### ✅ Pros:

* Simple.
* Works fine in single-threaded apps.

### ❌ Cons:

* **Not thread-safe** (two threads could create two instances simultaneously).

---

## 🧵 2. Thread-Safe Singleton (with Lock)

Using Python’s `threading` module.

```python
import threading

class Singleton:
    _instance = None
    _lock = threading.Lock()  # ensure thread safety

    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:  # double-checked locking
                    print("Creating new thread-safe instance")
                    cls._instance = super().__new__(cls)
        return cls._instance
```

### ✅ Pros:

* Thread-safe.
* Lazy initialization.

---

## 🧠 3. Singleton using a **Decorator**

A **decorator** makes any class a Singleton automatically.

```python
def singleton(cls):
    instances = {}

    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]

    return get_instance


@singleton
class Logger:
    def __init__(self):
        print("Initializing Logger")

    def log(self, msg):
        print(f"[LOG]: {msg}")


# Usage
l1 = Logger()
l2 = Logger()
print(l1 is l2)  # True
```

### ✅ Pros:

* Clean syntax.
* Reusable.
* Easy to apply to multiple classes.

### ❌ Cons:

* Doesn’t support inheritance easily.

---

## 🧩 4. Singleton using a **Metaclass**

Metaclasses allow controlling **class creation**, making them powerful for patterns like Singleton.

```python
class SingletonMeta(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            print(f"Creating new instance for {cls.__name__}")
            instance = super().__call__(*args, **kwargs)
            cls._instances[cls] = instance
        return cls._instances[cls]


class DatabaseConnection(metaclass=SingletonMeta):
    def __init__(self):
        print("Connecting to database...")


# Usage
db1 = DatabaseConnection()
db2 = DatabaseConnection()
print(db1 is db2)  # True
```

### ✅ Pros:

* Clean, extensible (works with inheritance).
* Centralized control via the metaclass.

### ❌ Cons:

* Metaclasses can be confusing to beginners.

---

## 🧱 5. Module-Level Singleton (Pythonic Way)

In Python, **modules are singletons by default** — they’re imported once per interpreter session.

```python
# logger.py
class Logger:
    def log(self, msg):
        print(f"[LOG]: {msg}")

logger = Logger()
```

Usage:

```python
# main.py
from logger import logger

logger.log("App started")
logger.log("Still the same instance")
```

### ✅ Pros:

* Simple.
* Pythonic.
* Automatically managed by the import system.

### ❌ Cons:

* Can’t easily subclass or reset instance.

---

## 🧰 6. Using `__new__` with Inheritance Support

If you need subclasses with independent singleton behavior:

```python
class SingletonBase:
    _instances = {}

    def __new__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__new__(cls)
        return cls._instances[cls]
```

### Example:

```python
class A(SingletonBase): pass
class B(SingletonBase): pass

a1, a2 = A(), A()
b1, b2 = B(), B()
print(a1 is a2, b1 is b2, a1 is b1)  # True True False
```

---

## 🧩 Real-World Example — Logging Service

```python
import threading

class Logger(metaclass=SingletonMeta):
    _lock = threading.Lock()

    def __init__(self):
        self._messages = []

    def log(self, msg):
        with self._lock:
            self._messages.append(msg)
            print(f"[LOG]: {msg}")

    def get_logs(self):
        return list(self._messages)
```

Usage:

```python
def worker():
    Logger().log("Worker started")

threads = [threading.Thread(target=worker) for _ in range(3)]
for t in threads:
    t.start()
for t in threads:
    t.join()
```

Output:

```
Creating new instance for Logger
[LOG]: Worker started
[LOG]: Worker started
[LOG]: Worker started
```

---

## ⚖️ When *Not* to Use Singleton in Python

Avoid Singletons when:

* You need **multiple independent instances** (testing, multiprocessing).
* They hide **global mutable state**, leading to hard-to-debug behavior.
* Your codebase can use **dependency injection** (more testable and flexible).

---
---

## **License**
This project is licensed under the MIT License.

---

Happy Coding!
