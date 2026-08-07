

# Lesson 21 — Observer Pattern

---

# Before We Start

If I had to rank the **Top 5 most important design patterns** for a C++/Qt Software Architect, Observer would definitely be on the list.

You cannot become a strong Qt developer without understanding Observer because one of Qt's core concepts is built on the same architectural idea:

* **Signals & Slots** (not a textbook Observer implementation, but it follows the same publish/subscribe principle)

Observer is everywhere:

* Qt Applications
* Medical Devices
* Stock Market Systems
* Weather Systems
* ERP Dashboards
* Chat Applications
* IoT
* Operating Systems
* Game Engines

---

# The Architect's Way of Thinking

Imagine you're building a **Treatment Planning System (TPS)**.

The user changes the beam angle.

Immediately, all of these need updating:

```text
Beam Angle Changed
        │
        ├── Dose Viewer
        ├── DVH Graph
        ├── 3D Viewer
        ├── Beam List
        ├── Status Bar
        ├── Plan Summary
        └── Save Indicator
```

A junior developer writes:

```cpp
beam.rotate();

doseViewer.update();

dvh.update();

viewer3D.update();

statusBar.update();

beamList.update();

planSummary.update();
```

Now suppose tomorrow management adds:

* AI Dose Predictor
* QA Validation Panel
* Treatment Timeline

Every place that changes a beam must now call three more update methods.

This violates the **Open/Closed Principle**.

An architect asks:

> **"Why should the Beam know every object interested in its changes?"**

Instead:

```text
Beam
 │
 │ notify()
 ▼
Observers
```

The Beam publishes an event.

Interested objects react automatically.

This is the **Observer Pattern**.

---

# 1. Introduction

## What is the Observer Pattern?

The **Observer Pattern** defines a **one-to-many dependency** between objects.

When one object changes state, all registered observers are automatically notified.

Simple definition:

> **One object publishes changes; many objects subscribe to receive notifications.**

---

## Why was it created?

Many systems have:

One source of truth

↓

Many consumers

Instead of hardcoding every consumer,

allow them to subscribe.

---

## Category

**Behavioral Pattern**

---

## What problem does it solve?

Without Observer:

```text
Beam

↓

Dose Viewer

↓

DVH

↓

Status Bar

↓

Logger

↓

3D View
```

The Beam knows everyone.

Adding a new view means modifying the Beam.

With Observer:

```text
Beam

↓

notify()

↓

Observer A

Observer B

Observer C

Observer D
```

The Beam knows only its subscribers.

---

# 2. Problem Statement

Imagine a **Stock Market Application**.

Stock Price:

```text
Reliance = ₹3000
```

Needs to update:

* Graph
* Portfolio
* Alerts
* News
* Watchlist

Without Observer:

```cpp
updateGraph();

updatePortfolio();

updateAlert();

updateNews();

updateWatchlist();
```

Every stock update contains UI logic.

Bad design.

---

# 3. Motivation

Architects noticed a common pattern:

One object changes.

Many objects care.

Examples:

* Temperature Sensor
* Beam
* Patient Record
* Database Row
* Network Status
* Chat Message

Instead of direct communication,

use a subscription model.

---

# 4. Real-World Analogy

## Newspaper Subscription

Newspaper Company

↓

Publishes Newspaper

↓

Subscribers Receive It

The newspaper doesn't know:

* where people live,
* how they read it,
* what they do afterward.

It simply publishes.

---

## Mapping

| Real World        | Software             |
| ----------------- | -------------------- |
| Newspaper Company | Subject              |
| Newspaper         | Event / Notification |
| Subscriber        | Observer             |
| Subscription      | Registration         |

---

# 5. Software Scenario

Observer appears wherever data changes must propagate.

### Desktop Applications

* Status bars
* Toolbars
* Property panels

---

### Qt Applications

This is the classic example.

Suppose:

```text
PatientModel
```

changes.

Interested views:

```text
Table View

Tree View

Chart View

Statistics Panel
```

Each updates automatically.

Qt's **signals and slots** provide this loosely coupled communication model.

---

### CAD Software

Object moves:

↓

Update:

* Property Panel
* Mini Map
* Selection Tree
* Coordinates

---

### Medical Imaging

Beam changes:

↓

Update:

* Dose Viewer
* DVH
* Optimization
* Statistics
* Beam Table

---

### IoT

Sensor:

↓

Dashboard

↓

Alarm

↓

Logger

---

# 6. UML Class Diagram

```text
               +----------------------+
               |      Subject         |
               +----------------------+
               | +attach()            |
               | +detach()            |
               | +notify()            |
               +----------+-----------+
                          ^
                          |
                +----------------------+
                |   ConcreteSubject    |
                +----------------------+

                          |
                 notifies |
                          v

               +----------------------+
               |      Observer        |
               +----------------------+
               | +update()            |
               +----------+-----------+
                          ^
                ----------|----------
                |                     |
      +----------------+    +----------------+
      | Observer A     |    | Observer B     |
      +----------------+    +----------------+
```

---

## Responsibilities

### Subject

Maintains observers.

Calls:

```cpp
notify();
```

---

### Observer

Defines:

```cpp
update();
```

---

### Concrete Subject

Stores state.

---

### Concrete Observer

Updates itself.

---

# 7. Participants

## Subject

Example:

```text
BeamModel
```

Provides:

```cpp
attach()

detach()

notify()
```

---

## Concrete Subject

Stores:

```text
Beam Angle

Energy

Weight
```

---

## Observer

Interface:

```cpp
update()
```

---

## Concrete Observers

Examples:

```text
DoseViewer

DVHPanel

StatusBar

BeamList
```

---

## Client

Registers observers.

---

# 8. Collaboration

Runtime Flow

```text
Beam Angle Changed

↓

BeamModel

↓

notify()

↓

Dose Viewer

↓

DVH

↓

Status Bar

↓

Beam List
```

Notice:

BeamModel never calls:

```cpp
doseViewer.update();
```

directly.

It simply notifies observers.

---

# 9. C++ Example

```cpp
#include <iostream>
#include <memory>
#include <vector>

// Observer
class Observer
{
public:
    virtual ~Observer() = default;
    virtual void update(int value) = 0;
};

// Subject
class Subject
{
public:
    void attach(std::shared_ptr<Observer> observer)
    {
        observers.push_back(observer);
    }

    void setValue(int value)
    {
        m_value = value;
        notify();
    }

private:
    void notify()
    {
        for (auto& observer : observers)
        {
            observer->update(m_value);
        }
    }

    int m_value = 0;
    std::vector<std::shared_ptr<Observer>> observers;
};

// Concrete Observer
class Display : public Observer
{
public:
    void update(int value) override
    {
        std::cout << "New Value = "
                  << value << '\n';
    }
};

int main()
{
    Subject subject;

    auto display =
        std::make_shared<Display>();

    subject.attach(display);

    subject.setValue(100);
}
```

---

## Design Focus

Notice:

The subject never knows:

```text
Display
```

It only knows:

```text
Observer
```

This is **programming to an interface**, one of the foundations of SOLID.

---

# 10. Qt Example

This is one of the most important examples in the course.

Suppose you have:

```cpp
BeamModel
```

When its angle changes:

```cpp
emit beamAngleChanged(newAngle);
```

Connected objects:

```cpp
connect(&beamModel,
        &BeamModel::beamAngleChanged,
        &doseView,
        &DoseView::updateDose);

connect(&beamModel,
        &BeamModel::beamAngleChanged,
        &dvhPanel,
        &DVHPanel::refresh);

connect(&beamModel,
        &BeamModel::beamAngleChanged,
        &statusBar,
        &StatusBar::showMessage);
```

One signal.

Multiple receivers.

This follows the Observer principle while providing additional Qt features such as type-safe connections and queued cross-thread delivery.

---

# 11. Medical Software Example

Imagine a **CT Viewer**.

Subject:

```text
CurrentSliceModel
```

Observers:

```text
Slice Viewer

Crosshair

Histogram

Patient Info

Thumbnail Panel

3D Viewer
```

Workflow:

```text
User moves slider

↓

CurrentSliceModel

↓

notify()

↓

All Views Update
```

The model doesn't know:

* how the histogram works,
* how the 3D viewer renders,
* how the patient panel formats text.

It only publishes:

> **"The current slice changed."**

---

# 12. Advantages

### Loose Coupling

Subject doesn't depend on concrete observers.

### Open/Closed Principle

New observers can be added without modifying the subject.

### Reusability

Observers can subscribe to multiple subjects.

### Scalability

Supports many listeners naturally.

### Event-Driven Design

Ideal for reactive applications.

---

# 13. Disadvantages

### Notification Order

Observers are usually notified in registration order unless specified otherwise.

### Cascading Updates

One notification may trigger many updates.

### Memory Management

Observers must be detached appropriately if they outlive the subject.

### When NOT to Use

Avoid Observer when:

* there is only one consumer,
* synchronous direct calls are simpler,
* notification overhead outweighs the benefit.

---

# 14. Best Practices

* Notify only when state actually changes.
* Pass enough information in notifications to avoid unnecessary queries.
* Clearly define ownership and lifetime.
* Consider weak references when observers may disappear independently.
* Keep observer callbacks lightweight.

---

# 15. Common Mistakes

### Mistake 1

Calling concrete observer methods directly.

Depend on the observer interface.

---

### Mistake 2

Forgetting to unregister observers.

This can lead to dangling pointers in manual-memory systems.

---

### Mistake 3

Putting heavy business logic inside `update()`.

Observers should react, not become workflow engines.

---

### Mistake 4

Using Observer where Mediator is more appropriate.

Observer broadcasts notifications.

Mediator coordinates interactions.

---

# 16. Pattern Variations

## 1. Push Model

The subject sends data.

```text
notify(value)
```

Observer receives everything it needs.

---

## 2. Pull Model

The subject only says:

```text
Something Changed
```

Observer queries the subject.

---

## 3. Event Bus

Events are published through a central dispatcher.

Common in large applications.

---

## 4. Reactive Streams

Modern reactive frameworks build on Observer concepts while adding asynchronous data streams and backpressure.

---

# 17. Related Patterns

| Pattern           | Difference                                                         |
| ----------------- | ------------------------------------------------------------------ |
| Observer          | One subject notifies many observers.                               |
| Mediator          | Coordinates communication among peers.                             |
| Command           | Represents an action as an object.                                 |
| State             | Changes behavior based on internal state.                          |
| Publish/Subscribe | General architectural style; Observer is its object-oriented form. |

---

## Observer vs Mediator (Interview Favorite)

### Observer

Question:

> **"Who needs to know that something changed?"**

One publisher.

Many subscribers.

---

### Mediator

Question:

> **"How should these objects coordinate?"**

Many objects.

One coordinator.

---

# 18. Industry Usage

Observer is one of the most widely used patterns in software engineering.

* **Microsoft:** WPF data binding, WinUI events, Visual Studio extensibility.
* **Google:** Event-driven systems, UI frameworks, reactive architectures.
* **Adobe:** Document updates, layer changes, property panels.
* **Qt:** Signals & Slots, `QAbstractItemModel` notifications (`dataChanged`, `rowsInserted`, `rowsRemoved`) embody Observer principles.
* **ZEISS / Siemens / Philips:** Medical imaging viewers, acquisition monitoring, treatment planning UI synchronization.
* **Autodesk:** Scene updates, property inspectors, CAD model notifications.
* **Financial Systems:** Stock price feeds and market data dissemination.
* **IoT:** Sensor readings propagated to dashboards and alert systems.

The architectural goal is:

> **Separate the producer of information from the consumers.**

---

# 19. Interview Questions

## Beginner

1. What problem does the Observer Pattern solve?
2. What is the difference between a Subject and an Observer?
3. Why is Observer a Behavioral Pattern?

---

## Intermediate

1. What is the difference between Push and Pull notification models?
2. How does Observer support the Open/Closed Principle?
3. What lifetime issues can occur with observers?

---

## Advanced

1. How would you implement a thread-safe Observer in C++?
2. How would you avoid notification storms in a large GUI?
3. How would you combine Observer with MVC or MVVM?

---

## Scenario-Based

Your TPS updates:

* Beam Model
* Dose View
* DVH
* Statistics
* Plan Summary

whenever beam parameters change.

Design an Observer-based architecture that allows adding future views without modifying the Beam Model.

---

## Architecture

Design an Observer architecture for your **College ERP**.

Subject:

```text
StudentModel
```

Observers:

* Attendance Panel
* Fee Panel
* Timetable
* Notification Panel
* Analytics Dashboard

Requirements:

* Multiple observers.
* Dynamic registration and removal.
* Efficient updates.
* Safe lifetime management.

Explain:

* subject responsibilities,
* observer responsibilities,
* notification strategy,
* and ownership.

---

# 20. Practice Exercises

### Beginner Exercise

Design a **Weather Station**.

Subject:

* Temperature Sensor

Observers:

* Mobile App
* LCD Display
* Alarm

Draw the UML and explain the update flow.

---

### Intermediate Exercise

Design a Qt application where a `PatientModel` notifies:

* Table View
* Tree View
* Statistics Panel

using the Observer Pattern (or Qt's equivalent mechanism).

Explain how updates propagate.

---

### Advanced Exercise

Design an **Observer architecture** for a **Treatment Planning System**.

Subject:

* TreatmentPlanModel

Observers:

* Beam Editor
* Dose Viewer
* DVH Panel
* 3D Viewer
* Statistics Panel
* Status Bar
* Auto-Save Service
* Audit Logger

Requirements:

* Thread-safe notifications.
* Efficient updates (avoid unnecessary refreshes).
* Support dynamic registration/unregistration.
* Explain how Qt Signals & Slots could be incorporated.

**Do not implement the solution yet.** Focus on architecture, notification flow, lifetime management, and scalability.

---

# ⭐ Architect's Insight: Observer + MVC + Qt

One of the most common architectures in Qt applications is:

```text
            User
              │
              ▼
            View
              │
      (signals/commands)
              │
              ▼
         Controller/ViewModel
              │
              ▼
            Model
              │
      emits change notification
              │
              ▼
      All Registered Views
```

For example, in a TPS:

```text
BeamModel
      │
      ├── Dose Viewer
      ├── DVH Panel
      ├── Beam List
      ├── Status Bar
      ├── Auto Save
      └── Audit Logger
```

The **BeamModel** doesn't know how each observer uses the information.

It simply announces:

> **"My state changed."**

Each observer decides how to react.

This architecture is one of the reasons Qt applications remain modular and maintainable.

---

# Key Architectural Takeaway

The **Observer Pattern** is about **decoupling publishers from subscribers**.

A junior developer thinks:

> "Whenever the model changes, I'll call every view manually."

A software architect thinks:

> **"The model shouldn't know who is interested. It will publish changes, and interested observers can subscribe or unsubscribe independently."**

This mindset underpins event-driven programming, Qt Signals & Slots, MVC/MVVM architectures, real-time dashboards, and many large-scale enterprise and medical applications.

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

Only five Behavioral Patterns remain:

1. **State Pattern**
2. **Strategy Pattern**
3. **Template Method Pattern**
4. **Visitor Pattern**
5. **Null Object Pattern**

These five patterns are among the most frequently discussed in senior C++/Qt architecture interviews.



---
[⬅️ Mediator Pattern ](/MementoPattern.md)        |  [Observer Pattern  ➡️](/ObserverPattern.md) 
---
## **License**
This project is licensed under the MIT License.

---

Happy Coding!
