

# Lesson 19 — Mediator Pattern 

---

# Before We Start

The **Mediator Pattern** solves one of the biggest problems in object-oriented software:

> **Too many objects communicating directly with each other.**

This is commonly called **spaghetti communication**.

It appears in:

* Qt GUI Applications
* Medical Device Software
* ERP Systems
* IDEs
* CAD Software
* Air Traffic Control
* Chat Applications
* Workflow Engines

---

# The Architect's Way of Thinking

Imagine you're building your **College ERP**.

The login page contains:

* Username TextBox
* Password TextBox
* Role ComboBox
* Remember Me CheckBox
* Login Button
* Forgot Password Link
* Error Label

A junior developer writes code like this:

```text
Username ------> Login Button
Username ------> Error Label
Password ------> Login Button
Password ------> Error Label
Role ----------> Login Button
Role ----------> Forgot Password
Remember ------> Login Button
Login Button --> Error Label
```

Every widget talks directly to every other widget.

With only 7 widgets, communication already looks like a web.

Now imagine:

* 40 widgets
* 80 widgets
* 200 widgets

Maintenance becomes a nightmare.

An architect asks:

> **"Why should every widget know about every other widget?"**

Instead:

```text
All Widgets
      ↓
   LoginMediator
```

Now widgets know only one object.

That object is the **Mediator**.

---

# 1. Introduction

## What is the Mediator Pattern?

The **Mediator Pattern** defines an object that encapsulates how a group of objects interact.

Instead of objects communicating directly with each other, they communicate **through a mediator**.

Simple definition:

> **Centralize object communication into one coordinating object.**

---

## Why was it created?

As systems grow:

```text
Object A ↔ Object B
Object A ↔ Object C
Object A ↔ Object D
Object B ↔ Object D
Object C ↔ Object D
...
```

Dependencies grow rapidly.

If there are **N objects**, the number of possible communication paths grows roughly as:

```text
N × (N - 1)
```

This makes maintenance difficult.

The Mediator reduces it to:

```text
Object A
     \
Object B ----> Mediator
     /
Object C
```

Each object knows only the mediator.

---

## Category

**Behavioral Pattern**

---

## What problem does it solve?

Without Mediator:

```text
Button ↔ TextBox
Button ↔ Label
TextBox ↔ Label
Label ↔ ComboBox
ComboBox ↔ CheckBox
...
```

Too many dependencies.

With Mediator:

```text
Button
   |
TextBox
   |
Label
   |
ComboBox
   |
Mediator
```

One communication hub.

---

# 2. Problem Statement

Imagine a **Flight Booking System**.

Components:

* Departure City
* Arrival City
* Date Picker
* Seat Selection
* Passenger List
* Payment Button
* Price Label

When the departure city changes:

* Available flights change.
* Seat map changes.
* Price changes.
* Passenger validation changes.

Without Mediator:

Each widget updates every other widget directly.

Soon every widget contains business logic.

---

# 3. Motivation

Architects observed that GUI controls and business components often become tightly coupled.

Example:

```text
Patient Window

↓

Toolbar

↓

Image Viewer

↓

Status Bar

↓

Measurement Panel

↓

Histogram
```

If every component knows every other component:

* testing becomes difficult,
* reuse becomes impossible,
* maintenance becomes expensive.

The solution:

Create one object responsible for coordination.

---

# 4. Real-World Analogy

## Air Traffic Control (ATC)

Imagine an airport.

Aircraft should **not** communicate directly with each other.

Without ATC:

```text
Plane A ↔ Plane B
Plane A ↔ Plane C
Plane B ↔ Plane C
```

Chaos.

Instead:

```text
Plane A
    \
Plane B ---> Air Traffic Controller
    /
Plane C
```

Every plane communicates only with ATC.

The ATC decides:

* who lands,
* who takes off,
* who waits.

---

## Mapping

| Airport                | Software          |
| ---------------------- | ----------------- |
| Air Traffic Controller | Mediator          |
| Aircraft               | Colleague Objects |
| Pilot                  | Client            |
| Runway Instructions    | Messages          |

---

# 5. Software Scenario

Mediator is useful whenever many objects collaborate.

### Desktop Applications

* Dialog boxes
* Wizards
* Forms
* Dashboards

---

### Qt Applications

Imagine a **Patient Registration Dialog**.

Widgets:

```text
Patient Name
Age
Gender
Department
Doctor
Submit Button
```

Rules:

* Submit button enabled only when mandatory fields are valid.
* Department selection updates available doctors.
* Age determines available examination options.

Instead of widgets communicating directly:

```text
Widgets

↓

RegistrationMediator
```

Qt's **signals and slots** reduce coupling between sender and receiver, but a complex dialog often still benefits from a mediator (or controller) that coordinates application logic.

---

### CAD Software

Changing:

```text
Selected Object

↓

Property Panel

↓

Toolbar

↓

Status Bar

↓

Scene
```

Mediator coordinates updates.

---

### Medical Imaging

Changing the active image should update:

* Slice Slider
* Window/Level Panel
* Histogram
* Crosshair
* Patient Information
* Thumbnail Panel

One mediator coordinates all updates.

---

# 6. UML Class Diagram

```text
                   +----------------------+
                   |      Mediator        |
                   +----------------------+
                   | +notify(sender,msg)  |
                   +----------+-----------+
                              ^
                              |
                  -------------------------
                  |           |           |
        +---------------+ +---------------+ +---------------+
        | Colleague A   | | Colleague B   | | Colleague C   |
        +---------------+ +---------------+ +---------------+
        | mediator       | | mediator      | | mediator      |
        +---------------+ +---------------+ +---------------+
```

---

## Responsibilities

### Mediator

Coordinates communication.

Contains workflow logic.

---

### Colleague

Performs its own task.

Notifies mediator about events.

Does **not** know other colleagues.

---

### Client

Creates:

* Mediator
* Colleagues

Connects them together.

---

# 7. Participants

## Mediator Interface

Defines:

```cpp
notify(sender, event)
```

---

## Concrete Mediator

Example:

```text
PatientDialogMediator
```

Coordinates:

* buttons,
* labels,
* text boxes,
* validation.

---

## Colleague

Examples:

```text
LoginButton
UsernameField
PasswordField
RoleComboBox
```

Each stores:

```cpp
Mediator*
```

---

## Client

Creates everything and wires the dependencies.

---

# 8. Collaboration

Runtime Flow

```text
User Types Username

↓

UsernameField

↓

Mediator.notify()

↓

Validate Input

↓

Enable Login Button

↓

Update Error Label
```

Notice:

UsernameField never talks directly to LoginButton.

The mediator decides what happens.

---

# 9. C++ Example

```cpp
#include <iostream>
#include <memory>
#include <string>

class Mediator
{
public:
    virtual ~Mediator() = default;
    virtual void notify(const std::string& sender) = 0;
};

class LoginButton;
class UsernameField;

class LoginMediator : public Mediator
{
public:
    void setControls(UsernameField* user,
                     LoginButton* button)
    {
        m_user = user;
        m_button = button;
    }

    void notify(const std::string& sender) override;

private:
    UsernameField* m_user = nullptr;
    LoginButton* m_button = nullptr;
};

class UsernameField
{
public:
    explicit UsernameField(Mediator& mediator)
        : m_mediator(mediator)
    {}

    void textChanged()
    {
        std::cout << "Username changed\n";
        m_mediator.notify("Username");
    }

private:
    Mediator& m_mediator;
};

class LoginButton
{
public:
    explicit LoginButton(Mediator&)
    {}

    void enable()
    {
        std::cout << "Login button enabled\n";
    }
};

void LoginMediator::notify(const std::string& sender)
{
    if (sender == "Username")
    {
        m_button->enable();
    }
}

int main()
{
    LoginMediator mediator;

    UsernameField user(mediator);
    LoginButton button(mediator);

    mediator.setControls(&user, &button);

    user.textChanged();
}
```

---

## Design Focus

Notice:

`UsernameField` never calls:

```cpp
button.enable();
```

Instead:

```cpp
mediator.notify(...);
```

The mediator coordinates the interaction.

---

# 10. Qt Example

Imagine a **Treatment Planning System** dialog.

Widgets:

```text
Beam Energy ComboBox
Beam Angle SpinBox
MLC Editor
Calculate Dose Button
Status Label
```

Rules:

* Invalid beam angle disables **Calculate Dose**.
* Changing energy updates available MLC options.
* Dose calculation progress updates the status label.

Architecture:

```text
BeamEnergyWidget
AngleWidget
MLCWidget
DoseButton
StatusLabel
        ↓
TreatmentPlanMediator
```

Every widget emits signals.

The mediator receives them and decides what to update.

This keeps each widget reusable.

---

# 11. Medical Software Example

Imagine a **CT Viewer**.

Components:

```text
Thumbnail Panel
Slice Slider
Image Viewer
Histogram
Patient Info
Measurement Tool
```

Scenario:

Radiologist selects a new study.

Without Mediator:

Each component updates all others.

With Mediator:

```text
ThumbnailPanel

↓

ViewerMediator

↓

Load Image

↓

Update Histogram

↓

Update Patient Info

↓

Reset Measurement Tool

↓

Move Slice Slider
```

Each component remains focused on its own responsibility.

---

# 12. Advantages

### Loose Coupling

Colleagues don't know each other.

### Easier Maintenance

Workflow changes are centralized.

### Better Reusability

Widgets can be reused in different dialogs.

### Simpler Testing

You can test colleagues independently by mocking the mediator.

### Clear Coordination

Business workflow is in one place.

---

# 13. Disadvantages

### Mediator Can Grow Large

A poorly designed mediator can become a **God Object**.

### Centralized Complexity

Moving all coordination into one class may make it complex.

### Extra Indirection

Communication goes through another object.

### When NOT to Use

Avoid Mediator when:

* only two objects communicate,
* interactions are simple,
* direct collaboration is clearer.

---

# 14. Best Practices

* Keep colleagues independent.
* Put workflow logic in the mediator, not in widgets.
* Split very large mediators into smaller mediators.
* Define a clear mediator interface.
* Avoid business logic duplication between colleagues and mediator.

---

# 15. Common Mistakes

### Mistake 1

Allowing colleagues to communicate directly.

This defeats the purpose.

---

### Mistake 2

Making the mediator perform business operations.

The mediator coordinates.

Business logic should remain in domain services or model objects.

---

### Mistake 3

One mediator for the entire application.

Prefer:

```text
LoginMediator
PatientMediator
BeamEditorMediator
ReportMediator
```

instead of:

```text
ApplicationMediator
```

---

### Mistake 4

Using Mediator when Observer is sufficient.

If objects only need notifications, Observer is often a better choice.

---

# 16. Pattern Variations

## 1. GUI Mediator

Coordinates dialog controls.

Most common.

---

## 2. Workflow Mediator

Coordinates business process steps.

---

## 3. Event-Based Mediator

Uses an event bus internally.

Useful for larger systems.

---

## 4. Distributed Mediator

Coordinates services across processes or machines.

Common in enterprise applications.

---

# 17. Related Patterns

| Pattern                   | Difference                                                                           |
| ------------------------- | ------------------------------------------------------------------------------------ |
| Mediator                  | Centralizes communication between colleagues.                                        |
| Observer                  | Broadcasts notifications to many observers.                                          |
| Command                   | Encapsulates an action.                                                              |
| Facade                    | Simplifies access to a subsystem.                                                    |
| Controller (MVC/MVP/MVVM) | Handles user interaction and application flow; may internally use mediator concepts. |

---

## Mediator vs Observer (Very Important)

This is a classic interview question.

### Observer

Question:

> **"Who should be notified?"**

One subject notifies many observers.

Example:

```text
Temperature Sensor

↓

Display

↓

Logger

↓

Alarm
```

---

### Mediator

Question:

> **"How should these objects coordinate?"**

Example:

```text
TextBox

↓

Mediator

↓

Button

↓

Label

↓

ComboBox
```

Mediator manages the interactions.

---

# 18. Industry Usage

Mediator is widely used in GUI-heavy and workflow-oriented applications.

* **Microsoft:** Complex dialog boxes and Visual Studio tool windows.
* **Google:** Workflow coordination in productivity applications.
* **Adobe:** Photoshop property panels and tool interactions.
* **Qt:** Dialog controllers, wizard pages, and application coordinators often follow mediator principles alongside signals and slots.
* **ZEISS / Siemens / Philips:** Medical device UIs coordinating viewers, measurement panels, patient information, and acquisition controls.
* **Autodesk:** CAD property inspectors, command panels, and scene synchronization.
* **Air Traffic Systems:** Air traffic control is a classic real-world analogy for mediator-based coordination.

The architectural goal is **reducing direct dependencies among collaborating objects**.

---

# 19. Interview Questions

## Beginner

1. What problem does the Mediator Pattern solve?
2. Why is Mediator considered a Behavioral Pattern?
3. What is a colleague object?

---

## Intermediate

1. How does Mediator reduce coupling?
2. How is Mediator different from Observer?
3. Why can a mediator become a God Object?

---

## Advanced

1. How would you split a very large mediator into smaller ones?
2. How would you unit test colleagues independently?
3. How would you combine Mediator with MVC or MVVM in a Qt application?

---

## Scenario-Based

Your TPS contains:

* Beam Editor
* Dose Viewer
* DVH Panel
* Patient Information
* Toolbar
* Status Bar

Changing the active beam updates all of them.

Design a Mediator-based architecture to coordinate these updates while keeping each component independent.

---

## Architecture

Design a Mediator architecture for your **College ERP** dashboard.

Components:

* Student Panel
* Attendance Panel
* Fee Panel
* Notification Panel
* Calendar
* Timetable

Requirements:

* Selecting a student updates all related panels.
* Panels do not communicate directly.
* New panels can be added without changing existing panels.

Explain:

* mediator responsibilities,
* colleague responsibilities,
* event flow,
* and how to avoid creating a God Mediator.

---

# 20. Practice Exercises

### Beginner Exercise

Design a login dialog using the Mediator Pattern.

Components:

* Username Field
* Password Field
* Login Button
* Error Label

The Login Button should enable only when both fields are non-empty.

---

### Intermediate Exercise

Design a Qt-based **Patient Registration** dialog.

Widgets:

* Patient Name
* Age
* Gender
* Department
* Doctor
* Submit Button

Explain how a mediator coordinates validation and updates.

---

### Advanced Exercise

Design a **Mediator architecture** for a **Treatment Planning System**.

Components:

* CT Viewer
* Beam Editor
* Structure Panel
* DVH Panel
* Dose Viewer
* Toolbar
* Status Bar
* Optimization Panel

Requirements:

* Selecting a beam updates all relevant components.
* Dose recalculation updates the DVH and Dose Viewer.
* Components remain reusable.
* Avoid a single God Mediator by proposing a modular mediator design.

**Do not implement the solution yet.** Focus on architecture, communication flow, responsibilities, and scalability.

---

# Key Architectural Takeaway

The **Mediator Pattern** is about **centralizing communication while keeping collaborating objects independent**.

A junior developer thinks:

> "I'll let every widget call every other widget directly."

A software architect thinks:

> **"Communication is becoming tangled. I'll introduce a mediator so each component has one dependency, and all coordination happens in a well-defined place."**

This approach produces systems that are easier to maintain, test, extend, and reuse—especially in Qt desktop applications, CAD systems, ERP platforms, and medical device software.

---

# ⭐ Architect's Insight: Mediator in Qt

In professional Qt applications, you'll often combine:

```text
Qt Signals & Slots
        ↓
    Mediator (or Controller)
        ↓
   Domain Services / Models
```

For example:

```text
User changes Beam Angle
        ↓
QDoubleSpinBox emits valueChanged()
        ↓
BeamEditorMediator
        ↓
BeamModel updates
        ↓
DoseEngine notified
        ↓
DVH Panel refreshed
        ↓
Status Bar updated
```

This keeps:

* **Widgets** responsible for presentation,
* **Mediator** responsible for coordination,
* **Models/Services** responsible for business logic.

This separation is a common architecture in complex Qt applications.

---
[⬅️ Iterator Pattern ](/IteratorPattern.md)        |  [Memento Pattern  ➡️](/MementoPattern.md) 
---
## **License**
This project is licensed under the MIT License.

---

Happy Coding!

