# Design Patterns Master Course

# Lesson 24 — Template Method Pattern (Behavioral Pattern)

---

# Before We Start

The **Template Method Pattern** is one of the most important patterns for anyone who wants to become a **framework architect**.

If Strategy answers:

> **"Which algorithm should I use?"**

then Template Method answers:

> **"The overall algorithm must always stay the same, but some steps may vary."**

You will see this pattern in:

* Qt Framework
* MFC
* Google Test
* Unreal Engine
* Game Loops
* Medical Imaging Frameworks
* Treatment Planning Systems
* Document Processing Systems

---

# The Architect's Way of Thinking

Imagine you're designing a **Treatment Planning System (TPS)**.

Every dose calculation follows the same high-level workflow:

```text
Load Patient
      ↓
Validate Data
      ↓
Prepare Geometry
      ↓
Calculate Dose
      ↓
Post Process
      ↓
Generate DVH
      ↓
Generate Report
```

Now suppose you support different algorithms:

* Pencil Beam
* Monte Carlo
* Collapsed Cone
* AI Dose Engine

A junior developer writes four completely separate implementations:

```text
PencilBeam.calculate()

MonteCarlo.calculate()

CollapsedCone.calculate()

AIDose.calculate()
```

Each implementation duplicates:

* loading,
* validation,
* report generation,
* logging,
* timing,
* error handling.

Only the **dose calculation step** is different.

An architect asks:

> **"Why duplicate the whole workflow when only one step changes?"**

Instead:

```text
DoseEngine
      │
      ▼
run()
      │
      ├── Load Patient      ✅ Fixed
      ├── Validate          ✅ Fixed
      ├── Calculate Dose    🔄 Variable
      ├── Generate DVH      ✅ Fixed
      └── Generate Report   ✅ Fixed
```

Only the variable step is overridden.

This is the **Template Method Pattern**.

---

# 1. Introduction

## What is the Template Method Pattern?

The **Template Method Pattern** defines the **skeleton of an algorithm** in a base class while allowing subclasses to redefine specific steps without changing the algorithm's structure.

Simple definition:

> **Define the workflow once. Allow subclasses to customize selected steps.**

---

## Why was it created?

Many systems have:

* A fixed workflow.
* A few variable steps.

Duplicating the entire workflow leads to:

* repeated code,
* inconsistent behavior,
* maintenance problems.

---

## Category

**Behavioral Pattern**

---

## What problem does it solve?

Without Template Method:

```text
Algorithm A

Load

Validate

Calculate

Save
```

```text
Algorithm B

Load

Validate

Calculate

Save
```

Large amounts of duplicated code.

With Template Method:

```text
Base Class

↓

run()

↓

Load

↓

Validate

↓

calculate() ← overridden

↓

Save
```

The common workflow exists only once.

---

# 2. Problem Statement

Imagine a **Document Export System**.

Supported formats:

* PDF
* Word
* HTML

Every export process performs:

```text
Open Document
↓

Validate

↓

Convert

↓

Save

↓

Log
```

Only **Convert** differs.

Without Template Method:

Every exporter duplicates four identical steps.

---

# 3. Motivation

Architects observed:

The sequence of operations is fixed.

Only some operations vary.

Instead of rewriting everything,

allow subclasses to customize only the variable steps.

---

# 4. Real-World Analogy

## Coffee vs Tea

Making coffee:

```text
Boil Water
↓

Add Coffee
↓

Pour Into Cup
↓

Add Sugar
```

Making tea:

```text
Boil Water
↓

Add Tea
↓

Pour Into Cup
↓

Add Lemon
```

The workflow is almost identical.

Only one or two steps differ.

The recipe itself is the **template**.

---

### Mapping

| Recipe | Software               |
| ------ | ---------------------- |
| Recipe | Template Method        |
| Coffee | Concrete Class         |
| Tea    | Concrete Class         |
| Step   | Hook / Abstract Method |

---

# 5. Software Scenario

Template Method is ideal when workflows remain fixed.

### Desktop Applications

Printing:

```text
Open File

↓

Prepare

↓

Print

↓

Close
```

---

### Qt Applications

Qt relies heavily on this pattern.

Example:

`QAbstractItemModel`

The framework defines the interaction pattern.

Subclasses override methods such as:

```cpp
rowCount()

columnCount()

data()

index()

parent()
```

Qt controls **when** these methods are called.

This is a classic example of the **Hollywood Principle**:

> **Don't call us. We'll call you.**

---

### Medical Imaging

Image loading:

```text
Load File

↓

Validate

↓

Decode Format

↓

Create Image

↓

Notify View
```

Only the decoding step changes.

---

### CAD Software

Import:

```text
Read File

↓

Parse

↓

Build Geometry

↓

Validate

↓

Display
```

---

# 6. UML Class Diagram

```text
              +----------------------+
              |   AbstractClass      |
              +----------------------+
              | +templateMethod()    |
              | +step1()             |
              | +step2() (abstract)  |
              | +step3()             |
              +----------+-----------+
                         ^
             ------------|------------
             |                        |
    +----------------+      +----------------+
    | ConcreteClassA |      | ConcreteClassB |
    +----------------+      +----------------+
    | step2()        |      | step2()        |
    +----------------+      +----------------+
```

---

## Responsibilities

### Abstract Class

* Defines the workflow.
* Calls the steps.
* Implements common behavior.

---

### Concrete Class

Overrides only the variable steps.

---

# 7. Participants

## Abstract Class

Example:

```cpp
DoseCalculationWorkflow
```

Contains:

```cpp
run()
```

Defines:

```cpp
loadPatient();

validate();

calculateDose(); // abstract

generateReport();
```

---

## Concrete Classes

Examples:

```text
MonteCarloWorkflow

PencilBeamWorkflow

CollapsedConeWorkflow
```

Each overrides:

```cpp
calculateDose()
```

---

## Client

Calls only:

```cpp
workflow.run();
```

---

# 8. Collaboration

Runtime Flow

```text
Client

↓

workflow.run()

↓

Load Patient

↓

Validate

↓

calculateDose()

↓

Generate Report

↓

Return
```

Notice:

The client never calls individual steps.

---

# 9. C++ Example

```cpp
#include <iostream>

// Abstract Class
class Beverage
{
public:
    virtual ~Beverage() = default;

    void prepare()
    {
        boilWater();
        brew();
        pourIntoCup();
        addExtras();
    }

protected:
    void boilWater()
    {
        std::cout << "Boiling water\n";
    }

    virtual void brew() = 0;

    void pourIntoCup()
    {
        std::cout << "Pouring into cup\n";
    }

    virtual void addExtras() = 0;
};

// Concrete Class
class Coffee : public Beverage
{
protected:
    void brew() override
    {
        std::cout << "Brewing coffee\n";
    }

    void addExtras() override
    {
        std::cout << "Adding sugar\n";
    }
};

class Tea : public Beverage
{
protected:
    void brew() override
    {
        std::cout << "Steeping tea\n";
    }

    void addExtras() override
    {
        std::cout << "Adding lemon\n";
    }
};

int main()
{
    Coffee coffee;
    coffee.prepare();

    std::cout << "-----\n";

    Tea tea;
    tea.prepare();
}
```

---

## Design Focus

Notice:

The client always calls:

```cpp
prepare();
```

The workflow never changes.

Only:

```cpp
brew()

addExtras()
```

are customized.

---

# 10. Qt Example

Suppose you're creating a custom model.

Qt defines the workflow.

You implement only:

```cpp
rowCount()

columnCount()

data()
```

Qt internally performs:

```text
View Requests Data

↓

Model Calls

↓

Paint Item

↓

Refresh View
```

Your subclass customizes only the required pieces.

This is one of the reasons Qt is considered a framework rather than just a library.

---

# 11. Medical Software Example

Imagine a **DICOM Import Framework**.

Every import performs:

```text
Open File
↓

Read Metadata
↓

Validate
↓

Decode Pixels
↓

Store Image
↓

Notify UI
```

Different formats:

* CT
* MRI
* PET
* Ultrasound

share the same workflow.

Only:

```text
Decode Pixels
```

differs.

Architecture:

```text
AbstractDicomImporter
       │
       ▼
decodePixels() ← overridden
```

The framework guarantees that validation always happens before decoding.

---

# 12. Advantages

### Code Reuse

Common workflow exists only once.

### Open/Closed Principle

New implementations require new subclasses, not changes to the base class.

### Consistency

The workflow cannot accidentally change.

### Easier Maintenance

Shared logic lives in one place.

### Framework-Friendly

Excellent for reusable frameworks.

---

# 13. Disadvantages

### Inheritance Required

Template Method is inheritance-based.

### Less Flexible Than Strategy

Changing behavior at runtime is difficult.

### Deep Hierarchies

Many subclasses can become hard to manage.

### When NOT to Use

Avoid Template Method when:

* the workflow itself changes frequently,
* runtime algorithm switching is required,
* composition is more appropriate than inheritance.

---

# 14. Best Practices

* Keep the template method non-virtual to preserve the workflow.
* Mark only variable steps as `virtual`.
* Minimize the number of overridable methods.
* Use hook methods for optional behavior.
* Avoid putting unrelated logic into the base class.

---

# 15. Common Mistakes

### Mistake 1

Making the template method virtual.

Then subclasses can replace the entire workflow, defeating the pattern.

---

### Mistake 2

Overriding every step.

Only steps that truly vary should be virtual.

---

### Mistake 3

Duplicating shared logic in subclasses.

Keep common behavior in the base class.

---

### Mistake 4

Using Template Method when Strategy is a better fit.

If algorithms must change at runtime, prefer Strategy.

---

# 16. Pattern Variations

## 1. Abstract Step

Subclass must implement it.

```cpp
virtual void calculate() = 0;
```

---

## 2. Hook Method

Optional override.

```cpp
virtual void afterLoad()
{
}
```

The default implementation does nothing.

---

## 3. Final Template

Prevent subclasses from changing the workflow.

```cpp
void run() final;
```

(or simply make `run()` non-virtual)

---

# 17. Related Patterns

| Pattern         | Difference                                     |
| --------------- | ---------------------------------------------- |
| Template Method | Workflow is fixed; subclasses customize steps. |
| Strategy        | Entire algorithm can be replaced at runtime.   |
| Factory Method  | Creates objects.                               |
| Command         | Represents actions.                            |
| State           | Changes behavior based on state.               |

---

# ⭐ Template Method vs Strategy (Most Important Comparison)

This is one of the most common interview questions.

| Template Method               | Strategy                      |
| ----------------------------- | ----------------------------- |
| Based on inheritance.         | Based on composition.         |
| Workflow fixed.               | Algorithm interchangeable.    |
| Compile-time customization.   | Runtime customization.        |
| Framework controls execution. | Client selects the algorithm. |

### Example

#### Template Method

```text
Medical Import

↓

Open File

↓

Validate

↓

Decode ← overridden

↓

Display
```

Workflow never changes.

---

#### Strategy

```text
Dose Engine

↓

Monte Carlo

Pencil Beam

Collapsed Cone
```

User chooses the algorithm.

---

# 18. Industry Usage

Template Method is fundamental to framework design.

* **Microsoft:** MFC message handling, testing frameworks.
* **Google:** Google Test (`SetUp()`, `TearDown()`, test lifecycle).
* **Adobe:** Document processing pipelines.
* **Qt:** `QAbstractItemModel`, `QIODevice`, `QThread::run()`, `QGraphicsItem::paint()`, event handling callbacks.
* **ZEISS / Siemens / Philips:** Image import frameworks, processing pipelines, device workflows.
* **Autodesk:** File import/export frameworks and rendering pipelines.
* **Game Engines:** Scene loading, entity lifecycle, render loops.

The architectural goal is:

> **Define the workflow once. Customize only the varying steps.**

---

# 19. Interview Questions

## Beginner

1. What problem does the Template Method Pattern solve?
2. What is a template method?
3. Why is it inheritance-based?

---

## Intermediate

1. How is Template Method different from Strategy?
2. What is a hook method?
3. Why should the template method usually be non-virtual?

---

## Advanced

1. How would you design a plugin framework using Template Method?
2. How would you combine Template Method with Factory Method?
3. When would you replace Template Method with Strategy?

---

## Scenario-Based

Your TPS imports:

* CT
* MRI
* PET
* CBCT

All formats share the same workflow but decode pixel data differently.

Design a Template Method architecture.

---

## Architecture

Design a Template Method framework for your **College ERP**.

Importers:

* Student CSV
* Teacher CSV
* Course CSV

Workflow:

```text
Open File

↓

Validate Format

↓

Parse Records

↓

Store Database

↓

Generate Report
```

Only parsing differs.

Explain:

* base class,
* hooks,
* abstract methods,
* and extension strategy.

---

# 20. Practice Exercises

### Beginner Exercise

Design a **Beverage Preparation** framework.

Workflow:

* Boil Water
* Brew
* Pour
* Add Extras

Support:

* Tea
* Coffee

---

### Intermediate Exercise

Design a Qt-based image loading framework.

Workflow:

* Open File
* Validate
* Decode
* Cache
* Display

Different image formats override only the decode step.

---

### Advanced Exercise

Design a **Template Method architecture** for a **Treatment Planning System**.

Workflow:

```text
Load Patient
↓

Validate Patient
↓

Prepare Geometry
↓

Calculate Dose
↓

Generate DVH
↓

Generate Report
↓

Save Plan
```

Requirements:

* Support multiple dose algorithms.
* Keep the workflow fixed.
* Allow only `calculateDose()` to vary.
* Add optional hooks for logging and performance profiling.
* Explain how Strategy could be combined to make the dose calculation step runtime-configurable.

**Do not implement the solution yet.** Focus on architecture, extensibility, and responsibilities.

---

# ⭐ Architect's Insight: Template Method + Strategy

These two patterns are often combined.

```text
Template Method
        │
Fixed Workflow
        │
calculateDose()
        │
        ▼
Strategy
        │
 ┌──────┼────────────┐
 ▼      ▼            ▼
MonteCarlo  PencilBeam  AI Dose
```

The **workflow remains fixed**, but one step delegates to a Strategy chosen at runtime.

This is a powerful architecture used in many scientific and medical applications because it provides:

* A stable processing pipeline.
* Flexible algorithm selection.
* High extensibility.

---

# Key Architectural Takeaway

The **Template Method Pattern** is about **fixing the overall workflow while allowing subclasses to customize specific steps**.

A junior developer thinks:

> "I'll duplicate the whole workflow for every variation."

A software architect thinks:

> **"The sequence of operations is stable. I'll implement it once and expose only the steps that legitimately vary."**

This mindset is fundamental to designing reusable frameworks, SDKs, medical processing pipelines, and many Qt-based libraries.

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
* ✅ Template Method

Only **two** Behavioral Patterns remain:

1. **Visitor Pattern**
2. **Null Object Pattern**

---

## What You'll Learn Next

Type **`NEXT`** to continue with **Lesson 25: Visitor Pattern**.

Visitor is one of the most advanced GoF patterns and is heavily used in:

* Compilers and ASTs
* CAD software
* Medical imaging scene graphs
* Geometry processing
* Report generation
* Static analysis tools

You'll also learn **why Visitor is considered difficult**, when it is the right choice, and when it should be avoided.
