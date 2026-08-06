# Design Patterns Master Course

# Lesson 14 — Private Class Data Pattern (Structural Pattern)

---

# Before We Start

This is probably the **least discussed** GoF-related pattern in many courses, but it is extremely valuable in:

* Medical Device Software
* Aerospace
* Banking
* Embedded Systems
* Automotive (AUTOSAR)
* Industrial Automation
* Safety-Critical Systems

Many developers have never heard of it.

However, if you work at companies like:

* ZEISS
* Siemens Healthineers
* Philips
* GE Healthcare
* Medtronic

you will often see architectures that follow this principle.

---

# The Architect's Way of Thinking

Suppose you're writing software for a **Radiotherapy Treatment Planning System (TPS)**.

You have a class:

```cpp
class DoseEngine
{
public:
    double dose;
    double beamEnergy;
    double machineCalibration;
};
```

Another developer writes:

```cpp
DoseEngine engine;

engine.machineCalibration = 0;
```

Now every patient's dose calculation is wrong.

The compiler doesn't complain.

The code compiles.

The software may even pass basic testing.

This is a disaster in medical software.

An architect immediately asks:

> **"Why was external code allowed to modify internal state directly?"**

The answer is:

**Poor encapsulation.**

The **Private Class Data Pattern** exists to solve this.

---

# A Common Misunderstanding

Many developers think:

> "Making members `private` solves the problem."

Not completely.

The pattern goes one step further.

Instead of storing sensitive data directly inside the public class, it places that data into a **separate private data object**.

This allows:

* Better encapsulation
* Easier validation
* Better security
* Stable public interfaces

---

# 1. Introduction

## What is the Private Class Data Pattern?

The **Private Class Data Pattern** separates an object's **sensitive internal state** into a private helper object that cannot be modified directly by clients.

Simple definition:

> **Keep critical data in a separate private object and expose only controlled operations.**

---

## Why was it created?

Large systems often contain data that must remain valid.

Examples:

* Machine calibration
* Beam energy
* Patient ID
* Encryption keys
* Database credentials
* Medical device configuration

Allowing arbitrary modification is dangerous.

---

## Category

**Structural Pattern**

Although it deals with encapsulation, it is commonly classified among Structural patterns because it reorganizes object structure.

---

## What problem does it solve?

Without the pattern:

```text
DoseEngine
 ├── dose
 ├── beamEnergy
 ├── calibration
```

Even if members are private, many setter methods may expose too much.

With the pattern:

```text
DoseEngine
    |
    | owns
    v
PrivateData
 ├── calibration
 ├── beamEnergy
 ├── limits
```

The public object controls every modification.

---

# 2. Problem Statement

Imagine a **Bank Account**.

Without protection:

```cpp
account.balance = 100000000;
```

Anyone can change it.

Even if `balance` is private:

```cpp
setBalance(value);
```

may still allow invalid values.

Architecturally, we want:

```cpp
deposit()

withdraw()

transfer()
```

Only these operations may change the balance.

The internal state remains protected.

---

# 3. Motivation

Architects noticed that many classes expose too much mutable state.

This causes:

* Invalid objects
* Broken invariants
* Difficult debugging
* Hidden dependencies
* Safety issues

Instead of exposing data, expose behavior.

This aligns with one of the core ideas of Object-Oriented Design:

> **Objects should protect their own consistency.**

---

# 4. Real-World Analogy

## Bank Vault

Customer:

```text
Deposit Money
```

The customer never enters the vault.

Instead:

```text
Customer

↓

Bank Teller

↓

Vault
```

The vault stores the real assets.

Only authorized operations can change them.

### Mapping

| Real World | Software        |
| ---------- | --------------- |
| Vault      | Private Data    |
| Teller     | Public Class    |
| Customer   | Client          |
| Money      | Sensitive State |

---

# 5. Software Scenario

Private Class Data is useful whenever internal state must remain protected.

### Desktop Applications

* License keys
* User preferences
* Internal caches

---

### Qt Applications

Qt uses the **Pimpl (Pointer to Implementation)** idiom extensively.

Example:

```cpp
class QPushButton
{
private:
    QPushButtonPrivate *d;
};
```

`QPushButtonPrivate` stores internal implementation details.

While the primary motivation of Qt's Pimpl is **binary compatibility (ABI stability)** and reduced compile-time dependencies, it also improves encapsulation by hiding implementation details.

---

### CAD Software

Protect:

* Geometry tolerances
* Internal caches
* Constraint solvers

---

### Medical Imaging

Protect:

* Calibration values
* Scanner configuration
* Dose limits

---

### Banking

Protect:

* Balance
* Security Tokens
* Encryption Keys

---

# 6. UML Class Diagram

```text
              +----------------------+
              |      Public Class    |
              +----------------------+
              | +operation()         |
              | - privateData        |
              +----------+-----------+
                         |
                         | owns
                         v
              +----------------------+
              |   PrivateClassData   |
              +----------------------+
              | sensitive members    |
              +----------------------+
```

---

## Responsibilities

### Public Class

* Exposes public API
* Validates requests
* Protects invariants

---

### Private Data

Stores:

* sensitive state
* internal configuration
* implementation details

---

### Client

Cannot access private data directly.

---

# 7. Participants

## Public Class

Example:

```text
DoseEngine
```

Public methods:

```cpp
calculateDose()

loadMachine()
```

---

## Private Class Data

Stores:

```text
Machine Calibration

Energy

Tolerance

Configuration
```

---

## Client

Uses only:

```cpp
engine.calculateDose();
```

Never:

```cpp
engine.calibration = ...
```

---

# 8. Collaboration

Runtime Flow

```text
Client

↓

calculateDose()

↓

DoseEngine

↓

Read Private Data

↓

Perform Validation

↓

Compute Dose

↓

Return Result
```

The client never interacts with the internal state.

---

# 9. C++ Example

```cpp
#include <iostream>
#include <memory>

// Private Data
class BankAccountData
{
public:
    explicit BankAccountData(double balance)
        : balance(balance)
    {}

    double balance;
};

// Public Class
class BankAccount
{
public:
    explicit BankAccount(double initialBalance)
        : m_data(std::make_unique<BankAccountData>(initialBalance))
    {}

    void deposit(double amount)
    {
        if (amount > 0)
            m_data->balance += amount;
    }

    void withdraw(double amount)
    {
        if (amount > 0 && amount <= m_data->balance)
            m_data->balance -= amount;
    }

    double getBalance() const
    {
        return m_data->balance;
    }

private:
    std::unique_ptr<BankAccountData> m_data;
};

int main()
{
    BankAccount account(1000);

    account.deposit(500);

    std::cout << account.getBalance();
}
```

---

## Design Focus

Notice:

The client cannot modify:

```cpp
balance
```

directly.

Only controlled operations are available.

This preserves object validity.

---

# 10. Qt Example

Qt's **Pimpl (d-pointer)** is one of the best-known examples.

```cpp
class MedicalImageWidget
{
public:
    void loadImage();

private:
    std::unique_ptr<MedicalImageWidgetPrivate> d;
};
```

The private class stores:

```text
OpenGL Context

Image Cache

Rendering State

Internal Flags
```

The public header remains stable even if the private implementation changes.

This is one reason Qt has maintained strong binary compatibility across many releases.

---

# 11. Medical Software Example

Imagine a **Linear Accelerator Configuration**.

Sensitive values:

```text
Machine Calibration

Beam Energy

Mechanical Limits

Safety Thresholds

Commissioning Data
```

Instead of exposing them:

```cpp
MachineConfiguration
```

owns:

```cpp
MachineConfigurationPrivate
```

Only approved operations can:

* load calibration,
* validate machine settings,
* update commissioning data.

No external module can arbitrarily modify critical parameters.

This reduces the risk of unsafe states.

---

# 12. Advantages

### Strong Encapsulation

Sensitive data stays hidden.

### Data Integrity

Object invariants are preserved.

### Easier Validation

All modifications pass through controlled methods.

### ABI Stability

When implemented using the Pimpl idiom, public headers remain stable while implementation evolves.

### Reduced Compile-Time Dependencies

Changes inside the private implementation often do not require recompiling dependent code.

---

# 13. Disadvantages

### Extra Indirection

Accessing private data requires one extra pointer dereference.

### More Classes

Adds a helper class.

### Slight Memory Overhead

Extra allocation for the private data object.

### When NOT to Use

Avoid this pattern when:

* the class is extremely small,
* no sensitive state exists,
* simplicity is more valuable than the extra abstraction.

---

# 14. Best Practices

* Keep all sensitive state inside the private data object.
* Expose behavior, not mutable data.
* Validate every state-changing operation.
* Prefer immutable configuration where possible.
* In Qt, follow the established Pimpl conventions for long-lived public APIs.

---

# 15. Common Mistakes

### Mistake 1

Adding public setters for every internal field.

This defeats encapsulation.

---

### Mistake 2

Returning mutable references to private data.

Example:

```cpp
BankAccountData&
getData();
```

Now external code can bypass validation.

---

### Mistake 3

Putting business logic inside the private data class.

The private data object should primarily store implementation details and state.

The public class coordinates behavior.

---

### Mistake 4

Using the pattern everywhere.

Reserve it for classes where encapsulation, ABI stability, or protected state truly matter.

---

# 16. Pattern Variations

## 1. Pimpl (Pointer to Implementation)

Most common in C++ libraries.

```text
Widget

↓

WidgetPrivate
```

---

## 2. Immutable Private Data

The private data is created once and never modified.

Useful for configuration objects.

---

## 3. Shared Private Data

Multiple public objects reference the same immutable private data.

Often combined with reference counting.

---

# 17. Related Patterns

| Pattern            | Difference                                  |
| ------------------ | ------------------------------------------- |
| Private Class Data | Protects internal state.                    |
| Facade             | Simplifies a subsystem.                     |
| Proxy              | Controls access to another object.          |
| Flyweight          | Shares immutable state across many objects. |
| Memento            | Captures and restores object state.         |

---

## Private Class Data vs Proxy

### Proxy

Controls access to **another object**.

### Private Class Data

Protects the **internal state of the same object**.

---

# 18. Industry Usage

This pattern is especially common in frameworks and safety-critical systems.

* **Microsoft:** COM libraries and internal implementation hiding.
* **Google:** Large C++ codebases where implementation details are hidden behind stable APIs.
* **Adobe:** Graphics engines with stable public interfaces.
* **Qt:** Extensive use of the **Pimpl (d-pointer)** idiom throughout public APIs.
* **ZEISS / Siemens / Philips:** Medical device software protecting calibration data, device configuration, and internal implementation details.
* **Autodesk:** Geometry kernels and CAD APIs with hidden implementation.
* **Embedded Systems:** Device drivers and hardware abstraction layers.

The architectural goal is **protecting critical state while maintaining a stable, maintainable public interface**.

---

# 19. Interview Questions

## Beginner

1. What problem does the Private Class Data Pattern solve?
2. How is it different from simply making members `private`?
3. Why is encapsulation important?

---

## Intermediate

1. What is the Pimpl idiom?
2. How does this pattern improve binary compatibility?
3. Why should clients interact through behavior instead of state?

---

## Advanced

1. How would you redesign a safety-critical class using Private Class Data?
2. What are the performance costs of the Pimpl idiom?
3. When would you combine Private Class Data with Flyweight or Proxy?

---

## Scenario-Based

Your medical device stores:

* Calibration
* Safety Limits
* Hardware Configuration

These values must never be modified directly by UI code.

Design an architecture using the Private Class Data Pattern.

---

## Architecture

Design a **Private Class Data architecture** for a **Treatment Planning System**.

The public class is:

```text
DoseEngine
```

Private state includes:

* Machine Calibration
* Beam Models
* Dose Tables
* Internal Caches
* Configuration Flags

Explain:

* what belongs in the private class,
* what belongs in the public class,
* how validation works,
* and how the design supports long-term maintenance and binary compatibility.

---

# 20. Practice Exercises

### Beginner Exercise

Design a `BankAccount` using Private Class Data.

Identify:

* Public API
* Private Data
* Validation Rules

---

### Intermediate Exercise

Design a Qt widget using the **Pimpl idiom**.

Store in the private class:

* Rendering State
* Cache
* Internal Timers
* OpenGL Resources

Explain why this improves maintainability.

---

### Advanced Exercise

Design a **Private Class Data architecture** for a **Treatment Planning System**.

Public classes:

* `DoseEngine`
* `BeamManager`
* `PatientManager`

Private classes store:

* Calibration Data
* Internal Lookup Tables
* Optimization Parameters
* Cached Results
* Thread Synchronization Objects

Requirements:

* Strong encapsulation
* Binary compatibility
* Safe updates
* Validation of all state changes
* Modern C++ ownership

**Do not implement the solution yet.** Focus on architecture and responsibilities.

---

# Key Architectural Takeaway

The **Private Class Data Pattern** is about **protecting an object's internal state and implementation details**.

A junior developer thinks:

> "I'll expose setters so other code can modify whatever it needs."

A software architect thinks:

> **"Critical state must remain valid throughout the object's lifetime. I'll hide sensitive data behind a stable public interface and allow changes only through carefully validated operations."**

This mindset is particularly important in **medical devices, banking, embedded systems, Qt libraries, and any long-lived C++ framework** where correctness, maintainability, and API stability matter.

---

