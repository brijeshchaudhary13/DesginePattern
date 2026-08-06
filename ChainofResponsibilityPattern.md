# Design Patterns Master Course

# Phase 3 — Behavioral Design Patterns

Congratulations.

You have now learned:

* ✅ Creational Patterns → **How objects are created**
* ✅ Structural Patterns → **How objects are organized**

Now we enter the most important category for architects.

---

# Why Behavioral Patterns Matter

When you start working on large applications like:

* Treatment Planning System (TPS)
* PACS Viewer
* Qt Creator
* Visual Studio
* Photoshop
* CAD Software
* ERP Systems
* Browsers

you quickly discover that **creating objects is not the difficult part**.

The real challenge is:

> **How should objects communicate without becoming tightly coupled?**

This is exactly what Behavioral Patterns solve.

---

# The Architect's Way of Thinking

Imagine a **College ERP**.

A student submits an application.

Should the student object directly call:

```text
Student

↓

Admission Office

↓

Accounts

↓

Library

↓

Hostel

↓

Transport

↓

ID Card

↓

Email
```

Now imagine adding:

* Scholarship Department
* Medical Department
* Alumni Office

Every time a new department is added, the `Student` class changes.

This violates the **Open/Closed Principle**.

An architect asks:

> **"Can I design the communication so new processing steps can be added without changing existing code?"**

The first Behavioral Pattern answers exactly that question.

---

# Lesson 15 — Chain of Responsibility Pattern

---

# Before We Start

One of the biggest mistakes beginners make is writing long `if-else` or `switch` chains.

Example:

```cpp
if(user == Student)
{
    ...
}
else if(user == Teacher)
{
    ...
}
else if(user == Principal)
{
    ...
}
else if(user == Admin)
{
    ...
}
else if(user == SuperAdmin)
{
    ...
}
```

Looks okay today.

Tomorrow management says:

Add:

* Parent
* Accountant
* Librarian
* HR
* Transport Manager
* Hostel Warden
* Placement Officer

Now your code becomes:

```text
if...

else if...

else if...

else if...

else if...

else if...

else if...

...
```

Hundreds of lines.

An architect immediately thinks:

> **"Every handler should decide whether it can process the request. If not, pass it to the next handler."**

That is the **Chain of Responsibility Pattern (CoR)**.

---

# 1. Introduction

## What is the Chain of Responsibility Pattern?

The **Chain of Responsibility (CoR)** Pattern lets you pass a request through a chain of handlers.

Each handler decides:

1. Can I handle this request?
2. If yes → process it.
3. If no → forward it to the next handler.

Simple definition:

> **A request travels through a chain until one handler processes it or the chain ends.**

---

## Why was it created?

Without CoR:

One object contains every decision.

With CoR:

Each handler has only one responsibility.

This follows the **Single Responsibility Principle (SRP)**.

---

## Category

**Behavioral Pattern**

---

## What problem does it solve?

Instead of:

```text
Client

↓

if

↓

else if

↓

else if

↓

else if
```

Use:

```text
Client

↓

Handler A

↓

Handler B

↓

Handler C

↓

Handler D
```

Each handler is independent.

---

# 2. Problem Statement

Imagine a **Hospital Approval Workflow**.

A medicine purchase request arrives.

Rules:

```text
Amount < ₹5,000

↓

Department Head
```

```text
₹5,000–₹50,000

↓

Hospital Manager
```

```text
Above ₹50,000

↓

Director
```

Without CoR:

```cpp
if(amount < 5000)
...

else if(amount < 50000)
...

else
...
```

Tomorrow new approval levels are added.

You keep modifying the same code.

The class grows continuously.

---

# 3. Motivation

Architects noticed that many systems process requests in stages.

Examples:

* HTTP middleware
* Event processing
* Approval workflows
* Exception handling
* Authentication
* Logging
* Input validation

Each stage should perform one task.

If it cannot finish the request, it forwards it.

---

# 4. Real-World Analogy

## Customer Support

You call customer care.

Level 1:

```text
Customer Support
```

Cannot solve.

↓

Transfers to:

```text
Technical Support
```

Still cannot solve.

↓

Transfers to:

```text
Senior Engineer
```

Finally solved.

Notice:

You don't know who will solve it.

You only submit the request.

---

## Mapping

| Real World        | Software             |
| ----------------- | -------------------- |
| Customer          | Client               |
| Support Staff     | Handler              |
| Technical Support | Concrete Handler     |
| Escalation        | Pass to Next Handler |

---

# 5. Software Scenario

Chain of Responsibility is useful whenever requests pass through multiple processing stages.

### Desktop Applications

* Keyboard event handling
* Mouse event propagation
* Validation pipelines

---

### Qt Applications

Qt itself uses a similar concept for **event propagation**.

Example:

```text
Mouse Click

↓

Child Widget

↓

Parent Widget

↓

Main Window

↓

Application
```

If a widget doesn't handle an event, it can propagate upward.

While Qt's event system is not a textbook GoF CoR implementation in every detail, it follows the same architectural principle.

---

### CAD Software

Mouse selection:

```text
Entity

↓

Layer

↓

Scene

↓

Viewport
```

---

### Medical Imaging

Image loading:

```text
Validate File

↓

Load Metadata

↓

Load Pixel Data

↓

Apply Window Level

↓

Render
```

Each stage performs one task.

---

### Web Servers

HTTP Request:

```text
Authentication

↓

Authorization

↓

Logging

↓

Controller

↓

Response
```

ASP.NET, Spring, Express.js, and many other frameworks use middleware pipelines based on this idea.

---

# 6. UML Class Diagram

```text
                 +----------------------+
                 |      Handler         |
                 +----------------------+
                 | +setNext()           |
                 | +handle(request)     |
                 +----------+-----------+
                            ^
                ------------|------------
                |                        |
     +------------------+      +------------------+
     | ConcreteHandlerA |      | ConcreteHandlerB |
     +------------------+      +------------------+
                |
                | next
                v
     +------------------+
     | ConcreteHandlerC |
     +------------------+

                ^
                |
             Client
```

---

## Responsibilities

### Handler

Defines:

```cpp
handle(request)
```

Stores reference to next handler.

---

### Concrete Handler

Processes the request.

If it cannot process:

```cpp
next->handle(request);
```

---

### Client

Starts the chain.

Knows only the first handler.

---

# 7. Participants

## Handler

Abstract interface.

Example:

```cpp
ApprovalHandler
```

---

## Concrete Handlers

Examples:

```text
DepartmentHeadHandler

HospitalManagerHandler

DirectorHandler
```

Each knows only:

* its own responsibility,
* the next handler.

---

## Client

Creates the chain.

Submits the request.

---

# 8. Collaboration

Runtime Flow

```text
Purchase Request

↓

Department Head

↓

Can Approve?

↓

No

↓

Manager

↓

Can Approve?

↓

No

↓

Director

↓

Approved
```

The request moves through the chain until handled.

---

# 9. C++ Example

```cpp
#include <iostream>
#include <memory>

// Request
struct PurchaseRequest
{
    int amount;
};

// Handler
class Approver
{
public:
    virtual ~Approver() = default;

    void setNext(std::shared_ptr<Approver> next)
    {
        m_next = std::move(next);
    }

    virtual void approve(const PurchaseRequest& request)
    {
        if (m_next)
            m_next->approve(request);
    }

protected:
    std::shared_ptr<Approver> m_next;
};

// Concrete Handlers
class Manager : public Approver
{
public:
    void approve(const PurchaseRequest& request) override
    {
        if (request.amount <= 5000)
        {
            std::cout << "Approved by Manager\n";
        }
        else
        {
            Approver::approve(request);
        }
    }
};

class Director : public Approver
{
public:
    void approve(const PurchaseRequest& request) override
    {
        if (request.amount <= 50000)
        {
            std::cout << "Approved by Director\n";
        }
        else
        {
            std::cout << "CEO Approval Required\n";
        }
    }
};

int main()
{
    auto manager = std::make_shared<Manager>();
    auto director = std::make_shared<Director>();

    manager->setNext(director);

    manager->approve({20000});
}
```

---

## Design Focus

Notice:

The client only knows:

```cpp
manager->approve(request);
```

It does **not** know who eventually handles the request.

The chain determines that dynamically.

---

# 10. Qt Example

Imagine your College ERP desktop application.

A **Login Request** passes through:

```text
Login Request

↓

Input Validation Handler

↓

Database Authentication Handler

↓

Role Loading Handler

↓

Permission Handler

↓

Dashboard Loader
```

Each handler performs one responsibility.

If validation fails:

The chain stops.

Otherwise, it continues.

This design is easier to extend than a single giant `login()` function.

---

# 11. Medical Software Example

Let's design a **Treatment Planning System**.

Before calculating dose:

```text
Calculate Dose

↓

Patient Validation

↓

CT Validation

↓

Structure Validation

↓

Beam Validation

↓

Machine Validation

↓

Dose Calculation

↓

DVH Calculation

↓

Report Generation
```

Each step is a handler.

Suppose a patient's CT is missing.

The chain stops immediately.

No dose calculation is attempted.

Benefits:

* Each validation is isolated.
* New validation steps can be inserted without changing existing handlers.
* The workflow is easy to understand and test.

This is a common architectural style in safety-critical software.

---

# 12. Advantages

### Loose Coupling

The sender does not know which object handles the request.

### Open/Closed Principle

New handlers can be added without modifying existing ones.

### Single Responsibility

Each handler performs one task.

### Flexible Ordering

Handlers can often be reordered to change behavior.

### Reusability

Handlers can participate in different chains.

---

# 13. Disadvantages

### Debugging

A request may pass through many handlers, making tracing more difficult.

### No Guarantee of Handling

If no handler accepts the request, it may remain unprocessed unless a default handler exists.

### Configuration Complexity

Very long chains can become hard to configure and understand.

### When NOT to Use

Avoid CoR when:

* there is only one handler,
* the processing sequence is fixed and simple,
* every step must always execute (a simple pipeline may be clearer).

---

# 14. Best Practices

* Give each handler one responsibility.
* Keep handlers independent.
* Consider a default handler for unprocessed requests.
* Document the chain order if it matters.
* Prefer composition over hardcoded `if-else` logic.

---

# 15. Common Mistakes

### Mistake 1

Creating one handler that performs every task.

That defeats the purpose.

---

### Mistake 2

Allowing handlers to know about distant handlers.

Each handler should know only its immediate successor.

---

### Mistake 3

Forgetting to forward the request.

If a handler neither processes nor forwards it, the chain breaks unexpectedly.

---

### Mistake 4

Using CoR when every handler must always run.

In that case, a dedicated processing pipeline may communicate intent more clearly.

---

# 16. Pattern Variations

## 1. Single Handler

Processing stops when the first suitable handler is found.

Example:

```text
Authentication

↓

Authorization

↓

Business Logic
```

If authentication fails, processing stops.

---

## 2. Multi-Handler

Every handler processes the request.

Example:

```text
Logging

↓

Metrics

↓

Audit

↓

Business Logic
```

All stages execute.

---

## 3. Dynamic Chain

Handlers are added or removed at runtime.

Example:

Plugins in an IDE.

---

# 17. Related Patterns

| Pattern                 | Difference                                  |
| ----------------------- | ------------------------------------------- |
| Chain of Responsibility | Passes requests through handlers.           |
| Command                 | Encapsulates a request as an object.        |
| Mediator                | Centralizes communication between objects.  |
| Observer                | Broadcasts notifications to many listeners. |
| Strategy                | Chooses one algorithm.                      |

---

## Chain of Responsibility vs Strategy

### Strategy

Question:

> **Which algorithm should I use?**

One strategy is selected.

---

### Chain of Responsibility

Question:

> **Who should handle this request?**

The request moves until someone handles it.

---

# 18. Industry Usage

Chain of Responsibility is common in workflow-oriented systems.

* **Microsoft:** ASP.NET middleware pipeline and message processing.
* **Google:** Backend request filters and service pipelines.
* **Adobe:** Image processing and event handling chains.
* **Qt:** Event propagation and event filters follow similar principles.
* **ZEISS / Siemens / Philips:** Validation workflows, image processing stages, and treatment planning pipelines.
* **Autodesk:** Command processing and geometry validation chains.
* **Browsers:** HTTP request/response processing and DOM event propagation.

The architectural goal is **decoupling the sender from the receiver while supporting extensible processing pipelines**.

---

# 19. Interview Questions

## Beginner

1. What problem does the Chain of Responsibility Pattern solve?
2. What is a handler?
3. Why is CoR a Behavioral Pattern?

---

## Intermediate

1. How does CoR support the Open/Closed Principle?
2. What happens if no handler processes the request?
3. When would you stop the chain versus continue processing?

---

## Advanced

1. How would you design a thread-safe validation chain?
2. How would you configure handlers dynamically from a configuration file?
3. How would you monitor performance across a long handler chain?

---

## Scenario-Based

Your TPS must validate:

* Patient
* CT
* Structures
* Machine
* Beams
* Prescription

before dose calculation.

Design a Chain of Responsibility that allows new validation stages to be inserted without modifying existing validators.

---

## Architecture

Design a CoR architecture for a **College ERP** login process:

* Input Validation
* CAPTCHA Validation
* Database Authentication
* Role Loading
* Permission Validation
* Audit Logging
* Dashboard Loading

Explain:

* the handler hierarchy,
* request flow,
* error handling,
* and where the chain should stop.

---

# 20. Practice Exercises

### Beginner Exercise

Design a purchase approval workflow with:

* Team Lead
* Manager
* Director

using the Chain of Responsibility Pattern.

---

### Intermediate Exercise

Design a Qt file-opening workflow that passes a file through:

* Extension Validator
* Permission Checker
* File Loader
* Parser
* Viewer

Explain how handlers collaborate.

---

### Advanced Exercise

Design a **Chain of Responsibility** for a **Treatment Planning System** before dose calculation.

The chain should include:

* Patient Validator
* CT Validator
* Structure Validator
* Machine Validator
* Beam Validator
* Prescription Validator
* Dose Calculator
* DVH Calculator
* Report Generator

Requirements:

* Stop on validation failure.
* Log every stage.
* Support insertion of future validation handlers without changing existing ones.
* Explain ownership, configuration, and testing strategy.

**Do not implement the solution yet.** Focus on architecture and object collaboration.

---

# Key Architectural Takeaway

The **Chain of Responsibility Pattern** is about **passing a request through a sequence of independent handlers until it is processed or the chain ends**.

A junior developer thinks:

> "I'll put every rule inside one big `if-else` block."

A software architect thinks:

> **"Each processing step has one responsibility. I'll organize them into a configurable chain so the workflow is extensible, testable, and easy to maintain."**

This mindset is fundamental in enterprise systems, middleware, Qt event processing, medical validation workflows, and large-scale request processing pipelines.

---

## What You'll Learn Next

Type **`NEXT`** to continue with **Lesson 16: Command Pattern**, one of the most important patterns for Qt applications, undo/redo systems, IDEs, CAD software, and Treatment Planning Systems. It is a cornerstone of applications like **Qt Creator**, **Visual Studio**, **Photoshop**, and **AutoCAD**.
