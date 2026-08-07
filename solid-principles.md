# **S.O.L.I.D Principles**

The **S.O.L.I.D principles** are a set of five key guidelines for designing object-oriented software systems. These principles were:

* **First introduced** by the renowned computer scientist **Robert C. Martin**, commonly known as *Uncle Bob*.
* The **acronym "S.O.L.I.D"** was later coined by **Michael Feathers** to help remember the five principles.

These principles serve as a **foundation for writing clean, maintainable, and scalable object-oriented code**. They provide a **set of rules and best practices** that guide developers in designing class structures that are robust and easy to manage.

---

## **Why Use S.O.L.I.D?**

> **"To create understandable, readable, and testable code that many developers can collaboratively work on."**

By adhering to the S.O.L.I.D principles, teams can:

* Reduce complexity in codebases
* Enhance code readability and testability
* Make systems easier to maintain and extend
* Encourage better collaboration among developers

---

## **What Does SOLID Stand For?**

| **Letter** | **Principle**                         | **Meaning**                                                                 |
| ---------- | ------------------------------------- | --------------------------------------------------------------------------- |
| **S**      | Single Responsibility Principle (SRP) | A class should have only **one reason to change**.                          |
| **O**      | Open/Closed Principle (OCP)           | Classes should be **open for extension**, but **closed for modification**.  |
| **L**      | Liskov Substitution Principle (LSP)   | Subclasses should be usable **without breaking** the parent class behavior. |
| **I**      | Interface Segregation Principle (ISP) | Don’t force a class to implement **unused interfaces**.                     |
| **D**      | Dependency Inversion Principle (DIP)  | Depend on **abstractions**, not concrete implementations.                   |

---

## **1. Single Responsibility Principle (SRP)**

> A class should do **only one job** and have **one reason to change**.

Every class should have a **single, well-defined responsibility**. This improves clarity, reduces coupling, and makes maintenance easier.


### Simple Idea:

Think of a **remote control**: one button for volume, one for channel. If one button did *everything*, it would be confusing.


### Bad Example

```cpp
#include <iostream>
#include <fstream>
using namespace std;

class Report {
public:
    void generateReport() {
        cout << "Report generated\n";
    }
    void saveToFile() {
        ofstream file("report.txt");
        file << "Saving report to file...";
    }
};
int main() {
    Report report;

    report.generateReport();   // Display message
    report.saveToFile();       // Save to file

    cout << "Report saved to report.txt\n";

    return 0;
}
```

The class `Report` is doing **two jobs**:

* Generating a report
* Saving it to a file

If file-saving logic changes, this class must also change — violating SRP.

---

### Good Example (SRP Applied)

```cpp
#include <iostream>
#include <fstream>
using namespace std;

class Report {
public:
    void generateReport() {
        cout << "Report generated\n";
    }
};

class FileSaver {
public:
    void saveReport() {
        ofstream file("report.txt");
        file << "Saving report to file...";
    }
};

int main() {
    Report report;
    FileSaver fileSaver;

    report.generateReport();   // Display report generation message
    fileSaver.saveReport();    // Save report to file

    cout << "Report saved to report.txt\n";

    return 0;
}

```

Each class now has a **single responsibility**.

---

## **2. Open/Closed Principle (OCP)**

> Classes should be **open for extension** but **closed for modification**.

Systems should be designed so that new functionality can be **added without modifying existing code**, often achieved through abstraction and polymorphism.

### Simple Idea:

Imagine a **phone** where you can add new apps (extend) without opening the device (modifying it).


### Bad Example

```cpp
class AreaCalculator {
public:
    double area(string shape, double a, double b) {
        if (shape == "rectangle") return a * b;
        if (shape == "triangle") return 0.5 * a * b;
        return 0;
    }
};

int main() {
    AreaCalculator calc;

    double rectWidth = 10;
    double rectHeight = 5;
    double triangleBase = 8;
    double triangleHeight = 4;

    cout << "Rectangle Area: " << calc.area("rectangle", rectWidth, rectHeight) << endl;
    cout << "Triangle Area: " << calc.area("triangle", triangleBase, triangleHeight) << endl;

    // Unknown shape test
    cout << "Circle Area: " << calc.area("circle", 10, 0) << endl;

    return 0;
}
```

If we add a new shape (circle, hexagon), we must **modify** this class — breaking OCP.

---

### Good Example (OCP Applied)

```cpp
#include <iostream>
using namespace std;

class Shape {
public:
    virtual double area() = 0;  // abstract method
};

class Rectangle : public Shape {
    double w, h;
public:
    Rectangle(double w, double h): w(w), h(h) {}
    double area() override { return w * h; }
};

class Triangle : public Shape {
    double b, h;
public:
    Triangle(double b, double h): b(b), h(h) {}
    double area() override { return 0.5 * b * h; }
};

void printArea(Shape* s) {
    cout << "Area: " << s->area() << endl;
}
```

 To add a new shape (like Circle), you **extend** the code (create a new class) without modifying existing ones.

or
```cpp
#include <iostream>
using namespace std;

// Abstract base class
class Shape {
public:
    virtual double area() = 0;  // Pure virtual function
    
};

class Rectangle : public Shape {
    double w, h;
public:
    Rectangle(double width, double height) {
        w = width;
        h = height;
    }

    double area() {
        return w * h;
    }
};

class Triangle : public Shape {
    double b, h;
public:
    Triangle(double base, double height) {
        b = base;
        h = height;
    }

    double area() {
        return 0.5 * b * h;
    }
};

void printArea(Shape* s) {
    cout << "Area: " << s->area() << endl;
}

int main() {
    Rectangle r(10, 5);
    Triangle t(8, 4);

    printArea(&r);
    printArea(&t);

    return 0;
}

```

---

## **3. Liskov Substitution Principle (LSP)**

> Subclasses should **replace** their base classes **without breaking** functionality.

 Objects of a subclass should be able to **replace objects of the superclass** without affecting the program’s behavior. This ensures that inheritance is used correctly.


### Simple Idea:

If a **bird** can fly, any specific bird (e.g., sparrow) should also fly. But a **penguin** can’t fly — so it **shouldn’t be a subclass of flying birds**.


### Bad Example

```cpp
class Bird {
public:
    virtual void fly() { cout << "Flying\n"; }
};

class Penguin : public Bird {
public:
    void fly() override { cout << "Penguins can’t fly!\n"; }  // Wrong behavior
};
```

`Penguin` violates LSP — it **can’t fly**, so substituting it for a `Bird` breaks the program logic.

---

### Good Example (LSP Applied)

```cpp
class Bird { public: virtual void eat() = 0; };
class FlyingBird : public Bird { public: virtual void fly() = 0; };

class Sparrow : public FlyingBird {
public:
    void eat() override { cout << "Sparrow eating\n"; }
    void fly() override { cout << "Sparrow flying\n"; }
};

class Penguin : public Bird {
public:
    void eat() override { cout << "Penguin eating\n"; }
};
```

Now, `Penguin` doesn’t break anything — it doesn’t belong to `FlyingBird`.

---

## **4. Interface Segregation Principle (ISP)**

> Don’t force a class to **implement interfaces it doesn’t use**.

Favor **many small, specific interfaces** over one large, general-purpose interface. This helps keep implementations clean and focused.

###  Simple Idea:

Don’t make a **printer** class also handle scanning and faxing if it doesn’t need to.


### Bad Example

```cpp
class IWorker {
public:
    virtual void work() = 0;
    virtual void eat() = 0;
};

class Robot : public IWorker {
public:
    void work() override { cout << "Robot working\n"; }
    void eat() override { cout << "Robot doesn't eat!\n"; } // unnecessary
};
```

---

### Good Example (ISP Applied)

```cpp
class IWorkable { public: virtual void work() = 0; };
class IEatable { public: virtual void eat() = 0; };

class Human : public IWorkable, public IEatable {
public:
    void work() override { cout << "Human working\n"; }
    void eat() override { cout << "Human eating\n"; }
};

class Robot : public IWorkable {
public:
    void work() override { cout << "Robot working\n"; }
};
```

Interfaces are now **segregated** — no unnecessary methods.

---

## **5. Dependency Inversion Principle (DIP)**

> High-level modules should not depend on low-level modules;
> both should depend on **abstractions**.

Code should depend on **interfaces or abstract classes**, not on concrete implementations. This **reduces tight coupling** and increases flexibility.

### Simple Idea:

If you're making tea, you don't care which **brand of kettle** boils the water — just that it does the job.


### Bad Example

```cpp
class LightBulb {
public:
    void turnOn() { cout << "Light ON\n"; }
};

class Switch {
    LightBulb bulb;
public:
    void operate() { bulb.turnOn(); }
};
```

 `Switch` is tightly coupled to `LightBulb`.
If we add a `Fan`, we must modify the `Switch` class.

---

###  Good Example (DIP Applied)

```cpp
#include <iostream>
using namespace std;

class Device {
public:
    virtual void turnOn() = 0;
};

class LightBulb : public Device {
public:
    void turnOn() override { cout << "Light ON\n"; }
};

class Fan : public Device {
public:
    void turnOn() override { cout << "Fan ON\n"; }
};

class Switch {
    Device* device;
public:
    Switch(Device* dev): device(dev) {}
    void operate() { device->turnOn(); }
};

int main() {
    LightBulb bulb;
    Fan fan;
    Switch s1(&bulb);
    Switch s2(&fan);

    s1.operate();
    s2.operate();
}
```

 Now the `Switch` depends on the **abstract interface**, not a specific class — it can work with any `Device`.

---

## **Summary Table**

| **Principle** | **Key Idea**          | **Example Concept**                           |
| ------------- | --------------------- | --------------------------------------------- |
| **S**         | Single Responsibility | One class → one job                           |
| **O**         | Open/Closed           | Extend, don’t modify                          |
| **L**         | Liskov Substitution   | Derived classes shouldn’t break base behavior |
| **I**         | Interface Segregation | Small, specific interfaces                    |
| **D**         | Dependency Inversion  | Depend on abstractions, not concrete classes  |

---
[⬅️ Introduction to Design Patterns](/introduction.md)        |  [Singleton Pattern ➡️](/singleton.md) 
---
## **License**
This project is licensed under the MIT License.

---

Happy Coding!



