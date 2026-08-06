# Design Patterns Master Course

# Lesson 9 — Composite Pattern (Structural Pattern)

---

# Before We Start

Imagine you are developing a **Treatment Planning System (TPS)**.

The patient has:

```text
Patient
```

Inside the patient:

```text
Structures
```

Inside Structures:

```text
PTV

CTV

Heart

Lung

Spinal Cord

Kidney
```

Now imagine another patient.

This patient has:

```text
Patient

↓

Structures

↓

Head

↓

Brain

↓

Tumor

↓

Brain Stem

↓

Optic Nerve
```

Some nodes contain children.

Some do not.

Question:

Should your code treat:

```text
Heart
```

differently from

```text
Structures
```

A junior developer says:

> Yes.

A software architect says:

> **No. Everything is a node.**

That is the **Composite Pattern**.

---

# The Architect's Way of Thinking

Suppose we have a file explorer.

```text
Folder

↓

Documents

↓

Resume.pdf

↓

Notes.txt
```

Question:

Should the application use different APIs?

```cpp
openFile()

openFolder()

openDirectory()

displayFolder()

displayFile()
```

No.

Architects prefer:

```cpp
node->display();
```

Whether it is a file or a folder, the client uses the same interface.

---

# 1. Introduction

## What is the Composite Pattern?

The **Composite Pattern** composes objects into **tree structures** to represent **part-whole hierarchies**.

It allows clients to treat:

* individual objects (**Leaf**)
* groups of objects (**Composite**)

uniformly.

---

## Simple Definition

> **The Composite Pattern allows individual objects and collections of objects to be treated the same way.**

---

## Category

**Structural Pattern**

---

## Why was it created?

Large software systems naturally organize data as trees.

Examples:

* File Systems
* Organization Charts
* CAD Assemblies
* Scene Graphs
* HTML DOM
* XML
* Medical Structure Hierarchies

Without Composite, every recursive operation becomes filled with type checks.

---

## What problem does it solve?

Without Composite:

```cpp
if(node is File)
{
    ...
}
else if(node is Folder)
{
    ...
}
```

Repeated everywhere.

With Composite:

```cpp
node->display();
```

No type checking.

No duplicated logic.

---

# 2. Problem Statement

Imagine a **Hospital ERP**.

Hierarchy:

```text
Hospital

↓

Departments

↓

Doctors

↓

Patients
```

Now suppose HR asks:

> "Print the hierarchy."

Without Composite:

```cpp
printHospital()

printDepartment()

printDoctor()

printPatient()
```

Four different APIs.

Tomorrow management adds:

```text
Research Center
```

Now more functions.

Then:

```text
Medical College
```

Even more.

Your code becomes increasingly difficult to maintain.

---

# 3. Motivation

Developers noticed a common pattern:

Many systems contain **hierarchical relationships**.

Examples:

```
Folder
    Folder
        Folder
            File
```

or

```
Company
    Department
        Team
            Employee
```

Instead of writing different code for every level, represent everything through one common interface.

---

# 4. Real-World Analogy

## Company Organization Chart

```text
CEO

↓

Engineering

↓

Software Team

↓

Developer
```

The CEO manages departments.

Departments manage teams.

Teams manage developers.

Question:

How do we print the organization?

We don't care whether the node is:

* CEO
* Department
* Team
* Developer

Every node responds to:

```text
display()
```

---

## Mapping

| Company      | Software       |
| ------------ | -------------- |
| CEO          | Root Composite |
| Department   | Composite      |
| Team         | Composite      |
| Employee     | Leaf           |
| Organization | Tree           |

---

# 5. Software Scenario

Composite is useful whenever objects naturally form a tree.

### Desktop Applications

* File Explorer
* Menu Systems
* Tree Views

---

### Qt Applications

Qt uses hierarchical structures extensively.

Examples:

* `QTreeWidget`
* `QTreeView`
* `QStandardItemModel`
* `QObject` parent-child ownership hierarchy
* Graphics scene graphs built from nested items

Each item may itself contain child items.

---

### CAD Software

Assemblies:

```text
Car

↓

Engine

↓

Piston

↓

Valve
```

---

### Medical Imaging

Patient

↓

Study

↓

Series

↓

Image

This is the DICOM hierarchy.

---

### Game Engines

Scene Graph

↓

Player

↓

Weapon

↓

Scope

↓

Crosshair

---

### HTML

```html
<html>

<body>

<div>

<button>

</button>

</div>

</body>

</html>
```

Every element is a node.

---

# 6. UML Class Diagram

```text
                   +----------------------+
                   |      Component       |
                   +----------------------+
                   | +operation()         |
                   | +add()               |
                   | +remove()            |
                   | +getChild()          |
                   +----------+-----------+
                              ^
                --------------|--------------
                |                             |
       +----------------+           +----------------+
       |      Leaf      |           |   Composite    |
       +----------------+           +----------------+
       | operation()    |           | operation()    |
       |                |           | add()          |
       |                |           | remove()       |
       +----------------+           +----------------+
                                           |
                                 contains many
                                           |
                                   +--------------+
                                   | Component    |
                                   +--------------+
```

---

## Responsibilities

### Component

Common interface.

Defines operations shared by both leaves and composites.

---

### Leaf

Represents an individual object.

Cannot contain children.

---

### Composite

Represents a group.

Contains children.

Delegates operations to its children.

---

### Client

Works only with `Component`.

Never asks:

```cpp
isLeaf()
```

or

```cpp
isComposite()
```

unless absolutely necessary.

---

# 7. Participants

## Component

Example:

```text
FileSystemNode
```

Methods:

```cpp
display()

size()
```

---

## Leaf

Example:

```text
File
```

No children.

---

## Composite

Example:

```text
Folder
```

Contains:

```text
vector<Component*>
```

---

## Client

Uses:

```cpp
node->display();
```

without knowing whether it is a file or folder.

---

# 8. Collaboration

Runtime Flow

```text
Client

↓

Root Folder

↓

display()

↓

Folder displays itself

↓

Calls display()

↓

Child Folder

↓

Calls display()

↓

Files

↓

Recursion Ends
```

Notice:

Each composite delegates work to its children.

This is recursive.

---

# 9. C++ Example

Let's implement a simple file system.

```cpp
#include <iostream>
#include <memory>
#include <string>
#include <vector>

// Component
class FileSystemNode
{
public:
    virtual ~FileSystemNode() = default;
    virtual void display(int indent = 0) const = 0;
};

// Leaf
class File : public FileSystemNode
{
public:
    explicit File(std::string name)
        : m_name(std::move(name))
    {}

    void display(int indent = 0) const override
    {
        std::cout << std::string(indent, ' ')
                  << "- File: " << m_name << '\n';
    }

private:
    std::string m_name;
};

// Composite
class Folder : public FileSystemNode
{
public:
    explicit Folder(std::string name)
        : m_name(std::move(name))
    {}

    void add(std::unique_ptr<FileSystemNode> node)
    {
        m_children.push_back(std::move(node));
    }

    void display(int indent = 0) const override
    {
        std::cout << std::string(indent, ' ')
                  << "+ Folder: " << m_name << '\n';

        for (const auto& child : m_children)
        {
            child->display(indent + 4);
        }
    }

private:
    std::string m_name;
    std::vector<std::unique_ptr<FileSystemNode>> m_children;
};

int main()
{
    auto root = std::make_unique<Folder>("Documents");

    root->add(std::make_unique<File>("Resume.pdf"));
    root->add(std::make_unique<File>("Notes.txt"));

    root->display();
}
```

---

## Design Focus

Notice:

The client only knows:

```cpp
FileSystemNode
```

It doesn't care whether it receives:

* File
* Folder

This is the key idea of Composite.

---

# 10. Qt Example

Suppose you're building a **College ERP** desktop application in Qt.

Hierarchy:

```text
College

↓

Departments

↓

Courses

↓

Students
```

Represent every node as:

```cpp
TreeNode
```

Display it using:

* `QTreeView`
* `QStandardItemModel`

Your UI code simply expands or collapses nodes.

It doesn't need different logic for departments versus courses.

Another Qt example is the `QObject` hierarchy:

```text
MainWindow

↓

CentralWidget

↓

Buttons

↓

Labels
```

Destroying the parent automatically destroys its children because of the parent-child tree relationship.

> **Important:** `QObject` is **not** a textbook Composite implementation because not every child is treated identically for all operations, but its hierarchical ownership model reflects many Composite ideas.

---

# 11. Medical Software Example

One of the best real-world examples is the **DICOM Patient Hierarchy**.

```text
Patient

↓

Study

↓

Series

↓

Image
```

Another example inside a TPS:

```text
Treatment Plan

↓

Beam Group

↓

Beam

↓

Control Point
```

Each level contains children.

Suppose you want to calculate total monitor units (MU).

Without Composite:

```cpp
calculatePlan()

calculateBeam()

calculateControlPoint()
```

Different functions.

With Composite:

```cpp
node->calculateMU();
```

Each composite sums the results of its children.

Example:

```text
Treatment Plan

↓

Beam Group

↓

Beam 1

↓

MU = 150

+

Beam 2

↓

MU = 200

↓

Total = 350
```

The recursive structure naturally mirrors the domain model.

---

# 12. Advantages

### Uniformity

Treat leaves and composites the same.

### Simplicity

Clients work with one interface.

### Scalability

Easy to add new node types.

### Extensibility

New composite or leaf types rarely require changes to client code.

### Recursive Operations

Tree traversal becomes elegant and reusable.

---

# 13. Disadvantages

### Interface Trade-Off

Should `add()` exist on leaves?

There are two common approaches:

* **Transparent Composite:** `Component` defines `add()`; leaves reject it.
* **Safe Composite:** Only `Composite` defines `add()`; clients know they need a composite to add children.

We'll discuss these shortly.

### Hard to Restrict Trees

It can be difficult to prevent invalid parent-child relationships if the model is too generic.

### Overkill

Don't use Composite if your data is not hierarchical.

---

# 14. Best Practices

* Use Composite only for true tree structures.
* Program against the `Component` interface.
* Keep recursive operations inside the composite.
* Prefer smart pointers for ownership in modern C++.
* Clearly define ownership rules for child nodes.

---

# 15. Common Mistakes

### Mistake 1

Using Composite for flat collections.

A `std::vector` is enough if there is no hierarchy.

---

### Mistake 2

Adding business logic to traversal code.

Traversal belongs in the composite or in dedicated visitors, not scattered throughout the application.

---

### Mistake 3

Allowing cycles.

Example:

```text
Folder A

↓

Folder B

↓

Folder A
```

This causes infinite recursion.

Architects ensure the structure remains a tree (or intentionally design a graph with different algorithms).

---

### Mistake 4

Using runtime type checks everywhere.

If you constantly write:

```cpp
dynamic_cast<Folder*>(node)
```

you are probably not taking full advantage of the pattern.

---

# 16. Pattern Variations

## 1. Transparent Composite

The `Component` interface includes:

```cpp
add()

remove()

getChild()
```

Leaves either ignore these operations or throw an error.

### Advantage

Uniform interface.

### Disadvantage

Leaves expose methods they cannot meaningfully support.

---

## 2. Safe Composite

Only `Composite` defines:

```cpp
add()

remove()
```

Leaves remain simple.

### Advantage

Cleaner interfaces.

### Disadvantage

Clients sometimes need to know they are working with a composite when building the tree.

---

## 3. Immutable Composite

Once built, the hierarchy cannot change.

Useful for:

* Medical plans after approval
* XML document snapshots
* CAD release versions

---

# 17. Related Patterns

| Pattern   | Difference                                                                       |
| --------- | -------------------------------------------------------------------------------- |
| Composite | Represents tree structures using a common interface.                             |
| Decorator | Wraps a single object to add behavior.                                           |
| Flyweight | Shares state between many objects to reduce memory.                              |
| Iterator  | Traverses a composite tree without exposing its internal structure.              |
| Visitor   | Performs operations across a composite hierarchy without modifying node classes. |

## Composite vs Decorator

This is a classic interview question.

### Composite

Represents:

```text
Folder

↓

Files
```

One object **contains many**.

### Decorator

Represents:

```text
Coffee

↓

Milk

↓

Sugar

↓

Whipped Cream
```

One object is **wrapped by another**.

Composite is about **hierarchies**.

Decorator is about **adding responsibilities**.

---

# 18. Industry Usage

Composite is one of the most frequently used patterns because hierarchical data is everywhere.

* **Microsoft:** Windows File Explorer, Office document object models, and UI trees.
* **Google:** DOM trees, Android view hierarchies, and file systems.
* **Adobe:** Photoshop layer groups, Illustrator object hierarchies, and document structures.
* **Qt:** `QTreeView`, `QStandardItemModel`, parent-child object hierarchies, and graphics item trees.
* **ZEISS / Siemens / Philips:** DICOM hierarchies, treatment plans, beam trees, anatomical structures, and workflow models.
* **Autodesk:** CAD assemblies, parts, subassemblies, and scene graphs.
* **Game Engines:** Scene graphs, entity hierarchies, skeletal bone trees, and UI widget trees.

The architectural goal is **treating individual objects and object groups uniformly**.

---

# 19. Interview Questions

## Beginner

1. What problem does the Composite Pattern solve?
2. What is the difference between a Leaf and a Composite?
3. Why is Composite considered a Structural Pattern?

---

## Intermediate

1. Explain transparent vs safe composite.
2. Why is recursion natural in Composite?
3. How does Composite reduce client complexity?

---

## Advanced

1. How would you prevent cycles in a Composite tree?
2. How would you implement lazy loading of children in a large DICOM hierarchy?
3. How would Composite work with Visitor to perform operations without changing node classes?

---

## Scenario-Based

Your DICOM Viewer displays:

* Patients
* Studies
* Series
* Images

Each level can contain children except Images.

Would you use Composite? Explain how your design simplifies expanding, collapsing, and traversing the hierarchy.

---

## Architecture

Design a Composite architecture for a **Treatment Planning System** representing:

```text
Treatment Plan
    ├── Beam Groups
    │     ├── Beams
    │     │     ├── Control Points
    ├── Structures
    │     ├── PTV
    │     ├── Heart
    │     ├── Lung
    ├── Dose Objectives
```

Explain:

* Which classes are Components?
* Which are Leaves?
* Which are Composites?
* How recursive operations such as validation or dose statistics would work.

---

# 20. Practice Exercises

### Beginner Exercise

Design a file system using Composite with:

* Folder
* File

Implement a common interface that allows displaying the hierarchy recursively.

**Do not write code yet. Draw the class diagram first.**

---

### Intermediate Exercise

Design a Qt-based menu system where:

* Menu
* Submenu
* Menu Item

all inherit from a common component.

Explain how rendering and enabling/disabling menu items can be implemented recursively.

---

### Advanced Exercise

Design a **Composite architecture** for a **Medical Imaging Platform** representing the DICOM hierarchy:

```text
Patient
    ├── Study
    │     ├── Series
    │     │     ├── Image
```

Requirements:

* Display the hierarchy in a `QTreeView`.
* Compute the total image count recursively.
* Support lazy loading of Series and Images.
* Prevent invalid parent-child relationships.
* Explain ownership and memory management using modern C++ smart pointers.

**Do not implement the solution yet.** Focus on architecture and object responsibilities.

---

# Key Architectural Takeaway

The **Composite Pattern** is not about trees for their own sake.

It is about **treating a single object and a group of objects through the same interface**.

A junior developer thinks:

> "A folder and a file are different, so I need different code for each."

A software architect thinks:

> **"Both are nodes in the same hierarchy. I'll give them a common interface so recursive operations become simple, extensible, and consistent."**

This mindset is fundamental in systems such as file explorers, CAD assemblies, scene graphs, DICOM hierarchies, Qt model/view trees, and treatment planning systems.

---

## What You'll Learn Next

Type **`NEXT`** to continue with **Lesson 10: Decorator Pattern**, one of the most commonly used patterns in Qt, GUI frameworks, stream libraries, and enterprise software.
