# Design Patterns Master Course

# Lesson 10 — Decorator Pattern (Structural Pattern)

---

# Before We Start

The **Decorator Pattern** is one of the most useful patterns in modern software.

It is used extensively in:

* Qt
* C++ I/O Streams
* Java I/O
* GUI Frameworks
* Network Libraries
* Image Processing
* Medical Software
* Game Engines

Many developers confuse it with:

* Inheritance
* Composite
* Proxy

Today, you'll understand **why Decorator exists** and **why architects often prefer it over inheritance**.

---

# The Architect's Way of Thinking

Suppose you have a `TextEditor`.

Users want different features:

* Spell Check
* Grammar Check
* Auto Save
* Syntax Highlighting
* Line Numbers
* Code Folding

A junior developer might design:

```text
TextEditor

↓

SpellCheckEditor

↓

GrammarSpellCheckEditor

↓

GrammarSpellCheckAutoSaveEditor

↓

GrammarSpellCheckAutoSaveSyntaxEditor

↓

GrammarSpellCheckAutoSaveSyntaxLineNumberEditor
```

This quickly becomes impossible to maintain.

Let's calculate it.

Suppose you have **6 optional features**.

Each feature can be:

* Enabled
* Disabled

Possible combinations:

```text
2⁶ = 64
```

Need **64 subclasses**.

Now imagine **10 optional features**.

```text
2¹⁰ = 1024
```

This is called the **Subclass Explosion Problem**.

Decorator solves this.

---

# 1. Introduction

## What is the Decorator Pattern?

The **Decorator Pattern** attaches additional responsibilities to an object **dynamically** without modifying its class.

Simple definition:

> **Decorator wraps an existing object and adds new behavior before or after delegating to it.**

Notice the word:

**Wraps**

The original object still exists.

---

## Why was it created?

Inheritance is static.

Once you create:

```text
PremiumCoffee
```

it always has that behavior.

But users often want:

Today:

```text
Coffee + Milk
```

Tomorrow:

```text
Coffee + Milk + Sugar
```

Later:

```text
Coffee + Milk + Sugar + Cream
```

You don't want a new subclass for every combination.

Instead, wrap the object.

---

## Category

**Structural Pattern**

---

## What problem does it solve?

It avoids creating dozens (or hundreds) of subclasses for every feature combination.

Instead of inheritance:

```text
Image

↓

ZoomableImage

↓

ZoomableAnnotatedImage

↓

ZoomableAnnotatedCachedImage

↓

ZoomableAnnotatedCachedPrintableImage
```

Use:

```text
Image

↓

AnnotationDecorator

↓

CacheDecorator

↓

PrintDecorator
```

Each feature becomes an independent wrapper.

---

# 2. Problem Statement

Imagine a **DICOM Viewer**.

Basic Viewer:

```text
Display Image
```

Customers request:

* Measurement Tool
* Annotation
* Zoom
* Window/Level
* Image Cache
* Watermark
* AI Overlay
* Dose Overlay

Without Decorator:

```text
Viewer

↓

ZoomViewer

↓

ZoomMeasurementViewer

↓

ZoomMeasurementAnnotationViewer

↓

ZoomMeasurementAnnotationCacheViewer

...
```

The number of subclasses grows rapidly.

---

## Why is maintenance difficult?

Suppose management asks:

> "Add AI Segmentation Overlay."

You now need to update or create many subclass combinations.

This violates the **Open/Closed Principle (OCP)**.

---

# 3. Motivation

Architects realized:

Many behaviors are **optional** and **independent**.

Examples:

* Logging
* Compression
* Encryption
* Caching
* Validation

These should be added **without modifying existing classes**.

Decorator allows behavior to be assembled at runtime.

---

# 4. Real-World Analogy

## Coffee Shop

Base product:

```text
Coffee
```

Optional additions:

```text
Milk

Sugar

Chocolate

Whipped Cream
```

Customer 1:

```text
Coffee
```

Customer 2:

```text
Coffee

↓

Milk
```

Customer 3:

```text
Coffee

↓

Milk

↓

Sugar
```

Customer 4:

```text
Coffee

↓

Milk

↓

Sugar

↓

Chocolate
```

No new coffee classes are created.

Each topping wraps the previous drink.

---

## Mapping

| Coffee Shop | Software         |
| ----------- | ---------------- |
| Coffee      | Component        |
| Milk        | Decorator        |
| Sugar       | Decorator        |
| Chocolate   | Decorator        |
| Final Drink | Decorated Object |

---

# 5. Software Scenario

Decorator appears whenever features can be added independently.

### Desktop Applications

* Spell checking
* Syntax highlighting
* Auto-save

---

### Qt Applications

Suppose you have a custom image widget.

Optional features:

* Grid Overlay
* Ruler
* Measurement Tool
* Annotation Layer
* Watermark

Instead of subclasses:

```text
ImageWidget

↓

GridDecorator

↓

MeasurementDecorator

↓

AnnotationDecorator
```

Each decorator adds one responsibility.

---

### CAD Software

Optional layers:

* Dimension Overlay
* Snap Grid
* Selection Highlight
* Constraint Visualization

---

### Medical Imaging

Optional processing:

* AI Segmentation
* Dose Overlay
* ROI Display
* Window/Level Filter
* Image Enhancement

---

### Network Libraries

Decorators add:

* Compression
* Encryption
* Logging

---

# 6. UML Class Diagram

```text
                 +----------------------+
                 |      Component       |
                 +----------------------+
                 | +operation()         |
                 +----------+-----------+
                            ^
                            |
                 +----------------------+
                 | ConcreteComponent    |
                 +----------------------+

                            ^
                            |
                 +----------------------+
                 |      Decorator       |
                 +----------------------+
                 | - component          |
                 | +operation()         |
                 +----------+-----------+
                            ^
                ------------|------------
                |                         |
      +------------------+      +------------------+
      | Decorator A      |      | Decorator B      |
      +------------------+      +------------------+
```

Notice:

Decorator **HAS A**

```text
Component
```

This is composition.

---

# Responsibilities

## Component

Common interface.

---

## Concrete Component

Original object.

---

## Decorator

Wraps another component.

Delegates calls.

---

## Concrete Decorator

Adds extra behavior.

---

# 7. Participants

## Component

Example:

```text
ImageViewer
```

---

## Concrete Component

```text
BasicImageViewer
```

---

## Decorator

Contains:

```cpp
Component*
```

Delegates work.

---

## Concrete Decorators

Examples:

```text
ZoomDecorator

AnnotationDecorator

CacheDecorator

MeasurementDecorator
```

---

## Client

Builds the desired chain.

---

# 8. Collaboration

Runtime Flow

```text
Basic Viewer

↓

Annotation Decorator

↓

Cache Decorator

↓

Measurement Decorator

↓

Display()
```

Execution order:

```text
Measurement

↓

Cache

↓

Annotation

↓

Basic Viewer
```

Each decorator:

1. Does some work.
2. Calls the wrapped object.
3. Optionally performs work afterward.

---

# 9. C++ Example

```cpp
#include <iostream>
#include <memory>

// Component
class Coffee
{
public:
    virtual ~Coffee() = default;
    virtual double cost() const = 0;
    virtual void description() const = 0;
};

// Concrete Component
class BasicCoffee : public Coffee
{
public:
    double cost() const override
    {
        return 50;
    }

    void description() const override
    {
        std::cout << "Coffee";
    }
};

// Decorator
class CoffeeDecorator : public Coffee
{
public:
    explicit CoffeeDecorator(std::shared_ptr<Coffee> coffee)
        : m_coffee(std::move(coffee))
    {}

protected:
    std::shared_ptr<Coffee> m_coffee;
};

// Concrete Decorator
class MilkDecorator : public CoffeeDecorator
{
public:
    using CoffeeDecorator::CoffeeDecorator;

    double cost() const override
    {
        return m_coffee->cost() + 10;
    }

    void description() const override
    {
        m_coffee->description();
        std::cout << " + Milk";
    }
};

class SugarDecorator : public CoffeeDecorator
{
public:
    using CoffeeDecorator::CoffeeDecorator;

    double cost() const override
    {
        return m_coffee->cost() + 5;
    }

    void description() const override
    {
        m_coffee->description();
        std::cout << " + Sugar";
    }
};

int main()
{
    std::shared_ptr<Coffee> coffee =
        std::make_shared<BasicCoffee>();

    coffee = std::make_shared<MilkDecorator>(coffee);
    coffee = std::make_shared<SugarDecorator>(coffee);

    coffee->description();
    std::cout << "\nCost = " << coffee->cost();
}
```

---

## Design Focus

Notice:

We never modified:

```text
BasicCoffee
```

We simply wrapped it.

This is exactly how Decorator works.

---

# 10. Qt Example

Suppose you're building a **Medical Image Widget**.

Base widget:

```text
MedicalImageWidget
```

Optional decorators:

```text
MeasurementDecorator

↓

AnnotationDecorator

↓

CrosshairDecorator

↓

WatermarkDecorator
```

Runtime:

```cpp
widget =
    WatermarkDecorator(
        AnnotationDecorator(
            MeasurementDecorator(
                MedicalImageWidget)));
```

Each feature is independent.

You can enable or disable them dynamically.

Another real-world analogy is Qt's paint pipeline, where multiple painting operations build on one another, although Qt does not expose a textbook GoF Decorator class hierarchy for painting.

---

# 11. Medical Software Example

Imagine a **CT Viewer**.

Base Viewer:

```text
CT Image
```

Radiologist enables:

```text
Window Level

↓

ROI

↓

Measurement

↓

Dose Overlay

↓

AI Tumor Detection

↓

Annotations
```

Instead of creating:

```text
CTViewerWithROIAndDoseAndAIAndMeasurement
```

Use decorators.

Example chain:

```text
Basic CT Viewer

↓

WindowLevelDecorator

↓

ROIDecorator

↓

MeasurementDecorator

↓

DoseDecorator

↓

AITumorDecorator
```

Each decorator adds one clinical capability while preserving the core viewer.

---

# 12. Advantages

### Open/Closed Principle

Add features without modifying existing classes.

### Runtime Flexibility

Features can be combined dynamically.

### Avoids Subclass Explosion

No need for hundreds of subclasses.

### Single Responsibility

Each decorator has one job.

### Reusability

Decorators can be reused with different components.

---

# 13. Disadvantages

### Many Small Classes

Each feature often becomes its own class.

### Debugging

Behavior depends on the order of decorators.

### Object Identity

Multiple wrappers can make it harder to identify the underlying object.

### When NOT to Use

Avoid Decorator when:

* behavior never changes,
* inheritance remains simple,
* optional features are not needed.

---

# 14. Best Practices

* Keep each decorator focused on one responsibility.
* Preserve the component interface.
* Document the expected order when decorators depend on each other.
* Use composition, not inheritance, to extend behavior.
* Avoid storing unrelated business state inside decorators.

---

# 15. Common Mistakes

### Mistake 1

Putting several responsibilities into one decorator.

Example:

```text
Logging

+

Caching

+

Encryption
```

These should usually be separate decorators.

---

### Mistake 2

Breaking the component interface.

Clients should not need to know whether they are talking to a decorator or the original component.

---

### Mistake 3

Decorator order bugs.

Example:

```text
Compression

↓

Encryption
```

is **not** equivalent to:

```text
Encryption

↓

Compression
```

The sequence matters.

---

### Mistake 4

Confusing Decorator with inheritance.

Decorators **wrap**.

Inheritance **extends**.

---

# 16. Pattern Variations

## 1. Transparent Decorator

Decorator exposes exactly the same interface as the component.

Most common approach.

---

## 2. Stateful Decorator

Decorator stores additional state.

Example:

```text
Zoom Level

Annotation Color

Measurement Units
```

---

## 3. Lazy Decorator

Performs work only when needed.

Useful for:

* image loading,
* caching,
* expensive calculations.

---

# 17. Related Patterns

| Pattern   | Difference                                                             |
| --------- | ---------------------------------------------------------------------- |
| Decorator | Adds behavior dynamically by wrapping an object.                       |
| Composite | Represents tree structures.                                            |
| Proxy     | Controls access to an object without changing its core responsibility. |
| Adapter   | Changes the interface of an object.                                    |
| Bridge    | Separates abstraction from implementation.                             |

---

## Decorator vs Proxy (Very Important)

Both contain another object.

### Proxy

Purpose:

```text
Control Access
```

Examples:

* Authentication
* Lazy loading
* Remote object

---

### Decorator

Purpose:

```text
Add Features
```

Examples:

* Logging
* Compression
* Watermark
* Measurement

This distinction is a favorite interview topic.

---

# 18. Industry Usage

Decorator is widely used because modern software frequently composes features at runtime.

* **Microsoft:** Stream libraries, middleware pipelines, and editor features.
* **Google:** Request/response middleware, logging, tracing, and caching layers.
* **Adobe:** Graphics effects, image filters, and document processing.
* **Qt:** Many extensible UI and paint-related designs rely on composition and wrapper concepts similar to Decorator.
* **ZEISS / Siemens / Philips:** Medical viewers with optional overlays, measurement tools, AI analysis, and annotation layers.
* **Autodesk:** CAD visualization features such as dimensions, grids, snapping, and highlighting.
* **Game Engines:** Buffs, visual effects, AI modifiers, and gameplay enhancements attached dynamically to entities.

The architectural goal is **adding responsibilities without modifying existing classes or creating countless subclasses**.

---

# 19. Interview Questions

## Beginner

1. What problem does the Decorator Pattern solve?
2. Why is Decorator preferred over inheritance for optional features?
3. What is the role of the Component interface?

---

## Intermediate

1. Explain how Decorator supports the Open/Closed Principle.
2. Why is composition central to Decorator?
3. How does the order of decorators affect behavior?

---

## Advanced

1. Design a decorator chain for an image-processing pipeline that performs logging, caching, and AI segmentation.
2. How would you debug a system with many nested decorators?
3. How would you ensure decorators remain independent and reusable?

---

## Scenario-Based

Your CT Viewer must support:

* Window/Level
* ROI Display
* Measurements
* Dose Overlay
* AI Segmentation

Each feature should be independently enabled or disabled.

Would you create subclasses or use Decorator? Explain your architecture.

---

## Architecture

Design a Decorator-based architecture for a **Treatment Planning System** where a `DoseEngine` can be extended with optional capabilities:

* Logging
* Performance Monitoring
* Result Caching
* Validation
* Audit Trail

Explain:

* the base component,
* each decorator,
* the execution order,
* and how new capabilities can be added without modifying the `DoseEngine`.

---

# 20. Practice Exercises

### Beginner Exercise

Design a `Coffee` system using Decorator with:

* Milk
* Sugar
* Chocolate

Draw the UML and explain how the decorators collaborate.

---

### Intermediate Exercise

Design a Qt image viewer that supports optional decorators for:

* Grid Overlay
* Crosshair
* Annotation
* Measurement

Explain how features can be enabled or disabled at runtime.

---

### Advanced Exercise

Design a **Decorator architecture** for a **Medical Imaging Platform** where an `ImageProcessor` can be dynamically extended with:

* Noise Reduction
* Contrast Enhancement
* AI Segmentation
* Watermark
* Audit Logging

Requirements:

* Each feature must be independent.
* The processing order should be configurable.
* New processing steps should be added without changing existing classes.
* Explain ownership, object composition, and how the client assembles the processing pipeline.

**Do not implement the solution yet.** Focus on architecture and responsibilities.

---

# Key Architectural Takeaway

The **Decorator Pattern** is about **extending behavior through composition instead of inheritance**.

A junior developer thinks:

> "I'll create another subclass with the extra feature."

A software architect thinks:

> **"These features are optional and combinable. I'll wrap the object with decorators so I can add or remove capabilities dynamically without changing the original class."**

This mindset produces software that is more flexible, extensible, and maintainable—especially in GUI frameworks, middleware, image-processing pipelines, and medical imaging applications.

---

## What You'll Learn Next

Type **`NEXT`** to continue with **Lesson 11: Facade Pattern**, where you'll learn how architects simplify complex subsystems behind a clean, easy-to-use interface.
