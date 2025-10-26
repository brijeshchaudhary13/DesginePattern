# 🔗 Topic 8: Adapter Pattern (Structural Design Pattern)

## 🧠 Overview

The **Adapter Pattern** is a *Structural Design Pattern* that allows **incompatible interfaces to work together**.

In simple words:
➡️ “It acts as a bridge between two classes that otherwise cannot communicate.”

---

## 🎯 Real-Life Example

Imagine you have a **USB device** and a **power socket**.
The USB plug doesn’t fit the socket directly — you need an **adapter**.
Similarly, in C++, an Adapter **translates one interface to another** so classes can work together.

---

## ⚙️ Key Idea

1. Adapter wraps an **existing class** with a new interface.
2. Client code interacts with the **expected interface**, not the adaptee.
3. Helps integrate **legacy or third-party code** without modification.

---

## 🧩 C++ Example (Simple and Clear)

```cpp
#include <iostream>
using namespace std;

// Step 1: Target interface (what client expects)
class Target {
public:
    virtual void request() = 0;
};

// Step 2: Adaptee (existing class with incompatible interface)
class Adaptee {
public:
    void specificRequest() {
        cout << "Specific request from Adaptee class 🔧" << endl;
    }
};

// Step 3: Adapter class (makes Adaptee compatible with Target)
class Adapter : public Target {
private:
    Adaptee* adaptee;
public:
    Adapter(Adaptee* a) : adaptee(a) {}
    void request() override {
        // Translate request to adaptee’s method
        adaptee->specificRequest();
    }
};

// Step 4: Client code
int main() {
    Adaptee* oldSystem = new Adaptee();
    Target* adapter = new Adapter(oldSystem);

    // Client calls request() on Target
    adapter->request();

    delete oldSystem;
    delete adapter;
    return 0;
}
```

---

## 🧾 Output

```
Specific request from Adaptee class 🔧
```

---

## 🔍 Explanation (In Simple Words)

* `Adaptee` has a method (`specificRequest`) that client **cannot use directly**.
* `Target` defines the **interface the client expects**.
* `Adapter` **wraps Adaptee** and makes it compatible with Target.
* Client code works with **Target interface only**, no changes needed to Adaptee.

---

## 🧠 When to Use Adapter Pattern

* You have **existing classes** with incompatible interfaces.
* You want to **reuse legacy or third-party code** without modification.
* You want to provide a **common interface** to different implementations.

---

## 🏁 Summary

| **Concept**  | **Description**                                                         |
| ------------ | ----------------------------------------------------------------------- |
| Purpose      | Allows incompatible interfaces to work together.                        |
| Key Feature  | Adapter converts interface of a class into another expected by clients. |
| Benefit      | Reuse existing code without changing it.                                |
| Example      | USB-to-socket adapter, legacy code integration.                         |
| Pattern Type | Structural Design Pattern                                               |

---

✅ **Quick Comparison with Structural Patterns**

| Pattern   | Purpose                                     |
| --------- | ------------------------------------------- |
| Adapter   | Convert interface to expected one           |
| Bridge    | Separate abstraction and implementation     |
| Composite | Treat group of objects like a single object |
| Decorator | Add functionality to objects dynamically    |
| Facade    | Provide simple interface to complex system  |
| Flyweight | Reduce memory usage by sharing objects      |
| Proxy     | Control access to objects                   |

---
---

Excellent — let’s do a **complete deep dive** into the **Adapter Pattern (Structural Design Pattern)** in **C# (.NET)** and **Python**, side by side.

We’ll go from **concept → structure → C# example → Python example → real-world analogy → when to use**.

---

# 🧩 Adapter Pattern — Overview

## 🧠 Intent

> The **Adapter Pattern** allows incompatible interfaces to work together.
> It acts as a **translator or bridge** between two unrelated classes.

---

### 💬 Real-Life Analogy

Think of a **power adapter**:

* Your **laptop charger** (client) expects a 3-pin socket.
* You’re in Europe where there are **2-pin sockets**.
* The **adapter** converts between the two interfaces so that the charger can work.

---

# ⚙️ Problem

You have a **client** class that depends on a certain interface (`ITarget`),
but you want to use an **existing class (Adaptee)** that doesn’t match this interface.

Instead of modifying the existing class (which might not be possible or desirable),
you write an **Adapter** that converts the interface.

---

# 🧩 Structure

```
Client ---> ITarget (expected interface)
               ↑
               |
         Adapter (wraps Adaptee)
               |
           Adaptee (existing incompatible class)
```

---

# 🧱 C# Implementation — Adapter Pattern

Let’s walk through a concrete example.

## Scenario:

You have a **modern payment system** expecting an interface `INewPaymentProcessor`,
but you need to integrate an **old legacy payment gateway**.

---

### Step 1️⃣ – Target Interface

```csharp
public interface INewPaymentProcessor
{
    void ProcessPayment(decimal amount);
}
```

---

### Step 2️⃣ – Adaptee (Existing / Legacy Class)

```csharp
public class LegacyPaymentSystem
{
    public void MakePayment(int amountInCents)
    {
        Console.WriteLine($"Legacy system processed payment: {amountInCents} cents");
    }
}
```

---

### Step 3️⃣ – Adapter

```csharp
public class PaymentAdapter : INewPaymentProcessor
{
    private readonly LegacyPaymentSystem _legacySystem;

    public PaymentAdapter(LegacyPaymentSystem legacySystem)
    {
        _legacySystem = legacySystem;
    }

    public void ProcessPayment(decimal amount)
    {
        int amountInCents = (int)(amount * 100);
        _legacySystem.MakePayment(amountInCents);
    }
}
```

---

### Step 4️⃣ – Client Code

```csharp
class Program
{
    static void Main()
    {
        LegacyPaymentSystem legacySystem = new LegacyPaymentSystem();
        INewPaymentProcessor paymentProcessor = new PaymentAdapter(legacySystem);

        paymentProcessor.ProcessPayment(99.99m);
    }
}
```

✅ **Output:**

```
Legacy system processed payment: 9999 cents
```

---

## 🔍 Notes on C# Implementation

* The adapter **implements the target interface** (`INewPaymentProcessor`).
* It **wraps the adaptee** (`LegacyPaymentSystem`) to translate method calls.
* This approach is **type-safe** and **enforces interface contracts**.

---

# 🐍 Python Implementation — Adapter Pattern

Now let’s do the same concept in **Python**.

---

### Step 1️⃣ – Target Interface (expected by client)

Python doesn’t have strict interfaces like C#, but we can define a target protocol.

```python
class NewPaymentProcessor:
    def process_payment(self, amount: float):
        raise NotImplementedError
```

---

### Step 2️⃣ – Adaptee (existing incompatible class)

```python
class LegacyPaymentSystem:
    def make_payment(self, amount_in_cents: int):
        print(f"Legacy system processed payment: {amount_in_cents} cents")
```

---

### Step 3️⃣ – Adapter

```python
class PaymentAdapter(NewPaymentProcessor):
    def __init__(self, legacy_system: LegacyPaymentSystem):
        self.legacy_system = legacy_system

    def process_payment(self, amount: float):
        amount_in_cents = int(amount * 100)
        self.legacy_system.make_payment(amount_in_cents)
```

---

### Step 4️⃣ – Client Code

```python
if __name__ == "__main__":
    legacy_system = LegacyPaymentSystem()
    payment_processor = PaymentAdapter(legacy_system)
    payment_processor.process_payment(99.99)
```

✅ **Output:**

```
Legacy system processed payment: 9999 cents
```

---

## 🧠 Comparison — C# vs Python

| Concept                | C# (.NET)                           | Python                              |
| ---------------------- | ----------------------------------- | ----------------------------------- |
| Interface              | Uses `interface` keyword            | Convention or abstract base class   |
| Static typing          | Strong (compile-time check)         | Dynamic (duck typing)               |
| Adapter implementation | Must explicitly implement interface | Can implement methods dynamically   |
| Usage                  | Common in enterprise integrations   | Very flexible due to dynamic nature |

---

## 💡 Real-World Use Cases

| Domain          | Example                                             |
| --------------- | --------------------------------------------------- |
| Payment Systems | Adapting legacy APIs to modern REST/GraphQL APIs    |
| Data Access     | Wrapping an old database driver                     |
| Logging         | Adapting between different logging frameworks       |
| Microservices   | Connecting old SOAP services to new HTTP clients    |
| Hardware        | Bridging different hardware communication protocols |

---

# 🧰 Modern C# Example — with Dependency Injection (ASP.NET Core)

In modern .NET apps, you’d often register adapters as services:

```csharp
// Startup.cs or Program.cs
services.AddTransient<INewPaymentProcessor, PaymentAdapter>();
```

And then inject it:

```csharp
public class PaymentController : ControllerBase
{
    private readonly INewPaymentProcessor _processor;

    public PaymentController(INewPaymentProcessor processor)
    {
        _processor = processor;
    }

    [HttpPost("pay")]
    public IActionResult Pay(decimal amount)
    {
        _processor.ProcessPayment(amount);
        return Ok();
    }
}
```

---

# 🧩 Alternative Python Example — Adapter for Third-Party Library

Suppose you use an **external library** with an incompatible API:

```python
# Third-party library
class StripeAPI:
    def pay(self, value):
        print(f"Stripe paid {value} USD")
```

Your system expects:

```python
class PaymentGateway:
    def process(self, amount):
        raise NotImplementedError
```

Adapter:

```python
class StripeAdapter(PaymentGateway):
    def __init__(self, stripe_api: StripeAPI):
        self.stripe_api = stripe_api

    def process(self, amount):
        self.stripe_api.pay(amount)
```

Client:

```python
gateway = StripeAdapter(StripeAPI())
gateway.process(150)
```

Output:

```
Stripe paid 150 USD
```

---

# 🧭 Summary

| Feature               | C# (.NET)                           | Python                                      |
| --------------------- | ----------------------------------- | ------------------------------------------- |
| Interface enforcement | Compile-time via `interface`        | Convention or ABC                           |
| Static typing         | Strongly typed                      | Duck typing                                 |
| Adapter type          | Class-based                         | Class-based or function-based               |
| Typical use           | Enterprise integration, API bridges | Wrapping libraries, flexible data adapters  |
| Code verbosity        | More verbose                        | Concise                                     |
| Dependency Injection  | Common in .NET Core                 | Possible via frameworks (FastAPI, Flask DI) |

---

# 🧠 Key Takeaways

* **Adapter Pattern** = “Translator” between incompatible interfaces.
* **C#** uses explicit interfaces and strong typing — more structured but verbose.
* **Python** uses duck typing — more flexible and concise.
* Both ensure **decoupling** and **reuse** of legacy or third-party code.

---

Would you like me to extend this with a **real-world integrated example**,
such as connecting a **modern REST payment API (C#)** and a **legacy SOAP service (Python)** using adapters to bridge both?
