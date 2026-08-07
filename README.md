# 🧩 Design Patterns in C++

A complete guide to **Design Patterns in C++**, categorized into **Creational**, **Structural**, and **Behavioral** types.
Use this as a structured roadmap to master reusable software design principles and write clean, extensible, and maintainable C++ code.

| **S. No.** | **Pattern Name**                                                   | **Category** | **Description**                                                                           |
| ---------- | ------------------------------------------------------------------ | ------------ | ----------------------------------------------------------------------------------------- |
| **1**      | [Introduction to Design Patterns](/introduction.md) | Overview     | What are design patterns, their importance, and classification in software development.   |
| **2**      | [SOLID Principles](/solid-principles.md)            | Overview     | The five key principles (SRP, OCP, LSP, ISP, DIP) that guide good object-oriented design. |

---

## 🏗️ **Creational Design Patterns**

Creational patterns deal with **object creation mechanisms**, trying to create objects in a manner suitable to the situation.

| **S. No.** | **Pattern Name**                                               | **Description**                                                                                                      |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **3**      | [Singleton Pattern](/singleton.md)              | Ensures that a class has only one instance and provides a global point of access to it.                              |
| **4**      | [Factory Method Pattern](/factorymethod.md)     | Defines an interface for creating an object but allows subclasses to alter the type of objects that will be created. |
| **5**      | [Abstract Factory Pattern](/abstractfactory.md) | Creates families of related or dependent objects without specifying their concrete classes.                          |
| **6**      | [Builder Pattern](/builder.md)                  | Separates object construction from its representation to build complex objects step-by-step.                         |
| **7**      | [Prototype Pattern](/prototype.md)              | Creates new objects by copying existing ones instead of creating from scratch.                                       |
| **8**      | [Object Pool Pattern](/objectpool.md)           | Manages reusable objects for resource-intensive operations.                                                          |

---

## 🧱 **Structural Design Patterns**

Structural patterns deal with **class and object composition** — how classes and objects can be combined to form larger structures.

| **S. No.** | **Pattern Name**                                                  | **Description**                                                                |
| ---------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| **9**      | [Adapter Pattern](/adapter.md)                     | Allows incompatible interfaces to work together by acting as a bridge.         |
| **10**     | [Bridge Pattern](/bridge.md)                       | Decouples abstraction from implementation so both can vary independently.      |
| **11**     | [Composite Pattern](/composite.md)                 | Composes objects into tree structures to represent part-whole hierarchies.     |
| **12**     | [Decorator Pattern](/decorator.md)                 | Dynamically adds new behaviors to objects without altering existing structure. |
| **13**     | [Facade Pattern](/facade.md)                       | Provides a simplified interface to a complex subsystem.                        |
| **14**     | [Flyweight Pattern](/flyweight.md)                 | Minimizes memory usage by sharing common object data across instances.         |
| **15**     | [Proxy Pattern](/proxy.md)                         | Provides a surrogate or placeholder for another object to control access.      |
| **16**     | [Private Class Data Pattern](/privateclassdata.md) | Restricts direct access to class data and ensures data integrity.              |

---

## ⚙️ **Behavioral Design Patterns**

Behavioral patterns focus on **communication between objects**, making it easier to assign responsibilities and manage control flow.

| **S. No.** | **Pattern Name**                                                            | **Description**                                                                                                |
| ---------- | --------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| **17**     | [Chain of Responsibility Pattern](/designpatterns/chainofresponsibility.md) | Passes a request along a chain of handlers until one handles it.                                               |
| **18**     | [Command Pattern](/designpatterns/command.md)                               | Encapsulates requests as objects, enabling parameterization, queuing, and undo operations.                     |
| **19**     | [Interpreter Pattern](/designpatterns/interpreter.md)                       | Defines a grammar and an interpreter to process language sentences.                                            |
| **20**     | [Iterator Pattern](/IteratorPattern.md)                             | Provides a way to sequentially access elements of an aggregate object without exposing its representation.     |
| **21**     | [Mediator Pattern](/MediatorPattern.md)                             | Defines an object that encapsulates how a set of objects interact, promoting loose coupling.                   |
| **22**     | [Memento Pattern](/MementoPattern.md)                               | Captures and restores an object's internal state without violating encapsulation.                              |
| **23**     | [Observer Pattern](/ObserverPattern.md)                             | Establishes a one-to-many dependency between objects; when one changes, dependents are notified automatically. |
| **24**     | [State Pattern](/designpatterns/state.md)                                   | Allows an object to change its behavior dynamically when its internal state changes.                           |
| **25**     | [Strategy Pattern](/designpatterns/strategy.md)                             | Defines a family of algorithms and makes them interchangeable at runtime.                                      |
| **26**     | [Template Method Pattern](/designpatterns/templatemethod.md)                | Defines the program skeleton in a base class but lets subclasses override certain steps.                       |
| **27**     | [Visitor Pattern](/designpatterns/visitor.md)                               | Lets you add new operations to existing class hierarchies without modifying them.                              |
| **28**     | [Null Object Pattern](/designpatterns/nullobject.md)                        | Provides a default behavior when no real object is available to handle a request.                              |

---

## 📘 **Additional References**

| **S. No.** | **Topic**                                                              | **Description**                                                               |
| ---------- | ---------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| **29**     | [Design Pattern Comparison](/designpatterns/comparison.md)             | Quick comparison between similar design patterns and when to use them.        |
| **30**     | [Real-world Examples in C++](/designpatterns/examples.md)              | Implementation of design patterns using real-world C++ projects.              |
| **31**     | [Best Practices and Anti-Patterns](/designpatterns/bestpractices.md)   | Common mistakes and guidelines to use design patterns effectively.            |
| **32**     | [Interview Questions on Design Patterns](/designpatterns/interview.md) | Frequently asked interview questions and examples related to design patterns. |

---

