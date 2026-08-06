# Design Patterns Master Course

# Lesson 4 — Builder Pattern (Creational Pattern)

---

# Before We Start

Many developers confuse **Builder** with **Factory**.

This is one of the biggest interview mistakes.

Let's understand the difference first.

Imagine you go to a restaurant.

### Factory

You say:

> "Give me a Pizza."

Kitchen:

> "Here is your Pizza."

Done.

---

### Builder

You say:

> "I want a Pizza."

Then the chef asks:

* Thin crust or thick crust?
* Cheese?
* Extra cheese?
* Mushroom?
* Paneer?
* Olives?
* Jalapeños?
* Bake level?

Now the pizza is built **step by step**.

That is Builder.

---

# The Architect's Way of Thinking

Factories answer:

> **"Which object should I create?"**

Builders answer:

> **"How should I construct this complex object?"**

---

# Factory vs Builder

| Factory                     | Builder                          |
| --------------------------- | -------------------------------- |
| Creates object in one step  | Creates object in multiple steps |
| Simple creation             | Complex construction             |
| Focus on object type        | Focus on object configuration    |
| Usually returns immediately | Object is assembled gradually    |
| Chooses implementation      | Chooses construction process     |

Think like this:

```text
Factory

Client

↓

Factory

↓

Car
```

Builder:

```text
Client

↓

Builder

↓

Engine

↓

Seats

↓

Doors

↓

Paint

↓

Car
```

The Builder separates **construction** from the **final object**.

---

# 1. Introduction

## What is the Builder Pattern?

The **Builder Pattern** separates the construction of a complex object from its representation so that the same construction process can create different representations.

In simple words:

> **The Builder Pattern constructs complex objects step by step.**

---

## Why was it created?

Some objects are simple.

Example:

```cpp
Point p(10,20);
```

Done.

---

Some objects are huge.

Imagine creating a Treatment Plan.

It contains:

* Patient Information
* Machine
* Beam List
* Dose Algorithm
* Prescription
* Optimization Settings
* Structures
* Constraints
* DVH Settings
* Report Settings
* Export Settings

Imagine putting all of these into one constructor.

```cpp
TreatmentPlan(
patient,
machine,
beamList,
algorithm,
optimizer,
structures,
constraints,
doseGrid,
prescription,
reports,
exportSettings,
...
...
...)
```

Nobody can remember the parameter order.

Even worse:

```cpp
TreatmentPlan(
patient,
algorithm,
machine,
beamList,
...
)
```

Compiler may accept it if the types match.

The plan may be configured incorrectly.

---

## Category

**Creational Pattern**

---

## What problem does it solve?

It solves the problem of constructing **large, configurable objects** without creating huge constructors or dozens of overloaded constructors.

---

# 2. Problem Statement

Imagine a **CT Scan Exporter**.

Export settings:

```text
Compression

Patient Name

Anonymize

Resolution

Image Format

Metadata

Watermark

Encryption

Color Space

Output Folder
```

Without Builder:

```cpp
Exporter(
true,
false,
1024,
PNG,
true,
false,
AES256,
...
)
```

Question:

What does

```cpp
true,false,true,false
```

mean?

Nobody knows.

The constructor becomes unreadable.

---

Another problem:

Tomorrow management says:

Add:

```text
Digital Signature
```

Now every constructor changes.

Every call changes.

Hundreds of files break.

---

# 3. Motivation

Developers realized that object construction and object usage are different responsibilities.

Building a complex object often requires:

* validation,
* default values,
* optional settings,
* ordered steps,
* conditional logic.

The Builder pattern keeps this complexity out of the object's constructor and places it in a dedicated builder.

---

# 4. Real-World Analogy

## Building a House

You don't build a house in one step.

Construction happens in stages:

```text
Foundation

↓

Walls

↓

Roof

↓

Doors

↓

Windows

↓

Painting

↓

Furniture

↓

Finished House
```

The order matters.

The builder knows the process.

The homeowner only specifies requirements.

### Mapping

| House Construction   | Software            |
| -------------------- | ------------------- |
| House                | Complex object      |
| Construction Company | Builder             |
| Engineer             | Director (optional) |
| Homeowner            | Client              |
| Construction Steps   | Builder methods     |

The client doesn't lay bricks. The builder performs the construction.

---

# 5. Software Scenario

Builder is common whenever objects have many optional parts or must be assembled in stages.

### Desktop Applications

* Report generation
* Document creation
* Export configuration

### Qt Applications

* Dialog configuration
* UI builders
* Complex model initialization

Qt itself uses a builder-like style in many APIs through chained configuration calls, even if they are not formal GoF Builder implementations.

### CAD Software

* CAD model creation
* Mesh generation
* Scene creation

### Medical Imaging

* Image processing pipeline
* Treatment plan
* DICOM export configuration

### Game Engines

* Character creation
* Level creation
* Scene construction

### Browsers

* HTTP request configuration
* Rendering pipeline setup

---

# 6. UML Class Diagram

```text
                    +----------------------+
                    |       Director       |
                    +----------------------+
                    | construct()          |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |       Builder        |
                    +----------------------+
                    | buildPartA()         |
                    | buildPartB()         |
                    | buildPartC()         |
                    | getResult()          |
                    +----------+-----------+
                               ^
                ----------------|----------------
                |                               |
     +----------------------+      +----------------------+
     | ConcreteBuilderA     |      | ConcreteBuilderB     |
     +----------------------+      +----------------------+
     | buildPartA()         |      | buildPartA()         |
     | buildPartB()         |      | buildPartB()         |
     | buildPartC()         |      | buildPartC()         |
     +----------+-----------+      +----------+-----------+
                |                               |
                v                               v
         +--------------+                +--------------+
         | Product A    |                | Product B    |
         +--------------+                +--------------+
```

---

## Responsibilities

### Builder

Defines how the product is assembled.

---

### Concrete Builder

Implements each construction step.

---

### Director (Optional)

Knows the order of construction.

Example:

```text
Build Foundation

↓

Walls

↓

Roof

↓

Paint
```

The client may also drive the builder directly, so a Director is optional in many modern designs.

---

### Product

Final object produced by the builder.

---

### Client

Requests the build and receives the completed product.

---

# 7. Participants

## Builder

Declares methods such as:

* buildEngine()
* buildSeats()
* buildDoors()
* buildRoof()
* getResult()

---

## Concrete Builder

Actually builds each part.

---

## Product

The finished object.

Example:

```text
Car

TreatmentPlan

Exporter

HTTPRequest
```

---

## Director

Coordinates construction.

---

## Client

Uses the builder to obtain the final product.

---

# 8. Collaboration

Runtime Flow

```text
Client

↓

Builder Created

↓

Director

↓

Build Engine

↓

Build Wheels

↓

Build Doors

↓

Build Paint

↓

Return Car
```

Or, without a Director:

```text
Client

↓

Builder

↓

setEngine()

↓

setColor()

↓

addBeam()

↓

enableEncryption()

↓

build()
```

---

# 9. C++ Example

Let's build a simple `Car`.

```cpp
#include <iostream>
#include <memory>
#include <string>

class Car
{
public:
    std::string engine;
    int doors = 0;
    std::string color;

    void show() const
    {
        std::cout << "Engine: " << engine
                  << ", Doors: " << doors
                  << ", Color: " << color << '\n';
    }
};

class CarBuilder
{
public:
    CarBuilder& setEngine(const std::string& type)
    {
        car.engine = type;
        return *this;
    }

    CarBuilder& setDoors(int count)
    {
        car.doors = count;
        return *this;
    }

    CarBuilder& setColor(const std::string& c)
    {
        car.color = c;
        return *this;
    }

    Car build()
    {
        return car;
    }

private:
    Car car;
};

int main()
{
    Car sportsCar = CarBuilder()
                        .setEngine("V8")
                        .setDoors(2)
                        .setColor("Red")
                        .build();

    sportsCar.show();
}
```

### Design Focus

Notice the architectural benefits:

* No massive constructor.
* Readable configuration.
* Optional steps are easy.
* New options can often be added without breaking existing client code.

---

# 10. Qt Example

Imagine creating a configurable dialog.

Instead of:

```cpp
CustomDialog(
title,
icon,
width,
height,
theme,
buttons,
shortcut,
helpText,
...
);
```

Use a builder:

```cpp
DialogBuilder builder;

builder.setTitle("Export")
       .setSize(800, 600)
       .setTheme("Dark")
       .addButton("Save")
       .addButton("Cancel")
       .enableHelp(true);

auto dialog = builder.build();
```

This is easier to read and extend.

A similar style is often seen when configuring `QNetworkRequest`, `QTextDocument`, or complex UI objects through a sequence of method calls.

---

# 11. Medical Software Example

Imagine building a **Treatment Plan**.

A plan contains:

```text
Patient

↓

Machine

↓

Dose Algorithm

↓

Prescription

↓

Structures

↓

Beams

↓

Optimization Parameters

↓

DVH Constraints

↓

Export Settings

↓

QA Settings

↓

Final Treatment Plan
```

Without Builder:

```cpp
TreatmentPlan(
patient,
machine,
algorithm,
structures,
beams,
constraints,
...
)
```

Difficult to understand and maintain.

With Builder:

```text
TreatmentPlanBuilder

↓

setPatient()

↓

setMachine()

↓

addBeam()

↓

setPrescription()

↓

setDoseAlgorithm()

↓

enableQA()

↓

build()
```

This mirrors the real clinical workflow and makes the code easier to follow.

---

# 12. Advantages

### Readability

Construction steps describe the object's configuration clearly.

### Maintainability

Adding optional fields usually doesn't require changing every constructor call.

### Reusability

The same builder can construct different configurations.

### Flexibility

Clients choose only the options they need.

### Encapsulation

Construction logic stays out of the product class.

### Validation

The builder can verify required fields before producing the final object.

---

# 13. Disadvantages

### More Classes

Adds a builder class.

### Additional Code

Simple objects don't need builders.

### Slight Runtime Overhead

Usually negligible, but there is an extra construction layer.

### When NOT to Use

Avoid Builder when:

* the object has only a few required parameters,
* construction is simple,
* there are no optional configurations.

---

# 14. Best Practices

* Use Builder for objects with many optional parameters.
* Keep validation inside the builder when appropriate.
* Make `build()` produce a valid object or report errors.
* Prefer expressive method names (`setDoseAlgorithm()`, `enableAnonymization()`).
* Keep the product immutable after construction if possible.

---

# 15. Common Mistakes

### Mistake 1

Using Builder for every class.

Not every object is complex enough to justify it.

### Mistake 2

Putting business logic inside the builder.

The builder should assemble the product, not perform its work.

### Mistake 3

Returning partially built objects before validation.

### Mistake 4

Ignoring required fields.

A builder should prevent creation of invalid products whenever possible.

---

# 16. Pattern Variations

1. **Fluent Builder**

   * Methods return the builder itself, enabling chained calls (as in the C++ example).

2. **Director-Based Builder**

   * A Director controls the construction sequence.

3. **Immutable Builder**

   * Each configuration step returns a new builder instance.

4. **Step Builder**

   * Enforces the order of required construction steps at compile time.

---

# 17. Related Patterns

| Pattern          | Difference                                            |
| ---------------- | ----------------------------------------------------- |
| Factory Method   | Creates one object in one step.                       |
| Abstract Factory | Creates families of related objects.                  |
| Builder          | Constructs one complex object through multiple steps. |
| Prototype        | Clones an existing object.                            |

A simple memory trick:

* **Factory** → *Which object?*
* **Abstract Factory** → *Which family?*
* **Builder** → *How should it be assembled?*

---

# 18. Industry Usage

Builder is widely used whenever configuration becomes complex.

* **Microsoft:** Request builders, document generation, and configuration APIs.
* **Google:** Fluent builders are common in large C++ and Java codebases for complex configuration objects.
* **Adobe:** Document export settings, rendering pipelines, and graphics configuration.
* **Qt:** Many APIs encourage incremental configuration through chained setter methods, following a builder-like style.
* **ZEISS / Siemens / Philips:** Treatment plans, imaging pipelines, acquisition protocols, and report generation.
* **Autodesk:** CAD model construction, scene configuration, and export options.
* **Game Engines:** Character creation, level construction, rendering pipeline configuration, and asset import settings.

The common theme is **building complex objects safely and readably**.

---

# 19. Interview Questions

## Beginner

1. What problem does the Builder Pattern solve?
2. When would you use Builder instead of a constructor?
3. What is the role of the Builder?

## Intermediate

1. What is the purpose of a Director?
2. Why is a fluent interface commonly used with Builder?
3. How does Builder improve readability?

## Advanced

1. How would you prevent clients from creating an invalid product?
2. How would you design a thread-safe builder?
3. When would you choose Builder over Abstract Factory?

## Scenario-Based

You need to create a DICOM export configuration with 20 optional settings.

Would you use overloaded constructors, setters, or a Builder? Explain your architectural decision.

## Architecture

Design a `TreatmentPlanBuilder` for a TPS that constructs:

* Patient
* Machine
* Beams
* Prescription
* Optimization Parameters
* QA Settings
* Export Configuration

Explain which fields are mandatory, which are optional, and where validation should occur.

---

# 20. Practice Exercises

### Beginner Exercise

Design a `PizzaBuilder` that supports optional toppings such as cheese, mushrooms, olives, and paneer. Focus on the builder methods and the final `Pizza` object.

---

### Intermediate Exercise

Design a `ReportBuilder` for a Qt desktop application that can configure:

* Title
* Author
* Page Size
* Header
* Footer
* Watermark
* Output Format

Explain how the builder improves readability compared to a constructor.

---

### Advanced Exercise

Design a **TreatmentPlanBuilder** for a medical **Treatment Planning System (TPS)** that supports:

* Patient Information
* Machine Selection
* Beam Definitions
* Dose Algorithm
* Prescription
* Structures
* DVH Constraints
* QA Options
* DICOM Export Settings

Your design should ensure that an invalid treatment plan cannot be built. Describe the builder's responsibilities, validation strategy, and collaboration with the rest of the system.

**Do not implement the solution yet.** Focus on the architecture.

---

# Key Architectural Takeaway

The **Builder Pattern** is about separating **construction** from **representation**.

A junior developer thinks:

> "I'll create the object by passing everything into the constructor."

A software architect thinks:

> **"This object is complex, configurable, and likely to evolve. I'll make its construction explicit, readable, and safe by moving it into a Builder."**

That mindset leads to APIs that are easier to understand, extend, and maintain.

When you're ready, type **`NEXT`** to continue with **Lesson 5: Prototype Pattern**.













Perfect, Brijesh! 👏
You’re progressing excellently through the **Creational Design Patterns** series.
Let’s now move on to the next one — the **Builder Pattern**, explained in **simple English** with **clear C++ code** examples.

---

# 🧱 Topic 6: Builder Pattern (Creational Design Pattern)

## 🧠 Overview

The **Builder Pattern** helps you **create complex objects step by step**.

Instead of writing a big constructor with too many parameters (which can be confusing),
the Builder Pattern separates **object construction** from **object representation** — so the same construction process can create **different representations**.

---

## 🎯 Real-Life Example

Imagine you’re ordering a **burger meal** at a restaurant 🍔🍟🥤.
You can choose what to include — bun type, patty, cheese, drink, etc.
The chef (builder) prepares your customized meal step by step.

Similarly, in C++, the Builder Pattern helps you **build complex objects** (like a Car, PC, or Report) in small, manageable steps.

---

## ⚙️ Key Idea

1. A **Builder** constructs complex objects step by step.
2. A **Director** controls the building process.
3. The **final product** is assembled by combining these steps.

---

## 🧩 C++ Example (Simple and Easy)

Let’s build a **Car** object with parts like engine, seats, and GPS.

```cpp
#include <iostream>
#include <string>
using namespace std;

// Step 1: The Product class
class Car {
public:
    string engine;
    string seats;
    string GPS;

    void showCar() {
        cout << "Car built with: " << engine << ", " << seats << ", " << GPS << endl;
    }
};

// Step 2: The Builder interface
class CarBuilder {
public:
    virtual void buildEngine() = 0;
    virtual void buildSeats() = 0;
    virtual void buildGPS() = 0;
    virtual Car* getCar() = 0;
};

// Step 3: Concrete Builders
class SportsCarBuilder : public CarBuilder {
private:
    Car* car;
public:
    SportsCarBuilder() { car = new Car(); }

    void buildEngine() override { car->engine = "V8 Engine"; }
    void buildSeats() override { car->seats = "2 Leather Seats"; }
    void buildGPS() override { car->GPS = "Advanced GPS System"; }

    Car* getCar() override { return car; }
};

class SUVCarBuilder : public CarBuilder {
private:
    Car* car;
public:
    SUVCarBuilder() { car = new Car(); }

    void buildEngine() override { car->engine = "V6 Engine"; }
    void buildSeats() override { car->seats = "7 Comfortable Seats"; }
    void buildGPS() override { car->GPS = "Basic GPS"; }

    Car* getCar() override { return car; }
};

// Step 4: Director class — controls the building steps
class Director {
public:
    void construct(CarBuilder* builder) {
        builder->buildEngine();
        builder->buildSeats();
        builder->buildGPS();
    }
};

// Step 5: Client code
int main() {
    Director director;

    SportsCarBuilder sportsBuilder;
    SUVCarBuilder suvBuilder;

    director.construct(&sportsBuilder);
    Car* sportsCar = sportsBuilder.getCar();
    sportsCar->showCar();

    director.construct(&suvBuilder);
    Car* suvCar = suvBuilder.getCar();
    suvCar->showCar();

    delete sportsCar;
    delete suvCar;

    return 0;
}
```

---

## 🧾 Output

```
Car built with: V8 Engine, 2 Leather Seats, Advanced GPS System
Car built with: V6 Engine, 7 Comfortable Seats, Basic GPS
```

---

## 🔍 Explanation (In Simple Words)

* **Car** → The final product we’re building.
* **CarBuilder** → The abstract blueprint (defines the building steps).
* **SportsCarBuilder** and **SUVCarBuilder** → Different ways to build cars.
* **Director** → Organizes the construction steps in the correct order.
* **main()** → Client just tells the director what type of car to build.

This way, we can **easily create different types of objects** (like sports or SUV cars) without rewriting the construction logic.

---

## 🧠 When to Use Builder Pattern

* When object creation involves **many steps or configurations**.
* When constructors with too many parameters become confusing.
* When you need **different representations** of the same object.

---

## 🏁 Summary

| **Concept**  | **Description**                              |
| ------------ | -------------------------------------------- |
| Purpose      | Builds complex objects step by step.         |
| Key Feature  | Separate construction and representation.    |
| Benefit      | Makes object creation flexible and readable. |
| Example      | Building cars, PCs, houses, etc.             |
| Pattern Type | Creational Design Pattern                    |

---

✅ **Quick Comparison with Factory Patterns**

| Pattern          | Focus                               | Example              |
| ---------------- | ----------------------------------- | -------------------- |
| Factory Method   | Creates one object                  | Car or Bike          |
| Abstract Factory | Creates families of related objects | Windows UI or Mac UI |
| Builder          | Creates complex object step by step | SportsCar or SUVCar  |

---

Would you like me to continue next with **Topic 7: Prototype Pattern (Creational Design Pattern)** — in the same simple format with C++ example and explanation?
