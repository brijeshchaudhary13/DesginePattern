# Design Patterns Master Course

# Lesson 18 — Iterator Pattern (Behavioral Pattern)

---

# Before We Start

The **Iterator Pattern** is probably the **most frequently used design pattern that developers don't realize they're using**.

If you've written C++, you've already used it:

```cpp
std::vector<int> numbers = {10, 20, 30};

for (auto it = numbers.begin(); it != numbers.end(); ++it)
{
    std::cout << *it << '\n';
}
```

`it` is an **Iterator**.

Likewise:

* STL Iterators
* Qt Iterators (`QListIterator`, `QMapIterator`)
* Database cursors
* Tree traversals
* Scene Graphs
* File System Walkers
* Medical Imaging Pipelines

all use the Iterator Pattern.

Today, however, we are **not** learning how to use STL iterators.

We are learning **why architects created the Iterator Pattern** and **how to design systems that hide complex data structures behind a simple traversal interface**.

---

# The Architect's Way of Thinking

Imagine you're developing a **Treatment Planning System (TPS)**.

A treatment plan contains:

```text
Treatment Plan
│
├── Patient
├── CT Images
├── Structures
├── Beams
├── Dose Matrix
├── DVH Curves
└── Reports
```

Tomorrow, management asks:

> "Export every beam."

A junior developer writes:

```cpp
for (size_t i = 0; i < plan.beams.size(); i++)
{
    ...
}
```

Later, the storage changes:

```text
vector
↓

linked list
```

Every loop in the application breaks.

An architect asks:

> **"Why should clients know how beams are stored?"**

The answer:

**They shouldn't.**

The client should only know:

```cpp
iterator.next();
```

That is the Iterator Pattern.

---

# 1. Introduction

## What is the Iterator Pattern?

The **Iterator Pattern** provides a way to access elements of a collection **sequentially without exposing its internal representation**.

Simple definition:

> **Separate traversal logic from the collection itself.**

---

## Why was it created?

Collections can be implemented using:

* Array
* Linked List
* Tree
* Graph
* Hash Table
* Database
* File

Clients should not need different code for every collection type.

---

## Category

**Behavioral Pattern**

---

## What problem does it solve?

Without Iterator:

```cpp
for(int i=0;i<vector.size();i++)
```

Tomorrow:

```text
vector

↓

Tree
```

Now:

```cpp
tree.left()

tree.right()
```

All client code changes.

With Iterator:

```cpp
while(iterator.hasNext())
{
    auto object = iterator.next();
}
```

Client code never changes.

---

# 2. Problem Statement

Imagine a **Hospital Patient Database**.

Today:

Patients stored in:

```text
Vector
```

Next year:

```text
Balanced Tree
```

Later:

```text
SQL Database
```

Without Iterator:

Every report:

Every export:

Every search:

Every UI component:

must change.

---

# 3. Motivation

Architects realized:

Traversal and storage are different responsibilities.

Storage:

```text
Array

Tree

Database
```

Traversal:

```text
Next

Previous

Current

End
```

These should be separated.

This follows the **Single Responsibility Principle**.

---

# 4. Real-World Analogy

## TV Playlist

Imagine Netflix.

You don't know:

* SQL tables
* Video servers
* File storage
* CDN

You simply press:

```text
Next Episode
```

Internally:

Netflix decides where the next episode comes from.

### Mapping

| Netflix     | Software   |
| ----------- | ---------- |
| Playlist    | Collection |
| Next Button | Iterator   |
| Episode     | Element    |
| User        | Client     |

---

# 5. Software Scenario

Iterator appears wherever collections need to be traversed.

### Desktop Applications

* File Explorer
* Playlist
* History
* Recent Files

---

### Qt Applications

Qt provides several iterator classes, including:

* `QListIterator`
* `QMutableListIterator`
* `QMapIterator`
* `QHashIterator`

Example:

```cpp
QList<QString> names;

QListIterator<QString> it(names);

while(it.hasNext())
{
    qDebug() << it.next();
}
```

Qt also encourages range-based loops and STL-style iterators for many containers.

---

### CAD Software

Traverse:

* Shapes
* Layers
* Constraints

---

### Medical Imaging

Traverse:

* Slices
* Voxels
* ROIs
* Beam Collections

---

### Databases

Database Cursor:

```text
First Row

↓

Next Row

↓

Next Row
```

This is another iterator concept.

---

# 6. UML Class Diagram

```text
             +----------------------+
             |      Iterator        |
             +----------------------+
             | +first()             |
             | +next()              |
             | +hasNext()           |
             | +current()           |
             +----------+-----------+
                        ^
                        |
             +----------------------+
             | ConcreteIterator     |
             +----------------------+
             | collection           |
             | currentIndex         |
             +----------+-----------+
                        |
                        |
             +----------------------+
             |     Aggregate        |
             +----------------------+
             | +createIterator()    |
             +----------+-----------+
                        ^
                        |
             +----------------------+
             | ConcreteAggregate    |
             +----------------------+
```

---

## Responsibilities

### Aggregate

Creates iterator.

---

### Iterator

Defines traversal operations.

---

### Concrete Iterator

Tracks traversal state.

---

### Client

Uses iterator.

Never accesses collection internals.

---

# 7. Participants

## Aggregate

Example:

```text
BeamCollection
```

Provides:

```cpp
createIterator()
```

---

## Concrete Aggregate

Stores:

```text
vector<Beam>
```

or

```text
list<Beam>
```

---

## Iterator

Interface:

```cpp
next()

current()

hasNext()
```

---

## Concrete Iterator

Maintains:

```text
Current Position
```

---

## Client

Uses iterator only.

---

# 8. Collaboration

Runtime Flow

```text
Client

↓

Collection

↓

Create Iterator

↓

Has Next?

↓

Yes

↓

Current

↓

Next

↓

Has Next?

↓

No

↓

Finish
```

Notice:

The client never sees the internal container.

---

# 9. C++ Example

```cpp
#include <iostream>
#include <memory>
#include <vector>

// Aggregate
class NumberCollection;

// Iterator Interface
class Iterator
{
public:
    virtual ~Iterator() = default;
    virtual bool hasNext() const = 0;
    virtual int next() = 0;
};

// Concrete Iterator
class NumberIterator : public Iterator
{
public:
    explicit NumberIterator(const std::vector<int>& numbers)
        : m_numbers(numbers)
    {}

    bool hasNext() const override
    {
        return m_index < m_numbers.size();
    }

    int next() override
    {
        return m_numbers[m_index++];
    }

private:
    const std::vector<int>& m_numbers;
    std::size_t m_index = 0;
};

// Aggregate
class NumberCollection
{
public:
    void add(int value)
    {
        m_numbers.push_back(value);
    }

    std::unique_ptr<Iterator> createIterator() const
    {
        return std::make_unique<NumberIterator>(m_numbers);
    }

private:
    std::vector<int> m_numbers;
};

int main()
{
    NumberCollection collection;

    collection.add(10);
    collection.add(20);
    collection.add(30);

    auto iterator = collection.createIterator();

    while(iterator->hasNext())
    {
        std::cout << iterator->next() << '\n';
    }
}
```

---

## Design Focus

Notice:

The client never accesses:

```cpp
m_numbers
```

directly.

It simply asks:

```cpp
iterator->next();
```

Tomorrow the collection changes to:

```text
Linked List
```

The client code remains unchanged.

---

# 10. Qt Example

Imagine a Qt-based **Medical Image Browser**.

Internally:

```text
QVector<Image>

↓

Later

↓

QList<Image>

↓

Later

↓

Database
```

The UI shouldn't change.

Instead:

```cpp
auto it = imageCollection.createIterator();

while(it->hasNext())
{
    auto image = it->next();

    display(image);
}
```

The iterator hides how images are stored.

This also makes testing easier because mock collections can provide the same iterator interface.

---

# 11. Medical Software Example

Imagine a **Treatment Planning System**.

Treatment Plan:

```text
Plan
│
├── Beam 1
├── Beam 2
├── Beam 3
├── Beam 4
```

Suppose beams are initially stored in:

```text
std::vector<Beam>
```

Later, optimization requires:

```text
std::list<Beam>
```

Or perhaps beams come directly from a database.

Without Iterator:

Every export routine:

Every optimization algorithm:

Every UI list:

must change.

With Iterator:

```cpp
auto it = plan.createBeamIterator();

while(it->hasNext())
{
    Beam beam = it->next();

    optimize(beam);
}
```

The optimizer doesn't care where beams come from.

---

# 12. Advantages

### Encapsulation

The collection hides its internal representation.

### Loose Coupling

Clients depend on the iterator interface, not the container.

### Flexibility

Storage implementation can change without affecting clients.

### Multiple Traversals

Different iterators can traverse the same collection in different ways.

Example:

* Forward
* Reverse
* Filtered
* Breadth-First
* Depth-First

---

# 13. Disadvantages

### Extra Classes

Requires iterator classes.

### Additional Indirection

Traversal goes through another object.

### Synchronization

Iterators may become invalid if the collection changes during traversal.

### When NOT to Use

Avoid creating custom iterators when:

* STL iterators already solve the problem.
* The collection is trivial.
* No abstraction benefit exists.

---

# 14. Best Practices

* Prefer existing STL or Qt iterators unless custom traversal is required.
* Keep traversal logic out of the collection.
* Support const iterators for read-only traversal.
* Clearly document iterator invalidation rules.
* Consider lazy iteration for expensive data sources.

---

# 15. Common Mistakes

### Mistake 1

Exposing the internal container.

Example:

```cpp
std::vector<Beam>& getBeams();
```

Now clients depend on the storage type.

---

### Mistake 2

Mixing traversal and modification.

Iterators should have a clear contract.

If modification is supported, define it explicitly.

---

### Mistake 3

Invalidating iterators unexpectedly.

For example, inserting into certain containers may invalidate existing iterators.

Architects document these behaviors carefully.

---

### Mistake 4

Writing custom iterators unnecessarily.

Use STL or Qt iterators when they already provide the required functionality.

---

# 16. Pattern Variations

## 1. Forward Iterator

```text
A → B → C → D
```

---

## 2. Reverse Iterator

```text
D → C → B → A
```

---

## 3. Filter Iterator

Returns only matching elements.

Example:

```text
Only Active Patients
```

---

## 4. Tree Iterator

Traversal options:

* Preorder
* Inorder
* Postorder
* Level Order

---

## 5. Lazy Iterator

Loads data only when requested.

Useful for:

* Databases
* Large DICOM studies
* Remote services

---

# 17. Related Patterns

| Pattern   | Difference                                       |
| --------- | ------------------------------------------------ |
| Iterator  | Traverses collections.                           |
| Composite | Organizes objects into trees.                    |
| Visitor   | Performs operations while traversing structures. |
| Flyweight | Shares state among objects.                      |
| Memento   | Stores object state.                             |

---

## Iterator + Composite

This combination is extremely common.

Example:

```text
Scene Graph

↓

Iterator

↓

Visit Every Node
```

Composite stores the hierarchy.

Iterator traverses it.

---

# 18. Industry Usage

Iterator is foundational in modern software.

* **Microsoft:** STL, .NET collections, Visual Studio project trees.
* **Google:** Container libraries and data processing frameworks.
* **Adobe:** Layer traversal, object collections, document models.
* **Qt:** Container iterators, model/view traversal, item models.
* **ZEISS / Siemens / Philips:** Image slices, beam collections, ROI traversal, patient datasets.
* **Autodesk:** CAD entity traversal and scene graphs.
* **Databases:** Cursor-based iteration over result sets.

The architectural goal is **decoupling traversal from storage**.

---

# 19. Interview Questions

## Beginner

1. What problem does the Iterator Pattern solve?
2. Why shouldn't clients know the collection's internal structure?
3. What is the role of the Iterator interface?

---

## Intermediate

1. How does Iterator support the Open/Closed Principle?
2. What is iterator invalidation?
3. Why are const iterators useful?

---

## Advanced

1. How would you implement a thread-safe iterator?
2. How would you design an iterator for a database that streams millions of rows?
3. How would you support multiple traversal strategies over the same collection?

---

## Scenario-Based

Your TPS stores beams in a `std::vector` today, but future versions may use a database-backed collection.

Design an Iterator-based architecture that keeps optimization algorithms unchanged.

---

## Architecture

Design an Iterator architecture for a **College ERP**.

Collections:

* Students
* Teachers
* Courses
* Departments

Requirements:

* Forward iteration
* Reverse iteration
* Filter by department
* Lazy loading from the database

Explain:

* iterator hierarchy,
* aggregate responsibilities,
* traversal strategies,
* and invalidation rules.

---

# 20. Practice Exercises

### Beginner Exercise

Design a `BookCollection` with a custom iterator.

Support:

* `hasNext()`
* `next()`

Draw the UML and explain the collaboration.

---

### Intermediate Exercise

Design a Qt-based image browser where image storage may switch between:

* `QVector`
* `QList`
* Database

Ensure the UI code remains unchanged using the Iterator Pattern.

---

### Advanced Exercise

Design an **Iterator architecture** for a **Treatment Planning System**.

Collections:

* CT Slices
* MRI Slices
* Structures
* Beams
* Dose Voxels

Requirements:

* Forward and reverse traversal
* Filter only active beams
* Lazy loading of image slices
* Support future database-backed storage
* Thread-safe read-only iteration for visualization

**Do not implement the solution yet.** Focus on architecture, traversal strategies, ownership, and iterator lifecycle.

---

# Key Architectural Takeaway

The **Iterator Pattern** is about **separating traversal from storage**.

A junior developer thinks:

> "I'll loop directly over the `std::vector`."

A software architect thinks:

> **"The storage mechanism may change, but the traversal contract should remain stable. I'll expose an iterator so clients depend on behavior, not implementation."**

This mindset enables flexible, maintainable systems where collections can evolve without forcing changes throughout the codebase.

---

# ⭐ Architect's Insight: Iterator + Visitor

A common enterprise combination is:

```text
Composite
      ↓
Iterator
      ↓
Visitor
```

For example, in a TPS:

```text
Treatment Plan
      ↓
Iterator visits every Beam
      ↓
Visitor performs:
    • Dose validation
    • Export
    • Statistics
    • Report generation
```

The responsibilities remain clean:

* **Composite** → organizes objects.
* **Iterator** → traverses them.
* **Visitor** → performs operations.

This separation is a hallmark of well-designed object-oriented architectures.

---

## What You'll Learn Next

Type **`NEXT`** to continue with **Lesson 19: Mediator Pattern**, where you'll learn how to eliminate the "spaghetti communication" that occurs when many objects talk directly to one another. This pattern is heavily used in GUI frameworks, dialogs, workflow engines, air traffic control systems, and complex medical device UIs.
