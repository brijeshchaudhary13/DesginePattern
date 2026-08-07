

# Lesson 20 — Memento Pattern

---

# Before We Start

The **Memento Pattern** is one of the most misunderstood patterns.

Many developers think:

> "Memento means Undo."

That is **not** correct.

A **Memento does not perform Undo**.

Instead:

> **A Memento captures an object's state so it can be restored later.**

Undo is usually implemented using:

```text
Command
        +
Memento
```

This combination powers:

* Photoshop
* AutoCAD
* Microsoft Word
* Qt Creator
* Visual Studio
* Blender
* Treatment Planning Systems (TPS)

---

# The Architect's Way of Thinking

Imagine you're building a **Treatment Planning System**.

A physicist modifies a treatment plan:

```text
Beam Angle = 0°
↓

Beam Angle = 20°
↓

Beam Angle = 45°
↓

Beam Angle = 90°
```

Now the physicist clicks:

```text
Undo
```

Question:

How can we restore the previous beam angle?

A junior developer writes:

```cpp
oldAngle = beam.angle;
```

Works for one property.

Tomorrow the beam contains:

```text
Angle

Energy

Weight

MLC Leaves

Collimator

Isocenter

Jaw Positions

Beam Name

Machine

Dose Constraints

Optimization Flags
```

Now what?

Store every variable manually?

Impossible.

An architect thinks:

> **"The object itself knows its complete state. Let it create a snapshot of itself."**

That snapshot is a **Memento**.

---

# 1. Introduction

## What is the Memento Pattern?

The **Memento Pattern** captures and stores an object's internal state without exposing its implementation details.

The object can later restore itself from that snapshot.

Simple definition:

> **Save an object's state so it can be restored later without breaking encapsulation.**

---

## Why was it created?

Many applications need:

* Undo
* Redo
* Checkpoints
* Savepoints
* Rollback
* Time Travel
* Version History

Instead of exposing internal variables, the object creates its own snapshot.

---

## Category

**Behavioral Pattern**

---

## What problem does it solve?

Without Memento:

```cpp
oldX

oldY

oldWidth

oldHeight

oldColor

oldRotation
```

Developers manually save every variable.

Easy to forget one.

With Memento:

```text
Rectangle

↓

createMemento()

↓

Snapshot
```

Everything is saved together.

---

# 2. Problem Statement

Imagine a **Photo Editor**.

User:

```text
Open Image

↓

Crop

↓

Rotate

↓

Brightness

↓

Contrast

↓

Undo
```

Without Memento:

Need to manually restore:

* pixels,
* filters,
* selection,
* zoom,
* rotation.

Complex and error-prone.

---

# 3. Motivation

Architects observed that objects should protect their own state.

Instead of external code reading internal variables,

let the object itself produce a snapshot.

This preserves **encapsulation**.

---

# 4. Real-World Analogy

## Video Game Save Point

You reach a checkpoint.

The game saves:

```text
Health

Weapons

Inventory

Position

Level

Score
```

Later:

```text
Load Game
```

Everything returns exactly as before.

The save file is the **Memento**.

---

## Mapping

| Game         | Software   |
| ------------ | ---------- |
| Player       | Originator |
| Save File    | Memento    |
| Save Manager | Caretaker  |

---

# 5. Software Scenario

Memento appears whenever state must be restored.

### Desktop Applications

* Undo
* Redo
* Auto-save
* Version history

---

### Qt Applications

Imagine a drawing application.

User changes:

```text
Rectangle

↓

Move

↓

Resize

↓

Rotate
```

Before each change:

```text
Rectangle

↓

createMemento()

↓

QUndoCommand stores snapshot
```

Qt's `QUndoCommand` does not automatically use Memento internally, but many applications combine these patterns to implement reliable undo for complex objects.

---

### CAD Software

Restore:

* Geometry
* Constraints
* Layers

---

### Medical Imaging

Restore:

* Window/Level
* Camera Position
* ROI Editing
* Beam Parameters

---

# 6. UML Class Diagram

```text
                +----------------------+
                |     Originator       |
                +----------------------+
                | +createMemento()     |
                | +restore()           |
                +----------+-----------+
                           |
                     creates/restores
                           |
                           v
                +----------------------+
                |      Memento         |
                +----------------------+
                | state                |
                +----------+-----------+
                           ^
                           |
                +----------------------+
                |     Caretaker        |
                +----------------------+
                | history              |
                +----------------------+
```

---

## Responsibilities

### Originator

Owns the real state.

Creates snapshots.

Restores snapshots.

---

### Memento

Stores state.

No business logic.

---

### Caretaker

Stores history.

Never modifies snapshots.

---

# 7. Participants

## Originator

Example:

```text
Beam
```

Methods:

```cpp
createMemento()

restore()
```

---

## Memento

Stores:

```text
Angle

Energy

Weight

MLC
```

---

## Caretaker

Example:

```text
Undo History
```

Stores:

```text
Snapshot 1

Snapshot 2

Snapshot 3
```

---

## Client

Triggers save and restore operations.

---

# 8. Collaboration

Runtime Flow

```text
User Rotates Beam

↓

Beam.createMemento()

↓

Undo Stack Stores Snapshot

↓

Beam Changes

↓

Undo

↓

Restore Snapshot
```

Notice:

The caretaker never understands the contents of the snapshot.

It only stores it.

---

# 9. C++ Example

```cpp
#include <iostream>
#include <memory>
#include <string>

// Memento
class TextMemento
{
public:
    explicit TextMemento(std::string state)
        : m_state(std::move(state))
    {}

    const std::string& getState() const
    {
        return m_state;
    }

private:
    std::string m_state;
};

// Originator
class TextEditor
{
public:
    void setText(const std::string& text)
    {
        m_text = text;
    }

    std::shared_ptr<TextMemento> createMemento() const
    {
        return std::make_shared<TextMemento>(m_text);
    }

    void restore(const std::shared_ptr<TextMemento>& memento)
    {
        m_text = memento->getState();
    }

    void print() const
    {
        std::cout << m_text << '\n';
    }

private:
    std::string m_text;
};

// Caretaker
class History
{
public:
    void push(std::shared_ptr<TextMemento> m)
    {
        m_history.push_back(std::move(m));
    }

    std::shared_ptr<TextMemento> pop()
    {
        auto m = m_history.back();
        m_history.pop_back();
        return m;
    }

private:
    std::vector<std::shared_ptr<TextMemento>> m_history;
};
```

---

## Design Focus

Notice:

The history object never knows:

```cpp
text

cursorPosition

selection
```

It simply stores snapshots.

The editor alone understands them.

---

# 10. Qt Example

Imagine a Qt CAD editor.

Before moving an item:

```text
QGraphicsItem

↓

createMemento()

↓

Move Item

↓

Undo

↓

restore()
```

A `QUndoCommand` may internally keep a snapshot of the item's previous state.

This allows complex objects to return to exactly their earlier configuration.

---

# 11. Medical Software Example

Imagine a **Treatment Planning System**.

A beam contains:

```text
Angle

Energy

Weight

MLC

Collimator

Jaw

Isocenter
```

Before editing:

```text
Beam

↓

createMemento()

↓

History
```

If the physicist clicks Undo:

```text
History

↓

Beam.restore()

↓

Entire Beam Restored
```

Benefits:

* Every property returns consistently.
* No risk of forgetting one field.
* Object integrity is preserved.

---

# 12. Advantages

### Encapsulation

Only the originator understands its state.

### Easy Rollback

Restore previous versions safely.

### Clean Undo Support

Works naturally with Command.

### Version History

Supports checkpoints and history.

### Simplicity for Clients

Clients don't manipulate internal fields.

---

# 13. Disadvantages

### Memory Usage

Large objects produce large snapshots.

### Performance

Frequent snapshots may be expensive.

### History Management

Unlimited history can consume significant memory.

### When NOT to Use

Avoid Memento when:

* the object is tiny and rollback isn't needed,
* reconstructing state is cheaper than storing it,
* snapshots would be prohibitively large.

---

# 14. Best Practices

* Keep mementos immutable.
* Let only the originator create and restore them.
* Store only the necessary state.
* Limit history size if memory is a concern.
* Compress or use incremental snapshots for very large objects.

---

# 15. Common Mistakes

### Mistake 1

Allowing clients to modify the memento.

Snapshots should represent a fixed point in time.

---

### Mistake 2

Putting business logic inside the memento.

A memento stores state—it doesn't perform actions.

---

### Mistake 3

Exposing internal implementation details.

The caretaker should not inspect or edit snapshot contents.

---

### Mistake 4

Saving snapshots too frequently.

For very large objects, snapshot creation itself can become a bottleneck.

---

# 16. Pattern Variations

## 1. Full Snapshot

Stores the complete object state.

Simple but memory-intensive.

---

## 2. Incremental Memento

Stores only the changes since the previous snapshot.

Efficient for large documents.

---

## 3. Persistent History

Snapshots stored on disk.

Useful for:

* Auto-save
* Crash recovery
* Version history

---

## 4. Immutable Memento

Most common.

Once created, it never changes.

---

# 17. Related Patterns

| Pattern   | Difference                                   |
| --------- | -------------------------------------------- |
| Memento   | Captures object state.                       |
| Command   | Represents an action.                        |
| Prototype | Clones an object.                            |
| State     | Changes behavior based on current state.     |
| Caretaker | Manages snapshots but does not inspect them. |

---

## Command vs Memento (Interview Favorite)

### Command

Stores:

```text
What happened?
```

Example:

```text
Rotate Beam 20°
```

---

### Memento

Stores:

```text
What was the previous state?
```

Example:

```text
Beam Angle = 0°
```

Together:

```text
RotateBeamCommand

↓

Before execute()

↓

createMemento()

↓

execute()

↓

Undo

↓

restore(memento)
```

This combination is used in many professional desktop applications.

---

# 18. Industry Usage

Memento is common in software requiring rollback and history.

* **Microsoft:** Word, Excel, Visual Studio undo history.
* **Google:** Document versioning and collaborative editing.
* **Adobe:** Photoshop history states.
* **Qt:** Applications built with `QUndoStack` often combine Command and Memento concepts.
* **ZEISS / Siemens / Philips:** Treatment plan editing, imaging workflows, and protocol rollback.
* **Autodesk:** CAD editing history and undo systems.
* **Game Engines:** Save games and checkpoints.

The architectural goal is **capturing and restoring state without exposing implementation details**.

---

# 19. Interview Questions

## Beginner

1. What problem does the Memento Pattern solve?
2. What is the difference between the Originator and the Caretaker?
3. Why does the Caretaker not inspect the snapshot?

---

## Intermediate

1. How does Memento preserve encapsulation?
2. What are the memory trade-offs?
3. When would you choose incremental snapshots?

---

## Advanced

1. How would you design undo for a 500 MB medical image?
2. How would you compress mementos?
3. How would you combine Command, Memento, and Observer?

---

## Scenario-Based

Your TPS allows editing:

* Beam Angles
* MLC Positions
* Isocenter
* Dose Constraints

Design an Undo system using Memento and explain how snapshots are managed efficiently.

---

## Architecture

Design a Memento architecture for a **College ERP**.

Support:

* Student Profile Editing
* Fee Updates
* Timetable Editing

Requirements:

* Undo
* Redo
* Version History
* Crash Recovery

Explain:

* Originators,
* Mementos,
* Caretakers,
* storage strategy,
* and memory management.

---

# 20. Practice Exercises

### Beginner Exercise

Design a `TextEditor` using the Memento Pattern.

Support:

* Type text
* Save snapshot
* Restore snapshot

Draw the UML and explain the collaboration.

---

### Intermediate Exercise

Design a Qt drawing application where each shape can restore:

* Position
* Rotation
* Size
* Color

using Memento.

---

### Advanced Exercise

Design a **Memento architecture** for a **Treatment Planning System**.

Originators:

* Beam
* Structure Set
* Dose Plan
* Optimization Settings

Requirements:

* Unlimited Undo/Redo (with configurable history limits)
* Incremental snapshots for large data
* Crash recovery
* Thread-safe snapshot creation
* Integration with the Command Pattern

Explain:

* snapshot ownership,
* lifecycle,
* memory optimization,
* persistence strategy,
* and recovery workflow.

**Do not implement the solution yet.** Focus on architecture and object responsibilities.

---

# ⭐ Architect's Insight: Command + Memento + Observer

In professional desktop software, these patterns often work together:

```text
User Action
      ↓
Command.execute()
      ↓
Originator.createMemento()
      ↓
Model Updated
      ↓
Observer Notifies UI
      ↓
View Refreshes
```

For example, in a TPS:

```text
Rotate Beam
      ↓
RotateBeamCommand
      ↓
Beam.createMemento()
      ↓
Beam rotates
      ↓
BeamModel emits change
      ↓
Dose View updates
      ↓
DVH Panel refreshes
```

Each pattern has a distinct responsibility:

* **Command** → Represents the action.
* **Memento** → Preserves the previous state.
* **Observer** → Notifies interested views about the change.

This separation is a hallmark of scalable desktop application architecture.

---

# Key Architectural Takeaway

The **Memento Pattern** is about **capturing an object's state while preserving encapsulation**.

A junior developer thinks:

> "I'll manually save every field before changing it."

A software architect thinks:

> **"The object already knows its own state. I'll let it create a snapshot of itself and restore that snapshot when needed, keeping history management separate from business logic."**

This approach scales well for complex domain objects in CAD systems, IDEs, graphics editors, and medical treatment planning software.

---
[⬅️ Mediator Pattern ](/MediatorPattern.md)        |  [Observer Pattern  ➡️](/ObserverPattern.md) 
---
## **License**
This project is licensed under the MIT License.

---

Happy Coding!



