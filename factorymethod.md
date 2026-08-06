# Design Patterns Master Course

# Lesson 2 — Factory Method Pattern (Creational Pattern)

Before we begin, remember the mindset shift:

* **Singleton answered:** *"How many objects should exist?"*
* **Factory Method answers:** *"Who should decide which object to create?"*

This is one of the most important architectural questions in software design.

---

# 1. Introduction

## What is the Factory Method Pattern?

The **Factory Method Pattern** defines an interface for creating objects, but lets subclasses decide **which concrete object** should actually be created.

Instead of writing:

```cpp
Circle shape;
```

or

```cpp
Rectangle shape;
```

the client writes:

```cpp
Shape* shape = factory->createShape();
```

The client does **not** know whether it received a `Circle`, `Rectangle`, or `Triangle`.

---

## Why was it created?

Imagine a large software application.

Many places need to create objects.

If every class writes:

```cpp
new Circle();
new MRIImage();
new CTImage();
new PNGExporter();
```

then every module becomes tightly coupled to concrete classes.

Whenever requirements change, hundreds of files must also change.

Factory Method removes this dependency.

---

## Category

**Creational Pattern**

It controls object creation while hiding the concrete implementation.

---

## What problem does it solve?

It separates:

* **Object creation**
* **Object usage**

These are two different responsibilities.

---

# 2. Problem Statement

Imagine you're developing a **Medical Imaging Software**.

It supports:

* CT
* MRI
* PET
* Ultrasound
* X-Ray

Without Factory Method:

```cpp
if(type == "CT")
    image = new CTImage();

else if(type == "MRI")
    image = new MRIImage();

else if(type == "PET")
    image = new PETImage();

else if(type == "US")
    image = new UltrasoundImage();
```

Now imagine two years later.

The software supports:

* CBCT
* Mammography
* Fluoroscopy
* OCT
* Nuclear Imaging

Now every place that creates images contains another long `if-else` or `switch`.

Soon your project contains hundreds of them.

---

## Why is this difficult?

Suppose you rename:

```text
MRIImage

↓

MRVolumeImage
```

Now every module must be modified.

Suppose MRI requires additional initialization.

Again...

Every module changes.

This violates the **Open/Closed Principle (OCP)**:

> Software entities should be open for extension but closed for modification.

---

# 3. Motivation

Developers observed a recurring problem:

> Clients should focus on **using objects**, not **knowing how to construct every concrete type**.

Object creation often becomes more complex over time:

* Validation
* Configuration
* Initialization
* Logging
* Resource allocation
* Plugin loading

By centralizing creation, these concerns stay in one place.

---

# 4. Real-World Analogy

## Restaurant

Imagine ordering food.

You tell the waiter:

> "I want a pizza."

Do you walk into the kitchen and decide:

* which flour?
* oven temperature?
* cheese brand?
* baking time?

No.

The kitchen decides **how** to prepare it.

### Mapping

| Restaurant  | Software                 |
| ----------- | ------------------------ |
| Customer    | Client                   |
| Waiter/Menu | Factory interface        |
| Kitchen     | Concrete Factory         |
| Pizza       | Product                  |
| Chef        | Concrete Product creator |

The customer only requests the product.

The kitchen decides how it is created.

---

# 5. Software Scenario

Factory Method appears wherever software creates families of related objects.

Examples:

### Desktop Applications

* Document creation
* Export formats
* Printer drivers

### Qt Applications

* Model creation
* Dialog creation
* Widget factories
* Style plugins

### CAD Software

* Shape creation
* Tool creation
* Rendering backend selection

### Medical Imaging

* Image loaders
* DICOM parsers
* Scanner-specific reconstruction
* Exporters

### Browsers

* HTML parsers
* Renderer selection

### Game Engines

* Enemy spawning
* Weapon creation
* Particle systems

### Operating Systems

* Driver creation
* File system objects

---

# 6. UML Class Diagram

```text
                    +----------------------+
                    |      Product         |
                    +----------------------+
                    | +operation()         |
                    +----------------------+
                              ▲
                ┌─────────────┴─────────────┐
                │                           │
      +------------------+       +------------------+
      | ConcreteProductA |       | ConcreteProductB |
      +------------------+       +------------------+

                    +----------------------+
                    |      Creator         |
                    +----------------------+
                    | +createProduct()     |
                    | +businessLogic()     |
                    +----------------------+
                              ▲
                ┌─────────────┴─────────────┐
                │                           │
      +------------------+       +------------------+
      | ConcreteCreatorA |       | ConcreteCreatorB |
      +------------------+       +------------------+
      | createProduct()  |       | createProduct()  |
      +------------------+       +------------------+
```

---

## Relationship Explanation

### Product

Defines the common interface.

---

### Concrete Products

Actual implementations.

Example:

* CTImage
* MRIImage
* PETImage

---

### Creator

Declares:

```text
createProduct()
```

It does **not** know which concrete product it creates.

---

### Concrete Creator

Overrides:

```text
createProduct()
```

and decides which object to instantiate.

---

# 7. Participants

## Product

Common interface used by the client.

Example:

```text
Image
```

---

## Concrete Product

Real implementations.

```text
CTImage

MRIImage

PETImage
```

---

## Creator

Declares the factory method.

May also contain common business logic that works with the `Product` interface rather than concrete classes.

---

## Concrete Creator

Creates a specific product.

```text
CTFactory

MRIFactory
```

---

## Client

Uses only:

* Product interface
* Creator interface

The client never calls:

```cpp
new MRIImage();
```

directly.

---

# 8. Collaboration

Runtime Flow

```text
Client

↓

asks Creator

↓

ConcreteCreator selected

↓

createProduct()

↓

Concrete Product created

↓

returned as Product*

↓

Client uses Product interface
```

Notice:

The client communicates with abstractions, not concrete implementations.

---

# 9. C++ Example

```cpp
#include <iostream>
#include <memory>

// Product
class Image
{
public:
    virtual ~Image() = default;
    virtual void display() = 0;
};

// Concrete Products
class CTImage : public Image
{
public:
    void display() override
    {
        std::cout << "Displaying CT Image\n";
    }
};

class MRIImage : public Image
{
public:
    void display() override
    {
        std::cout << "Displaying MRI Image\n";
    }
};

// Creator
class ImageFactory
{
public:
    virtual ~ImageFactory() = default;
    virtual std::unique_ptr<Image> createImage() = 0;
};

// Concrete Creators
class CTFactory : public ImageFactory
{
public:
    std::unique_ptr<Image> createImage() override
    {
        return std::make_unique<CTImage>();
    }
};

class MRIFactory : public ImageFactory
{
public:
    std::unique_ptr<Image> createImage() override
    {
        return std::make_unique<MRIImage>();
    }
};

int main()
{
    std::unique_ptr<ImageFactory> factory =
        std::make_unique<CTFactory>();

    auto image = factory->createImage();

    image->display();
}
```

### Design Focus

Notice what changed:

* The client never creates `CTImage` directly.
* The factory owns the creation decision.
* If another image type is added, existing client code usually remains unchanged.

---

# 10. Qt Example

Suppose you're building a Qt application that supports different chart widgets.

Instead of:

```cpp
if(type == "Line")
    widget = new LineChartWidget;

if(type == "Bar")
    widget = new BarChartWidget;
```

You create:

```text
ChartFactory
        |
        +---- createChart()
```

Concrete factories:

```text
LineChartFactory

BarChartFactory

PieChartFactory
```

The main window only knows:

```cpp
factory->createChart();
```

Another common Qt scenario is a plugin architecture, where different plugins provide their own factories to create custom editors, importers, or views without changing the main application.

---

# 11. Medical Software Example

Imagine a **Treatment Planning System (TPS)**.

Different vendors produce different LINAC machines.

```text
Varian

Elekta

Siemens
```

Each machine has its own beam model.

Instead of:

```cpp
if(machine=="Varian")
    beam = new VarianBeam();

if(machine=="Elekta")
    beam = new ElektaBeam();
```

Use:

```text
BeamFactory
```

Each vendor implements:

```text
VarianBeamFactory

ElektaBeamFactory

SiemensBeamFactory
```

The Dose Engine only knows:

```cpp
BeamFactory::createBeam();
```

This makes supporting a new vendor far easier because you add a new factory and beam implementation rather than modifying the dose engine.

---

# 12. Advantages

## Flexibility

Creation logic is isolated.

---

## Maintainability

Changes occur in one place.

---

## Reusability

Factories can be reused by multiple clients.

---

## Scalability

Adding a new product usually means adding a new concrete product and creator.

---

## Loose Coupling

Clients depend on interfaces rather than concrete classes.

---

## Testability

Factories can be mocked or substituted during testing.

---

# 13. Disadvantages

## More Classes

The pattern introduces additional abstractions.

---

## More Files

Each new product often requires a new creator.

---

## Learning Curve

It can feel like over-engineering for very small applications.

---

## When NOT to Use

Avoid Factory Method when:

* there is only one implementation,
* object creation is trivial,
* you don't expect variation in product types.

---

# 14. Best Practices

* Return abstractions (`Image`, `Exporter`, `Beam`) rather than concrete types.
* Keep object creation inside the factory; avoid leaking construction details to clients.
* Name factories after what they create (`ImageFactory`, `BeamFactory`, `ExporterFactory`).
* Let business logic operate on the `Product` interface, not on concrete implementations.
* Combine with dependency injection in larger architectures so clients receive the appropriate factory from configuration.

---

# 15. Common Mistakes

### Mistake 1

Factories that simply wrap:

```cpp
return new Object();
```

without adding any architectural value.

---

### Mistake 2

Putting business logic inside the factory.

Factories should create objects, not perform the object's work.

---

### Mistake 3

Clients checking concrete product types after creation:

```cpp
if (dynamic_cast<CTImage*>(image))
```

This defeats the purpose of programming to the interface.

---

### Mistake 4

Creating a single "God Factory" with hundreds of `switch` statements.

Experienced architects often split responsibilities into focused factories or move toward Abstract Factory or plugin-based registration when the product family grows.

---

# 16. Pattern Variations

1. **Parameterized Factory Method**

   * The creator chooses the product based on input parameters.

2. **Plugin-Based Factory**

   * New factories register themselves dynamically.

3. **Template-Based Factory**

   * Uses C++ templates to reduce boilerplate for compile-time scenarios.

4. **Self-Registering Factory**

   * Products register automatically, avoiding large conditional blocks.

---

# 17. Related Patterns

| Pattern          | Difference                                           |
| ---------------- | ---------------------------------------------------- |
| Singleton        | Controls the number of instances.                    |
| Factory Method   | Decides which product to create through subclassing. |
| Abstract Factory | Creates entire families of related products.         |
| Builder          | Constructs one complex object step by step.          |
| Prototype        | Creates objects by cloning existing ones.            |

Think of it this way:

* **Factory Method:** "Which product should I create?"
* **Abstract Factory:** "Which family of products should I create?"
* **Builder:** "How do I assemble this complex product?"

---

# 18. Industry Usage

Factory Method is one of the most widely used patterns in professional software because extensibility is a common requirement.

* **Microsoft:** Different document types, file handlers, UI controls, and platform-specific implementations are often created through factories.
* **Google:** Browser engines, service implementations, and backend components commonly rely on factory abstractions combined with dependency injection.
* **Adobe:** Import/export filters, graphics formats, and plugin systems.
* **Qt:** Many extensible systems (such as SQL drivers, image format plugins, and style plugins) rely on factory-style object creation behind the scenes.
* **ZEISS / Siemens / Philips:** Different imaging modalities, hardware adapters, reconstruction algorithms, and vendor-specific device implementations.
* **Autodesk:** CAD entities, rendering pipelines, and import/export plugins.
* **Game Engines:** Factories are widely used for entity spawning, asset loading, AI behaviors, and scripting integration.

The common architectural theme is **extensibility without modifying existing client code**.

---

# 19. Interview Questions

## Beginner

1. What problem does the Factory Method Pattern solve?
2. Why is it a Creational Pattern?
3. What is the difference between a Product and a Creator?
4. Why should clients avoid calling `new` directly?

## Intermediate

1. How does Factory Method support the Open/Closed Principle?
2. When would you choose Factory Method over a simple constructor?
3. What responsibilities belong in a factory?
4. How would you unit test code that depends on a factory?

## Advanced

1. How would you implement a self-registering factory for medical imaging plugins?
2. How would you load new product types at runtime without recompiling the application?
3. How does Factory Method interact with dependency injection?
4. How would you handle object creation failures inside a factory?

## Scenario-Based

Your DICOM Viewer must support new image formats every year from third-party vendors.

Would you keep adding `if-else` statements or introduce Factory Method? Explain the architectural trade-offs.

## Architecture

You're designing a CAD application with:

* Shape Importers
* File Exporters
* Rendering Backends
* Measurement Tools

Which components are good candidates for Factory Method, and why?

---

# 20. Practice Exercises

### Beginner Exercise

Design a `DocumentFactory` that can create:

* PDFDocument
* WordDocument
* TextDocument

Focus on the class design and interactions, not the implementation details.

---

### Intermediate Exercise

Design a Qt-based `ChartFactory` capable of creating different chart widgets (Line, Bar, Pie). Explain how the main window remains independent of concrete chart classes.

---

### Advanced Exercise

Design a factory architecture for a **Treatment Planning System** that supports multiple LINAC vendors (Varian, Elekta, Siemens) and allows adding a new vendor without modifying the Dose Engine. Draw the class relationships and explain how the factories collaborate with the rest of the system.

**Do not implement the solution yet.** Focus on the architecture and object responsibilities.

---

## Key Architectural Takeaway

The Factory Method Pattern is not about hiding the `new` keyword.

It is about **moving the responsibility of object creation to the right place**.

A junior developer thinks:

> "I need an `MRIImage`, so I'll create one."

A software architect thinks:

> **"My code shouldn't need to know which concrete image class exists. It should ask an abstraction to provide the correct implementation."**

That mindset leads to software that is easier to extend, test, and maintain.

When you're ready, type **`NEXT`** to continue with **Lesson 3: Abstract Factory Pattern**.














# Factory Method Pattern (Creational Design Pattern)


The **Factory Method Pattern** is another *Creational Design Pattern* that provides an **interface for creating objects**, but **lets subclasses decide which class to instantiate**.

In simple words:
➡️ Instead of creating objects directly using `new`, you use a **factory** (a special function or class) that decides **which object to create** based on input or logic.

---

## Real-Life Example

Imagine a **vehicle factory**.
You ask the factory for a vehicle type (like *Car* or *Bike*).
You don’t care *how* it’s made — you just get the right object.

Similarly, in C++, a Factory Method hides object creation details and provides flexibility.

---

##  Key Idea

* Define an **interface (base class)** for creating objects.
* Let **derived classes** decide which object to create.
* The **client code** uses only the base class — not the concrete classes.

---

##  C++ Example (Simple and Clear)

```cpp
#include <iostream>
using namespace std;

// Step 1: Create an interface (abstract base class)
class Vehicle {
public:
    virtual void start() = 0; // pure virtual function
};

// Step 2: Create concrete classes
class Car : public Vehicle {
public:
    void start() override {
        cout << "Car started 🚗" << endl;
    }
};

class Bike : public Vehicle {
public:
    void start() override {
        cout << "Bike started 🏍️" << endl;
    }
};

// Step 3: Create a Factory class
class VehicleFactory {
public:
    static Vehicle* createVehicle(string type) {
        if (type == "car")
            return new Car();
        else if (type == "bike")
            return new Bike();
        else
            return nullptr;
    }
};

// Step 4: Client code
int main() {
    Vehicle* v1 = VehicleFactory::createVehicle("car");
    Vehicle* v2 = VehicleFactory::createVehicle("bike");

    if (v1) v1->start();
    if (v2) v2->start();

    // Clean up
    delete v1;
    delete v2;

    return 0;
}
```

---

## 🧾 Output

```
Car started 🚗
Bike started 🏍️
```

---

## 🔍 Explanation (In Simple Words)

* The **`Vehicle`** class defines a common interface.
* The **`Car`** and **`Bike`** classes implement it differently.
* The **`VehicleFactory`** decides which object to create based on input.
* The **main()** function only calls `createVehicle()` — it doesn’t care how the objects are made.

So, if tomorrow you add a **Truck** class, you can just update the factory — not the client code.
✅ That’s flexibility and maintainability in action!

---

## When to Use Factory Method

* When the exact type of object to create is **determined at runtime**.
* When you want to **avoid tight coupling** between code and object creation.
* When creating an object involves **complex logic**.

---

## 🏁 Summary

| **Concept**  | **Description**                                                    |
| ------------ | ------------------------------------------------------------------ |
| Purpose      | Creates objects without exposing the creation logic to the client. |
| Key Feature  | Uses a common interface, returns derived class instances.          |
| Benefit      | Promotes loose coupling and flexibility.                           |
| Example      | VehicleFactory creating Car or Bike.                               |
| Pattern Type | Creational Design Pattern                                          |

---

Would you like me to continue with **Topic 5: Abstract Factory Pattern** next — explained in the same simple English + C++ code format?
