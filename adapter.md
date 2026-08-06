# Design Patterns Master Course

# Phase 2 – Structural Design Patterns

Congratulations! You have completed all six **Creational Patterns**.

You now know how architects think about **creating objects**.

The next question is:

> **"Now that I have these objects, how do I organize them so they can work together?"**

That is the purpose of **Structural Design Patterns**.

---

# Design Pattern Roadmap

## ✅ Creational Patterns (Completed)

* Singleton
* Factory Method
* Abstract Factory
* Builder
* Prototype
* Object Pool

---

## Structural Patterns (Starting Today)

1. **Adapter Pattern** ← Today
2. Bridge Pattern
3. Composite Pattern
4. Decorator Pattern
5. Facade Pattern
6. Flyweight Pattern
7. Proxy Pattern
8. Private Class Data Pattern

---

# Before We Start

Imagine this situation.

Your company purchases a new CT Scanner.

Old Scanner API:

```text
getImage()
```

New Scanner API:

```text
captureSlice()
```

Your application contains:

```text
1000 files
```

Every file calls:

```cpp
scanner.getImage();
```

Should you modify all 1000 files?

A junior developer says:

> "Yes."

A software architect says:

> **"No. I'll adapt the new scanner so the existing software doesn't change."**

That is the **Adapter Pattern**.

---

# 1. Introduction

## What is the Adapter Pattern?

The **Adapter Pattern** converts the interface of one class into another interface that the client expects.

In simple words:

> **It makes two incompatible classes work together without changing either one.**

It acts like a **translator**.

---

## Why was it created?

Software constantly changes.

Examples:

* New hardware
* New APIs
* Third-party libraries
* Legacy code
* Vendor SDK updates
* Operating system changes

Often you cannot modify:

* Legacy code
* Third-party libraries
* Vendor SDKs

Instead of rewriting your application, you insert an **adapter**.

---

## Category

**Structural Pattern**

Structural patterns focus on:

* relationships between classes,
* composition,
* object collaboration,
* system organization.

---

## What problem does it solve?

Imagine:

Your application expects:

```cpp
Image image = scanner.getImage();
```

But the new SDK provides:

```cpp
Image image = scanner.captureSlice();
```

Interfaces don't match.

Without Adapter:

```text
Application

↓

Compilation Error
```

With Adapter:

```text
Application

↓

Adapter

↓

New Scanner SDK
```

The application remains unchanged.

---

# 2. Problem Statement

Imagine you're developing a **Hospital Information System (HIS)**.

Your software already integrates with:

```text
Philips CT
```

using:

```cpp
loadPatientImage();
```

The hospital buys a new scanner from Siemens.

Its SDK provides:

```cpp
fetchPatientScan();
```

The names and interfaces differ.

Without Adapter:

```text
Every Module

↓

Modify Code

↓

Retest Everything

↓

Deploy Again
```

Hundreds of files must change.

This increases:

* development time,
* testing effort,
* risk of bugs.

---

# 3. Motivation

Software architects noticed that systems often need to integrate with components that were designed independently.

Changing either side is often impossible:

* vendor libraries,
* legacy systems,
* operating system APIs,
* external services.

Instead of changing both sides, introduce one class that translates between them.

The Adapter Pattern formalizes this idea.

---

# 4. Real-World Analogy

## Mobile Phone Charger Adapter

You travel from India to Europe.

Indian plug:

```text
Type D
```

European socket:

```text
Type C
```

They don't fit.

Do you redesign the house?

Do you redesign your charger?

No.

You buy an adapter.

```text
Phone Charger

↓

Travel Adapter

↓

European Socket
```

The adapter translates the connection.

### Mapping

| Real World      | Software             |
| --------------- | -------------------- |
| Phone Charger   | Client               |
| Travel Adapter  | Adapter              |
| European Socket | Existing Library/API |
| Electricity     | Requested Service    |

The charger doesn't know the adapter exists.

---

# 5. Software Scenario

Adapter appears whenever different interfaces must cooperate.

### Desktop Applications

* Legacy library integration
* Printer drivers
* File format compatibility

---

### Qt Applications

Suppose your application expects a custom logging interface:

```cpp
ILogger
```

But a third-party library exposes:

```cpp
ThirdPartyLogger::writeMessage()
```

Create:

```text
QtLoggerAdapter

↓

ILogger
```

The rest of your Qt application continues using `ILogger`.

---

### CAD Software

Different CAD kernels:

* OpenCascade
* Parasolid
* ACIS

Each exposes different APIs.

An adapter presents one common interface to the application.

---

### Medical Imaging

Different DICOM SDKs:

* DCMTK
* GDCM
* Vendor SDK

Each has different function names and data structures.

Adapters allow the application to work with a single imaging interface.

---

### Game Engines

Different controller APIs.

---

### Browsers

Different operating system networking APIs.

---

# 6. UML Class Diagram

```text
                    +----------------------+
                    |        Target        |
                    +----------------------+
                    | request()            |
                    +----------+-----------+
                               ^
                               |
                    +----------------------+
                    |       Adapter        |
                    +----------------------+
                    | request()            |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |       Adaptee        |
                    +----------------------+
                    | specificRequest()    |
                    +----------------------+

              Client
                 |
                 v
              Target
```

---

## Responsibilities

### Target

The interface expected by the client.

---

### Adapter

Translates calls.

---

### Adaptee

Existing class with an incompatible interface.

---

### Client

Works only with the Target interface.

---

# 7. Participants

## Target

Example:

```text
ImageLoader
```

Defines:

```cpp
loadImage()
```

---

## Adapter

Implements:

```cpp
loadImage()
```

Internally calls:

```cpp
vendorSDK.fetchImage()
```

---

## Adaptee

Third-party SDK.

Cannot be modified.

---

## Client

Uses only:

```cpp
loadImage()
```

---

# 8. Collaboration

Runtime Flow

```text
Client

↓

loadImage()

↓

Adapter

↓

fetchImage()

↓

Vendor SDK

↓

Image Returned

↓

Adapter Converts

↓

Client Receives Image
```

Notice:

The client never knows translation occurred.

---

# 9. C++ Example

Suppose our application expects a `Printer` interface.

```cpp
#include <iostream>
#include <string>

// Target
class Printer
{
public:
    virtual ~Printer() = default;
    virtual void print(const std::string& text) = 0;
};

// Adaptee (Third-party library)
class LegacyPrinter
{
public:
    void printText(const std::string& text)
    {
        std::cout << "Legacy Printer: " << text << '\n';
    }
};

// Adapter
class PrinterAdapter : public Printer
{
public:
    explicit PrinterAdapter(LegacyPrinter& printer)
        : m_printer(printer)
    {}

    void print(const std::string& text) override
    {
        m_printer.printText(text);
    }

private:
    LegacyPrinter& m_printer;
};

int main()
{
    LegacyPrinter legacy;

    PrinterAdapter adapter(legacy);

    Printer* printer = &adapter;

    printer->print("Hello Adapter Pattern");
}
```

---

## Design Focus

The client knows only:

```cpp
Printer
```

It has no dependency on:

```cpp
LegacyPrinter
```

If tomorrow another printer library arrives, you write another adapter.

The client code remains unchanged.

---

# 10. Qt Example

Suppose your Qt application uses:

```cpp
class ImageLoader
{
public:
    virtual QImage load(const QString&) = 0;
};
```

One project uses **DCMTK**.

Another uses **GDCM**.

Create:

```text
DCMTKAdapter

↓

ImageLoader
```

and

```text
GDCMAdapter

↓

ImageLoader
```

Your `MainWindow`, thumbnail view, and image viewer all use only:

```cpp
loader->load(fileName);
```

Changing the DICOM library means replacing the adapter, not modifying the UI.

---

# 11. Medical Software Example

Imagine a **Treatment Planning System (TPS)** that supports multiple LINAC vendors.

The Dose Engine expects:

```cpp
BeamData getBeam();
```

Vendor APIs expose:

```text
Varian

↓

readBeam()
```

```text
Elekta

↓

loadBeam()
```

```text
Siemens

↓

fetchBeam()
```

Instead of changing the Dose Engine:

Create:

```text
VarianBeamAdapter

↓

BeamProvider
```

```text
ElektaBeamAdapter

↓

BeamProvider
```

```text
SiemensBeamAdapter

↓

BeamProvider
```

Now the Dose Engine simply calls:

```cpp
beamProvider->getBeam();
```

The vendor-specific differences are isolated inside adapters.

This dramatically reduces coupling and simplifies support for new hardware.

---

# 12. Advantages

### Reuse

Integrates existing code without modification.

### Loose Coupling

Clients depend on the Target interface.

### Maintainability

Vendor-specific logic is isolated.

### Scalability

Adding another library usually means adding another adapter.

### Backward Compatibility

Old code continues working with new components.

---

# 13. Disadvantages

### Additional Layer

Adds another class.

### Translation Overhead

There may be minor runtime overhead due to conversion.

### Can Hide Design Problems

If many adapters accumulate, it may indicate the architecture needs redesign.

### When NOT to Use

Avoid Adapter when:

* both interfaces can be changed directly,
* the interfaces are already compatible,
* no translation is required.

---

# 14. Best Practices

* Keep adapters focused on interface translation.
* Avoid putting business logic inside the adapter.
* Program against the Target interface.
* One adapter should typically adapt one external interface.
* Isolate third-party dependencies behind adapters to protect the rest of the system.

---

# 15. Common Mistakes

### Mistake 1

Putting business logic inside the adapter.

Adapters translate interfaces—they should not implement domain rules.

---

### Mistake 2

Letting the client know about the adaptee.

If client code still calls vendor-specific APIs, the benefit is lost.

---

### Mistake 3

Creating one giant adapter for multiple unrelated libraries.

Architects prefer small, focused adapters.

---

### Mistake 4

Using Adapter when a simple wrapper or interface redesign would be sufficient.

Choose the simplest design that solves the problem.

---

# 16. Pattern Variations

### 1. Object Adapter (Most Common)

Uses composition.

```text
Adapter

↓

Has Adaptee
```

Preferred in modern C++ because it is flexible and works with classes that cannot be inherited.

---

### 2. Class Adapter

Uses inheritance.

```text
Adapter

↓

inherits Target

↓

inherits Adaptee
```

This requires language support for multiple inheritance and is less common in modern designs.

---

### 3. Two-Way Adapter

Allows communication in both directions between two incompatible interfaces.

Used less frequently.

---

# 17. Related Patterns

| Pattern   | Difference                                                                  |
| --------- | --------------------------------------------------------------------------- |
| Adapter   | Makes incompatible interfaces work together.                                |
| Bridge    | Separates abstraction from implementation before they evolve independently. |
| Facade    | Simplifies a complex subsystem with a new interface.                        |
| Proxy     | Controls access to another object without changing its interface.           |
| Decorator | Adds responsibilities while keeping the same interface.                     |

### A Simple Memory Trick

* **Adapter** → *Translate*
* **Bridge** → *Separate*
* **Facade** → *Simplify*
* **Proxy** → *Control*
* **Decorator** → *Enhance*

---

# 18. Industry Usage

Adapter is everywhere because software constantly integrates with external systems.

* **Microsoft:** Windows API wrappers and legacy component integration.
* **Google:** Service adapters for different backend implementations.
* **Adobe:** Import/export plugins supporting many file formats.
* **Qt:** Platform abstraction layers and integrations with native operating system services.
* **ZEISS / Siemens / Philips:** Hardware abstraction layers that adapt different imaging devices and vendor SDKs to a common internal interface.
* **Autodesk:** CAD kernel adapters allowing different geometry engines to work with the same application.
* **Game Engines:** Input device adapters, physics engine adapters, and graphics API adapters.

The architectural goal is **protecting the application from external interface changes**.

---

# 19. Interview Questions

## Beginner

1. What problem does the Adapter Pattern solve?
2. Why is it called a Structural Pattern?
3. What are the roles of Target, Adapter, and Adaptee?

---

## Intermediate

1. What is the difference between Object Adapter and Class Adapter?
2. Why is composition generally preferred over inheritance for adapters?
3. How does Adapter reduce coupling?

---

## Advanced

1. How would you design adapters for multiple DICOM libraries (DCMTK, GDCM, vendor SDKs)?
2. How would you test an adapter independently of the external SDK?
3. What architectural risks arise if adapters begin containing business logic?

---

## Scenario-Based

Your hospital upgrades from one CT scanner vendor to another with a completely different SDK.

Would you modify every module in the application or introduce an adapter? Explain your reasoning and describe the responsibilities of the adapter.

---

## Architecture

Design an adapter-based architecture for a medical imaging platform that supports:

* Multiple DICOM libraries
* Multiple scanner vendors
* Multiple PACS providers

Explain:

* the Target interfaces,
* the Adaptees,
* the Adapters,
* and how the rest of the application remains isolated from vendor-specific code.

---

# 20. Practice Exercises

### Beginner Exercise

Design a `PaymentGateway` interface expected by your application. Adapt an existing third-party payment SDK that exposes a different API. Draw the class relationships.

---

### Intermediate Exercise

Design a Qt application that supports both **DCMTK** and **GDCM** for DICOM loading using the Adapter Pattern. Explain how the image viewer remains independent of the chosen library.

---

### Advanced Exercise

Design an adapter architecture for a **Treatment Planning System (TPS)** that supports three LINAC vendors:

* Varian
* Elekta
* Siemens

Each vendor provides different APIs for:

* Beam Data
* Machine Configuration
* MLC Control
* Patient Setup

Design:

* A common `MachineProvider` interface.
* Vendor-specific adapters.
* The collaboration between the Dose Engine and the adapters.
* Error handling for vendor-specific failures.

**Do not implement the solution yet.** Focus on architecture, interface design, and dependency management.

---

# Key Architectural Takeaway

The **Adapter Pattern** is not about changing existing code.

It is about **protecting your application from incompatible interfaces**.

A junior developer thinks:

> "The new SDK is different. I'll update all the existing code."

A software architect thinks:

> **"The rest of my application already works. I'll isolate the incompatibility behind an adapter so only one place knows about the external API."**

That mindset keeps large systems maintainable, reduces the impact of third-party changes, and is especially valuable in domains like medical imaging, CAD, enterprise software, and hardware integration.

When you're ready, type **`NEXT`** to continue with **Lesson 8: Bridge Pattern**.













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
