# Design Patterns Master Course

# Lesson 5 — Prototype Pattern (Creational Pattern)

---

# Before We Start

The **Prototype Pattern** is one of the least understood design patterns.

Many developers read its definition:

> "Create new objects by copying existing objects."

Then they think:

> "Why don't I just use the copy constructor?"

A software architect asks a different question:

> **"Why am I copying this object instead of constructing a new one?"**

The answer is the Prototype Pattern.

---

# Understanding the Journey So Far

Let's review the previous patterns:

| Pattern          | Main Question                                         |
| ---------------- | ----------------------------------------------------- |
| Singleton        | How many objects should exist?                        |
| Factory Method   | Which object should be created?                       |
| Abstract Factory | Which family of objects should be created?            |
| Builder          | How should a complex object be constructed?           |
| Prototype        | Can I create a new object by copying an existing one? |

Notice how each pattern solves a different architectural problem.

---

# 1. Introduction

## What is the Prototype Pattern?

The **Prototype Pattern** creates new objects by **cloning an existing object**, called the **prototype**, instead of constructing a new object from scratch.

Instead of:

```text
New Object

↓

Initialize

↓

Configure

↓

Validate

↓

Ready
```

We do:

```text
Existing Object

↓

Clone

↓

Modify Required Fields

↓

Done
```

---

## Why was it created?

Some objects are:

* expensive to create,
* complex to configure,
* loaded from external resources,
* contain many default values.

Recreating them every time wastes time and increases complexity.

Instead, create one well-configured object and clone it.

---

## Category

**Creational Pattern**

---

## What problem does it solve?

It solves the problem of **repeatedly creating similar objects** whose construction is expensive or complicated.

---

# 2. Problem Statement

Imagine a **Medical Imaging Viewer**.

Every CT View contains:

* Window Width
* Window Level
* Zoom Settings
* Annotation Style
* Color Map
* Grid Settings
* Measurement Tools
* Layout
* GPU Configuration

Without Prototype:

```cpp
CTView view;

view.setWindowWidth(...);
view.setWindowLevel(...);
view.setZoom(...);
view.setGrid(...);
view.setColorMap(...);
view.setMeasurement(...);
view.setAnnotation(...);
...
```

Now imagine you need **100 CT views**.

You repeat all this configuration 100 times.

This is error-prone and inefficient.

---

Another example:

A TPS contains a beam configuration with:

* Energy
* MLC
* Wedge
* Collimator
* Gantry
* Couch
* SSD
* Dose Rate

Creating each beam from scratch becomes repetitive.

Often, only one or two parameters differ.

---

# 3. Motivation

Developers observed that many new objects are almost identical to existing ones.

Examples:

* Duplicate a slide in PowerPoint.
* Duplicate a layer in Photoshop.
* Copy a CAD object.
* Duplicate a treatment beam in a TPS.

Rather than reconstructing everything, clone the existing object and change only what is different.

The Prototype Pattern formalizes this idea.

---

# 4. Real-World Analogy

## Photocopy Machine

Imagine you have a completed document.

Need another copy?

Do you rewrite the document?

No.

You place it in a photocopier.

```text
Original Document

↓

Photocopy

↓

New Document
```

If needed, you write a different name on the copied version.

### Mapping

| Real World        | Software   |
| ----------------- | ---------- |
| Original Document | Prototype  |
| Photocopier       | clone()    |
| Copy              | New Object |
| Person            | Client     |

The original stays unchanged.

The copy becomes independent.

---

# 5. Software Scenario

Prototype appears whenever duplication is more practical than reconstruction.

### Desktop Applications

* Copy/Paste objects
* Duplicate documents
* Duplicate diagrams

### Qt Applications

* Duplicate graphics items in a scene
* Copy drawing objects
* Duplicate editor configurations

### CAD Software

* Copy geometric models
* Duplicate assemblies
* Copy constraints

### Medical Imaging

* Duplicate image processing pipelines
* Copy viewer settings
* Duplicate treatment beams
* Clone contour templates

### Game Engines

* Duplicate enemies
* Clone NPC templates
* Copy particle emitters

### IDEs

* Duplicate project configurations
* Copy code templates

---

# 6. UML Class Diagram

```text
                 +----------------------+
                 |      Prototype       |
                 +----------------------+
                 | +clone()             |
                 +----------+-----------+
                            ^
                ------------|------------
                |                       |
     +-------------------+   +-------------------+
     | ConcretePrototype |   | ConcretePrototype |
     |        A          |   |        B          |
     +-------------------+   +-------------------+
     | clone()           |   | clone()           |
     +-------------------+   +-------------------+

                 +----------------------+
                 |       Client         |
                 +----------------------+
                 | prototype.clone()    |
                 +----------------------+
```

---

## Responsibilities

### Prototype

Declares the cloning interface.

---

### Concrete Prototype

Implements cloning.

Knows how to copy itself.

---

### Client

Requests copies.

The client does not know the internal details of copying.

---

# 7. Participants

## Prototype

Defines:

```text
clone()
```

---

## Concrete Prototype

Example:

```text
CTView

Beam

MRIViewport

PatientTemplate
```

Implements the cloning process.

---

## Client

Calls:

```cpp
prototype->clone();
```

The client does not call constructors directly.

---

# 8. Collaboration

Runtime Flow

```text
Client

↓

Prototype Exists

↓

clone()

↓

New Object Created

↓

Modify Required Fields

↓

Use Object
```

Example:

```text
Beam 1

↓

Clone

↓

Beam 2

↓

Change Gantry Angle
```

---

# 9. C++ Example

Let's implement a simple Prototype.

```cpp
#include <iostream>
#include <memory>
#include <string>

class Shape
{
public:
    virtual ~Shape() = default;

    virtual std::unique_ptr<Shape> clone() const = 0;

    virtual void draw() const = 0;
};

class Circle : public Shape
{
public:
    Circle(int radius, std::string color)
        : m_radius(radius), m_color(std::move(color))
    {}

    std::unique_ptr<Shape> clone() const override
    {
        return std::make_unique<Circle>(*this);
    }

    void draw() const override
    {
        std::cout << "Circle Radius=" << m_radius
                  << " Color=" << m_color << '\n';
    }

private:
    int m_radius;
    std::string m_color;
};

int main()
{
    Circle original(50, "Red");

    auto copy = original.clone();

    original.draw();
    copy->draw();
}
```

### Design Focus

Notice:

The client never calls:

```cpp
new Circle(...)
```

for the copy.

Instead:

```cpp
clone()
```

creates another object with the same state.

---

# 10. Qt Example

Imagine a vector drawing application built with **QGraphicsScene**.

The user selects a rectangle and clicks **Duplicate**.

Without Prototype:

```text
Read Width

Read Height

Read Color

Read Border

Read Rotation

Read Opacity

Create New Item

Set Every Property
```

With Prototype:

```text
Selected QGraphicsItem

↓

clone()

↓

Move Slightly

↓

Done
```

Every duplicated item inherits the original's visual properties, making the duplication feature simple and consistent.

---

# 11. Medical Software Example

Let's consider a **Treatment Planning System (TPS)**.

A physicist creates **Beam 1**.

Configuration:

```text
Energy = 6 MV

Collimator = 30°

MLC Pattern

Dose Rate = 600 MU/min

SSD = 100 cm

Wedge = Off
```

Now they need **Beam 2**.

Only one thing changes:

```text
Gantry

0°

↓

90°
```

Without Prototype:

Reconfigure every field again.

With Prototype:

```text
Beam 1

↓

clone()

↓

Beam 2

↓

Change Gantry = 90°
```

Everything else stays identical.

This reduces repetitive work and lowers the risk of accidental configuration differences.

Other medical examples include:

* Cloning image processing pipelines.
* Duplicating contour templates.
* Copying viewer presets.
* Creating protocol templates.

---

# 12. Advantages

### Performance

Avoids repeating expensive initialization.

### Simplicity

Copying is easier than rebuilding.

### Reusability

One configured object can produce many similar objects.

### Flexibility

Clients don't need to know construction details.

### Reduced Configuration Errors

Defaults remain consistent across copies.

---

# 13. Disadvantages

### Deep vs Shallow Copy

This is the biggest challenge.

### Complex Object Graphs

Objects containing pointers or shared resources require careful copying.

### Hidden Copy Costs

Copying very large objects may be expensive.

### When NOT to Use

Avoid Prototype when:

* construction is cheap,
* objects are simple,
* copying semantics are unclear.

---

# 14. Best Practices

* Clearly define whether cloning performs a deep copy or shallow copy.
* Ensure cloned objects are independent when required.
* Document ownership of dynamically allocated resources.
* Prefer smart pointers and value semantics where possible.
* Validate cloned state before exposing it to clients if necessary.

---

# 15. Common Mistakes

### Mistake 1

Using shallow copy when deep copy is required.

Example:

```text
Original

↓

Pointer

↓

Shared Memory

↓

Clone
```

Now both objects modify the same data unintentionally.

---

### Mistake 2

Forgetting to copy newly added member variables.

The clone becomes incomplete.

---

### Mistake 3

Sharing mutable state accidentally.

Architects explicitly decide which state should be shared and which should be copied.

---

### Mistake 4

Using Prototype for every object.

Only objects that benefit from duplication should implement it.

---

# 16. Pattern Variations

### 1. Shallow Prototype

Copies member values only.

Suitable when members are value types or intentionally shared.

---

### 2. Deep Prototype

Recursively copies owned objects.

Preferred when independence is required.

---

### 3. Prototype Registry

Store reusable prototypes in a registry.

Example:

```text
Prototype Registry

↓

Default CT View

↓

Default MRI View

↓

Default PET View
```

Clients request a prototype by name and receive a clone.

---

### 4. Copy-on-Write

Initially share data between copies.

Create independent data only when a modification occurs.

This technique can improve performance but adds implementation complexity.

---

# 17. Related Patterns

| Pattern          | Difference                                                           |
| ---------------- | -------------------------------------------------------------------- |
| Factory Method   | Creates new objects through a factory.                               |
| Abstract Factory | Creates families of related objects.                                 |
| Builder          | Builds objects step by step.                                         |
| Prototype        | Creates objects by cloning existing ones.                            |
| Flyweight        | Shares intrinsic state between many objects instead of copying them. |

A useful memory trick:

* **Factory** → Create.
* **Builder** → Assemble.
* **Prototype** → Copy.

---

# 18. Industry Usage

Prototype is common wherever duplication is a core user workflow.

* **Microsoft:** Duplicating slides, shapes, and document templates.
* **Google:** Configuration templates and editor duplication.
* **Adobe:** Layers, artboards, brushes, and graphic objects.
* **Qt:** Graphics editors and scene duplication often implement clone-like behavior for custom items.
* **ZEISS / Siemens / Philips:** Viewer presets, imaging protocols, beam templates, and examination configurations.
* **Autodesk:** CAD object duplication and assembly copying.
* **Game Engines:** Prefabs, entity templates, and particle system duplication.

The architectural goal is **fast, consistent duplication without repeating complex construction logic**.

---

# 19. Interview Questions

## Beginner

1. What is the Prototype Pattern?
2. Why would you clone an object instead of creating a new one?
3. What is the purpose of the `clone()` method?

---

## Intermediate

1. What is the difference between shallow copy and deep copy?
2. When is Prototype better than Builder?
3. What types of objects are good candidates for Prototype?

---

## Advanced

1. How would you implement deep cloning for an object graph with shared ownership?
2. How would you avoid cyclic references during cloning?
3. How would you design a prototype registry for a plugin-based application?

---

## Scenario-Based

Your TPS allows physicists to duplicate treatment beams and modify only the gantry angle.

Would Prototype be appropriate? Explain your reasoning and discuss how you would handle shared versus copied data.

---

## Architecture

Design a prototype-based architecture for a DICOM Viewer that supports duplicating viewer layouts, annotation presets, and image processing pipelines. Explain which components should be deeply copied and which could safely be shared.

---

# 20. Practice Exercises

### Beginner Exercise

Design a `Shape` hierarchy (`Circle`, `Rectangle`, `Triangle`) where every shape can clone itself. Explain whether value members require deep or shallow copying.

---

### Intermediate Exercise

Design a `ViewerSettings` prototype for a Qt application containing:

* Theme
* Window/Level
* Zoom
* Measurement Units
* Annotation Preferences

Explain how new viewer windows can be created by cloning these settings instead of configuring each one from scratch.

---

### Advanced Exercise

Design a **Prototype-based architecture** for a **Treatment Planning System (TPS)** that allows users to duplicate:

* Treatment Plans
* Beam Configurations
* Dose Calculation Profiles
* Optimization Templates

For each object, decide:

* Should cloning be **deep** or **shallow**?
* Which resources can be shared?
* Which resources must be copied?
* How will you prevent accidental modification of shared state?

**Do not implement the solution yet.** Focus on architectural reasoning.

---

# Key Architectural Takeaway

The **Prototype Pattern** is not about copying objects because it's easier to code.

It's about recognizing when an existing, fully configured object is the **best starting point** for creating another object.

A junior developer thinks:

> "I'll create a new object and configure it again."

A software architect thinks:

> **"This object already represents a valid configuration. I'll clone it, preserve the known-good state, and change only what needs to be different."**

That approach reduces duplication, minimizes configuration errors, and simplifies workflows in domains like CAD, graphics, and medical imaging.

When you're ready, type **`NEXT`** to continue with **Lesson 6: Object Pool Pattern**.














# 🌀 Topic 7: Prototype Pattern (Creational Design Pattern)

## 🧠 Overview

The **Prototype Pattern** is a *Creational Design Pattern* that allows you to **create new objects by copying (cloning) an existing object**, rather than creating a new instance from scratch.

In simple words:
➡️ “If creating a new object is expensive, just clone an existing one.”

---

## 🎯 Real-Life Example

Imagine you’re designing a **video game** 🕹️.
You have a **monster object** with health, weapons, and skills.
Instead of creating every monster from scratch, you can **clone an existing monster** and make small modifications.

This saves time and resources, especially when object creation is costly.

---

## ⚙️ Key Idea

1. A class provides a **clone method**.
2. New objects are created by **copying the existing object**.
3. Useful when **creating a new object is heavy** (like database, files, network, or large objects).

---

## 🧩 C++ Example (Simple and Clear)

```cpp
#include <iostream>
#include <string>
using namespace std;

// Step 1: Prototype class
class Monster {
public:
    string type;
    int health;
    string weapon;

    Monster(string t, int h, string w) : type(t), health(h), weapon(w) {}

    // Step 2: Clone method
    Monster* clone() {
        return new Monster(type, health, weapon);
    }

    void showInfo() {
        cout << "Monster Type: " << type
             << ", Health: " << health
             << ", Weapon: " << weapon << endl;
    }
};

int main() {
    // Step 3: Original object
    Monster* dragon = new Monster("Dragon", 500, "Fire Breath");
    dragon->showInfo();

    // Step 4: Cloned object
    Monster* dragonClone = dragon->clone();
    dragonClone->weapon = "Ice Breath";  // modify cloned object
    dragonClone->showInfo();

    delete dragon;
    delete dragonClone;
    return 0;
}
```

---

## 🧾 Output

```
Monster Type: Dragon, Health: 500, Weapon: Fire Breath
Monster Type: Dragon, Health: 500, Weapon: Ice Breath
```

---

## 🔍 Explanation (In Simple Words)

* `Monster` is our **prototype object**.
* `clone()` method **creates a copy** of the object.
* We can **modify the cloned object** without affecting the original.
* This pattern is **very useful** when creating an object is expensive or complex.

---

## 🧠 When to Use Prototype Pattern

* When object creation is **time-consuming or costly**.
* When you want to **avoid repeated initialization code**.
* When you need **many similar objects** with minor differences.

---

## 🏁 Summary

| **Concept**  | **Description**                               |
| ------------ | --------------------------------------------- |
| Purpose      | Create new objects by cloning existing ones.  |
| Key Feature  | `clone()` method duplicates the object.       |
| Benefit      | Saves resources, avoids heavy initialization. |
| Example      | Video game monsters, document templates.      |
| Pattern Type | Creational Design Pattern                     |

---

✅ **Quick Comparison with Other Creational Patterns**

| Pattern          | Focus                     | Example                  |
| ---------------- | ------------------------- | ------------------------ |
| Singleton        | Only one object           | Logger, DB Connection    |
| Factory Method   | One type of object        | Car or Bike              |
| Abstract Factory | Families of objects       | Windows UI or Mac UI     |
| Builder          | Step-by-step construction | SportsCar or SUVCar      |
| Prototype        | Clone existing object     | Game monsters, templates |

---

Do you want me to **continue next with Topic 8: Adapter Pattern (Structural Design Pattern)** in the same style?
