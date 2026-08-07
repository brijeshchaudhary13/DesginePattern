# Design Patterns Master Course

# Lesson 22 — State Pattern (Behavioral Pattern)

---

# Before We Start

The **State Pattern** is one of the most valuable patterns for eliminating large `if-else` and `switch` statements.

If you work in:

* Medical Devices
* Qt Applications
* Embedded Systems
* ATM Software
* Workflow Engines
* Industrial Automation
* Robotics
* CAD Software

you will encounter systems where behavior changes depending on the **current state**.

The State Pattern is the architect's solution.

---

# The Architect's Way of Thinking

Imagine you're building a **Treatment Planning System (TPS)**.

A treatment plan has the following workflow:

```text
Draft
   │
   ▼
Validated
   │
   ▼
Approved
   │
   ▼
Delivered
   │
   ▼
Archived
```

Business rules:

* **Draft**

  * Can edit beams
  * Can delete structures
  * Cannot export

* **Validated**

  * Can optimize
  * Cannot delete structures

* **Approved**

  * Cannot modify plan
  * Can export DICOM RT

* **Delivered**

  * Read-only

A junior developer writes:

```cpp
if(plan.state == Draft)
{
    ...
}
else if(plan.state == Validated)
{
    ...
}
else if(plan.state == Approved)
{
    ...
}
else if(plan.state == Delivered)
{
    ...
}
```

Now imagine 25 operations:

* Add Beam
* Delete Beam
* Rotate Beam
* Edit Prescription
* Export
* Print
* Optimize
* Calculate Dose
* Save
* Approve
* Archive

Every function contains:

```cpp
if(state == ...)
```

The application becomes difficult to maintain.

An architect asks:

> **"Why should every function know every possible state?"**

Instead:

```text
TreatmentPlan
      │
      ▼
Current State Object
      │
      ▼
Behavior
```

The current state object decides what is allowed.

That is the **State Pattern**.

---

# 1. Introduction

## What is the State Pattern?

The **State Pattern** allows an object to change its behavior when its internal state changes.

Instead of using conditionals, different **state objects** implement the behavior.

Simple definition:

> **Represent each state as an object and delegate behavior to it.**

---

## Why was it created?

Objects often behave differently depending on their current state.

Instead of:

```cpp
if(state == Open)
...
```

Use:

```cpp
currentState->handle();
```

---

## Category

**Behavioral Pattern**

---

## What problem does it solve?

Without State:

```text
if

else if

else if

else if

switch

switch

switch
```

With State:

```text
Context

↓

Current State

↓

Behavior
```

No giant conditionals.

---

# 2. Problem Statement

Imagine an **ATM Machine**.

States:

```text
No Card

↓

Card Inserted

↓

PIN Verified

↓

Transaction

↓

Cash Dispensed
```

Without State:

Every button press checks:

```cpp
if(state == ...)
```

The logic becomes scattered across many methods.

---

# 3. Motivation

Architects noticed:

Behavior depends on state.

Instead of asking:

```text
"What state am I in?"
```

Ask the current state object:

```text
"What should happen?"
```

This moves state-specific logic into dedicated classes.

---

# 4. Real-World Analogy

## Traffic Light

States:

```text
Red

↓

Green

↓

Yellow

↓

Red
```

Behavior changes:

* Red → Stop
* Green → Go
* Yellow → Prepare to Stop

The traffic light doesn't execute:

```cpp
if(color == Red)
```

Instead, each state defines its own behavior.

---

## Mapping

| Traffic Light | Software       |
| ------------- | -------------- |
| Light         | Context        |
| Red           | Concrete State |
| Green         | Concrete State |
| Yellow        | Concrete State |

---

# 5. Software Scenario

### Desktop Applications

Document lifecycle:

```text
New

↓

Modified

↓

Saved

↓

Closed
```

---

### Qt Applications

Qt provides the **Qt State Machine Framework** (`QStateMachine`, `QState`, `QFinalState`).

Example:

```text
Disconnected

↓

Connecting

↓

Connected

↓

Disconnected
```

Each state handles events differently.

---

### Medical Devices

Linear Accelerator:

```text
Power Off

↓

Standby

↓

Ready

↓

Beam On

↓

Emergency Stop
```

Different buttons are enabled in each state.

---

### CAD Software

Drawing Tool:

```text
Idle

↓

Selecting

↓

Dragging

↓

Resizing
```

Mouse events behave differently in each state.

---

# 6. UML Class Diagram

```text
                +----------------------+
                |      State           |
                +----------------------+
                | +handle()            |
                +----------+-----------+
                           ^
              -------------|-------------
              |                          |
      +---------------+         +---------------+
      | DraftState    |         | ApprovedState |
      +---------------+         +---------------+

                 ^
                 |
          +----------------------+
          |      Context         |
          +----------------------+
          | currentState         |
          | +request()           |
          +----------------------+
```

---

## Responsibilities

### Context

* Maintains current state.
* Delegates requests to the current state.
* May change to another state.

---

### State

Defines state-specific behavior.

---

### Concrete State

Implements behavior for one state.

---

# 7. Participants

## Context

Example:

```text
TreatmentPlan
```

Stores:

```cpp
State* currentState;
```

---

## State Interface

Example:

```cpp
class PlanState
{
public:
    virtual void calculateDose() = 0;
};
```

---

## Concrete States

Examples:

```text
DraftState

ValidatedState

ApprovedState

DeliveredState
```

---

## Client

Uses the context.

Never talks directly to state objects.

---

# 8. Collaboration

Runtime Flow

```text
User Clicks Export

↓

TreatmentPlan

↓

currentState->export()

↓

ApprovedState

↓

Export DICOM RT
```

If the state changes:

```text
Draft

↓

Approved
```

The same call:

```cpp
plan.export();
```

now produces different behavior.

---

# 9. C++ Example

```cpp
#include <iostream>
#include <memory>

class State
{
public:
    virtual ~State() = default;
    virtual void pressButton() = 0;
};

class RedState : public State
{
public:
    void pressButton() override
    {
        std::cout << "Cannot cross. Red light.\n";
    }
};

class GreenState : public State
{
public:
    void pressButton() override
    {
        std::cout << "You may cross.\n";
    }
};

class TrafficLight
{
public:
    void setState(std::unique_ptr<State> state)
    {
        m_state = std::move(state);
    }

    void pressButton()
    {
        m_state->pressButton();
    }

private:
    std::unique_ptr<State> m_state;
};

int main()
{
    TrafficLight light;

    light.setState(std::make_unique<RedState>());
    light.pressButton();

    light.setState(std::make_unique<GreenState>());
    light.pressButton();
}
```

---

## Design Focus

Notice:

The client never checks:

```cpp
if(light == Red)
```

Instead:

```cpp
light.pressButton();
```

Behavior changes because the state object changes.

---

# 10. Qt Example

Imagine a Qt application controlling a medical scanner.

States:

```text
Disconnected
↓

Connecting
↓

Ready
↓

Scanning
↓

Error
```

Using `QStateMachine`:

* In `Ready`, the **Start Scan** button is enabled.
* In `Scanning`, it is disabled.
* In `Error`, only **Reset** is enabled.

The UI behavior is driven by the active state, not by scattered `if` statements.

---

# 11. Medical Software Example

Consider a **Linear Accelerator**.

Machine states:

```text
Power Off
↓

Standby
↓

Calibration
↓

Ready
↓

Beam On
↓

Emergency Stop
```

Behavior:

| State          | Beam On Allowed? | Settings Editable? |
| -------------- | ---------------- | ------------------ |
| Power Off      | ❌                | ❌                  |
| Standby        | ❌                | ✅                  |
| Calibration    | ❌                | Limited            |
| Ready          | ✅                | ✅                  |
| Beam On        | Already Active   | ❌                  |
| Emergency Stop | ❌                | ❌                  |

Instead of checking:

```cpp
if(machineState == Ready)
```

everywhere,

each state object implements its own rules.

This makes safety-critical behavior easier to reason about and test.

---

# 12. Advantages

### Eliminates Large Conditionals

No repeated `if-else` or `switch`.

### Open/Closed Principle

Add new states without modifying existing ones.

### Single Responsibility

Each state manages only its own behavior.

### Easier Testing

Each state can be tested independently.

### Better Readability

State transitions become explicit.

---

# 13. Disadvantages

### More Classes

Each state becomes a separate class.

### Transition Management

Many transitions can become complex.

### Overkill

For only two or three simple states, a basic conditional may be sufficient.

### When NOT to Use

Avoid State when:

* behavior barely changes,
* there are very few states,
* transitions are trivial.

---

# 14. Best Practices

* Keep each state focused on one behavior.
* Make transitions explicit.
* Keep shared logic in the context.
* Prefer immutable state objects when possible.
* Clearly document allowed transitions.

---

# 15. Common Mistakes

### Mistake 1

Keeping `if(state == ...)` inside state classes.

That defeats the purpose.

---

### Mistake 2

Making one giant state class.

Split behavior by state.

---

### Mistake 3

Letting clients manipulate state directly.

Clients should interact with the context.

---

### Mistake 4

Mixing workflow logic with UI logic.

The state machine should govern business behavior, not widget layout.

---

# 16. Pattern Variations

## 1. Static States

One shared instance per state.

Useful when states have no mutable data.

---

## 2. Dynamic States

A new state object is created on each transition.

Useful when state objects hold transition-specific data.

---

## 3. Hierarchical State Machines

Supported by Qt's `QStateMachine`.

Example:

```text
Connected
├── Idle
├── Downloading
└── Uploading
```

---

## 4. Table-Driven State Machine

Transitions are defined in a table instead of code.

Often used in embedded systems.

---

# 17. Related Patterns

| Pattern         | Difference                                         |
| --------------- | -------------------------------------------------- |
| State           | Behavior changes with internal state.              |
| Strategy        | Algorithm is selected externally.                  |
| Template Method | Defines a fixed algorithm with customizable steps. |
| Command         | Represents actions.                                |
| Mediator        | Coordinates interactions among objects.            |

---

## State vs Strategy (Most Important Interview Question)

Both patterns have similar UML diagrams.

### Strategy

Question:

> **"Which algorithm should I use?"**

Example:

```text
Compression

↓

ZIP

RAR

7z
```

The client chooses the algorithm.

---

### State

Question:

> **"How should I behave in my current state?"**

Example:

```text
ATM

↓

No Card

Card Inserted

PIN Verified
```

The object changes behavior automatically as its state changes.

---

# 18. Industry Usage

State machines are common in systems with workflows and modes.

* **Microsoft:** Document editing modes, connection states, installers.
* **Google:** Network protocols, Android lifecycle components.
* **Adobe:** Editing modes, tool states.
* **Qt:** `QStateMachine` framework, animation workflows, UI states.
* **ZEISS / Siemens / Philips:** Medical device operating modes, acquisition workflows, treatment lifecycle management.
* **Autodesk:** Drawing modes, selection modes, editing workflows.
* **Embedded Systems:** Protocol handlers, vending machines, printers, elevators.

The architectural goal is:

> **Represent behavior through state objects instead of conditional logic.**

---

# 19. Interview Questions

## Beginner

1. What problem does the State Pattern solve?
2. What is the role of the Context?
3. Why is State considered a Behavioral Pattern?

---

## Intermediate

1. How is State different from Strategy?
2. What are the advantages of representing states as objects?
3. When should you avoid the State Pattern?

---

## Advanced

1. How would you implement a thread-safe state machine?
2. How would you model hierarchical states in Qt?
3. How would you validate illegal state transitions?

---

## Scenario-Based

Your TPS workflow is:

```text
Draft
↓

Validated
↓

Approved
↓

Delivered
↓

Archived
```

Each state allows different operations.

Design a State-based architecture that eliminates repeated state checks throughout the application.

---

## Architecture

Design a State architecture for your **College ERP**.

Student admission lifecycle:

```text
Applied
↓

Documents Verified
↓

Fees Paid
↓

Admitted
↓

Graduated
↓

Alumni
```

Requirements:

* Different operations allowed in each state.
* Explicit transition rules.
* Easy addition of future states (e.g., "Suspended").

Explain:

* context responsibilities,
* state hierarchy,
* transition validation,
* and testing strategy.

---

# 20. Practice Exercises

### Beginner Exercise

Design a **Traffic Light** using the State Pattern.

States:

* Red
* Yellow
* Green

Explain how button presses are handled.

---

### Intermediate Exercise

Design a Qt-based **Network Connection Manager**.

States:

* Disconnected
* Connecting
* Connected
* Error

Describe how UI buttons change behavior based on the current state.

---

### Advanced Exercise

Design a **State architecture** for a **Treatment Planning System**.

Treatment Plan states:

* Draft
* Validated
* Approved
* Delivered
* Archived

Requirements:

* State-specific permissions.
* Explicit transitions.
* Audit logging of transitions.
* Integration with Observer to notify the UI.
* Integration with Command so transitions can be undone before approval.

**Do not implement the solution yet.** Focus on state hierarchy, transitions, responsibilities, and extensibility.

---

# ⭐ Architect's Insight: State + Observer + Command

In many enterprise applications, these patterns work together:

```text
User Action
      │
      ▼
Command
      │
      ▼
Context
      │
changes state
      ▼
State Object
      │
updates model
      ▼
Observer notifies UI
```

For example, in a TPS:

```text
Approve Plan Command
        ↓
TreatmentPlan
        ↓
Current State changes:
Validated → Approved
        ↓
Observer notifications:
• Toolbar updates
• Export enabled
• Editing disabled
• Audit log refreshed
```

Each pattern has a separate responsibility:

* **Command** → Represents the user's action.
* **State** → Defines behavior based on lifecycle.
* **Observer** → Updates interested components.

This combination is common in workflow-driven desktop applications.

---

# Key Architectural Takeaway

The **State Pattern** is about **moving state-specific behavior into dedicated state objects**.

A junior developer thinks:

> "I'll add another `if(state == ...)`."

A software architect thinks:

> **"Behavior belongs to the state itself. I'll model each state as an object so new states can be added without scattering conditional logic throughout the system."**

This approach leads to cleaner, safer, and more maintainable architectures—especially in workflow engines, Qt applications, embedded systems, and medical devices.

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

Only four lessons remain:

1. **Strategy Pattern**
2. **Template Method Pattern**
3. **Visitor Pattern**
4. **Null Object Pattern**

These are among the most frequently asked patterns in senior C++/Qt and software architecture interviews.

---

## What You'll Learn Next

Type **`NEXT`** to continue with **Lesson 23: Strategy Pattern**.

You'll learn one of the most important interview topics:

> **State vs Strategy**

We'll explore why their UML diagrams look almost identical, why candidates often confuse them, and how architects decide which one to use in real systems such as payment gateways, compression engines, dose calculation algorithms (Monte Carlo vs Pencil Beam), image processing filters, and Qt applications.
