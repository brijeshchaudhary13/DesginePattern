

# Lesson 6 — Object Pool Pattern 

---

# Before We Start

The **Object Pool Pattern** is one of the most practical patterns in high-performance software.

It is widely used in:

* Game Engines
* Database Systems
* Web Servers
* Medical Imaging Software
* GPU Programming
* Network Servers
* Real-Time Systems

Unlike the previous patterns, this pattern is **primarily about performance and resource management**, not just object creation.

---

# The Architect's Way of Thinking

Let's compare the patterns we've learned.

| Pattern          | Main Question                                              |
| ---------------- | ---------------------------------------------------------- |
| Singleton        | How many objects should exist?                             |
| Factory Method   | Which object should be created?                            |
| Abstract Factory | Which family should be created?                            |
| Builder          | How should the object be constructed?                      |
| Prototype        | Can I copy an existing object?                             |
| Object Pool      | Can I reuse existing objects instead of creating new ones? |

This is the mindset of a performance engineer.

---

# 1. Introduction

## What is the Object Pool Pattern?

The **Object Pool Pattern** maintains a collection of reusable objects.

Instead of creating and destroying objects repeatedly, the application:

1. Creates a pool of objects.
2. Hands them out when needed.
3. Returns them to the pool after use.
4. Reuses them later.

Think of it as **borrowing** an object rather than **owning** it forever.

---

## Why was it created?

Creating an object is not always cheap.

Some objects require:

* Memory allocation
* GPU memory allocation
* File opening
* Database connections
* Thread creation
* Network sockets
* DICOM parsing
* Large image buffers

Repeatedly creating and destroying these objects wastes CPU time and increases memory fragmentation.

---

## Category

**Creational Pattern**

---

## What problem does it solve?

Suppose a TPS needs to calculate dose for 500 beams.

Without a pool:

```text
Create BeamBuffer

↓

Use

↓

Destroy

↓

Create BeamBuffer

↓

Destroy

↓

Create BeamBuffer

↓

Destroy

...
```

The CPU spends significant time allocating and freeing memory.

With Object Pool:

```text
Create Pool

↓

Borrow Buffer

↓

Use

↓

Return Buffer

↓

Borrow Same Buffer

↓

Use

↓

Return
```

Only one allocation may be needed for many uses.

---

# 2. Problem Statement

Imagine a **Medical Image Processing Pipeline**.

Every image filter needs:

* Temporary image buffer
* Histogram buffer
* GPU texture
* Scratch memory

Without Object Pool:

```cpp
processImage()
{
    ImageBuffer buffer;

    ...
}
```

This happens thousands of times.

Each call performs:

* allocation
* initialization
* destruction

Now imagine processing:

```text
10,000 CT slices
```

Performance drops significantly.

---

Another example:

Database server.

Every request creates:

```text
Database Connection

↓

Query

↓

Close Connection
```

Creating connections repeatedly is expensive.

Instead:

```text
Connection Pool
```

reuses existing connections.

---

# 3. Motivation

Developers noticed that many objects have **short lifetimes** but are **expensive to create**.

Examples:

* Database connections
* Thread objects
* Image buffers
* GPU memory blocks
* Network sockets

Instead of destroying them, keep them ready for reuse.

The Object Pool Pattern formalizes this optimization.

---

# 4. Real-World Analogy

## Library

Imagine a university library.

There are:

```text
100 Books
```

Students borrow books.

```text
Student

↓

Borrow Book

↓

Read

↓

Return Book

↓

Another Student Borrows Same Book
```

The library does **not** print a new copy every time someone wants to read it.

### Mapping

| Library | Software        |
| ------- | --------------- |
| Book    | Reusable Object |
| Student | Client          |
| Library | Object Pool     |
| Borrow  | Acquire         |
| Return  | Release         |

The key idea:

Objects are **shared over time**, not simultaneously by multiple clients.

---

# 5. Software Scenario

Object Pool appears whenever creating objects is expensive and objects can be safely reused.

### Desktop Applications

* Printer jobs
* Document buffers
* Image caches

---

### Qt Applications

Possible candidates:

* Reusable image buffers (`QImage`)
* Temporary rendering resources
* Worker objects for controlled thread usage (though `QThreadPool` is often a better fit for thread management)

Qt already provides pooling concepts in places such as `QThreadPool`, which manages reusable worker threads.

---

### CAD Software

* Geometry buffers
* Rendering resources
* Temporary meshes

---

### Medical Imaging

* CT slice buffers
* MRI reconstruction buffers
* GPU textures
* Volume rendering memory
* DICOM parsing buffers

---

### Game Engines

* Bullets
* Enemies
* Particle systems
* Audio channels

---

### Databases

* Database connection pools

---

### Browsers

* HTTP connection pools
* Renderer process pools

---

# 6. UML Class Diagram

```text
                  +----------------------+
                  |      ObjectPool      |
                  +----------------------+
                  | acquire()            |
                  | release()            |
                  +----------+-----------+
                             |
               -----------------------------
               |             |             |
         +-----------+ +-----------+ +-----------+
         | Object 1  | | Object 2  | | Object 3  |
         +-----------+ +-----------+ +-----------+

                    ^
                    |
               Client borrows
                    |
               Client returns
```

---

## Responsibilities

### Object Pool

Responsible for:

* creating reusable objects,
* tracking available objects,
* lending objects,
* accepting returned objects.

---

### Pooled Object

A reusable resource.

Example:

```text
ImageBuffer

DatabaseConnection

BeamBuffer

GPUTexture
```

---

### Client

Borrows an object.

Uses it.

Returns it.

---

# 7. Participants

## Object Pool

Maintains:

```text
Available Objects

+

In-Use Objects
```

---

## Pooled Object

The reusable object.

---

## Client

Requests:

```cpp
buffer = pool.acquire();
```

Later:

```cpp
pool.release(buffer);
```

---

# 8. Collaboration

Runtime Flow

```text
Application Starts

↓

Pool Creates 20 Objects

↓

Client Requests Object

↓

Pool Returns Object

↓

Client Uses Object

↓

Client Returns Object

↓

Pool Marks Object Available

↓

Next Client Uses Same Object
```

The object is reused instead of destroyed.

---

# 9. C++ Example

```cpp
#include <iostream>
#include <memory>
#include <vector>

class ImageBuffer
{
public:
    void process()
    {
        std::cout << "Processing image...\n";
    }
};

class ImageBufferPool
{
public:
    ImageBufferPool(std::size_t size)
    {
        for (std::size_t i = 0; i < size; ++i)
        {
            m_available.push_back(std::make_unique<ImageBuffer>());
        }
    }

    std::unique_ptr<ImageBuffer> acquire()
    {
        if (m_available.empty())
        {
            return nullptr;
        }

        auto buffer = std::move(m_available.back());
        m_available.pop_back();
        return buffer;
    }

    void release(std::unique_ptr<ImageBuffer> buffer)
    {
        m_available.push_back(std::move(buffer));
    }

private:
    std::vector<std::unique_ptr<ImageBuffer>> m_available;
};

int main()
{
    ImageBufferPool pool(2);

    auto buffer = pool.acquire();

    if (buffer)
    {
        buffer->process();
        pool.release(std::move(buffer));
    }
}
```

### Design Focus

Architecturally, notice:

* The pool owns the objects.
* Clients borrow them temporarily.
* Ownership returns to the pool after use.
* The pool can enforce limits on the number of active objects.

In production systems, pools often use RAII wrappers or smart handles so objects are automatically returned even if exceptions occur.

---

# 10. Qt Example

Suppose a Qt DICOM Viewer displays hundreds of thumbnails.

Instead of creating a new `QImage` buffer for every thumbnail:

```text
ThumbnailPool

↓

Acquire Image Buffer

↓

Load Thumbnail

↓

Display

↓

Release Buffer
```

Another real Qt example is `QThreadPool`, which keeps worker threads alive and reuses them for new tasks instead of creating and destroying threads repeatedly.

---

# 11. Medical Software Example

Consider a **CT Volume Renderer**.

Each rendering operation requires:

```text
Volume Buffer

↓

Gradient Buffer

↓

GPU Texture

↓

Temporary Memory
```

Allocating these resources for every frame is expensive.

Instead:

```text
RenderingResourcePool

↓

Acquire GPU Texture

↓

Render

↓

Release GPU Texture
```

Similarly, in a **Treatment Planning System**, a dose engine may reuse:

* Dose grids
* Temporary voxel buffers
* Matrix workspaces
* Interpolation buffers

instead of reallocating them for every calculation.

This improves performance and reduces memory fragmentation.

---

# 12. Advantages

### Performance

Reduces repeated allocation and deallocation.

### Memory Efficiency

Encourages controlled reuse of expensive resources.

### Predictable Resource Usage

A fixed-size pool can limit memory consumption.

### Reduced Fragmentation

Frequent allocations and deallocations are minimized.

### Scalability

Useful for high-throughput systems handling many repeated operations.

---

# 13. Disadvantages

### Complexity

Requires lifecycle management.

### Limited Capacity

If all objects are in use, clients may have to wait, fail, or trigger pool expansion depending on the design.

### Resource Leaks

If a client forgets to return an object, the pool gradually becomes exhausted.

### Stale State

Objects must be reset before being reused.

### When NOT to Use

Avoid Object Pool when:

* object creation is inexpensive,
* objects are lightweight,
* reuse provides little measurable benefit,
* modern allocators already make allocation fast enough.

Always measure performance before introducing a pool.

---

# 14. Best Practices

* Pool only expensive-to-create resources.
* Reset objects before returning them to the pool.
* Use RAII to guarantee automatic release.
* Define clear ownership rules.
* Make the pool thread-safe if multiple threads access it.
* Monitor pool usage to determine the appropriate pool size.

---

# 15. Common Mistakes

### Mistake 1

Pooling small objects like simple integers or tiny value types.

The management overhead often exceeds the benefit.

---

### Mistake 2

Returning objects without resetting their state.

The next client receives stale data.

---

### Mistake 3

Forgetting to return borrowed objects.

This effectively leaks pool capacity.

---

### Mistake 4

Using one global pool for unrelated resource types.

Architects typically create focused pools for specific resource categories.

---

# 16. Pattern Variations

### 1. Fixed-Size Pool

A predefined number of objects.

Simple and predictable.

---

### 2. Dynamic Pool

Expands when demand increases.

May shrink later depending on policy.

---

### 3. Thread-Local Pool

Each thread owns its own pool.

Reduces locking overhead.

---

### 4. RAII Pool Handle

The borrowed object is wrapped in a handle that automatically returns it to the pool when the handle goes out of scope.

This is a common modern C++ approach.

---

# 17. Related Patterns

| Pattern        | Difference                                                            |
| -------------- | --------------------------------------------------------------------- |
| Singleton      | Ensures one shared instance.                                          |
| Factory Method | Creates new objects.                                                  |
| Builder        | Constructs complex objects.                                           |
| Prototype      | Copies existing objects.                                              |
| Object Pool    | Reuses existing objects.                                              |
| Flyweight      | Shares intrinsic state between many logical objects to reduce memory. |

A simple memory trick:

* **Factory** → Create.
* **Builder** → Assemble.
* **Prototype** → Copy.
* **Object Pool** → Reuse.

---

# 18. Industry Usage

Object Pool is common in performance-critical systems.

* **Microsoft:** Database connection pools, networking resources, and graphics buffers.
* **Google:** High-performance servers reuse network connections, buffers, and worker resources.
* **Adobe:** Image processing buffers and rendering resources.
* **Qt:** `QThreadPool` is a strong example of pooling worker threads.
* **ZEISS / Siemens / Philips:** Medical imaging pipelines, reconstruction buffers, GPU resources, and acquisition workspaces.
* **Autodesk:** Geometry buffers and rendering caches.
* **Game Engines:** Particle systems, bullets, AI entities, audio channels, and rendering resources.

The common architectural goal is **reuse of expensive resources to improve performance and predictability**.

---

# 19. Interview Questions

## Beginner

1. What problem does the Object Pool Pattern solve?
2. When should you use an Object Pool?
3. Why is repeatedly allocating and destroying expensive objects inefficient?

---

## Intermediate

1. How does an Object Pool differ from a cache?
2. What should happen if the pool is empty?
3. Why should pooled objects be reset before reuse?

---

## Advanced

1. How would you design a thread-safe object pool?
2. How would you prevent clients from forgetting to return borrowed objects?
3. How would you choose the optimal pool size for a high-throughput medical imaging application?

---

## Scenario-Based

Your CT reconstruction engine allocates a 512 MB temporary voxel buffer for every reconstruction.

Would you introduce an Object Pool? Explain your reasoning, including lifecycle management and thread safety.

---

## Architecture

Design an object pool architecture for a **Treatment Planning System** that manages:

* Dose Calculation Buffers
* Image Processing Buffers
* GPU Textures
* Matrix Workspaces

Explain:

* which resources should be pooled,
* when they are acquired,
* when they are released,
* how stale state is cleared,
* and how concurrent calculations are handled safely.

---

# 20. Practice Exercises

### Beginner Exercise

Design an `ImageBufferPool` that manages reusable image buffers. Explain the lifecycle of a borrowed buffer from acquisition to release.

---

### Intermediate Exercise

Design a Qt-based thumbnail generation system that uses a pool of image buffers and worker threads. Explain why pooling improves performance compared to allocating new resources for every thumbnail.

---

### Advanced Exercise

Design a **thread-safe resource pool** for a **Treatment Planning System (TPS)** supporting concurrent dose calculations.

The pool should manage:

* Dose Grids
* Voxel Buffers
* GPU Textures
* Temporary Interpolation Buffers

For each resource, explain:

* Why it should (or should not) be pooled.
* How clients acquire and release it.
* How you prevent stale data.
* How you handle pool exhaustion.
* How RAII can guarantee automatic resource return.

**Do not implement the solution yet.** Focus on architecture, ownership, concurrency, and lifecycle management.

---

# Key Architectural Takeaway

The **Object Pool Pattern** is not about avoiding object creation at all costs.

It's about recognizing when **object creation and destruction are expensive enough that reuse provides measurable benefits**.

A junior developer thinks:

> "I'll create a new buffer whenever I need one."

A software architect thinks:

> **"This buffer is expensive to allocate, used repeatedly, and has a short lifetime. I'll reuse it through a controlled pool to improve performance, reduce fragmentation, and keep resource usage predictable."**

That mindset is common in high-performance domains such as game engines, graphics software, databases, and medical imaging systems.

When you're ready, type **`NEXT`** to begin the **Structural Design Patterns** section with **Lesson 7: Adapter Pattern**.

















Got it, Brijesh! ✅
Let’s explain **Topic 8: Object Pool Pattern** in **simple English** with a **C++ example**.

---

# 🔄 Topic 8: Object Pool Pattern (Creational / Structural Design Pattern)

## 🧠 Overview

The **Object Pool Pattern** is a **creational/structural pattern** that **manages the reuse of objects** that are expensive to create.

In simple words:
➡️ “Instead of creating and destroying objects repeatedly, keep a pool of ready-to-use objects and reuse them.”

---

## 🎯 Real-Life Example

Think of a **database connection** in a server application:

* Opening a connection is **slow**.
* Instead of creating a new connection each time, we **reuse existing connections** from a pool.
* Once done, we **return it to the pool** for later use.

---

## ⚙️ Key Idea

1. Maintain a **pool of reusable objects**.
2. When an object is needed → **borrow from the pool**.
3. When done → **return it to the pool**.
4. Avoid **expensive creation/destruction** for each use.

---

## 🧩 C++ Example (Simple and Clear)

```cpp
#include <iostream>
#include <vector>
using namespace std;

// Step 1: The resource class (expensive object)
class Connection {
private:
    int id;
public:
    Connection(int i) : id(i) { cout << "Creating connection " << id << endl; }
    void connect() { cout << "Using connection " << id << endl; }
    int getID() { return id; }
};

// Step 2: Object Pool
class ConnectionPool {
private:
    vector<Connection*> available;
    vector<Connection*> inUse;
    int size;

public:
    ConnectionPool(int s) : size(s) {
        for (int i = 1; i <= size; i++)
            available.push_back(new Connection(i));
    }

    // Borrow connection
    Connection* acquire() {
        if (available.empty()) {
            cout << "No available connections!" << endl;
            return nullptr;
        }
        Connection* conn = available.back();
        available.pop_back();
        inUse.push_back(conn);
        return conn;
    }

    // Return connection
    void release(Connection* conn) {
        for (auto it = inUse.begin(); it != inUse.end(); ++it) {
            if (*it == conn) {
                inUse.erase(it);
                available.push_back(conn);
                return;
            }
        }
    }

    ~ConnectionPool() {
        for (auto conn : available) delete conn;
        for (auto conn : inUse) delete conn;
    }
};

// Step 3: Client code
int main() {
    ConnectionPool pool(3);

    Connection* c1 = pool.acquire();
    Connection* c2 = pool.acquire();

    if (c1) c1->connect();
    if (c2) c2->connect();

    pool.release(c1);

    Connection* c3 = pool.acquire();
    if (c3) c3->connect();

    return 0;
}
```

---

## 🧾 Output

```
Creating connection 1
Creating connection 2
Creating connection 3
Using connection 2
Using connection 1
Using connection 1
```

---

## 🔍 Explanation (In Simple Words)

* `ConnectionPool` **manages a fixed number of connections**.
* `acquire()` → borrow a connection from the pool.
* `release()` → return it to the pool for reuse.
* Saves **time and resources**, especially for **expensive objects** like DB connections or threads.

---

## 🧠 When to Use Object Pool Pattern

* When object creation is **expensive** (time or resources).
* When you need **frequent reuse** of objects.
* When system performance depends on **reducing object creation overhead**.

---

## 🏁 Summary

| **Concept**  | **Description**                                                    |
| ------------ | ------------------------------------------------------------------ |
| Purpose      | Reuse expensive objects instead of creating/destroying repeatedly. |
| Key Feature  | Pool manages available and in-use objects.                         |
| Benefit      | Saves resources, improves performance.                             |
| Example      | Database connections, thread pools, network sockets.               |
| Pattern Type | Creational / Structural Design Pattern                             |

---

[⬅️ Prototype Pattern](/prototype.md)        |  [Adapter Pattern ➡️](/adapter.md) 
---
## **License**
This project is licensed under the MIT License.

---

Happy Coding!

