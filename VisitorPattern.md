# Design Patterns Master Course

# Lesson 25 — Visitor Pattern (Behavioral Pattern)

---

# Before We Start

The **Visitor Pattern** is one of the **most powerful** and **most misunderstood** GoF design patterns.

It is also one of the most common interview topics for **Senior C++ Engineers** because it tests whether you understand:

* Open/Closed Principle
* Double Dispatch
* Object-Oriented Design
* Extensibility
* Separation of Responsibilities

Many developers avoid Visitor because its implementation looks complicated.

The good news is:

> **The idea is much simpler than the code.**

Once you understand the **problem** it solves, the pattern becomes logical.

---

# The Architect's Way of Thinking

Imagine you're building a **Treatment Planning System (TPS)**.

Your system contains many object types:

```text
Treatment Plan

├── Beam
├── Structure
├── CT Image
├── Dose Matrix
├── DVH
└── Isocenter
```

Now management requests a new feature:

> "Export the treatment plan to PDF."

A junior developer adds:

```cpp
class Beam
{
public:
    void exportPDF();
};

class Structure
{
public:
    void exportPDF();
};

class CTImage
{
public:
    void exportPDF();
};
```

Three months later:

> "Export to Excel."

Now every class gets:

```cpp
exportExcel();
```

Next:

> "Generate QA Report."

Again:

```cpp
generateQAReport();
```

Then:

> "Calculate statistics."

Again:

```cpp
calculateStatistics();
```

Eventually:

```text
Beam

├── exportPDF()
├── exportExcel()
├── exportCSV()
├── exportDICOM()
├── statistics()
├── audit()
├── print()
├── validate()
├── ...
```

The classes become huge.

An architect asks:

> **"Why should Beam know how to export itself to every possible format?"**

Instead:

```text
Beam
Structure
Dose
DVH

        │
        ▼

PDF Visitor

Excel Visitor

Statistics Visitor

Validation Visitor
```

Now the **operations** become separate objects.

That is the **Visitor Pattern**.

---

# 1. Introduction

## What is the Visitor Pattern?

The **Visitor Pattern** lets you define **new operations** on existing object structures **without modifying those object classes**.

Simple definition:

> **Move operations out of your objects into visitor classes.**

---

## Why was it created?

Sometimes:

The object hierarchy is stable.

But new operations are added frequently.

Instead of modifying every class every time,

add a new visitor.

---

## Category

**Behavioral Pattern**

---

## What problem does it solve?

Without Visitor:

```text
Beam

↓

Export

↓

Print

↓

Validate

↓

Statistics

↓

Audit
```

The class grows forever.

With Visitor:

```text
Beam

↓

accept(visitor)

↓

Visitor decides operation
```

The object stays small.

---

# 2. Problem Statement

Imagine a **CAD Software**.

Shapes:

* Circle
* Rectangle
* Polygon
* Line

Operations:

* Draw
* Print
* Export
* Area
* Perimeter
* Collision
* Serialization

Without Visitor:

Every shape class gets new methods forever.

---

# 3. Motivation

Architects observed:

Objects often represent **data**.

Operations evolve much faster than data.

Instead of mixing them,

separate them.

---

# 4. Real-World Analogy

## Hospital Rounds

Hospital rooms contain patients.

Doctors visit every patient.

Later:

* Dietician visits.
* Insurance Officer visits.
* Physiotherapist visits.
* Cleaning Staff visits.

Patients don't know every future visitor.

Instead:

Visitors know what to do.

---

### Mapping

| Hospital  | Software         |
| --------- | ---------------- |
| Patient   | Element          |
| Doctor    | Visitor          |
| Dietician | Visitor          |
| Nurse     | Visitor          |
| Hospital  | Object Structure |

---

# 5. Software Scenario

Visitor appears when:

* Object hierarchy is stable.
* Operations change frequently.

---

### Desktop Applications

Document objects:

```text
Paragraph

Table

Image

Chart
```

Visitors:

```text
Print

Export PDF

Spell Check

Statistics
```

---

### Qt Applications

Suppose a graphics scene contains:

```text
QGraphicsItem

↓

Rectangle

Ellipse

Polygon
```

Visitors:

* Export SVG
* Collision Analysis
* Statistics
* Serialization

Instead of modifying every item class, create visitor objects.

---

### Medical Imaging

Objects:

```text
Beam

Structure

DVH

Dose

Image
```

Visitors:

* DICOM Export
* QA Report
* Statistics
* Validation
* Print

---

# 6. UML Class Diagram

```text
               +----------------------+
               |      Visitor         |
               +----------------------+
               | visit(Beam)          |
               | visit(Structure)     |
               | visit(DVH)           |
               +----------+-----------+
                          ^
               -----------|------------
               |                       |
     +------------------+    +------------------+
     | PDFVisitor       |    | StatisticsVisitor|
     +------------------+    +------------------+

                        

      +----------------------+
      |      Element         |
      +----------------------+
      | accept(visitor)      |
      +----------+-----------+
                 ^
      -----------|-------------
      |                        |
 +------------+          +-------------+
 | Beam       |          | Structure   |
 +------------+          +-------------+
```

---

## Responsibilities

### Element

Defines:

```cpp
accept(visitor)
```

---

### Visitor

Defines:

```cpp
visit(Beam)

visit(Structure)
```

---

### Concrete Visitor

Implements one operation.

Example:

```text
PDF Export
```

---

### Object Structure

Stores elements.

---

# 7. Participants

## Element

Example:

```cpp
Beam
```

Contains:

```cpp
accept()
```

---

## Concrete Elements

Examples:

```text
Beam

Structure

Dose

DVH
```

---

## Visitor

Example:

```cpp
TreatmentVisitor
```

---

## Concrete Visitors

Examples:

```text
PDFVisitor

QAVisitor

StatisticsVisitor

PrintVisitor
```

---

## Object Structure

Example:

```text
TreatmentPlan
```

Contains all elements.

---

# 8. Collaboration

Runtime Flow

```text
TreatmentPlan

↓

Beam.accept()

↓

PDFVisitor.visit(Beam)

↓

Export Beam

↓

Structure.accept()

↓

PDFVisitor.visit(Structure)

↓

Export Structure
```

Notice:

The Beam never exports itself.

The visitor performs the operation.

---

# 9. C++ Example

```cpp
#include <iostream>
#include <memory>

// Forward declarations
class Circle;
class Rectangle;

// Visitor Interface
class ShapeVisitor
{
public:
    virtual ~ShapeVisitor() = default;

    virtual void visit(Circle&) = 0;
    virtual void visit(Rectangle&) = 0;
};

// Element
class Shape
{
public:
    virtual ~Shape() = default;
    virtual void accept(ShapeVisitor& visitor) = 0;
};

// Concrete Elements
class Circle : public Shape
{
public:
    void accept(ShapeVisitor& visitor) override
    {
        visitor.visit(*this);
    }
};

class Rectangle : public Shape
{
public:
    void accept(ShapeVisitor& visitor) override
    {
        visitor.visit(*this);
    }
};

// Concrete Visitor
class PrintVisitor : public ShapeVisitor
{
public:
    void visit(Circle&) override
    {
        std::cout << "Printing Circle\n";
    }

    void visit(Rectangle&) override
    {
        std::cout << "Printing Rectangle\n";
    }
};

int main()
{
    Circle circle;
    Rectangle rectangle;

    PrintVisitor printer;

    circle.accept(printer);
    rectangle.accept(printer);
}
```

---

## Design Focus

Notice:

The shape classes never implement:

```cpp
print();

export();

statistics();
```

Instead:

```cpp
accept(visitor);
```

Adding a new operation means adding a new visitor.

No modification to `Circle` or `Rectangle`.

---

# 10. Qt Example

Imagine a Qt-based CAD application using `QGraphicsScene`.

Items:

```text
QGraphicsItem

↓

Line

Rectangle

Ellipse

Text
```

Now add new operations:

* Export SVG
* Export PDF
* Calculate Bounding Statistics
* Accessibility Audit

Architecture:

```text
Scene

↓

Items

↓

accept(visitor)

↓

SVGVisitor

StatisticsVisitor

AuditVisitor
```

Each new feature is implemented as a new visitor.

Existing graphics items remain unchanged.

---

# 11. Medical Software Example

Imagine a **Treatment Planning System**.

Elements:

```text
Beam

Structure

Dose

DVH

Patient
```

Operations:

```text
QA Validation

↓

DICOM Export

↓

Clinical Statistics

↓

PDF Report

↓

Audit
```

Instead of adding five new methods to every class:

```text
Beam

↓

accept(visitor)
```

Examples:

```text
Beam.accept(pdfVisitor)

Beam.accept(validationVisitor)

Beam.accept(statisticsVisitor)
```

The Beam remains focused on representing beam data.

---

# 12. Advantages

### Open/Closed Principle

Add new operations without modifying existing element classes.

### Separation of Concerns

Data and operations are separated.

### Single Responsibility

Elements represent data.

Visitors implement behavior.

### Easy to Add Features

Need a new report?

Create a new visitor.

---

# 13. Disadvantages

### Hard to Add New Element Types

Suppose you add:

```text
Bolus
```

Now every visitor needs:

```cpp
visit(Bolus)
```

This is the biggest trade-off.

### Double Dispatch Complexity

Requires understanding how `accept()` and `visit()` interact.

### More Classes

Every operation becomes another class.

### When NOT to Use

Avoid Visitor when:

* the object hierarchy changes frequently,
* only a few operations exist,
* the hierarchy is still evolving rapidly.

---

# 14. Best Practices

* Use Visitor only when the element hierarchy is stable.
* Keep visitors focused on a single operation.
* Name visitors after the operation they perform (`PdfExportVisitor`, `ValidationVisitor`).
* Keep element classes free of unrelated business operations.
* Consider combining with Composite when traversing object trees.

---

# 15. Common Mistakes

### Mistake 1

Using Visitor when new element types are added frequently.

Visitor is optimized for **new operations**, not **new element types**.

---

### Mistake 2

Putting unrelated responsibilities into one visitor.

Instead of:

```text
EverythingVisitor
```

create:

* `PdfVisitor`
* `AuditVisitor`
* `StatisticsVisitor`

---

### Mistake 3

Confusing Visitor with Strategy.

Strategy changes **how** one algorithm runs.

Visitor changes **what operation** is performed on many element types.

---

### Mistake 4

Ignoring traversal.

Visitor often works together with Composite or Iterator to visit many objects.

---

# 16. Pattern Variations

## 1. Classic Visitor

Each visitor implements methods for every element type.

---

## 2. Reflective Visitor

Uses runtime type information instead of overloaded methods.

Simpler to extend but sacrifices compile-time checking.

---

## 3. Acyclic Visitor

Reduces coupling by splitting visitor interfaces.

Useful in large systems.

---

## 4. Composite + Visitor

Very common.

```text
Scene Graph

↓

Iterator

↓

Visitor
```

Traverse a hierarchy and perform operations.

---

# 17. Related Patterns

| Pattern   | Difference                                    |
| --------- | --------------------------------------------- |
| Visitor   | Adds new operations to an existing hierarchy. |
| Strategy  | Replaces one algorithm with another.          |
| Command   | Encapsulates a request.                       |
| Iterator  | Traverses collections.                        |
| Composite | Organizes hierarchical object structures.     |

---

# ⭐ Visitor vs Strategy

This is another common interview topic.

### Strategy

Question:

> **"How should I perform this task?"**

Example:

```text
Compression

↓

ZIP

RAR

7z
```

One algorithm is selected.

---

### Visitor

Question:

> **"What operation should I perform on these different object types?"**

Example:

```text
Beam

Structure

Dose

↓

PDF Export Visitor
```

The visitor knows how to process each element type.

---

# ⭐ Understanding Double Dispatch

This is the heart of the Visitor Pattern.

Many developers memorize it without understanding it.

Let's simplify it.

Suppose you call:

```cpp
shape.accept(visitor);
```

Inside `Circle`:

```cpp
void Circle::accept(ShapeVisitor& visitor)
{
    visitor.visit(*this);
}
```

Now C++ knows:

* The object is a `Circle`.
* The visitor has an overloaded `visit(Circle&)`.

The call becomes:

```cpp
visitor.visit(circle);
```

The operation depends on:

1. **Which element received the call?**
2. **Which visitor was passed?**

That's why it's called **double dispatch**.

---

# 18. Industry Usage

Visitor is common in systems with **stable object hierarchies** and **frequently changing operations**.

* **Microsoft:** Compiler internals, code analysis tools.
* **Google:** Static analysis frameworks and syntax trees.
* **Adobe:** Document processing and export pipelines.
* **Qt:** Qt itself does not expose a generic Visitor framework, but many applications built on Qt use Visitor for scene graphs, document models, and serialization.
* **ZEISS / Siemens / Philips:** Treatment plan validation, DICOM export, report generation, QA analysis.
* **Autodesk:** CAD geometry processing, file export, model validation.
* **Compilers:** Abstract Syntax Tree (AST) traversal is one of the classic Visitor use cases.

The architectural goal is:

> **Keep object classes stable while allowing new operations to be added easily.**

---

# 19. Interview Questions

## Beginner

1. What problem does the Visitor Pattern solve?
2. Why is Visitor considered a Behavioral Pattern?
3. What is the role of `accept()`?

---

## Intermediate

1. What is double dispatch?
2. Why is Visitor suitable for stable hierarchies?
3. What is the biggest disadvantage of Visitor?

---

## Advanced

1. How would you design a Visitor for an AST in a compiler?
2. How would you combine Visitor with Composite and Iterator?
3. When would you replace Visitor with polymorphism or Strategy?

---

## Scenario-Based

Your TPS contains:

* Beam
* Structure
* Dose
* DVH

New operations arrive every release:

* QA Validation
* PDF Export
* DICOM Export
* Clinical Statistics

Design a Visitor-based architecture that keeps the domain classes unchanged.

---

## Architecture

Design a Visitor architecture for your **College ERP**.

Elements:

* Student
* Teacher
* Course
* Department

Visitors:

* Report Visitor
* Export Visitor
* Analytics Visitor
* Audit Visitor

Requirements:

* New reports added frequently.
* Element classes remain stable.
* Easy testing of each visitor.

Explain:

* visitor hierarchy,
* element hierarchy,
* traversal,
* and extension strategy.

---

# 20. Practice Exercises

### Beginner Exercise

Design a **Shape** hierarchy:

Elements:

* Circle
* Rectangle
* Triangle

Visitor:

* Area Calculator

Draw the UML and explain how `accept()` and `visit()` interact.

---

### Intermediate Exercise

Design a Qt-based graphics application.

Elements:

* Line
* Rectangle
* Ellipse
* Text

Visitors:

* SVG Export
* Statistics
* Print Preview

Explain how new visitors can be added without modifying the graphics items.

---

### Advanced Exercise

Design a **Visitor architecture** for a **Treatment Planning System**.

Elements:

* Beam
* Structure
* Dose Matrix
* DVH
* Patient
* Prescription

Visitors:

* DICOM RT Export
* PDF Report
* QA Validation
* Dose Statistics
* Clinical Audit
* AI Quality Analyzer

Requirements:

* Stable element hierarchy.
* Frequent addition of new operations.
* Integration with Composite for hierarchical treatment plans.
* Iterator for traversal.
* Thread-safe report generation.

**Do not implement the solution yet.** Focus on architecture, double dispatch, object responsibilities, and extensibility.

---

# ⭐ Architect's Insight: Composite + Iterator + Visitor

In large engineering applications, these three patterns are frequently used together.

```text
Treatment Plan
       │
       ▼
Composite Tree
       │
       ▼
Iterator Traverses
       │
       ▼
Visitor Performs Operation
```

Example:

```text
Treatment Plan
├── Beam 1
├── Beam 2
├── Structure Set
│   ├── PTV
│   ├── Heart
│   └── Lung
└── Dose Matrix

        │
        ▼

PDFVisitor
```

Responsibilities remain clean:

* **Composite** → Organizes the hierarchy.
* **Iterator** → Traverses the hierarchy.
* **Visitor** → Executes one operation on each element.

This is a common architecture in CAD systems, compilers, scene graphs, and medical imaging software.

---

# Key Architectural Takeaway

The **Visitor Pattern** is about **adding new operations without modifying an existing object hierarchy**.

A junior developer thinks:

> "I'll add another method to every class."

A software architect thinks:

> **"The object hierarchy is stable, but new operations keep arriving. I'll move those operations into visitor classes so the domain model stays focused and extensible."**

This approach is especially valuable when you have a mature object model that must support a growing set of reporting, exporting, validation, and analysis features.

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
* ✅ Visitor

Only **one** Behavioral Pattern remains:

1. **Null Object Pattern**

---

## What You'll Learn Next

Type **`NEXT`** to continue with **Lesson 26: Null Object Pattern**, the final Behavioral Pattern.

You'll learn how to eliminate repetitive `nullptr` checks, simplify client code, and design APIs that are safer and easier to use. We'll also explore how this pattern appears in logging systems, authentication, permissions, medical device software, and enterprise applications.
