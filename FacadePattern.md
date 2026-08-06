# Design Patterns Master Course

# Lesson 11 — Facade Pattern (Structural Pattern)

---

# Before We Start

Imagine you join **ZEISS** and your task is:

> "Display a CT image."

You think:

> "Easy! I'll write `viewer.showImage()`."

Then you open the codebase.

You discover that before displaying one image, the system must:

```text
Read DICOM File
↓

Parse Metadata
↓

Validate Patient Information
↓

Load Pixel Data
↓

Decompress Image
↓

Apply Window/Level
↓

Convert Pixel Format
↓

Create GPU Texture
↓

Initialize Renderer
↓

Display Image
```

There are **20 classes** involved.

Should every developer know all these classes?

A junior developer might say:

> "Yes, I'll call each class one by one."

A software architect says:

> **"No. I'll provide one simple interface that hides all this complexity."**

That is the **Facade Pattern**.

---

# The Architect's Way of Thinking

Imagine driving a car.

To start the engine internally, many things happen:

```text
Battery

↓

Fuel Pump

↓

ECU

↓

Starter Motor

↓

Ignition

↓

Engine Starts
```

Do you call each subsystem?

No.

You simply turn the key or press the Start button.

The **Start button is a Facade**.

It hides a complex subsystem.

---

# 1. Introduction

## What is the Facade Pattern?

The **Facade Pattern** provides a **simple, unified interface** to a complex subsystem.

Simple definition:

> **Facade hides complexity behind a simple API.**

---

## Why was it created?

Large systems naturally become complex.

Example:

A Medical Imaging System contains:

* DICOM Loader
* Database
* GPU Renderer
* Cache
* Logger
* Image Filters
* AI Engine
* Report Generator

A new developer should not need to understand all of them to display one image.

Facade reduces the learning curve.

---

## Category

**Structural Pattern**

---

## What problem does it solve?

Without Facade:

```cpp
DicomReader reader;

MetadataParser parser;

GPUTexture texture;

ImageCache cache;

Renderer renderer;

WindowLevel wl;

...

...
```

The client becomes tightly coupled to many subsystem classes.

With Facade:

```cpp
ImageViewer viewer;

viewer.display("patient001.dcm");
```

One call.

Same functionality.

---

# 2. Problem Statement

Imagine a **College ERP System**.

To admit a student, the client must call:

```text
Validate Documents

↓

Create Student

↓

Create Roll Number

↓

Allocate Department

↓

Allocate Class

↓

Generate Fee Record

↓

Generate ERP Login

↓

Send Email

↓

Generate ID Card
```

Without Facade:

```cpp
validate();

createStudent();

createRoll();

allocateDepartment();

generateFee();

createLogin();

sendMail();

createID();
```

Every client repeats this workflow.

If admission rules change, every client changes.

---

# 3. Motivation

Architects noticed two recurring problems:

1. Clients know too much about subsystem internals.
2. Changes inside the subsystem ripple into client code.

The solution:

Create one class that exposes only the operations clients actually need.

---

# 4. Real-World Analogy

## ATM Machine

Inside an ATM:

```text
Card Reader

↓

PIN Validator

↓

Bank Server

↓

Account Verification

↓

Balance Check

↓

Cash Dispenser

↓

Receipt Printer
```

Do you call each component?

No.

You simply press:

```text
Withdraw Cash
```

The ATM coordinates everything.

The ATM user interface is a **Facade**.

---

## Mapping

| ATM            | Software  |
| -------------- | --------- |
| ATM Screen     | Facade    |
| Bank Server    | Subsystem |
| Cash Dispenser | Subsystem |
| Card Reader    | Subsystem |
| Customer       | Client    |

---

# 5. Software Scenario

Facade appears whenever a subsystem becomes too complex.

### Desktop Applications

* Export Document
* Print Document
* Open Project

---

### Qt Applications

Imagine a video player.

Internally:

* File Reader
* Decoder
* Audio Engine
* Video Renderer
* Subtitle Loader

Expose only:

```cpp
player.play(file);
```

The player acts as a facade over the multimedia subsystem.

---

### CAD Software

Opening a CAD project requires:

* Load geometry
* Load materials
* Load constraints
* Build scene graph
* Initialize renderer

Expose:

```cpp
cad.openProject(file);
```

---

### Medical Imaging

Opening a DICOM study requires:

* Read DICOM
* Validate tags
* Decode pixels
* Build image pyramid
* Load overlays
* Prepare renderer

Expose:

```cpp
viewer.openStudy(folder);
```

---

### Game Engines

Loading a game level:

```text
Load Meshes

↓

Load Textures

↓

Load Physics

↓

Load Audio

↓

Spawn Objects
```

Expose:

```cpp
engine.loadLevel();
```

---

# 6. UML Class Diagram

```text
                 +----------------------+
                 |        Client        |
                 +----------+-----------+
                            |
                            v
                 +----------------------+
                 |        Facade        |
                 +----------------------+
                 | +operation()         |
                 +----------+-----------+
                            |
         ---------------------------------------------
         |              |             |              |
         v              v             v              v
+---------------+ +---------------+ +---------------+ +---------------+
| Subsystem A   | | Subsystem B   | | Subsystem C   | | Subsystem D   |
+---------------+ +---------------+ +---------------+ +---------------+
```

---

## Responsibilities

### Facade

Coordinates subsystem objects.

Provides simplified APIs.

---

### Subsystems

Perform actual work.

Remain independent.

They do **not** know the Facade exists.

---

### Client

Uses only the Facade.

---

# 7. Participants

## Facade

Example:

```text
ImageViewerFacade
```

Methods:

```cpp
openImage()

exportImage()

printImage()
```

---

## Subsystems

Examples:

```text
DicomLoader

ImageDecoder

Renderer

Cache

Logger
```

---

## Client

Uses:

```cpp
viewer.openImage();
```

instead of calling five different subsystem classes.

---

# 8. Collaboration

Runtime Flow

```text
Client

↓

Facade.openImage()

↓

DicomLoader

↓

Decoder

↓

WindowLevel

↓

Renderer

↓

Display
```

Notice:

The client knows only one class.

---

# 9. C++ Example

```cpp
#include <iostream>
#include <string>

// Subsystems
class DicomLoader
{
public:
    void load(const std::string& file)
    {
        std::cout << "Loading DICOM: " << file << '\n';
    }
};

class ImageDecoder
{
public:
    void decode()
    {
        std::cout << "Decoding image\n";
    }
};

class Renderer
{
public:
    void render()
    {
        std::cout << "Rendering image\n";
    }
};

// Facade
class ImageViewerFacade
{
public:
    void openImage(const std::string& file)
    {
        loader.load(file);
        decoder.decode();
        renderer.render();
    }

private:
    DicomLoader loader;
    ImageDecoder decoder;
    Renderer renderer;
};

int main()
{
    ImageViewerFacade viewer;

    viewer.openImage("patient001.dcm");
}
```

---

## Design Focus

Notice:

The client knows nothing about:

* Decoder
* Renderer
* Loader

Everything is hidden.

---

# 10. Qt Example

Suppose your Qt application exports a report.

Without Facade:

```cpp
QPrinter

QPainter

QPdfWriter

ReportGenerator

ImageExporter

Logger
```

With Facade:

```cpp
ReportFacade report;

report.exportPDF();
```

Internally:

```text
Create PDF

↓

Draw Images

↓

Draw Charts

↓

Save

↓

Log Export
```

The UI remains simple.

---

# 11. Medical Software Example

Imagine a **Treatment Planning System**.

To calculate dose:

Without Facade:

```text
Load Patient

↓

Load CT

↓

Load Structures

↓

Load Machine

↓

Load Beam

↓

Load Prescription

↓

Run Dose Engine

↓

Store Dose

↓

Refresh DVH
```

Every caller must remember this order.

Instead:

```cpp
DoseFacade.calculateDose();
```

Internally:

```text
DoseFacade

↓

PatientManager

↓

BeamManager

↓

MachineManager

↓

DoseEngine

↓

DVHCalculator

↓

Database
```

The physicist-facing UI doesn't need to understand internal workflow.

This makes the system safer and easier to maintain.

---

# 12. Advantages

### Simplicity

Clients see one clean API.

### Loose Coupling

Clients depend on the facade, not many subsystem classes.

### Easier Learning

New developers can start using the system quickly.

### Better Maintainability

Subsystem changes often remain hidden behind the facade.

### Layered Architecture

Facade provides a clear entry point into a subsystem.

---

# 13. Disadvantages

### Can Become a God Object

If every subsystem operation is placed into one huge facade, it becomes difficult to maintain.

### May Hide Useful Functionality

Advanced users may still need direct access to subsystem classes.

### Extra Layer

Adds another abstraction.

### When NOT to Use

Avoid Facade when:

* the subsystem is already simple,
* clients genuinely require fine-grained control,
* hiding complexity provides little benefit.

---

# 14. Best Practices

* Keep the facade focused on common workflows.
* Do not move subsystem logic into the facade; let subsystems perform the work.
* Create multiple facades for large systems instead of one giant facade.
* Allow advanced clients to bypass the facade if necessary.
* Use meaningful names (`ImageViewerFacade`, `DoseCalculationFacade`, `AdmissionFacade`).

---

# 15. Common Mistakes

### Mistake 1

Putting all business logic inside the facade.

The facade should coordinate, not replace, subsystem classes.

---

### Mistake 2

Creating one facade for the entire application.

Large applications often need several focused facades.

---

### Mistake 3

Making subsystem classes depend on the facade.

Subsystems should remain reusable and independent.

---

### Mistake 4

Confusing Facade with Mediator.

Facade simplifies access to a subsystem.

Mediator manages communication between peer objects.

---

# 16. Pattern Variations

## 1. Simple Facade

One class wrapping one subsystem.

---

## 2. Layered Facade

Example:

```text
UI

↓

Application Facade

↓

Business Facade

↓

Subsystems
```

Common in enterprise software.

---

## 3. Service Facade

Frequently used in distributed systems.

Example:

```text
REST API

↓

OrderFacade

↓

Inventory

Payment

Shipping
```

---

# 17. Related Patterns

| Pattern   | Difference                                      |
| --------- | ----------------------------------------------- |
| Facade    | Simplifies access to a subsystem.               |
| Adapter   | Converts one interface into another.            |
| Bridge    | Separates abstraction from implementation.      |
| Decorator | Adds responsibilities dynamically.              |
| Mediator  | Coordinates communication between peer objects. |

---

## Facade vs Adapter (Very Important)

### Adapter

Problem:

```text
Old API

↓

New API
```

Need translation.

---

### Facade

Problem:

```text
Too Many APIs

↓

Need Simplicity
```

Need simplification.

**Adapter changes an interface.**

**Facade hides complexity.**

---

# 18. Industry Usage

Facade is common in almost every large software system.

* **Microsoft:** Office automation APIs, Windows service layers, and framework entry points.
* **Google:** Service-layer APIs that coordinate multiple backend services.
* **Adobe:** Document import/export workflows and rendering pipelines.
* **Qt:** High-level classes often coordinate lower-level platform services behind simpler APIs.
* **ZEISS / Siemens / Philips:** Imaging workflows, acquisition pipelines, reconstruction workflows, and dose calculation services.
* **Autodesk:** Project loading, rendering initialization, and export workflows.
* **Game Engines:** Scene loading, asset management, and gameplay initialization.

The architectural goal is **reducing subsystem complexity exposed to clients**.

---

# 19. Interview Questions

## Beginner

1. What problem does the Facade Pattern solve?
2. Why is Facade considered a Structural Pattern?
3. What is the role of the Facade class?

---

## Intermediate

1. How does Facade reduce coupling?
2. Can clients bypass the Facade?
3. Why should subsystem classes remain independent?

---

## Advanced

1. How would you design multiple facades for a large Treatment Planning System?
2. How would you prevent a Facade from becoming a God Object?
3. How would you test a Facade independently from subsystem implementations?

---

## Scenario-Based

Your DICOM Viewer requires eight subsystem classes to display a study.

Would you expose all eight classes to UI developers or provide a Facade? Explain the benefits and trade-offs.

---

## Architecture

Design a **Facade architecture** for a **Treatment Planning System** with the following subsystems:

* Patient Manager
* CT Loader
* Structure Manager
* Beam Manager
* Dose Engine
* DVH Calculator
* Report Generator
* Database

Design a `TreatmentPlanningFacade` that provides operations such as:

* `loadPatient()`
* `calculateDose()`
* `generateReport()`
* `savePlan()`

Explain which responsibilities belong in the facade and which remain inside subsystem classes.

---

# 20. Practice Exercises

### Beginner Exercise

Design a `HomeTheaterFacade` that simplifies:

* DVD Player
* Projector
* Sound System
* Lights

into operations like:

* `watchMovie()`
* `stopMovie()`

---

### Intermediate Exercise

Design a Qt `ReportFacade` that exports reports to PDF by coordinating:

* Chart Generator
* Image Exporter
* PDF Writer
* Logger

Draw the class relationships.

---

### Advanced Exercise

Design a **Facade architecture** for a **Medical Imaging Platform**.

Subsystems:

* DICOM Import
* Image Decoder
* Image Cache
* GPU Renderer
* AI Segmentation
* Annotation Manager
* Report Generator

The UI should expose only:

* `openStudy()`
* `runAISegmentation()`
* `exportReport()`
* `closeStudy()`

Explain:

* the responsibilities of the facade,
* the subsystem interactions,
* error handling,
* and how the design avoids turning the facade into a God Object.

**Do not implement the solution yet.** Focus on architecture and responsibilities.

---

# Key Architectural Takeaway

The **Facade Pattern** is not about adding new functionality.

It is about **making complex subsystems easy to use**.

A junior developer thinks:

> "I'll call every subsystem directly."

A software architect thinks:

> **"Most clients don't need to understand the internal workflow. I'll provide a clean, high-level interface that coordinates the subsystem while keeping the implementation hidden."**

This mindset is fundamental in enterprise software, Qt desktop applications, CAD systems, and medical imaging platforms, where complex workflows should be presented through simple, stable APIs.

---

## What You'll Learn Next

Type **`NEXT`** to continue with **Lesson 12: Flyweight Pattern**, where you'll learn how architects dramatically reduce memory usage by sharing common object state in large-scale systems such as CAD software, game engines, and medical imaging applications.
