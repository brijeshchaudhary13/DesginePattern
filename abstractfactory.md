# Design Patterns Master Course

# Lesson 3 — Abstract Factory Pattern (Creational Pattern)

---

# Before We Start

Many beginners confuse **Factory Method** and **Abstract Factory** because both create objects.

As an architect, don't memorize definitions. Think about the problem.

Ask yourself:

### Factory Method asks:

> **"Which object should I create?"**

### Abstract Factory asks:

> **"Which family of related objects should I create?"**

This small difference changes the architecture of an entire application.

---

# Factory Method vs Abstract Factory

| Factory Method            | Abstract Factory                      |
| ------------------------- | ------------------------------------- |
| Creates **one product**   | Creates **multiple related products** |
| One factory → One product | One factory → Family of products      |
| Uses inheritance          | Uses composition (most commonly)      |
| Easier                    | More powerful                         |
| Small object creation     | Entire platform/framework creation    |

Think of it this way:

```
Factory Method

ImageFactory
      |
      +------ CT Image
```

```
Abstract Factory

MedicalFactory
      |
      +------ CT Image
      |
      +------ CT Viewer
      |
      +------ CT Exporter
      |
      +------ CT Processor
```

Notice the difference?

One creates **one object**.

The other creates an **entire ecosystem**.

---

# 1. Introduction

## What is the Abstract Factory Pattern?

The **Abstract Factory Pattern** provides an interface for creating **families of related or dependent objects** without specifying their concrete classes.

Instead of asking:

> "Create me a CT image."

You ask:

> "Give me everything needed for a CT system."

---

## Why was it created?

Large software systems often support multiple platforms, vendors, themes, hardware models, or product editions.

Example:

A medical software supports:

```
ZEISS

Siemens

Philips

GE
```

Each vendor requires:

* Viewer
* Image Loader
* Reconstruction Algorithm
* Report Generator
* Exporter

Without Abstract Factory, every module contains vendor-specific logic.

---

## Category

**Creational Pattern**

---

## What problem does it solve?

It guarantees that related objects work together.

For example:

Wrong combination:

```
Philips Viewer

+

GE Image Loader

+

Siemens Exporter
```

These components may be incompatible.

Abstract Factory ensures consistency by creating all components from the same family.

---

# 2. Problem Statement

Imagine you're developing a **Treatment Planning System (TPS)**.

The system supports three LINAC vendors:

```
Varian

Elekta

Siemens
```

Each vendor provides:

```
Beam Model

Machine Configuration

Dose Calculator

MLC Controller

Calibration Data
```

Without Abstract Factory:

```cpp
if(vendor=="Varian")
{
    beam = new VarianBeam();

    calculator = new SiemensDoseCalculator();

    mlc = new ElektaMLC();
}
```

This compiles.

But architecturally it's dangerous.

Now your application mixes incompatible vendor components.

In a medical device, this could produce incorrect calculations or unsupported hardware behavior.

---

## Why does maintenance become difficult?

Suppose you add a fourth vendor:

```
Accuray
```

Now you must update every place that creates:

* Beam
* Dose Calculator
* Exporter
* Report
* Machine Configuration
* MLC

The changes spread throughout the codebase.

---

# 3. Motivation

Developers recognized that related objects should be created together.

If one component belongs to Vendor A, all collaborating components should come from Vendor A unless the architecture explicitly supports mixing.

Abstract Factory centralizes this decision.

The client requests a family, not individual products.

---

# 4. Real-World Analogy

## Car Factory

Imagine buying a car.

You choose:

```
BMW
```

The factory provides:

* Engine
* Steering
* Dashboard
* Seats
* Gearbox

All are designed to work together.

You do **not** receive:

```
BMW Engine

Toyota Dashboard

Tesla Steering

Audi Gearbox
```

Those parts may not fit together.

### Mapping

| Car Factory | Software         |
| ----------- | ---------------- |
| BMW Factory | Concrete Factory |
| Engine      | Product A        |
| Dashboard   | Product B        |
| Steering    | Product C        |
| Customer    | Client           |
| Car Brand   | Product Family   |

The factory ensures compatibility across all products.

---

# 5. Software Scenario

Abstract Factory is useful whenever an application supports multiple coherent families of components.

### Desktop Applications

* Light Theme
* Dark Theme
* High Contrast Theme

Each theme creates:

* Buttons
* Icons
* Colors
* Fonts

---

### Qt Applications

A cross-platform Qt application may support:

* Windows
* Linux
* macOS

Each platform provides:

* Native dialogs
* File pickers
* Menus
* Clipboard integration

The application asks a platform factory to provide the correct set of components.

---

### CAD Software

Different rendering backends:

* OpenGL
* Vulkan
* DirectX

Each backend provides:

* Renderer
* Shader Manager
* Texture Manager
* Buffer Manager

---

### Medical Imaging

Different scanner vendors provide:

* Image Loader
* Metadata Parser
* Reconstruction Pipeline
* Calibration Manager

All must be consistent.

---

### Game Engines

Different graphics APIs:

* DirectX
* Vulkan
* Metal

Each API has its own family of graphics objects.

---

# 6. UML Class Diagram

```
                        +----------------------+
                        |   AbstractFactory    |
                        +----------------------+
                        | +createProductA()    |
                        | +createProductB()    |
                        | +createProductC()    |
                        +----------+-----------+
                                   ^
                 ------------------|------------------
                 |                                    |
     +-----------------------+            +-----------------------+
     | ConcreteFactoryA      |            | ConcreteFactoryB      |
     +-----------------------+            +-----------------------+
     | createProductA()      |            | createProductA()      |
     | createProductB()      |            | createProductB()      |
     | createProductC()      |            | createProductC()      |
     +-----------+-----------+            +-----------+-----------+
                 |                                    |
        Creates Product Family A             Creates Product Family B


          +--------------------+
          | ProductA           |
          +--------------------+
                   ^
        ------------|------------
        |                       |
+----------------+      +----------------+
| ProductA_V1    |      | ProductA_V2    |
+----------------+      +----------------+

          +--------------------+
          | ProductB           |
          +--------------------+
                   ^
        ------------|------------
        |                       |
+----------------+      +----------------+
| ProductB_V1    |      | ProductB_V2    |
+----------------+      +----------------+
```

---

## Responsibilities

### Abstract Factory

Defines methods to create every product in the family.

---

### Concrete Factory

Implements creation for one specific family.

Example:

```
VarianFactory

creates:

VarianBeam

VarianDoseCalculator

VarianMLC
```

---

### Product Interfaces

Common interfaces for all products.

---

### Concrete Products

Actual implementations.

---

### Client

Works only with interfaces.

The client never knows which vendor-specific classes it received.

---

# 7. Participants

## Abstract Factory

Declares methods such as:

* createBeam()
* createDoseCalculator()
* createMLC()

---

## Concrete Factory

Creates one complete product family.

Example:

```
VarianFactory
```

---

## Abstract Products

Examples:

```
Beam

DoseCalculator

Exporter

ImageLoader
```

---

## Concrete Products

```
VarianBeam

ElektaBeam

SiemensBeam
```

---

## Client

Uses only:

```
Factory

Beam

DoseCalculator

Exporter
```

No vendor-specific code appears in the client.

---

# 8. Collaboration

Runtime Flow

```
Application Starts

↓

Configuration says:

Vendor = Siemens

↓

Create SiemensFactory

↓

Factory creates:

Beam

↓

Dose Calculator

↓

MLC

↓

Exporter

↓

Application uses interfaces only
```

Notice that **all products come from the same family**, ensuring compatibility.

---

# 9. C++ Example

```cpp
#include <iostream>
#include <memory>

// ---------- Abstract Products ----------
class Button {
public:
    virtual ~Button() = default;
    virtual void draw() = 0;
};

class CheckBox {
public:
    virtual ~CheckBox() = default;
    virtual void draw() = 0;
};

// ---------- Windows Products ----------
class WindowsButton : public Button {
public:
    void draw() override {
        std::cout << "Windows Button\n";
    }
};

class WindowsCheckBox : public CheckBox {
public:
    void draw() override {
        std::cout << "Windows CheckBox\n";
    }
};

// ---------- Linux Products ----------
class LinuxButton : public Button {
public:
    void draw() override {
        std::cout << "Linux Button\n";
    }
};

class LinuxCheckBox : public CheckBox {
public:
    void draw() override {
        std::cout << "Linux CheckBox\n";
    }
};

// ---------- Abstract Factory ----------
class GUIFactory {
public:
    virtual ~GUIFactory() = default;

    virtual std::unique_ptr<Button> createButton() = 0;
    virtual std::unique_ptr<CheckBox> createCheckBox() = 0;
};

// ---------- Concrete Factories ----------
class WindowsFactory : public GUIFactory {
public:
    std::unique_ptr<Button> createButton() override {
        return std::make_unique<WindowsButton>();
    }

    std::unique_ptr<CheckBox> createCheckBox() override {
        return std::make_unique<WindowsCheckBox>();
    }
};

class LinuxFactory : public GUIFactory {
public:
    std::unique_ptr<Button> createButton() override {
        return std::make_unique<LinuxButton>();
    }

    std::unique_ptr<CheckBox> createCheckBox() override {
        return std::make_unique<LinuxCheckBox>();
    }
};

int main() {
    std::unique_ptr<GUIFactory> factory =
        std::make_unique<WindowsFactory>();

    auto button = factory->createButton();
    auto checkbox = factory->createCheckBox();

    button->draw();
    checkbox->draw();
}
```

### Design Focus

The client chooses **one factory**, and that factory guarantees all created UI components belong to the same platform family.

---

# 10. Qt Example

Imagine building a Qt application with multiple visual themes.

```
DarkThemeFactory

creates

↓

DarkPushButton
DarkToolBar
DarkIcons
DarkStylesheet
```

```
LightThemeFactory

creates

↓

LightPushButton
LightToolBar
LightIcons
LightStylesheet
```

The `MainWindow` asks only for a `ThemeFactory`. It never contains code like:

```cpp
if (darkMode)
    createDarkButton();
else
    createLightButton();
```

Switching themes becomes a matter of changing the factory.

---

# 11. Medical Software Example

Consider a DICOM Viewer supporting multiple scanner vendors.

Each vendor requires:

```
GE Factory

↓

GE Image Loader

GE Metadata Parser

GE Calibration

GE Reconstruction
```

```
Philips Factory

↓

Philips Image Loader

Philips Metadata Parser

Philips Calibration

Philips Reconstruction
```

The viewer simply requests:

```
factory->createImageLoader()
factory->createMetadataParser()
factory->createReconstruction()
```

If a new vendor is added, you introduce a new concrete factory without modifying the viewer's logic.

---

# 12. Advantages

### Consistency

All related objects are compatible.

### Maintainability

Creation logic is centralized.

### Scalability

Adding a new family means adding a new factory and product implementations.

### Loose Coupling

Clients depend only on interfaces.

### Testability

Factories can be replaced with mock implementations for testing.

---

# 13. Disadvantages

### More Classes

Each product family requires multiple concrete classes and a factory.

### Harder to Add New Product Types

Adding a **new family** is easy.

Adding a **new product** (e.g., `Slider`) requires changing the abstract factory and every concrete factory.

### Higher Initial Complexity

For small applications, the pattern may be unnecessary.

### When NOT to Use

Avoid it when:

* You only create one type of object.
* Product families don't need to remain compatible.
* The application is small and unlikely to grow.

---

# 14. Best Practices

* Design product interfaces first.
* Keep product families internally consistent.
* Name factories after the family they produce (`WindowsFactory`, `VarianFactory`).
* Avoid exposing concrete products to clients.
* Inject the appropriate factory during application startup based on configuration or platform detection.

---

# 15. Common Mistakes

### Mistake 1

Using Abstract Factory when Factory Method is sufficient.

### Mistake 2

Mixing products from different families after creation.

### Mistake 3

Putting business logic into factories.

Factories should create objects, not coordinate workflows.

### Mistake 4

Adding vendor-specific `if-else` logic back into the client.

This defeats the purpose of the pattern.

---

# 16. Pattern Variations

1. **Runtime Factory Selection**

   * Choose the factory from configuration files or plugins.

2. **Plugin-Based Abstract Factory**

   * Vendors register their own factories dynamically.

3. **Dependency Injection**

   * The application receives the correct factory from an IoC container.

4. **Factory Registry**

   * A registry maps identifiers to available factories.

---

# 17. Related Patterns

| Pattern          | Difference                                                                     |
| ---------------- | ------------------------------------------------------------------------------ |
| Factory Method   | Creates one product.                                                           |
| Abstract Factory | Creates a family of related products.                                          |
| Builder          | Builds one complex object step by step.                                        |
| Prototype        | Clones existing objects.                                                       |
| Bridge           | Separates abstraction from implementation but doesn't create product families. |

A useful mental model:

* **Factory Method:** One object.
* **Abstract Factory:** Many related objects.
* **Builder:** One complex object assembled over time.

---

# 18. Industry Usage

Large software platforms frequently use Abstract Factory when supporting multiple environments or vendors.

* **Microsoft:** Cross-platform UI components, rendering backends, and service families.
* **Google:** Platform-specific service implementations selected through injected factories.
* **Adobe:** Graphics engines, import/export subsystems, and plugin ecosystems.
* **Qt:** Platform abstraction layers and style/plugin systems rely on factory families internally.
* **ZEISS / Siemens / Philips:** Different imaging devices, scanner families, and hardware abstraction layers.
* **Autodesk:** Rendering engines, CAD kernels, and platform integrations.
* **Game Engines:** Graphics API abstraction (DirectX, Vulkan, Metal) through families of rendering objects.

The architectural benefit is **consistent object families with minimal client knowledge**.

---

# 19. Interview Questions

## Beginner

1. What is the Abstract Factory Pattern?
2. How is it different from Factory Method?
3. Why does it create families of objects?

## Intermediate

1. When would you choose Abstract Factory over Factory Method?
2. What happens when you add a new product family?
3. Why do clients depend on abstract products?

## Advanced

1. How would you implement an extensible vendor plugin architecture using Abstract Factory?
2. How does Abstract Factory support dependency injection?
3. What are the trade-offs when adding new product types?

## Scenario-Based

Your medical imaging application must support five scanner vendors, each with its own loader, parser, calibration manager, and reconstruction algorithm.

Would you use Abstract Factory? Explain your design.

## Architecture

Design the product families for a cross-platform Qt application supporting Windows, Linux, and macOS with native dialogs, menus, clipboard handlers, and file pickers. Explain why Abstract Factory fits this architecture.

---

# 20. Practice Exercises

### Beginner Exercise

Design an abstract `FurnitureFactory` that creates:

* Chair
* Table

Implement two families:

* Modern
* Classic

Focus on the relationships between the factory and the products.

---

### Intermediate Exercise

Design a Qt `ThemeFactory` that creates:

* Toolbar
* Icons
* Color Palette
* Stylesheet

Explain how switching themes only requires replacing the factory.

---

### Advanced Exercise

Design an **Abstract Factory architecture** for a **Treatment Planning System (TPS)** supporting:

* Varian
* Elekta
* Siemens

Each vendor must provide:

* Beam Model
* Dose Calculator
* MLC Controller
* Machine Configuration
* DICOM Exporter

Draw the class relationships and explain how the client remains independent of vendor-specific implementations.

**Do not implement the solution yet.** Focus on architectural responsibilities and interactions.

---

# Key Architectural Takeaway

The **Abstract Factory Pattern** is about maintaining **consistency across a family of related objects**.

A junior developer thinks:

> "I need a Beam and a Dose Calculator."

A software architect thinks:

> **"These objects must belong to the same vendor family so they work together correctly. I'll let one factory create the entire compatible set."**

That mindset prevents incompatible combinations, simplifies extension, and keeps large systems maintainable.

When you're ready, type **`NEXT`** to continue with **Lesson 4: Builder Pattern**.

















# Abstract Factory Pattern (Creational Design Pattern)

The **Abstract Factory Pattern** is an advanced version of the **Factory Method Pattern**.

While a **Factory Method** creates one type of object,
an **Abstract Factory** creates **families (groups) of related objects** — without specifying their concrete classes.

In simple words:
> “A factory of factories” that can create a set of related products that belong together.

---

##  Real-Life Example

Imagine a company that makes **UI components** for two operating systems:

* **Windows** (buttons, checkboxes with Windows look)
* **Mac** (buttons, checkboxes with Mac look)

You want to create UI elements based on the OS type — but you don’t want to modify your main code.

That’s where the **Abstract Factory Pattern** helps.
It lets you create related objects (like Button + Checkbox) for a specific platform in one go.

---

##  Key Idea

1. Define **interfaces** for families of products.
2. Create **concrete factories** that make related products.
3. The **client** uses the factory, not the concrete classes.

---

##  C++ Example (Simple and Clear)

```cpp
#include <iostream>
using namespace std;

// Step 1: Create product interfaces
class Button {
public:
    virtual void click() = 0;
};

class Checkbox {
public:
    virtual void check() = 0;
};

// Step 2: Create concrete products for Windows
class WindowsButton : public Button {
public:
    void click() override {
        cout << "Windows Button clicked 💻" << endl;
    }
};

class WindowsCheckbox : public Checkbox {
public:
    void check() override {
        cout << "Windows Checkbox checked ☑️" << endl;
    }
};

// Step 3: Create concrete products for Mac
class MacButton : public Button {
public:
    void click() override {
        cout << "Mac Button clicked 🍎" << endl;
    }
};

class MacCheckbox : public Checkbox {
public:
    void check() override {
        cout << "Mac Checkbox checked ✅" << endl;
    }
};

// Step 4: Abstract factory (interface for creating products)
class GUIFactory {
public:
    virtual Button* createButton() = 0;
    virtual Checkbox* createCheckbox() = 0;
};

// Step 5: Concrete factories
class WindowsFactory : public GUIFactory {
public:
    Button* createButton() override {
        return new WindowsButton();
    }
    Checkbox* createCheckbox() override {
        return new WindowsCheckbox();
    }
};

class MacFactory : public GUIFactory {
public:
    Button* createButton() override {
        return new MacButton();
    }
    Checkbox* createCheckbox() override {
        return new MacCheckbox();
    }
};

// Step 6: Client code
int main() {
    GUIFactory* factory;
    string osType = "Mac";

    if (osType == "Windows")
        factory = new WindowsFactory();
    else
        factory = new MacFactory();

    Button* btn = factory->createButton();
    Checkbox* chk = factory->createCheckbox();

    btn->click();
    chk->check();

    delete btn;
    delete chk;
    delete factory;

    return 0;
}
```

---

## 🧾 Output

```
Mac Button clicked 🍎
Mac Checkbox checked ✅
```

---

## 🔍 Explanation (In Simple Words)

* `Button` and `Checkbox` are interfaces for products.
* `WindowsButton` and `MacButton` are **different versions** of the same product.
* `GUIFactory` defines a family of objects (button + checkbox).
* `WindowsFactory` and `MacFactory` create specific product sets.
* The main code (`main()`) only uses `GUIFactory`, not any concrete class.

So, you can **easily switch between Windows and Mac UI** without changing your main logic.

---

##  When to Use Abstract Factory

* When you want to create **families of related objects** (like themes, UIs, database connections).
* When your system should be **independent of how products are created**.
* When adding new product families should be **easy and modular**.

---

## 🏁 Summary

| **Concept**  | **Description**                                                      |
| ------------ | -------------------------------------------------------------------- |
| Purpose      | Create families of related objects without specifying their classes. |
| Key Feature  | Factory of factories — creates related products together.            |
| Benefit      | Promotes consistency among related objects and easy scalability.     |
| Example      | WindowsFactory, MacFactory (UI creation).                            |
| Pattern Type | Creational Design Pattern                                            |

---

 **Quick Comparison:**

| Pattern          | Creates                     | Example                    |
| ---------------- | --------------------------- | -------------------------- |
| Factory Method   | One type of object          | Car or Bike                |
| Abstract Factory | Families of related objects | Windows or Mac UI elements |

---


