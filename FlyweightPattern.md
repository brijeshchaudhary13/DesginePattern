

# Lesson 12 — Flyweight Pattern 
---

# Before We Start

The **Flyweight Pattern** is one of the most misunderstood patterns because its purpose is **not** to simplify code.

Its purpose is:

> **Reduce memory usage by sharing common data.**

This is a **performance optimization pattern**.

It is heavily used in:

* Qt
* Browsers
* Game Engines
* CAD Software
* Medical Imaging
* GIS Systems (Google Maps)
* Text Editors

---

# The Architect's Way of Thinking

Imagine you are developing a TPS.

The patient CT contains:

```text
512 × 512 × 400

=

104,857,600 voxels
```

Suppose each voxel stores:

```text
Position
Color
Material
Opacity
Dose
```

Memory becomes enormous.

Question:

Do all voxels need their own copy of:

```text
Color Table
```

No.

The **Color Table** is identical for all voxels.

An architect asks:

> **"Which data is unique, and which data can be shared?"**

That question leads directly to the Flyweight Pattern.

---

# Memory Problem

Imagine creating:

```text
1,000,000 Trees
```

Each tree stores:

```text
Species Name
Texture
Leaf Texture
Branch Texture
Height
Position
Rotation
```

Suppose:

```text
Texture = 10 MB
```

Memory:

```text
10 MB × 1,000,000

=

10,000,000 MB
```

Impossible.

But wait.

All Oak trees use the same texture.

So instead:

```text
Oak Texture

↓

Shared

↓

1,000,000 Trees
```

Now only one texture exists.

---

# 1. Introduction

## What is the Flyweight Pattern?

The **Flyweight Pattern** shares common object state among many objects to reduce memory consumption.

Simple definition:

> **Store shared data once and reuse it across many objects.**

---

## Why was it created?

Many software systems create:

* millions of objects
* billions of pixels
* millions of particles
* thousands of CAD entities

Often:

Most object data is identical.

Duplicating it wastes memory.

---

## Category

**Structural Pattern**

---

## What problem does it solve?

Instead of:

```text
Object A

↓

Texture
```

```text
Object B

↓

Texture
```

```text
Object C

↓

Texture
```

Every object stores the same texture.

Flyweight:

```text
Shared Texture

↓

Object A

↓

Object B

↓

Object C
```

Only one copy exists.

---

# 2. Problem Statement

Imagine a **Google Maps**-style application.

There are:

```text
1,000,000 Map Pins
```

Each stores:

```text
Hospital Icon

Hospital Color

Hospital Font

Hospital Symbol
```

Every hospital pin has identical icon data.

Only these differ:

```text
Latitude

Longitude

Hospital Name
```

Without Flyweight:

Duplicate icon 1,000,000 times.

Huge waste.

---

# 3. Motivation

Architects noticed that objects contain two kinds of state.

## Intrinsic State

Shared.

Never changes between objects.

Example:

```text
Hospital Icon

Tree Texture

Character Font

DICOM Color LUT
```

---

## Extrinsic State

Unique.

Changes per object.

Example:

```text
Position

Rotation

Patient Name

Beam Angle

Voxel Coordinate
```

Flyweight stores:

Intrinsic state once.

Extrinsic state separately.

---

# 4. Real-World Analogy

## Library Books

A university has:

```text
100 students
```

Need the same textbook.

Does each student buy:

```text
Algorithms Book
```

No.

Library stores:

```text
One Book

↓

Many Students Read It
```

The book is shared.

Each student stores only:

```text
Borrow Date

Return Date
```

---

## Mapping

| Library     | Software          |
| ----------- | ----------------- |
| Book        | Flyweight         |
| Student     | Client            |
| Library     | Flyweight Factory |
| Borrow Info | Extrinsic State   |

---

# 5. Software Scenario

Flyweight is useful when many objects share identical information.

### Desktop Applications

* Font glyphs
* Icons
* Images

---

### Qt Applications

Qt internally shares data in several classes through **implicit sharing (copy-on-write)**, such as:

* `QString`
* `QImage`
* `QPixmap`
* `QByteArray`

These are not textbook GoF Flyweight implementations, but they apply the same architectural principle: **share data until modification is required**.

---

### CAD Software

Millions of bolts.

Every bolt:

Same geometry.

Different position.

---

### Medical Imaging

Millions of voxels.

Same color lookup table.

Different coordinates.

---

### Game Engines

Thousands of trees.

Same mesh.

Different position.

---

### Browsers

Thousands of letters.

Same font glyph.

Different location.

---

# 6. UML Class Diagram

```text
                 +----------------------+
                 |      Flyweight       |
                 +----------------------+
                 | operation(state)     |
                 +----------+-----------+
                            ^
                ------------|------------
                |                       |
       +----------------+     +----------------+
       | ConcreteFlyweight |   | ConcreteFlyweight |
       +----------------+     +----------------+

                 +----------------------+
                 |  FlyweightFactory    |
                 +----------------------+
                 | getFlyweight()       |
                 +----------+-----------+
                            |
                     shares objects
                            |
                 +----------------------+
                 |       Client         |
                 +----------------------+
```

---

## Responsibilities

### Flyweight

Stores intrinsic state.

---

### Concrete Flyweight

Actual shared object.

---

### Flyweight Factory

Creates flyweights.

Returns existing ones when possible.

---

### Client

Stores extrinsic state.

---

# 7. Participants

## Flyweight

Example:

```text
TreeModel
```

Contains:

```text
Texture

Mesh

Material
```

---

## Flyweight Factory

Keeps:

```cpp
std::unordered_map<std::string,
                   std::shared_ptr<TreeModel>>
```

---

## Client

Stores:

```text
Position

Rotation

Scale
```

---

# 8. Collaboration

Runtime Flow

```text
Client

↓

Request Tree

↓

Factory

↓

Tree Already Exists?

↓

Yes

↓

Return Shared Tree

↓

Client Stores Position
```

Notice:

The shared object is reused.

---

# 9. C++ Example

```cpp
#include <iostream>
#include <memory>
#include <unordered_map>

// Flyweight
class TreeType
{
public:
    explicit TreeType(std::string name)
        : m_name(std::move(name))
    {}

    void draw(int x, int y) const
    {
        std::cout << "Drawing "
                  << m_name
                  << " at "
                  << x << "," << y
                  << '\n';
    }

private:
    std::string m_name;
};

// Factory
class TreeFactory
{
public:
    std::shared_ptr<TreeType>
    getTree(const std::string& name)
    {
        auto it = trees.find(name);

        if (it != trees.end())
            return it->second;

        auto tree =
            std::make_shared<TreeType>(name);

        trees[name] = tree;

        return tree;
    }

private:
    std::unordered_map<
        std::string,
        std::shared_ptr<TreeType>> trees;
};

int main()
{
    TreeFactory factory;

    auto oak = factory.getTree("Oak");

    oak->draw(10,20);

    auto oak2 = factory.getTree("Oak");

    oak2->draw(30,40);
}
```

---

## Design Focus

Notice:

Only one:

```text
Oak
```

exists.

Different trees have different:

```text
Position
```

This is Flyweight.

---

# 10. Qt Example

Suppose your Qt application displays:

```text
100,000 Hospital Icons
```

Instead of:

```text
Each Marker

↓

Own QPixmap
```

Use:

```text
Shared QPixmap

↓

Marker 1

Marker 2

Marker 3
```

Each marker stores only:

```text
Position

Hospital Name
```

The image resource is shared.

Qt's implicit sharing further reduces copying costs for many value classes.

---

# 11. Medical Software Example

Imagine a **Dose Viewer**.

You have:

```text
100 Million Voxels
```

Every voxel uses:

```text
Same Dose Color LUT
```

Without Flyweight:

```text
Voxel

↓

Own LUT
```

Memory explodes.

Instead:

```text
Shared LUT

↓

Voxel 1

Voxel 2

Voxel 3
```

Each voxel stores only:

```text
Dose Value

Coordinate
```

Similarly:

A TPS may display:

```text
Thousands of Beams
```

Each beam uses:

Same machine model.

Different:

* Gantry Angle
* Collimator
* MU

Machine configuration can be shared.

---

# 12. Advantages

### Huge Memory Savings

Primary advantage.

### Better Cache Performance

Shared objects improve CPU cache locality.

### Faster Object Creation

Reuse instead of repeatedly allocating identical data.

### Scalability

Supports millions of lightweight objects.

---

# 13. Disadvantages

### Complexity

Requires separating intrinsic and extrinsic state.

### Thread Safety

Shared flyweights may require synchronization if mutable.

### More Indirection

Objects depend on a factory and shared instances.

### When NOT to Use

Avoid Flyweight when:

* objects are few,
* shared data is small,
* intrinsic/extrinsic separation is unclear.

---

# 14. Best Practices

* Make flyweights immutable whenever possible.
* Store only intrinsic state in flyweights.
* Keep extrinsic state in clients.
* Use a factory to manage flyweight lifetime.
* Measure memory usage before and after applying the pattern.

---

# 15. Common Mistakes

### Mistake 1

Putting unique state into the flyweight.

Example:

```text
Patient Name
```

This should not be shared.

---

### Mistake 2

Making flyweights mutable.

If one client changes shared data, all clients are affected.

---

### Mistake 3

Using Flyweight when memory is not actually a problem.

Every optimization has a cost.

---

### Mistake 4

Ignoring ownership.

Factories should clearly own shared flyweights.

---

# 16. Pattern Variations

## 1. Immutable Flyweight

Most common.

Shared object never changes.

---

## 2. Cached Flyweight

Factory caches objects.

Creates only when needed.

---

## 3. Copy-on-Write

Initially shared.

Copied only when modified.

Qt uses this technique extensively.

---

# 17. Related Patterns

| Pattern     | Difference                            |
| ----------- | ------------------------------------- |
| Flyweight   | Shares object state to reduce memory. |
| Object Pool | Reuses expensive objects over time.   |
| Singleton   | One shared instance.                  |
| Prototype   | Creates objects by copying.           |
| Composite   | Represents hierarchical trees.        |

---

## Flyweight vs Object Pool (Very Important)

Many interviewers ask this.

### Object Pool

Purpose:

```text
Reuse objects

↓

Over Time
```

Example:

Database connections.

One client uses it.

Returns it.

Another client uses it.

---

### Flyweight

Purpose:

```text
Share object

↓

Simultaneously
```

Example:

One font.

Thousands of labels use it at the same time.

---

# 18. Industry Usage

Flyweight is widely used in systems where memory efficiency is critical.

* **Microsoft:** Font rendering, icons, and graphics resources.
* **Google:** Maps markers, browser rendering, and document editing.
* **Adobe:** Shared fonts, brushes, and image resources.
* **Qt:** Implicit sharing in value classes (`QString`, `QImage`, `QPixmap`) reflects Flyweight principles.
* **ZEISS / Siemens / Philips:** Color lookup tables, machine models, calibration data, and rendering resources shared across viewers.
* **Autodesk:** Shared CAD geometry templates, materials, and symbol libraries.
* **Game Engines:** Tree meshes, textures, particle meshes, and character models.

The architectural goal is **sharing immutable common state across many logical objects**.

---

# 19. Interview Questions

## Beginner

1. What problem does the Flyweight Pattern solve?
2. What is intrinsic state?
3. What is extrinsic state?

---

## Intermediate

1. Why is immutability important in Flyweight?
2. How does Flyweight reduce memory usage?
3. What is the role of the Flyweight Factory?

---

## Advanced

1. How would you design a Flyweight system for a CAD application with one million bolts?
2. How would you make a Flyweight Factory thread-safe?
3. When would you choose Flyweight over Object Pool?

---

## Scenario-Based

Your TPS visualizes 100 million voxels. Each voxel uses the same color lookup table and rendering settings.

Would you use Flyweight? Identify which state is intrinsic and which is extrinsic.

---

## Architecture

Design a **Flyweight architecture** for a **Medical Imaging Platform** displaying:

* CT Voxels
* MRI Voxels
* PET Voxels

Shared resources:

* Color LUT
* Rendering Parameters
* Machine Calibration

Unique per voxel:

* Coordinates
* Dose Value
* Hounsfield Unit

Explain:

* how the Flyweight Factory manages shared resources,
* ownership,
* thread safety,
* and why immutable flyweights are beneficial.

---

# 20. Practice Exercises

### Beginner Exercise

Design a text editor where thousands of characters share the same font information. Identify intrinsic and extrinsic state.

---

### Intermediate Exercise

Design a Qt map application displaying 500,000 location markers. Explain how marker icons can be shared while each marker stores its own position and metadata.

---

### Advanced Exercise

Design a **Flyweight architecture** for a **Treatment Planning System (TPS)** visualizing a 3D dose distribution containing over **100 million voxels**.

Requirements:

* Share the dose color lookup table.
* Share rendering configuration.
* Store voxel-specific coordinates and dose values separately.
* Support multithreaded rendering safely.
* Explain memory ownership and lifecycle.

**Do not implement the solution yet.** Focus on architecture, intrinsic vs. extrinsic state, and scalability.

---

# Key Architectural Takeaway

The **Flyweight Pattern** is about **sharing immutable common state to dramatically reduce memory consumption**.

A junior developer thinks:

> "Every object should store all of its own data."

A software architect thinks:

> **"Most of this data is identical across millions of objects. I'll store the shared state once and let every object reference it, keeping only the truly unique state locally."**

This mindset is essential for high-performance systems such as CAD software, game engines, browsers, GIS platforms, and medical imaging applications that manage millions of objects efficiently.

---
[⬅️ Facade Pattern](/FacadePattern.md)        |  [Proxy Pattern  ➡️](/ProxyPattern.md ) 
---
## **License**
This project is licensed under the MIT License.

---

Happy Coding!

