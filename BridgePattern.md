# Design Patterns Master Course

# Lesson 8 — Bridge Pattern (Structural Pattern)

---

# Before We Start

The **Bridge Pattern** is one of the most misunderstood design patterns.

Many developers confuse it with:

* Adapter
* Strategy
* Abstract Factory

This is because all of them use interfaces.

However, **their purpose is completely different**.

---

# First Understand the Difference

## Adapter

Adapter asks:

> **"How can I make two incompatible classes work together?"**

Problem:

```text
Application

↓

Old API

New API
```

Solution:

Translate the interface.

---

## Bridge

Bridge asks:

> **"How can I prevent an explosion of subclasses when two independent dimensions change?"**

Problem:

```text
CircleOpenGL
CircleVulkan
CircleDirectX

RectangleOpenGL
RectangleVulkan
RectangleDirectX

TriangleOpenGL
TriangleVulkan
TriangleDirectX
```

The number of classes keeps growing.

Bridge separates these dimensions.

---

# The Architect's Way of Thinking

Suppose you are designing a **Medical Imaging Viewer**.

You have:

Shapes:

```text
CT Viewer

MRI Viewer

PET Viewer
```

Rendering Engines:

```text
CPU Rendering

GPU Rendering

OpenGL

Vulkan
```

Without Bridge:

```text
CTViewerOpenGL

CTViewerCPU

CTViewerGPU

MRIViewerOpenGL

MRIViewerCPU

MRIViewerGPU

PETViewerOpenGL

PETViewerCPU

PETViewerGPU
```

Every new viewer multiplies with every renderer.

This is called the **Cartesian Product Problem**.

---

## Mathematical View

Suppose:

4 viewers

×

5 rendering engines

=

20 classes

Now add:

3 export methods

Now:

```text
4 × 5 × 3

=

60 Classes
```

As the application grows:

```text
N × M × K × ...
```

The number of subclasses explodes.

Bridge was created to solve exactly this problem.

---

# 1. Introduction

## What is the Bridge Pattern?

The **Bridge Pattern** separates an abstraction from its implementation so that both can vary independently.

In simple words:

> **Split one big inheritance hierarchy into two independent hierarchies connected by composition.**

---

## Why was it created?

Large systems often have **multiple dimensions of change**.

Example:

Medical Viewer

can vary by:

* Viewer Type

AND

* Rendering Backend

These dimensions are independent.

Using inheritance alone combines them, creating too many subclasses.

---

## Category

**Structural Pattern**

---

## What problem does it solve?

It avoids subclass explosion by replacing inheritance with composition.

Instead of:

```text
CTViewerOpenGL
CTViewerVulkan
CTViewerCPU
```

Use:

```text
CTViewer

↓

Renderer
```

Now any viewer can work with any renderer.

---

# 2. Problem Statement

Imagine you are developing a **CAD Application**.

You have:

Shapes:

```text
Circle

Rectangle

Polygon
```

Rendering APIs:

```text
OpenGL

Vulkan

DirectX
```

Without Bridge:

```text
CircleOpenGL

CircleVulkan

CircleDirectX

RectangleOpenGL

RectangleVulkan

RectangleDirectX

PolygonOpenGL

PolygonVulkan

PolygonDirectX
```

Nine classes.

Now add:

Metal

Now:

```text
CircleMetal

RectangleMetal

PolygonMetal
```

Every renderer requires modifying every shape hierarchy.

Maintenance becomes expensive.

---

# 3. Motivation

Architects observed that some inheritance hierarchies actually represent **multiple independent reasons to change**.

In the previous example:

Reason 1:

New Shape

Reason 2:

New Renderer

These reasons are unrelated.

Changing one should not require modifying the other.

Bridge separates them.

---

# 4. Real-World Analogy

## TV and Remote Control

Imagine televisions.

TV Brands:

```text
Sony

Samsung

LG
```

Remote Controls:

```text
Basic Remote

Smart Remote

Voice Remote
```

Without Bridge:

```text
SonyBasicRemote

SonySmartRemote

SonyVoiceRemote

SamsungBasicRemote

SamsungSmartRemote

SamsungVoiceRemote
```

Too many combinations.

Instead:

```text
Remote

↓

TV
```

Any remote can control any TV that implements the expected interface.

### Mapping

| Real World   | Software             |
| ------------ | -------------------- |
| Remote       | Abstraction          |
| TV           | Implementor          |
| Sony TV      | Concrete Implementor |
| Smart Remote | Refined Abstraction  |

The remote delegates operations to the TV.

---

# 5. Software Scenario

Bridge is useful whenever two independent dimensions evolve separately.

### Desktop Applications

* Document types × Export formats
* Charts × Rendering engines

### Qt Applications

Suppose you have custom widgets:

```text
MedicalImageWidget

DoseWidget

DVHWidget
```

Rendering options:

```text
QPainter

OpenGL

Vulkan
```

The widget holds a pointer to a renderer interface instead of inheriting from every renderer combination.

### CAD Software

Shapes:

* Circle
* Line
* Polygon

Renderers:

* OpenGL
* Vulkan
* Software Renderer

### Medical Imaging

Viewers:

* CT Viewer
* MRI Viewer
* PET Viewer

Renderers:

* CPU
* GPU
* OpenGL
* Vulkan

### Game Engines

Entities:

* Player
* Enemy
* NPC

Rendering backends:

* DirectX
* Vulkan
* Metal

---

# 6. UML Class Diagram

```text
                 +----------------------+
                 |     Abstraction      |
                 +----------------------+
                 | - implementor        |
                 | +operation()         |
                 +----------+-----------+
                            |
                            | has-a
                            |
                            v
                 +----------------------+
                 |     Implementor      |
                 +----------------------+
                 | +operationImpl()     |
                 +----------+-----------+
                            ^
              --------------|--------------
              |                             |
     +------------------+        +------------------+
     | ConcreteImplA    |        | ConcreteImplB    |
     +------------------+        +------------------+

              ^
              |
    +----------------------+
    | RefinedAbstraction   |
    +----------------------+
```

Notice:

This is **composition**, not inheritance, between the abstraction and implementor.

---

# Responsibilities

### Abstraction

High-level logic.

Delegates work to the implementor.

---

### Implementor

Defines implementation interface.

---

### Concrete Implementor

Real implementations.

Example:

```text
OpenGLRenderer

VulkanRenderer

CPURenderer
```

---

### Refined Abstraction

Specialized abstraction.

Example:

```text
CTViewer

MRIViewer
```

---

# 7. Participants

## Abstraction

Example:

```text
ImageViewer
```

Contains:

```text
Renderer*
```

---

## Refined Abstraction

Examples:

```text
CTViewer

MRIViewer
```

---

## Implementor

Example:

```text
Renderer
```

Declares:

```cpp
drawImage()
```

---

## Concrete Implementors

Examples:

```text
OpenGLRenderer

VulkanRenderer

SoftwareRenderer
```

---

## Client

Creates:

```text
Viewer

+

Renderer
```

and connects them.

---

# 8. Collaboration

Runtime Flow

```text
Client

↓

Create VulkanRenderer

↓

Create CTViewer

↓

Inject Renderer

↓

CTViewer.display()

↓

Renderer.drawImage()
```

The viewer delegates rendering to the renderer.

---

# 9. C++ Example

```cpp
#include <iostream>
#include <memory>

// Implementor
class Renderer
{
public:
    virtual ~Renderer() = default;
    virtual void drawCircle(float radius) = 0;
};

// Concrete Implementors
class OpenGLRenderer : public Renderer
{
public:
    void drawCircle(float radius) override
    {
        std::cout << "Drawing Circle using OpenGL: "
                  << radius << '\n';
    }
};

class VulkanRenderer : public Renderer
{
public:
    void drawCircle(float radius) override
    {
        std::cout << "Drawing Circle using Vulkan: "
                  << radius << '\n';
    }
};

// Abstraction
class Shape
{
public:
    explicit Shape(std::shared_ptr<Renderer> renderer)
        : m_renderer(std::move(renderer))
    {}

    virtual ~Shape() = default;
    virtual void draw() = 0;

protected:
    std::shared_ptr<Renderer> m_renderer;
};

// Refined Abstraction
class Circle : public Shape
{
public:
    Circle(std::shared_ptr<Renderer> renderer,
           float radius)
        : Shape(std::move(renderer)),
          m_radius(radius)
    {}

    void draw() override
    {
        m_renderer->drawCircle(m_radius);
    }

private:
    float m_radius;
};

int main()
{
    auto renderer =
        std::make_shared<OpenGLRenderer>();

    Circle circle(renderer, 10);

    circle.draw();
}
```

---

## Design Focus

Notice:

The `Circle` knows **what** to draw.

The renderer knows **how** to draw.

These responsibilities are independent.

---

# 10. Qt Example

Imagine a Qt medical viewer.

Widgets:

```text
CTWidget

MRIWidget

PETWidget
```

Renderers:

```text
QPainterRenderer

OpenGLRenderer

VulkanRenderer
```

Each widget stores:

```cpp
std::unique_ptr<Renderer>
```

When the hospital upgrades from OpenGL to Vulkan:

Only replace:

```text
OpenGLRenderer

↓

VulkanRenderer
```

The widgets remain unchanged.

This is exactly how Bridge reduces maintenance.

---

# 11. Medical Software Example

Imagine a **Treatment Planning System (TPS)**.

View Types:

```text
2D Beam View

3D Dose View

DVH View
```

Rendering Backends:

```text
CPU Renderer

CUDA Renderer

OpenGL Renderer

Vulkan Renderer
```

Without Bridge:

```text
DoseViewCPU

DoseViewCUDA

DoseViewOpenGL

DoseViewVulkan

BeamViewCPU

BeamViewCUDA

BeamViewOpenGL

BeamViewVulkan
```

Eight classes already.

Now add:

```text
Structure View
```

Four more classes.

Instead:

```text
DoseView

↓

Renderer
```

The renderer becomes interchangeable.

This architecture allows:

* Software rendering for testing.
* GPU rendering for production.
* Future Vulkan migration without redesigning every view.

This is a common architectural approach in high-performance visualization software.

---

# 12. Advantages

### Independent Evolution

Abstraction and implementation evolve separately.

### Avoids Class Explosion

No need for every possible subclass combination.

### Loose Coupling

High-level code depends only on the implementor interface.

### Extensibility

Add a new renderer without changing existing viewers.

Add a new viewer without changing existing renderers.

### Better Maintainability

Each hierarchy has a single reason to change.

---

# 13. Disadvantages

### More Classes

Bridge introduces additional abstractions.

### More Indirection

Calls pass through an extra layer.

### Overengineering

For small applications with only one implementation, Bridge may be unnecessary.

### When NOT to Use

Avoid Bridge when:

* there is only one implementation,
* there is no expectation that the abstraction and implementation will evolve independently,
* inheritance alone remains simple.

---

# 14. Best Practices

* Identify independent dimensions of change before introducing Bridge.
* Prefer composition over inheritance between the abstraction and implementation.
* Keep the implementor interface focused and stable.
* Inject the implementor through the constructor or dependency injection.
* Avoid leaking implementor-specific details into the abstraction.

---

# 15. Common Mistakes

### Mistake 1

Confusing Bridge with Adapter.

**Adapter** fixes incompatibility after the fact.

**Bridge** is designed upfront to allow independent evolution.

---

### Mistake 2

Putting rendering logic inside the abstraction.

The abstraction should delegate, not implement low-level details.

---

### Mistake 3

Creating one implementor interface with dozens of unrelated methods.

Keep interfaces cohesive.

---

### Mistake 4

Using inheritance instead of composition between the abstraction and implementation.

That defeats the purpose of the pattern.

---

# 16. Pattern Variations

### 1. Runtime Bridge

The implementor is selected at runtime.

Example:

```text
Configuration

↓

Choose OpenGL

↓

Inject Renderer
```

---

### 2. Plugin-Based Bridge

New renderers are added as plugins without modifying the viewer hierarchy.

---

### 3. Multi-Level Bridge

More than two dimensions are separated.

Example:

```text
Viewer

↓

Renderer

↓

GPU Driver
```

Each layer abstracts another independent concern.

---

# 17. Related Patterns

| Pattern          | Difference                                                                  |
| ---------------- | --------------------------------------------------------------------------- |
| Adapter          | Makes incompatible interfaces compatible.                                   |
| Bridge           | Separates abstraction from implementation so both can evolve independently. |
| Strategy         | Encapsulates interchangeable algorithms or behaviors.                       |
| Abstract Factory | Creates related families of objects.                                        |
| Decorator        | Adds responsibilities dynamically.                                          |

## Bridge vs Strategy (Important Interview Question)

Many candidates confuse these two.

### Strategy

Focuses on **behavior**.

Example:

```text
Compression

↓

ZIP

RAR

7Z
```

You choose one algorithm.

---

### Bridge

Focuses on **architecture**.

Example:

```text
Viewer

↓

Renderer
```

The goal is to separate two dimensions that evolve independently.

---

# 18. Industry Usage

Bridge is widely used in systems where multiple implementations must support multiple abstractions.

* **Microsoft:** UI frameworks separating controls from platform-specific rendering.
* **Google:** Platform abstraction layers and graphics systems.
* **Adobe:** Document models separated from rendering engines and output devices.
* **Qt:** The Qt Platform Abstraction (QPA) concept separates Qt APIs from platform-specific implementations. While it is not a textbook GoF Bridge everywhere, it reflects the same architectural principle of separating abstraction from platform implementation.
* **ZEISS / Siemens / Philips:** Medical viewers separated from CPU/GPU rendering engines and hardware-specific visualization backends.
* **Autodesk:** CAD models separated from graphics APIs and rendering pipelines.
* **Game Engines:** Scene objects separated from DirectX, Vulkan, Metal, or OpenGL renderers.

The architectural goal is **independent evolution of high-level features and low-level implementations**.

---

# 19. Interview Questions

## Beginner

1. What problem does the Bridge Pattern solve?
2. Why is composition preferred over inheritance in Bridge?
3. What are the roles of Abstraction and Implementor?

---

## Intermediate

1. How does Bridge prevent class explosion?
2. How is Bridge different from Adapter?
3. When would you choose Bridge over inheritance?

---

## Advanced

1. How would you design a rendering architecture supporting OpenGL, Vulkan, CUDA, and Software rendering?
2. How would you add a new rendering backend without modifying existing viewers?
3. How would dependency injection improve a Bridge-based architecture?

---

## Scenario-Based

Your medical imaging application supports:

* CT Viewer
* MRI Viewer
* PET Viewer

It must also support:

* CPU Rendering
* OpenGL
* Vulkan
* CUDA

Would you create 12 subclasses or introduce a Bridge? Explain the architectural reasoning.

---

## Architecture

Design a Bridge architecture for a **Treatment Planning System** with:

**View Types**

* Beam View
* Dose View
* DVH View
* Structure View

**Rendering Backends**

* CPU
* OpenGL
* Vulkan
* CUDA

Explain:

* the abstraction hierarchy,
* the implementor hierarchy,
* how a new renderer is added,
* how a new view is added,
* and why this design scales better than inheritance.

---

# 20. Practice Exercises

### Beginner Exercise

Design a `Shape` hierarchy (`Circle`, `Rectangle`) that works with two renderers (`OpenGLRenderer`, `VulkanRenderer`) using the Bridge Pattern. Draw the UML and explain the collaboration.

---

### Intermediate Exercise

Design a Qt charting framework where:

* `LineChart`
* `BarChart`
* `PieChart`

can each render using:

* `QPainter`
* `OpenGL`
* `SVG`

Use Bridge to avoid subclass explosion.

---

### Advanced Exercise

Design a **Bridge-based architecture** for a **Medical Imaging Platform** supporting:

**Image Viewers**

* CT
* MRI
* PET
* Ultrasound

**Rendering Engines**

* CPU
* OpenGL
* Vulkan
* CUDA

Additionally, the system should support future rendering engines without modifying existing viewers.

Explain:

* the class hierarchy,
* ownership relationships,
* runtime object creation,
* dependency injection strategy,
* and how this design supports future scalability.

**Do not implement the solution yet.** Focus on architecture and responsibilities.

---

# Key Architectural Takeaway

The **Bridge Pattern** is not about translating interfaces.

It is about **preventing combinatorial class explosion** by separating two independent dimensions of change.

A junior developer thinks:

> "I'll create a new subclass for every combination."

A software architect thinks:

> **"These are two separate concerns. I'll model them as two independent hierarchies connected through composition, so each can evolve without affecting the other."**

This mindset is essential for building scalable systems in CAD, graphics, Qt applications, and medical imaging software where both features and implementations evolve continuously.

When you're ready, type **`NEXT`** to continue with **Lesson 9: Composite Pattern**.
