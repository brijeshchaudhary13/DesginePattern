

# Lesson 13 — Proxy Pattern 

---

# Before We Start

The **Proxy Pattern** is everywhere.

It is used in:

* Qt
* Operating Systems
* Browsers
* Databases
* Cloud Computing
* Medical Software
* Game Engines
* Network Applications

Yet many developers misunderstand it.

They think:

> "Proxy is just another wrapper."

No.

A software architect asks a different question:

> **"Should the client have direct access to the real object?"**

If the answer is **No**, then a **Proxy** may be the solution.

---

# The Architect's Way of Thinking

Imagine a **Hospital PACS (Picture Archiving and Communication System)**.

The hospital stores:

```text
10 Million DICOM Images
```

A doctor opens the patient list.

Should the system immediately load:

```text
10 Million Images
```

No.

It would:

* Consume enormous memory
* Waste CPU
* Freeze the application
* Slow the network

Instead:

```text
Patient List

↓

Thumbnail Placeholder

↓

Doctor Clicks Image

↓

Load Real DICOM Image
```

The doctor thinks the image was always there.

Actually, a **Proxy** delayed loading until it was needed.

This is called **Virtual Proxy**.

---

# What is the Difference?

Many developers confuse these patterns.

| Pattern   | Purpose                                  |
| --------- | ---------------------------------------- |
| Adapter   | Change interface                         |
| Decorator | Add behavior                             |
| Facade    | Simplify subsystem                       |
| Bridge    | Separate abstraction from implementation |
| Proxy     | Control access                           |

Remember this table.

It appears in interviews frequently.

---

# 1. Introduction

## What is the Proxy Pattern?

The **Proxy Pattern** provides a **placeholder (surrogate)** for another object.

The proxy controls access to the real object.

The client communicates with the proxy.

The proxy decides:

* Should the request proceed?
* Should it be delayed?
* Should it be cached?
* Should permissions be checked?
* Should it go over the network?

---

## Simple Definition

> **A Proxy controls access to another object while presenting the same interface.**

---

## Category

**Structural Pattern**

---

## Why was it created?

Some objects are expensive.

Examples:

* MRI Volume
* Large CT Dataset
* Database Connection
* Remote Server
* Cloud Storage
* AI Model
* GPU Resources

Sometimes the client shouldn't interact with them directly.

The Proxy stands in front.

---

# 2. Problem Statement

Imagine a **Medical Viewer**.

Each CT study is:

```text
4 GB
```

The hospital has:

```text
2000 Patients
```

Without Proxy:

```text
Application Starts

↓

Load Every CT

↓

8 TB Loaded

↓

Crash
```

Clearly impossible.

Instead:

```text
Patient Selected

↓

Proxy

↓

Load CT

↓

Display
```

Only the requested study is loaded.

---

# 3. Motivation

Architects observed that not every request should immediately reach the real object.

Reasons include:

* Delayed loading
* Security
* Caching
* Logging
* Network communication
* Synchronization

The client should not care about these concerns.

The Proxy handles them transparently.

---

# 4. Real-World Analogy

## Bank Locker

Imagine a bank locker.

You want access to your valuables.

Do you walk directly into the vault?

No.

You first meet:

```text
Bank Officer
```

The officer checks:

* Identity
* Signature
* Authorization
* Locker Availability

Only then do you access the locker.

### Mapping

| Bank         | Software           |
| ------------ | ------------------ |
| Locker       | Real Object        |
| Bank Officer | Proxy              |
| Customer     | Client             |
| Vault        | Expensive Resource |

The officer controls access.

---

# 5. Software Scenario

Proxy appears whenever access needs to be controlled.

### Desktop Applications

* Lazy image loading
* File access
* Print spoolers

---

### Qt Applications

Suppose a Qt application displays:

```text
100,000 Images
```

Instead of loading every image:

```text
ImageProxy

↓

Real Image
```

The real image loads only when visible.

Qt's Model/View framework often follows similar lazy-loading principles, although it is not a textbook Proxy implementation.

---

### CAD Software

Massive CAD assemblies.

Load only visible parts.

---

### Medical Imaging

Load:

* CT
* MRI
* PET

Only when the radiologist opens them.

---

### Cloud Applications

Cloud document proxy.

Actual file downloaded on demand.

---

### Browsers

Images load when scrolling.

This is another form of lazy access.

---

# 6. UML Class Diagram

```text
                 +----------------------+
                 |      Subject         |
                 +----------------------+
                 | +request()           |
                 +----------+-----------+
                            ^
                ------------|------------
                |                         |
       +----------------+      +----------------+
       | RealSubject    |      | Proxy          |
       +----------------+      +----------------+
       | request()      |      | request()      |
       +----------------+      | - realSubject  |
                               +----------------+
```

---

## Responsibilities

### Subject

Common interface.

---

### Real Subject

Actual object.

Performs the work.

---

### Proxy

Controls access.

Creates the real object if needed.

---

### Client

Uses only:

```cpp
Subject*
```

It doesn't know whether it is talking to:

* Proxy
* Real Object

---

# 7. Participants

## Subject

Example:

```text
Image
```

---

## Real Subject

```text
DicomImage
```

---

## Proxy

```text
DicomImageProxy
```

Contains:

```cpp
std::unique_ptr<DicomImage>
```

---

## Client

Uses:

```cpp
image->display();
```

No knowledge of the proxy.

---

# 8. Collaboration

Runtime Flow

```text
Client

↓

display()

↓

Proxy

↓

Real Object Exists?

↓

No

↓

Load Image

↓

Display

↓

Next Request

↓

Already Loaded

↓

Display Immediately
```

Notice:

Only the first request performs the expensive work.

---

# 9. C++ Example

```cpp
#include <iostream>
#include <memory>
#include <string>

// Subject
class Image
{
public:
    virtual ~Image() = default;
    virtual void display() = 0;
};

// Real Subject
class RealImage : public Image
{
public:
    explicit RealImage(const std::string& file)
        : m_file(file)
    {
        loadFromDisk();
    }

    void display() override
    {
        std::cout << "Displaying " << m_file << '\n';
    }

private:
    void loadFromDisk()
    {
        std::cout << "Loading image from disk...\n";
    }

    std::string m_file;
};

// Proxy
class ImageProxy : public Image
{
public:
    explicit ImageProxy(std::string file)
        : m_file(std::move(file))
    {}

    void display() override
    {
        if (!m_realImage)
        {
            m_realImage = std::make_unique<RealImage>(m_file);
        }

        m_realImage->display();
    }

private:
    std::string m_file;
    std::unique_ptr<RealImage> m_realImage;
};

int main()
{
    ImageProxy image("CT001.dcm");

    image.display();

    image.display();
}
```

---

## Design Focus

Observe the output.

First call:

```text
Loading image...

Displaying image...
```

Second call:

```text
Displaying image...
```

Loading happens only once.

This is **Lazy Loading**, the most common Proxy implementation.

---

# 10. Qt Example

Suppose you build a DICOM browser.

Left panel:

```text
Patient A

Patient B

Patient C

...
```

Instead of:

```text
Load Every Thumbnail

↓

Load Every CT

↓

Load Every MRI
```

Use:

```text
ThumbnailProxy

↓

Real Thumbnail
```

When the thumbnail becomes visible:

```text
Proxy

↓

Load QImage

↓

Display
```

Scrolling remains smooth because invisible images are never loaded.

---

# 11. Medical Software Example

Let's design a **PACS Viewer**.

Hospital Database:

```text
Patient

↓

Study

↓

Series

↓

Image
```

Each image:

```text
512 MB
```

Without Proxy:

Opening one patient loads everything.

Instead:

```text
StudyProxy

↓

Real Study
```

Workflow:

```text
Doctor Opens Study

↓

StudyProxy

↓

Load Images

↓

Cache

↓

Display
```

Similarly, in a **Treatment Planning System (TPS)**:

Large dose matrices may be loaded only when the user opens the Dose View, rather than during initial patient loading.

---

# 12. Advantages

### Performance

Avoids unnecessary loading.

### Lazy Initialization

Objects created only when required.

### Security

Proxy can check permissions before forwarding requests.

### Caching

Previously loaded objects can be reused.

### Logging

Proxy can record every access.

### Network Transparency

Remote proxies can hide network communication from clients.

---

# 13. Disadvantages

### Extra Layer

Adds one more object.

### Increased Complexity

More classes and delegation.

### Potential Latency

The first request may be slower because the object is created or fetched.

### When NOT to Use

Avoid Proxy when:

* direct access is acceptable,
* the object is inexpensive,
* access control is unnecessary.

---

# 14. Best Practices

* Keep the proxy interface identical to the real subject.
* Keep the proxy focused on access control, not business logic.
* Cache expensive objects when appropriate.
* Use smart pointers to manage object lifetime safely.
* Clearly document whether the proxy owns the real object.

---

# 15. Common Mistakes

### Mistake 1

Putting business logic inside the proxy.

The proxy should manage access, not perform the domain work.

---

### Mistake 2

Breaking interface compatibility.

The client should not need different code for the proxy and the real object.

---

### Mistake 3

Never releasing cached resources.

A proxy that holds everything forever can become a memory leak.

---

### Mistake 4

Confusing Proxy with Decorator.

Proxy controls access.

Decorator adds responsibilities.

---

# 16. Pattern Variations

## 1. Virtual Proxy

Creates the object lazily.

Most common.

Example:

```text
Large CT Volume

↓

Load On Demand
```

---

## 2. Protection Proxy

Checks permissions.

Example:

```text
Doctor

↓

Can View?

↓

Yes

↓

Open Patient
```

---

## 3. Remote Proxy

Represents an object on another machine.

Example:

```text
Client

↓

Proxy

↓

Hospital Server
```

The client behaves as if the object were local.

---

## 4. Cache Proxy

Stores results.

Future requests reuse cached data.

---

## 5. Smart Reference Proxy

Performs extra management such as:

* reference counting,
* locking,
* logging,
* synchronization.

---

# 17. Related Patterns

| Pattern   | Difference                         |
| --------- | ---------------------------------- |
| Proxy     | Controls access to an object.      |
| Decorator | Adds responsibilities dynamically. |
| Facade    | Simplifies a subsystem.            |
| Adapter   | Changes an interface.              |
| Flyweight | Shares common state.               |

---

## Proxy vs Decorator (Interview Favorite)

Both wrap another object.

### Proxy

Purpose:

```text
Access Control
```

Examples:

* Lazy loading
* Authentication
* Remote access

---

### Decorator

Purpose:

```text
Behavior Extension
```

Examples:

* Logging
* Compression
* Encryption
* Watermark

The key question is:

> **Are you controlling access or adding functionality?**

---

# 18. Industry Usage

Proxy is used extensively wherever expensive resources or controlled access are involved.

* **Microsoft:** COM proxies, distributed objects, virtual file systems, and cloud storage clients.
* **Google:** Remote service proxies, API clients, and lazy resource loading.
* **Adobe:** Large document loading, linked assets, and cloud synchronization.
* **Qt:** Model/view implementations often defer loading data until needed, reflecting proxy principles.
* **ZEISS / Siemens / Philips:** PACS viewers, DICOM studies, remote imaging devices, and lazy loading of large datasets.
* **Autodesk:** Large assembly loading and remote model access.
* **Game Engines:** Texture streaming, asset streaming, and remote multiplayer objects.

The architectural goal is **transparent access control while preserving the same client interface**.

---

# 19. Interview Questions

## Beginner

1. What problem does the Proxy Pattern solve?
2. What is the role of the Proxy?
3. Why does the Proxy implement the same interface as the Real Subject?

---

## Intermediate

1. Explain the difference between Virtual Proxy and Protection Proxy.
2. How does Proxy improve performance?
3. When should lazy loading be used?

---

## Advanced

1. Design a Remote Proxy for a PACS server.
2. How would you implement thread-safe lazy initialization in a Virtual Proxy?
3. How would you add caching and cache invalidation to a Proxy?

---

## Scenario-Based

Your DICOM Viewer displays a list of 50,000 studies. Each study contains gigabytes of image data.

Would you load every study during startup or introduce a Virtual Proxy? Explain the architecture and expected performance benefits.

---

## Architecture

Design a **Proxy architecture** for a **Treatment Planning System** that manages:

* CT Volumes
* Dose Matrices
* Structure Sets
* AI Segmentation Models

Requirements:

* Lazy loading
* Permission checks
* Result caching
* Logging
* Thread-safe initialization

Explain:

* the `Subject` interface,
* the `RealSubject`,
* the `Proxy`,
* ownership,
* caching strategy,
* and concurrency considerations.

---

# 20. Practice Exercises

### Beginner Exercise

Design an `ImageProxy` that delays loading a large image until `display()` is called.

Draw the UML and explain the object interactions.

---

### Intermediate Exercise

Design a Qt application where thumbnails are represented by proxies. Explain how scrolling remains responsive while images load on demand.

---

### Advanced Exercise

Design a **Proxy architecture** for a **Medical Imaging Platform** that provides controlled access to:

* DICOM Studies
* CT Volumes
* MRI Volumes
* AI Models

Requirements:

* Lazy loading
* Remote server support
* Permission validation
* Caching
* Automatic resource cleanup

Explain:

* ownership,
* lifecycle,
* thread safety,
* cache invalidation,
* and how the client remains unaware of whether it is talking to a proxy or the real object.

**Do not implement the solution yet.** Focus on architecture and responsibilities.

---

# Key Architectural Takeaway

The **Proxy Pattern** is not about changing functionality.

It is about **controlling access** to another object while preserving the same interface.

A junior developer thinks:

> "I'll let the client use the real object directly."

A software architect thinks:

> **"This object is expensive, remote, protected, or shared. I'll place a proxy in front of it so I can manage access, optimize performance, enforce security, and keep the client code unchanged."**

This way of thinking is fundamental in enterprise systems, cloud applications, PACS viewers, CAD software, and treatment planning systems where efficient and controlled resource access is critical.


---
[⬅️ Flyweight Pattern ](/FlyweightPattern.md)        |  [Private Class Data Pattern ➡️](/PrivateClassDataPattern.md) 
---
## **License**
This project is licensed under the MIT License.

---

Happy Coding!

