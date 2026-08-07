

# Lesson 16 — Command Pattern 

---

# Before We Start

If I had to choose **one design pattern that every C++/Qt developer must master**, it would be the **Command Pattern**.

Why?

Because it is everywhere:

* Qt (`QUndoCommand`, `QUndoStack`)
* Visual Studio
* Qt Creator
* Photoshop
* AutoCAD
* Blender
* Microsoft Office
* Treatment Planning Systems (TPS)
* CAD Software

If you understand Command deeply, you'll understand how professional desktop applications are designed.

---

# The Architect's Way of Thinking

Imagine you're building a **Treatment Planning System (TPS)**.

A medical physicist performs these actions:

```text
Open Patient
↓

Add Beam
↓

Rotate Beam
↓

Move Isocenter
↓

Change Dose
↓

Delete Beam
↓

Optimize Plan
```

Now management asks:

> "We need Undo and Redo."

A junior developer says:

```cpp
void undoRotateBeam();
void undoMoveBeam();
void undoDeleteBeam();
void undoDoseChange();
```

Soon you'll have:

```text
undoBeamRotation()

undoBeamDeletion()

undoDoseChange()

undoROI()

undoStructure()

undoOptimization()

undoPrescription()

undo...
```

Hundreds of undo functions.

An architect asks a different question:

> **"Can every user action become an object?"**

That idea leads directly to the **Command Pattern**.

---

# 1. Introduction

## What is the Command Pattern?

The **Command Pattern** encapsulates a request as an object.

Instead of calling a function directly:

```cpp
beam.rotate(20);
```

Create an object:

```cpp
RotateBeamCommand(beam, 20)
```

Now the request itself is an object.

---

## Simple Definition

> **Turn an operation into an object.**

That object can then be:

* Stored
* Queued
* Logged
* Undone
* Redone
* Serialized
* Sent over the network

---

## Why was it created?

Developers wanted to:

* Undo/Redo
* Queue operations
* Record macros
* Schedule jobs
* Decouple UI from business logic

Instead of storing function calls, they stored **objects representing those calls**.

---

## Category

**Behavioral Pattern**

---

## What problem does it solve?

Without Command:

```text
Button

↓

Beam.rotate()

↓

Dose.calculate()

↓

UpdateUI()
```

Button knows too much.

With Command:

```text
Button

↓

RotateBeamCommand

↓

Beam
```

The button knows only the command.

---

# 2. Problem Statement

Imagine a **CAD Application**.

Toolbar:

```text
Move

Rotate

Scale

Delete

Copy

Paste

Undo

Redo
```

Without Command:

Each button directly calls model objects.

Undo becomes difficult.

Macro recording becomes impossible.

Remote execution becomes difficult.

Testing UI becomes harder.

---

# 3. Motivation

Architects realized:

A request has information:

```text
Receiver

Operation

Parameters

Time

User

Context
```

Instead of throwing that information away after calling a function,

store it inside an object.

That object becomes reusable.

---

# 4. Real-World Analogy

## Restaurant

Customer:

```text
"I want Pizza."
```

Does the chef hear it directly?

No.

The waiter writes an order.

```text
Customer

↓

Waiter

↓

Order Slip

↓

Chef
```

The order slip is the **Command**.

It contains:

* What to cook
* Size
* Toppings
* Table Number

The chef executes it later.

---

## Mapping

| Restaurant | Software |
| ---------- | -------- |
| Customer   | Client   |
| Waiter     | Invoker  |
| Order Slip | Command  |
| Chef       | Receiver |

---

# 5. Software Scenario

Command appears whenever requests need to become objects.

### Desktop Applications

* Undo/Redo
* Toolbar actions
* Menu commands

---

### Qt Applications

Qt provides one of the best real-world examples:

```text
QUndoCommand

↓

QUndoStack
```

Every edit:

```text
Insert Text

Delete Text

Move Item

Resize Rectangle
```

is represented as a command object.

---

### CAD Software

Commands:

* Rotate
* Scale
* Mirror
* Offset

---

### Medical Imaging

Commands:

* Add ROI
* Remove ROI
* Window Level
* Zoom
* Rotate Volume

---

### Game Engines

Commands:

* Move Player
* Fire Weapon
* Jump

---

# 6. UML Class Diagram

```text
                  +----------------------+
                  |      Command         |
                  +----------------------+
                  | +execute()           |
                  | +undo()              |
                  +----------+-----------+
                             ^
                -------------|-------------
                |                           |
      +-------------------+      +-------------------+
      | RotateCommand     |      | DeleteCommand     |
      +-------------------+      +-------------------+
                |
                | uses
                v
          +----------------------+
          |      Receiver        |
          +----------------------+
          | rotate()             |
          | delete()             |
          +----------------------+

                ^
                |
           +------------+
           | Invoker    |
           +------------+

                ^
                |
             Client
```

---

# Responsibilities

## Command

Defines:

```cpp
execute()

undo()
```

---

## Concrete Command

Stores:

* Receiver
* Parameters
* Previous State (for Undo)

---

## Receiver

Actually performs the work.

Example:

```text
Beam

Image

Document

Patient
```

---

## Invoker

Calls:

```cpp
command->execute();
```

Never knows what the command does.

---

## Client

Creates the command.

---

# 7. Participants

## Command

Abstract interface.

Example:

```cpp
ICommand
```

---

## Concrete Command

Example:

```text
RotateBeamCommand
```

Stores:

```text
Beam

Angle

Old Angle
```

---

## Receiver

```text
Beam
```

Provides:

```cpp
rotate()
```

---

## Invoker

Examples:

```text
Toolbar

Menu

Shortcut

Undo Stack
```

---

## Client

Creates command objects.

---

# 8. Collaboration

Runtime Flow

```text
User Clicks Rotate

↓

Toolbar

↓

RotateBeamCommand

↓

Beam.rotate()

↓

Undo Stack Stores Command
```

Later:

```text
Undo

↓

Undo Stack

↓

RotateBeamCommand.undo()
```

This is why Command is ideal for Undo/Redo.

---

# 9. C++ Example

```cpp
#include <iostream>
#include <memory>

// Receiver
class Light
{
public:
    void on()
    {
        std::cout << "Light ON\n";
    }

    void off()
    {
        std::cout << "Light OFF\n";
    }
};

// Command
class Command
{
public:
    virtual ~Command() = default;
    virtual void execute() = 0;
};

// Concrete Command
class LightOnCommand : public Command
{
public:
    explicit LightOnCommand(Light& light)
        : m_light(light)
    {}

    void execute() override
    {
        m_light.on();
    }

private:
    Light& m_light;
};

// Invoker
class RemoteControl
{
public:
    void setCommand(std::shared_ptr<Command> command)
    {
        m_command = std::move(command);
    }

    void pressButton()
    {
        if (m_command)
            m_command->execute();
    }

private:
    std::shared_ptr<Command> m_command;
};

int main()
{
    Light light;

    auto command =
        std::make_shared<LightOnCommand>(light);

    RemoteControl remote;

    remote.setCommand(command);

    remote.pressButton();
}
```

---

## Design Focus

Notice:

The remote never knows:

```cpp
light.on();
```

Instead:

```cpp
command.execute();
```

The command hides the receiver.

---

# 10. Qt Example

This is one of the most important Qt examples.

Qt provides:

```text
QUndoCommand
```

Each user action becomes:

```text
MoveRectangleCommand

ResizeRectangleCommand

DeleteItemCommand

AddNodeCommand
```

Stored in:

```text
QUndoStack
```

Undo:

```cpp
undoStack.undo();
```

Redo:

```cpp
undoStack.redo();
```

This is a textbook Command implementation.

If you're preparing for Qt interviews, understanding `QUndoCommand` and `QUndoStack` is essential.

---

# 11. Medical Software Example

Imagine a **Treatment Planning System**.

Physicist actions:

```text
Add Beam

Delete Beam

Move Isocenter

Rotate Gantry

Change Energy

Modify MLC

Optimize Plan
```

Each action becomes a command.

Example:

```text
AddBeamCommand

↓

execute()

↓

BeamManager.addBeam()
```

Undo:

```text
AddBeamCommand

↓

undo()

↓

BeamManager.removeBeam()
```

Now the TPS automatically supports:

* Undo
* Redo
* Macro Recording
* Session Replay
* Audit Trail

Many professional TPS and CAD systems follow this architectural style.

---

# 12. Advantages

### Undo/Redo

The biggest advantage.

### Loose Coupling

Invoker doesn't know business logic.

### Macro Recording

Store commands.

Replay later.

### Queuing

Commands can execute later.

### Logging

Every action becomes an object.

Easy to audit.

### Testing

Commands can be tested independently.

---

# 13. Disadvantages

### More Classes

One command per operation.

### Additional Memory

Commands store state.

### Complexity

Simple applications may not benefit.

### When NOT to Use

Avoid Command when:

* actions are extremely simple,
* undo is unnecessary,
* requests never need to be stored or delayed.

---

# 14. Best Practices

* One command should represent one user intention.
* Keep business logic inside the receiver.
* Store only the state needed for undo.
* Use an invoker (toolbar, menu, shortcut) to execute commands.
* Keep commands small and focused.

---

# 15. Common Mistakes

### Mistake 1

Putting business logic inside the invoker.

The invoker should only execute commands.

---

### Mistake 2

Making commands know about UI widgets.

Commands should operate on the domain model, not directly on buttons or dialogs.

---

### Mistake 3

Saving too much state for undo.

Store only what is required to restore the previous state.

---

### Mistake 4

Creating one giant command that performs many unrelated actions.

Prefer one command per user action.

---

# 16. Pattern Variations

## 1. Undoable Command

Supports:

```cpp
execute()

undo()
```

Most common in desktop applications.

---

## 2. Macro Command

Contains multiple commands.

Example:

```text
Optimize Plan

↓

Move Beam

↓

Rotate Beam

↓

Calculate Dose
```

The macro executes them in sequence.

---

## 3. Asynchronous Command

Queued for later execution.

Common in job schedulers and background workers.

---

## 4. Network Command

Serialized and sent to another machine.

Used in multiplayer games and distributed systems.

---

# 17. Related Patterns

| Pattern                 | Difference                                  |
| ----------------------- | ------------------------------------------- |
| Command                 | Encapsulates a request as an object.        |
| Chain of Responsibility | Passes a request through multiple handlers. |
| Strategy                | Chooses one algorithm.                      |
| Memento                 | Stores object state for undo.               |
| Observer                | Notifies many listeners of changes.         |

---

## Command vs Chain of Responsibility

This is a popular interview question.

### Command

Question:

> **"What action should be performed?"**

The action is packaged as an object.

---

### Chain of Responsibility

Question:

> **"Who should handle the request?"**

The request travels through handlers.

---

# 18. Industry Usage

Command is one of the most widely used patterns in desktop software.

* **Microsoft:** Office Undo/Redo, Visual Studio editor actions, ribbon commands.
* **Google:** Editor actions and command-based operations in productivity tools.
* **Adobe:** Photoshop operations (brush stroke, crop, filter, transform) are modeled as commands to support undo history.
* **Qt:** `QUndoCommand` and `QUndoStack` are textbook implementations.
* **ZEISS / Siemens / Philips:** Treatment planning actions, image annotation, contour editing, and workflow operations.
* **Autodesk:** AutoCAD commands (`MOVE`, `ROTATE`, `COPY`, `UNDO`) follow the Command concept.
* **Game Engines:** Input commands, replay systems, AI actions, and editor tools.

The architectural goal is **treating user actions as first-class objects**.

---

# 19. Interview Questions

## Beginner

1. What problem does the Command Pattern solve?
2. What are the responsibilities of the Command, Receiver, and Invoker?
3. Why is Command useful for Undo/Redo?

---

## Intermediate

1. How would you implement undo for a `RotateBeamCommand`?
2. Why shouldn't the Invoker contain business logic?
3. What is a Macro Command?

---

## Advanced

1. How would you serialize commands for session replay?
2. How would you implement thread-safe command execution?
3. How would you combine Command with Memento to support complex undo operations?

---

## Scenario-Based

Your TPS must support:

* Add Beam
* Delete Beam
* Rotate Beam
* Change MLC
* Optimize Plan

with unlimited Undo/Redo.

Design a Command-based architecture and explain how the undo history is managed.

---

## Architecture

Design a Command architecture for a **College ERP** where the following actions are commands:

* Add Student
* Delete Student
* Update Student
* Assign Course
* Pay Fees
* Generate ID Card

Requirements:

* Undo/Redo
* Audit Logging
* Macro Commands (e.g., "Student Admission")
* Queueing background commands

Explain:

* command hierarchy,
* receivers,
* invokers,
* and history management.

---

# 20. Practice Exercises

### Beginner Exercise

Design a TV remote using the Command Pattern.

Commands:

* Power On
* Power Off
* Volume Up
* Volume Down

Draw the UML and explain the interaction.

---

### Intermediate Exercise

Design a Qt drawing application using:

* `QUndoCommand`
* `QUndoStack`

Support:

* Move Shape
* Resize Shape
* Delete Shape

Explain how Undo and Redo work.

---

### Advanced Exercise

Design a **Command architecture** for a **Treatment Planning System**.

Commands:

* Add Beam
* Delete Beam
* Rotate Beam
* Change Energy
* Move Isocenter
* Optimize Plan
* Calculate Dose

Requirements:

* Unlimited Undo/Redo
* Command History
* Macro Commands
* Audit Logging
* Session Replay
* Thread-safe execution of long-running commands

**Do not implement the solution yet.** Focus on architecture, command lifecycle, and object collaboration.

---

# Key Architectural Takeaway

The **Command Pattern** is about **turning actions into objects**.

A junior developer thinks:

> "When the button is clicked, I'll call the function directly."

A software architect thinks:

> **"A user action has value beyond immediate execution. I'll represent it as an object so it can be stored, undone, replayed, logged, queued, or sent across the network."**

This mindset is fundamental in professional desktop software, CAD tools, IDEs, Qt applications, and medical treatment planning systems.

---

# ⭐ Architect's Insight: Command + Memento

Here's an important preview:

In real-world applications, **Command** and **Memento** are often used together.

* **Command** represents *what happened* (e.g., "Rotate Beam by 15°").
* **Memento** stores *the previous state* (e.g., the beam's original angle).

Together they provide a clean and scalable Undo/Redo architecture.

This combination is used in applications like Photoshop, Qt Creator, AutoCAD, and many medical imaging systems.

---
[⬅️ Chain of Responsibility Pattern ](/ChainofResponsibilityPattern.md)        |  [Interpreter Pattern ➡️](/InterpreterPattern.md) 
---
## **License**
This project is licensed under the MIT License.

---

Happy Coding!

