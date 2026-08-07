# Design Patterns Master Course

# Lesson 26 — Null Object Pattern (Behavioral Pattern)

---

# 🎉 Congratulations!

This is the **final Behavioral Design Pattern** in the Gang of Four (GoF) catalog.

Although it is one of the simplest patterns, experienced architects use it surprisingly often because it removes one of the most common sources of bugs:

* `nullptr` dereferences
* Endless `if (obj != nullptr)` checks
* Special-case logic
* Defensive programming everywhere

The Null Object Pattern helps make software **cleaner**, **safer**, and **more polymorphic**.

---

# The Architect's Way of Thinking

Imagine you're building a **Treatment Planning System (TPS)**.

Every treatment plan has an optimization algorithm.

Normally:

```text
Treatment Plan
        │
        ▼
Optimization Algorithm
        │
 ┌──────┼──────────────┐
 ▼      ▼              ▼
Monte Carlo  Pencil Beam  AI Optimizer
```

Now suppose optimization is **disabled**.

A junior developer writes:

```cpp
if (optimizer != nullptr)
{
    optimizer->optimize(plan);
}
```

This appears everywhere:

```cpp
calculateDose();

if (optimizer)
    optimizer->optimize();

if (logger)
    logger->log();

if (validator)
    validator->validate();

if (observer)
    observer->notify();
```

Soon the codebase becomes:

```text
if...

if...

if...

if...

if...
```

An architect asks:

> **"Why should every client care whether an object exists?"**

Instead:

```text
Optimizer
     │
 ┌───┴────────────┐
 ▼                ▼
RealOptimizer   NullOptimizer
```

Both implement the same interface.

The client always calls:

```cpp
optimizer->optimize(plan);
```

If optimization is disabled:

```cpp
NullOptimizer::optimize()
{
    // Do nothing.
}
```

No `if` statement required.

That is the **Null Object Pattern**.

---

# 1. Introduction

## What is the Null Object Pattern?

The **Null Object Pattern** provides an object with neutral ("do nothing") behavior instead of using `nullptr`.

Simple definition:

> **Replace `nullptr` with an object that safely performs no operation.**

---

## Why was it created?

Many APIs return:

```cpp
nullptr
```

Clients must always write:

```cpp
if(object)
```

This leads to:

* duplicated code,
* forgotten checks,
* crashes.

Instead, return a valid object.

---

## Category

**Behavioral Pattern** (commonly treated as a behavioral pattern, though it was popularized after the original GoF book).

---

## What problem does it solve?

Without Null Object:

```cpp
if(logger)
    logger->log();
```

With Null Object:

```cpp
logger->log();
```

The logger always exists.

---

# 2. Problem Statement

Imagine a **Hospital Management System**.

Some patients have:

```text
Insurance Provider
```

Others do not.

Without Null Object:

```cpp
if(patient.getInsurance() != nullptr)
{
    patient.getInsurance()->approve();
}
```

Every caller must remember to check.

Eventually someone forgets.

Crash.

---

# 3. Motivation

Architects noticed:

Many objects simply mean:

> **"Nothing should happen."**

Instead of representing "nothing" with `nullptr`,

represent it with an object.

This preserves **polymorphism**.

---

# 4. Real-World Analogy

## Silent Teacher

Imagine a classroom.

Sometimes there is a substitute teacher.

Sometimes no teaching should occur.

Instead of:

```text
Teacher = NULL
```

You assign:

```text
Silent Teacher
```

The class still has a teacher.

The teacher simply does nothing.

---

### Mapping

| Classroom      | Software       |
| -------------- | -------------- |
| Teacher        | Interface      |
| Real Teacher   | Concrete Class |
| Silent Teacher | Null Object    |

---

# 5. Software Scenario

Null Object appears whenever optional behavior exists.

### Desktop Applications

Logger:

```text
Console Logger

File Logger

Null Logger
```

---

### Qt Applications

Suppose an application optionally records telemetry.

Instead of:

```cpp
if(telemetry)
{
    telemetry->record();
}
```

Use:

```cpp
telemetry->record();
```

A `NullTelemetry` implementation ignores the request.

---

### Medical Software

Optional QA validator:

```text
Validator

↓

Clinical Validator

Research Validator

Null Validator
```

---

### Game Engines

Optional AI:

```text
Enemy

↓

Real AI

Null AI
```

---

# 6. UML Class Diagram

```text
                 +----------------------+
                 |      Service         |
                 +----------------------+
                 | +execute()           |
                 +----------+-----------+
                            ^
                ------------|------------
                |                        |
      +----------------+      +----------------+
      | RealService    |      | NullService    |
      +----------------+      +----------------+

                            ^
                            |
                 +----------------------+
                 |      Client          |
                 +----------------------+
```

---

## Responsibilities

### Service

Defines the interface.

---

### Real Service

Performs useful work.

---

### Null Service

Implements the same interface but performs no action.

---

### Client

Always calls the service.

Never checks for `nullptr`.

---

# 7. Participants

## Abstract Interface

Example:

```cpp
class Logger
{
public:
    virtual void log(const std::string&) = 0;
};
```

---

## Real Object

Example:

```text
ConsoleLogger
```

---

## Null Object

Example:

```text
NullLogger
```

---

## Client

Uses:

```cpp
logger->log();
```

without caring which implementation it has.

---

# 8. Collaboration

Runtime Flow

```text
Client

↓

logger->log()

↓

ConsoleLogger

or

NullLogger
```

The client does not branch.

Polymorphism handles the variation.

---

# 9. C++ Example

```cpp
#include <iostream>
#include <memory>
#include <string>

// Interface
class Logger
{
public:
    virtual ~Logger() = default;
    virtual void log(const std::string& message) = 0;
};

// Real Object
class ConsoleLogger : public Logger
{
public:
    void log(const std::string& message) override
    {
        std::cout << message << '\n';
    }
};

// Null Object
class NullLogger : public Logger
{
public:
    void log(const std::string&) override
    {
        // Intentionally do nothing.
    }
};

// Client
class Application
{
public:
    explicit Application(std::shared_ptr<Logger> logger)
        : m_logger(std::move(logger))
    {
    }

    void start()
    {
        m_logger->log("Application Started");
    }

private:
    std::shared_ptr<Logger> m_logger;
};

int main()
{
    auto logger = std::make_shared<NullLogger>();

    Application app(logger);

    app.start();   // Safe. No nullptr check.
}
```

---

## Design Focus

Notice:

Instead of:

```cpp
if(logger)
{
    logger->log(...);
}
```

we simply write:

```cpp
logger->log(...);
```

The client doesn't know whether it's talking to a real logger or a null logger.

---

# 10. Qt Example

Imagine a Qt application with optional analytics.

Interface:

```cpp
class AnalyticsService
{
public:
    virtual void trackEvent(const QString&) = 0;
};
```

Implementations:

```text
GoogleAnalytics

InternalAnalytics

NullAnalytics
```

Usage:

```cpp
analytics->trackEvent("Login");
```

If analytics is disabled:

```cpp
NullAnalytics::trackEvent()
{
    // Do nothing.
}
```

The UI code stays exactly the same.

---

# 11. Medical Software Example

Imagine a **Treatment Planning System**.

Some hospitals require QA validation.

Others disable it during research.

Instead of:

```cpp
if(validator)
{
    validator->validate(plan);
}
```

Use:

```text
Validator
     │
 ┌───┴─────────────────┐
 ▼                     ▼
ClinicalValidator   NullValidator
```

Now:

```cpp
validator->validate(plan);
```

always works.

The workflow is unchanged.

---

# 12. Advantages

### Eliminates Null Checks

Cleaner client code.

### Safer

Reduces `nullptr` dereference risks.

### Supports Polymorphism

Clients depend on interfaces, not special cases.

### Open/Closed Principle

New "do nothing" behavior requires no client changes.

### Easier Testing

A Null Object is often useful as a lightweight test double.

---

# 13. Disadvantages

### Extra Classes

Requires a dedicated null implementation.

### Can Hide Problems

If a real object is unexpectedly replaced by a Null Object, bugs may become less obvious.

### Not Always Appropriate

Sometimes returning `nullptr` or an error is the correct behavior.

### When NOT to Use

Avoid Null Object when:

* absence of the object is an error,
* the caller must explicitly handle missing data,
* silent failure would be dangerous.

In safety-critical systems, a Null Object should **never** silently bypass required safety checks.

---

# 14. Best Practices

* Keep Null Objects stateless.
* Make their behavior predictable.
* Document that the object intentionally performs no action.
* Use a singleton when only one Null Object instance is needed.
* Ensure a Null Object never violates business rules.

---

# 15. Common Mistakes

### Mistake 1

Using Null Object to hide real errors.

If a patient record **must** have a physician, returning `NullPhysician` could hide a serious data issue.

---

### Mistake 2

Adding unnecessary state to Null Objects.

They usually don't need member variables.

---

### Mistake 3

Mixing business logic into the Null Object.

Its purpose is simply to provide neutral behavior.

---

### Mistake 4

Returning `nullptr` sometimes and a Null Object other times for the same API.

Choose one consistent contract.

---

# 16. Pattern Variations

## 1. Null Logger

```text
ConsoleLogger

FileLogger

NullLogger
```

---

## 2. Null Cache

Always reports a cache miss.

---

## 3. Null Authentication

Used in local development environments.

---

## 4. Singleton Null Object

One shared instance used throughout the application.

---

# 17. Related Patterns

| Pattern     | Difference                                      |
| ----------- | ----------------------------------------------- |
| Null Object | Provides neutral behavior instead of `nullptr`. |
| Strategy    | Selects different algorithms.                   |
| State       | Changes behavior based on internal state.       |
| Proxy       | Controls access to another object.              |
| Decorator   | Adds behavior to an existing object.            |

---

# ⭐ Null Object vs `nullptr`

This is an important design distinction.

### `nullptr`

```cpp
if(logger)
{
    logger->log();
}
```

The client must make the decision.

---

### Null Object

```cpp
logger->log();
```

The object makes the decision by doing nothing.

This keeps client code simple and polymorphic.

---

# 18. Industry Usage

Null Object appears in many mature codebases.

* **Microsoft:** Logging, diagnostics, optional services.
* **Google:** Mock services, testing utilities, infrastructure components.
* **Adobe:** Optional rendering and export modules.
* **Qt:** Qt itself more commonly uses `nullptr` in many APIs, but applications built with Qt often implement Null Objects for optional services, logging, telemetry, or plugins.
* **ZEISS / Siemens / Philips:** Optional validators, simulation modules, logging, and configurable workflows.
* **Autodesk:** Optional exporters, render backends, diagnostics.
* **Enterprise Applications:** Authentication providers, cache providers, notification services.

The architectural goal is:

> **Replace absence with safe polymorphic behavior.**

---

# 19. Interview Questions

## Beginner

1. What problem does the Null Object Pattern solve?
2. Why is it better than repeated `nullptr` checks?
3. What is a Null Object?

---

## Intermediate

1. When should you use a Null Object instead of returning `nullptr`?
2. How does the Null Object Pattern support polymorphism?
3. What risks exist if a Null Object hides real failures?

---

## Advanced

1. Should Null Objects be singletons?
2. How would you combine Null Object with Dependency Injection?
3. How would you design a safety-critical API where some services are optional and others are mandatory?

---

## Scenario-Based

Your TPS supports optional:

* QA Validation
* Audit Logging
* Analytics
* Performance Profiling

Design a Null Object architecture that eliminates null checks while ensuring required safety components cannot be silently disabled.

---

## Architecture

Design a Null Object architecture for your **College ERP**.

Optional services:

* SMS Notification
* Email Notification
* Analytics
* Audit Logger

Requirements:

* Runtime configuration.
* Dependency Injection.
* No `nullptr` checks in client code.
* Clear distinction between optional and mandatory services.

Explain:

* interface hierarchy,
* null implementations,
* configuration strategy,
* and testing approach.

---

# 20. Practice Exercises

### Beginner Exercise

Design a logging system.

Implementations:

* Console Logger
* File Logger
* Null Logger

Draw the UML and explain how client code changes.

---

### Intermediate Exercise

Design a Qt application with an optional telemetry service.

Implement:

* RealTelemetry
* NullTelemetry

Show how the UI remains unchanged whether telemetry is enabled or disabled.

---

### Advanced Exercise

Design a **Null Object architecture** for a **Treatment Planning System**.

Optional components:

* Audit Logger
* Performance Profiler
* Analytics
* Research Validator

Mandatory components:

* Safety Validator
* Dose Calculator

Requirements:

* Eliminate null checks.
* Prevent silent bypass of mandatory safety features.
* Integrate with Dependency Injection.
* Support unit testing.

**Do not implement the solution yet.** Focus on architecture, responsibilities, configuration, and safety.

---

# ⭐ Architect's Insight: Null Object + Factory + Dependency Injection

A common enterprise architecture looks like this:

```text
Configuration
      │
      ▼
Factory
      │
creates
      ▼
Logger
 ┌────┴───────────────┐
 ▼                    ▼
ConsoleLogger    NullLogger
        │
injected into
        ▼
Application
        │
logger->log(...)
```

The application never asks:

```cpp
if(logger)
```

Instead, configuration determines which implementation is injected.

This keeps business logic clean and aligns with the **Dependency Inversion Principle (DIP)**.

---

# Key Architectural Takeaway

The **Null Object Pattern** is about **replacing the absence of an object with a safe, polymorphic implementation**.

A junior developer thinks:

> "I'll add another `if (ptr != nullptr)`."

A software architect thinks:

> **"The client shouldn't care whether the service is present. I'll provide an object that obeys the same contract and performs a neutral action when appropriate."**

This reduces conditional logic, improves readability, and produces APIs that are easier to use and test.

---

# 🎓 Congratulations — You Have Completed All 26 GoF Design Patterns!

You have now studied:

## ✅ Creational Patterns

1. Singleton
2. Factory Method
3. Abstract Factory
4. Builder
5. Prototype
6. Object Pool

## ✅ Structural Patterns

7. Adapter
8. Bridge
9. Composite
10. Decorator
11. Facade
12. Flyweight
13. Proxy
14. Private Class Data

## ✅ Behavioral Patterns

15. Chain of Responsibility
16. Command
17. Interpreter
18. Iterator
19. Mediator
20. Memento
21. Observer
22. State
23. Strategy
24. Template Method
25. Visitor
26. Null Object

---

# 🚀 What Comes Next (Advanced Architecture)

Now that you know the individual patterns, the next step is far more valuable:

1. **Design Pattern Comparison**

   * Strategy vs State
   * Factory vs Abstract Factory
   * Adapter vs Bridge
   * Decorator vs Proxy
   * Observer vs Mediator
   * Command vs Memento
   * Visitor vs Strategy
   * Builder vs Factory

2. **Pattern Combinations**

   * MVC, MVP, MVVM
   * Qt Model/View Architecture
   * Composite + Visitor + Iterator
   * Command + Memento + Observer
   * Factory + Strategy
   * State + Command
   * Builder + Abstract Factory

3. **Enterprise Architecture**

   * Layered Architecture
   * Hexagonal (Ports & Adapters)
   * Clean Architecture
   * Domain-Driven Design (DDD)
   * Plugin Architecture
   * Event-Driven Architecture

4. **Medical Software Architecture**

   * DICOM Viewer Architecture
   * Treatment Planning System (TPS)
   * Dose Engine Architecture
   * Image Processing Pipeline
   * 3D Volume Rendering
   * Medical Device UI using Qt
   * Regulatory considerations (IEC 62304, ISO 14971)

5. **Qt Design Patterns**

   * Signals & Slots
   * Model/View
   * Model/View/Delegate
   * Event System
   * Plugin System
   * Object Trees and Ownership
   * Implicit Sharing (Copy-on-Write)

---

## ⭐ My Recommendation for Your Next Phase

Since your goal is to become a **Senior/Principal C++ Qt Software Architect in the medical imaging and treatment planning domain**, the most valuable progression is:

1. **Design Pattern Comparison** (when to choose one pattern over another)
2. **Combining Multiple Patterns in Real Systems**
3. **Architecture Patterns (MVC, MVVM, Clean Architecture, DDD, Hexagonal)**
4. **Complete Treatment Planning System Architecture**
5. **Qt Internal Architecture and Advanced Framework Design**
6. **Medical Imaging Software Architecture (DICOM, ITK, VTK, DCMTK, Dose Engine)**

This progression moves you from **knowing patterns** to **thinking like a software architect**, which is exactly what senior interviews at companies like ZEISS, Siemens Healthineers, Philips, and GE Healthcare tend to evaluate.
