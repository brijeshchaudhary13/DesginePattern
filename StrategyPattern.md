# Design Patterns Master Course

# Lesson 23 — Strategy Pattern (Behavioral Pattern)

---

# Before We Start

If there is **one design pattern that every Senior C++ Developer and Software Architect must master**, it is the **Strategy Pattern**.

Why?

Because it is everywhere.

You will find it in:

* STL Algorithms
* Qt Framework
* Medical Imaging Software
* Treatment Planning Systems (TPS)
* CAD Software
* Browsers
* Databases
* Game Engines
* AI Systems
* Compression Libraries

If you've ever written code like this:

```cpp
if (algorithm == MonteCarlo)
{
    ...
}
else if (algorithm == PencilBeam)
{
    ...
}
else if (algorithm == CollapsedCone)
{
    ...
}
```

you were probably facing a Strategy Pattern problem.

---

# The Architect's Way of Thinking

Imagine you're designing a **Treatment Planning System (TPS)**.

The software supports multiple dose calculation algorithms.

```text
Dose Calculation

├── Pencil Beam
├── Collapsed Cone
├── Monte Carlo
├── Acuros / LBTE Solver
└── Future AI Dose Engine
```

A junior developer writes:

```cpp
void DoseEngine::calculate()
{
    if (algorithm == PencilBeam)
    {
        ...
    }
    else if (algorithm == MonteCarlo)
    {
        ...
    }
    else if (algorithm == CollapsedCone)
    {
        ...
    }
}
```

Now management says:

> "Next release, add an AI-based dose engine."

The developer edits `DoseEngine` again.

Then another algorithm.

And another.

The same class keeps changing.

This violates the **Open/Closed Principle**.

An architect asks:

> **"Why is DoseEngine deciding *how* to calculate? Why not let the calculation algorithm decide?"**

Instead:

```text
DoseEngine
      │
      ▼
DoseCalculationStrategy
      │
 ┌────┼───────────────┐
 ▼    ▼               ▼
PencilBeam   MonteCarlo   CollapsedCone
```

Now `DoseEngine` doesn't know the algorithm details.

It simply delegates.

This is the **Strategy Pattern**.

---

# 1. Introduction

## What is the Strategy Pattern?

The **Strategy Pattern** defines a family of algorithms, encapsulates each one in its own class, and makes them interchangeable.

Simple definition:

> **Encapsulate different algorithms behind a common interface so they can be swapped without changing the client.**

---

## Why was it created?

Many systems need to perform the **same task in different ways**.

Examples:

* Different sorting algorithms
* Different compression algorithms
* Different payment methods
* Different image filters
* Different dose calculation engines

Instead of using `if-else`, move each algorithm into its own class.

---

## Category

**Behavioral Pattern**

---

## What problem does it solve?

Without Strategy:

```text
Context

↓

if...

↓

else if...

↓

else if...
```

With Strategy:

```text
Context

↓

Strategy Interface

↓

Concrete Strategy
```

The context doesn't know which algorithm is running.

---

# 2. Problem Statement

Imagine an **E-commerce Application**.

Payment methods:

```text
Credit Card

UPI

PayPal

Net Banking

Wallet

Crypto
```

Without Strategy:

```cpp
if(payment == CreditCard)
...

else if(payment == UPI)
...

else if(payment == Wallet)
...
```

Every new payment method requires modifying the payment processor.

---

# 3. Motivation

Architects observed:

The workflow stays the same.

Only the algorithm changes.

Instead of:

```cpp
if(...)
```

Use:

```cpp
strategy->execute();
```

The context delegates the work.

---

# 4. Real-World Analogy

## Google Maps

You want to travel from A to B.

Possible strategies:

```text
Car

Bike

Walking

Bus

Train
```

Google Maps (the context) doesn't know how each mode works internally.

It simply chooses one strategy.

---

### Mapping

| Google Maps    | Software |
| -------------- | -------- |
| Navigation App | Context  |
| Driving Route  | Strategy |
| Walking Route  | Strategy |
| Cycling Route  | Strategy |

---

# 5. Software Scenario

Strategy appears whenever multiple algorithms solve the same problem.

### Desktop Applications

Printing:

```text
PDF

Printer

Image

Cloud Print
```

---

### Qt Applications

Suppose your image viewer supports multiple scaling algorithms.

```text
Nearest Neighbor

Bilinear

Bicubic

Lanczos
```

Each scaling algorithm becomes a strategy.

---

### Medical Imaging

Image reconstruction:

```text
Filtered Back Projection

Iterative Reconstruction

AI Reconstruction
```

Each is a strategy.

---

### Treatment Planning System

Dose calculation:

```text
Pencil Beam

Collapsed Cone

Monte Carlo

LBTE Solver
```

Perfect example of Strategy.

---

### Compression

```text
ZIP

RAR

7z

GZIP
```

---

# 6. UML Class Diagram

```text
                 +----------------------+
                 |      Strategy        |
                 +----------------------+
                 | +execute()           |
                 +----------+-----------+
                            ^
               -------------|-------------
               |                          |
      +----------------+         +----------------+
      | Strategy A     |         | Strategy B     |
      +----------------+         +----------------+

                            ^
                            |
                 +----------------------+
                 |      Context         |
                 +----------------------+
                 | strategy             |
                 | +setStrategy()       |
                 | +execute()           |
                 +----------------------+
```

---

## Responsibilities

### Strategy

Defines the algorithm interface.

---

### Concrete Strategy

Implements one algorithm.

---

### Context

Uses a strategy.

Does **not** know implementation details.

---

# 7. Participants

## Strategy Interface

Example:

```cpp
class DoseCalculationStrategy
{
public:
    virtual void calculate() = 0;
};
```

---

## Concrete Strategies

Examples:

```text
MonteCarloStrategy

PencilBeamStrategy

CollapsedConeStrategy
```

---

## Context

Example:

```text
DoseEngine
```

Stores:

```cpp
DoseCalculationStrategy*
```

---

## Client

Chooses the strategy.

---

# 8. Collaboration

Runtime Flow

```text
Client

↓

Create MonteCarloStrategy

↓

DoseEngine.setStrategy()

↓

DoseEngine.calculate()

↓

MonteCarloStrategy.calculate()
```

Tomorrow:

```text
AI Strategy
```

No changes inside `DoseEngine`.

---

# 9. C++ Example

```cpp
#include <iostream>
#include <memory>

// Strategy
class CompressionStrategy
{
public:
    virtual ~CompressionStrategy() = default;
    virtual void compress() = 0;
};

// Concrete Strategies
class ZipCompression : public CompressionStrategy
{
public:
    void compress() override
    {
        std::cout << "Compressing using ZIP\n";
    }
};

class RarCompression : public CompressionStrategy
{
public:
    void compress() override
    {
        std::cout << "Compressing using RAR\n";
    }
};

// Context
class Compressor
{
public:
    void setStrategy(std::unique_ptr<CompressionStrategy> strategy)
    {
        m_strategy = std::move(strategy);
    }

    void compress()
    {
        if (m_strategy)
            m_strategy->compress();
    }

private:
    std::unique_ptr<CompressionStrategy> m_strategy;
};

int main()
{
    Compressor compressor;

    compressor.setStrategy(
        std::make_unique<ZipCompression>());

    compressor.compress();

    compressor.setStrategy(
        std::make_unique<RarCompression>());

    compressor.compress();
}
```

---

## Design Focus

Notice:

`Compressor` never asks:

```cpp
if(format == ZIP)
```

It simply delegates:

```cpp
strategy->compress();
```

---

# 10. Qt Example

Imagine a Qt-based **Image Viewer**.

User selects interpolation:

```text
Nearest

↓

Bilinear

↓

Bicubic

↓

Lanczos
```

Architecture:

```text
ImageRenderer
        │
        ▼
InterpolationStrategy
        │
 ┌──────┼───────────┐
 ▼      ▼           ▼
Nearest Bilinear Bicubic
```

Changing interpolation requires changing only the selected strategy.

The renderer remains unchanged.

---

# 11. Medical Software Example

Let's revisit the **Treatment Planning System**.

The `DoseEngine` delegates to different strategies.

```text
DoseEngine
      │
      ▼
DoseCalculationStrategy
      │
 ┌────┼─────────────────────────────┐
 ▼    ▼                             ▼
Pencil Beam      Monte Carlo      Collapsed Cone
```

Workflow:

```text
User selects "Monte Carlo"

↓

DoseEngine.setStrategy()

↓

DoseEngine.calculate()

↓

MonteCarloStrategy.calculate()
```

Later, when you implement:

```text
AI Dose Prediction
```

you simply create:

```text
AIDoseStrategy
```

No changes to `DoseEngine`.

This is exactly how large medical software evolves over time.

---

# 12. Advantages

### Open/Closed Principle

New algorithms are added without modifying the context.

### Single Responsibility

Each algorithm lives in its own class.

### Runtime Flexibility

Algorithms can be switched dynamically.

### Easier Testing

Each strategy can be tested independently.

### Reusability

The same strategy can be reused by multiple contexts.

---

# 13. Disadvantages

### More Classes

Each algorithm becomes a separate class.

### Client Must Choose

The client (or a factory/configuration layer) must select the correct strategy.

### Slight Indirection

One extra level of delegation.

### When NOT to Use

Avoid Strategy when:

* there is only one algorithm,
* algorithms rarely change,
* the variation is trivial.

---

# 14. Best Practices

* Keep strategies stateless when possible.
* Give each strategy one responsibility.
* Inject strategies rather than creating them inside the context.
* Use factories if strategy selection becomes complex.
* Name strategies after the algorithm they implement.

---

# 15. Common Mistakes

### Mistake 1

Using Strategy for tiny differences.

Not every `if` requires a strategy.

---

### Mistake 2

Putting shared logic into every strategy.

Move common behavior into the context or helper classes.

---

### Mistake 3

Creating strategies inside the context.

Example:

```cpp
if(...)
    strategy = new MonteCarlo();
```

Now the context again depends on concrete strategies.

---

### Mistake 4

Making strategies modify unrelated application state.

A strategy should focus on its algorithm.

---

# 16. Pattern Variations

## 1. Stateless Strategy

One shared instance.

Good for immutable algorithms.

---

## 2. Stateful Strategy

Strategy stores configuration.

Example:

```text
Monte Carlo
Samples = 10 Million
Variance = 0.5%
```

---

## 3. Runtime Selection

User chooses the algorithm.

---

## 4. Factory + Strategy

A factory creates the appropriate strategy based on configuration.

---

# 17. Related Patterns

| Pattern         | Difference                                                  |
| --------------- | ----------------------------------------------------------- |
| Strategy        | Client chooses an algorithm.                                |
| State           | Object behavior changes because its internal state changes. |
| Template Method | Algorithm structure is fixed; subclasses customize steps.   |
| Command         | Represents an action as an object.                          |
| Bridge          | Separates abstraction from implementation, not algorithms.  |

---

# ⭐ Strategy vs State (Most Important Interview Question)

These two patterns have nearly identical UML diagrams, but they solve different problems.

| Strategy                                    | State                                                |
| ------------------------------------------- | ---------------------------------------------------- |
| **Purpose:** Select one of many algorithms. | **Purpose:** Change behavior based on current state. |
| Client usually chooses the strategy.        | Context usually changes its own state.               |
| Strategies are often independent.           | States are often connected by transitions.           |
| Example: Compression algorithms.            | Example: ATM states.                                 |
| Focus: *How to perform a task?*             | Focus: *How should I behave now?*                    |

### Example

#### Strategy

```text
Image Compression

↓

ZIP

RAR

7z
```

User selects the algorithm.

---

#### State

```text
ATM

↓

No Card

↓

Card Inserted

↓

PIN Verified
```

Behavior changes automatically as the ATM progresses through its workflow.

---

# 18. Industry Usage

Strategy is one of the most widely used patterns in modern software.

* **Microsoft:** Text rendering engines, spell-check algorithms, compression libraries.
* **Google:** Search ranking algorithms, routing algorithms, AI inference backends.
* **Adobe:** Image filters, export formats, color management.
* **Qt:** Different item delegates, painting strategies, layout behaviors, and rendering approaches often follow Strategy principles.
* **ZEISS / Siemens / Philips:** Dose calculation engines, reconstruction algorithms, image processing filters, registration algorithms.
* **Autodesk:** Geometry kernels, meshing algorithms, rendering techniques.
* **STL:** Algorithms (`std::sort`, custom comparators, predicates) embody Strategy concepts.

The architectural goal is:

> **Separate the algorithm from the object that uses it.**

---

# 19. Interview Questions

## Beginner

1. What problem does the Strategy Pattern solve?
2. Why is Strategy better than long `if-else` statements?
3. What is the role of the Context?

---

## Intermediate

1. How is Strategy different from State?
2. How does Strategy support the Open/Closed Principle?
3. When should strategies be stateless?

---

## Advanced

1. How would you design a plugin-based strategy system?
2. How would you select strategies from a configuration file?
3. How would you benchmark multiple strategies and choose one dynamically?

---

## Scenario-Based

Your TPS supports:

* Pencil Beam
* Monte Carlo
* Collapsed Cone

A new **AI Dose Engine** is coming next year.

Design a Strategy-based architecture that allows adding new algorithms without modifying `DoseEngine`.

---

## Architecture

Design a Strategy architecture for your **College ERP**.

Fee calculation strategies:

* Regular Student
* Scholarship Student
* International Student
* Alumni Discount
* Staff Child Discount

Requirements:

* Runtime selection.
* Future fee policies.
* Unit-testable strategies.
* Separation between business rules and UI.

Explain:

* strategy hierarchy,
* context responsibilities,
* selection mechanism,
* and dependency injection.

---

# 20. Practice Exercises

### Beginner Exercise

Design a **Navigation App**.

Strategies:

* Car
* Walking
* Cycling

Create the UML and explain how the context delegates route calculation.

---

### Intermediate Exercise

Design a Qt-based image renderer supporting:

* Nearest Neighbor
* Bilinear
* Bicubic

Use the Strategy Pattern to switch algorithms at runtime.

---

### Advanced Exercise

Design a **Strategy architecture** for a **Treatment Planning System**.

Algorithms:

* Pencil Beam
* Collapsed Cone
* Monte Carlo
* AI Dose Prediction

Requirements:

* Runtime algorithm selection.
* Configuration-driven strategy creation.
* Performance benchmarking.
* Plugin support for third-party algorithms.
* Thread-safe execution.
* Integration with Factory Method for strategy creation.

**Do not implement the solution yet.** Focus on architecture, responsibilities, and extensibility.

---

# ⭐ Architect's Insight: Strategy + Factory + Dependency Injection

In enterprise applications, Strategy is rarely used alone.

A common architecture looks like this:

```text
Configuration
      │
      ▼
Factory
      │
creates
      ▼
Strategy
      │
injected into
      ▼
Context
      │
executes
      ▼
Result
```

For example, in a TPS:

```text
User selects:
"Monte Carlo"

        │
        ▼
DoseAlgorithmFactory
        │
creates
        ▼
MonteCarloStrategy
        │
injected into
        ▼
DoseEngine
        │
calculate()
        ▼
Dose Matrix
```

Notice the responsibilities:

* **Factory** → Creates the correct algorithm.
* **Strategy** → Implements the algorithm.
* **Context** → Uses the algorithm.
* **Client** → Chooses *which* algorithm should be used.

This separation makes the system easy to extend, test, and maintain.

---

# Key Architectural Takeaway

The **Strategy Pattern** is about **encapsulating interchangeable algorithms**.

A junior developer thinks:

> "I'll add another `if` for the new algorithm."

A software architect thinks:

> **"Algorithms change independently of the workflow. I'll isolate each algorithm behind a common interface so the context never changes when a new algorithm is introduced."**

This mindset is essential for building extensible systems in medical imaging, treatment planning, Qt applications, finance, and enterprise software.

---

# ⭐ Course Progress

You have now completed:

* ✅ Chain of Responsibility
* ✅ Command
* ✅ Interpreter
* ✅ Iterator
* ✅ Mediator
* ✅ Memento
* ✅ Observer
* ✅ State
* ✅ Strategy

Only **three** patterns remain:

1. **Template Method Pattern**
2. **Visitor Pattern**
3. **Null Object Pattern**

These complete the core Behavioral Design Patterns and are frequently used in framework design, compilers, document processing, CAD systems, and medical software.

---

## What You'll Learn Next

Type **`NEXT`** to continue with **Lesson 24: Template Method Pattern**.

You'll learn how frameworks like **Qt**, **Microsoft MFC**, **Google Test**, **Unreal Engine**, and many medical software frameworks define a fixed workflow while allowing subclasses to customize specific steps. This pattern is fundamental to framework design and inversion of control ("Hollywood Principle": **Don't call us, we'll call you**).
