

# Lesson 17 — Interpreter Pattern 

---

# Before We Start

This is one of the **least used** Gang of Four (GoF) patterns in modern software.

Many developers study it only for interviews.

However, that doesn't mean it's unimportant.

If you're building:

* Formula Engines
* Rule Engines
* SQL Parsers
* Expression Evaluators
* Medical Protocol Languages
* CAD Scripting
* Mathematical Expression Systems

then the **Interpreter Pattern** becomes extremely useful.

---

# Architect's Reality Check

Before teaching the pattern, here's something experienced architects know:

> **Most production compilers, SQL databases, and programming language interpreters do NOT use the classic GoF Interpreter Pattern alone.**

Instead, they use:

* Parsers
* Abstract Syntax Trees (AST)
* Visitors
* Compilers
* Bytecode Interpreters

The GoF Interpreter Pattern is best suited for:

* Small languages
* Configuration rules
* Business rule engines
* Mathematical expressions
* DSLs (Domain Specific Languages)

Understanding this distinction is important in interviews.

---

# The Architect's Way of Thinking

Suppose your **Treatment Planning System (TPS)** allows physicists to define dose constraints like:

```text
PTV.D95 >= 95%
```

or

```text
Heart.MeanDose < 20 Gy
```

or

```text
Lung.V20 < 30%
```

Question:

Should you hardcode every possible rule?

```cpp
if(rule == "PTV")
...

if(rule == "Heart")
...

if(rule == "Lung")
...
```

No.

Tomorrow users invent new rules.

Instead, architects think:

> **"What if the rules themselves become objects that know how to evaluate themselves?"**

That is the Interpreter Pattern.

---

# 1. Introduction

## What is the Interpreter Pattern?

The **Interpreter Pattern** defines a representation for a language and provides an interpreter to evaluate sentences written in that language.

Simple definition:

> **Represent grammar rules as classes and evaluate expressions using object collaboration.**

---

## Why was it created?

Many applications must understand user-defined expressions.

Examples:

```text
3 + 5

Age > 18

Salary > 50000

Dose < 20

Patient.Age >= 60
```

Instead of writing many `if-else` statements, model the language itself.

---

## Category

**Behavioral Pattern**

---

## What problem does it solve?

Without Interpreter:

```cpp
if(expression == "A+B")
...

else if(expression == "A-B")
...

else if(expression == "A*B")
...

else if(...)
```

The evaluator grows endlessly.

With Interpreter:

```text
Expression

↓

AddExpression

MultiplyExpression

SubtractExpression
```

Each expression knows how to evaluate itself.

---

# 2. Problem Statement

Imagine a **Hospital Rule Engine**.

Rules:

```text
Age > 60

AND

Diabetes == TRUE

AND

BloodPressure > 140
```

Without Interpreter:

Hundreds of nested `if` statements.

Hard to extend.

Hard to test.

Hard to maintain.

Tomorrow management adds:

```text
OR

NOT

BMI > 30

HeartRate > 100
```

Everything changes.

---

# 3. Motivation

Architects observed that many systems need to interpret user-defined languages.

Examples:

* SQL
* Mathematical formulas
* Search queries
* Spreadsheet formulas
* Medical protocols
* Business rules

Instead of hardcoding every rule,

represent each grammar rule as an object.

---

# 4. Real-World Analogy

## Calculator

You type:

```text
5 + 8 × 2
```

The calculator understands:

* Numbers
* Operators
* Precedence

The calculator interprets the expression.

---

### Mapping

| Calculator | Software                |
| ---------- | ----------------------- |
| Number     | Terminal Expression     |
| +          | Non-Terminal Expression |
| Formula    | Expression Tree         |
| Calculator | Interpreter             |

---

# 5. Software Scenario

Interpreter appears when software must understand a small language.

### Desktop Applications

* Search filters
* Calculator
* Formula editors

---

### Qt Applications

Suppose your ERP supports custom search filters:

```text
Department = "CS"

AND

Year = 3
```

Instead of many filters in code,

build an expression tree.

---

### CAD Software

Constraint language:

```text
Length > 20

AND

Angle < 45
```

---

### Medical Imaging

ROI filtering:

```text
Volume > 100

AND

Dose < 20
```

---

### Databases

Simple query interpreters.

---

# 6. UML Class Diagram

```text
                   +----------------------+
                   |      Expression      |
                   +----------------------+
                   | +interpret()         |
                   +----------+-----------+
                              ^
                --------------|--------------
                |                             |
      +------------------+         +------------------+
      | TerminalExpr     |         | NonTerminalExpr  |
      +------------------+         +------------------+
                                           |
                                 contains expressions
```

---

## Responsibilities

### Expression

Defines:

```cpp
interpret(context)
```

---

### Terminal Expression

Represents:

```text
Number

Variable

Constant
```

---

### Non-Terminal Expression

Represents operations.

Example:

```text
Addition

Subtraction

AND

OR
```

---

### Context

Stores variables.

Example:

```text
A = 10

B = 20
```

---

# 7. Participants

## Abstract Expression

Example:

```cpp
Expression
```

---

## Terminal Expression

Examples:

```text
Number

Variable

PatientAge
```

---

## Non-Terminal Expression

Examples:

```text
AddExpression

MultiplyExpression

AndExpression
```

---

## Context

Stores runtime values.

---

## Client

Builds the expression tree.

---

# 8. Collaboration

Runtime Flow

Expression:

```text
(5 + 3) × 2
```

Tree:

```text
         *
       /   \
      +     2
     / \
    5   3
```

Evaluation:

```text
Multiply

↓

Addition

↓

5

↓

3

↓

Result = 8

↓

Multiply

↓

2

↓

16
```

Each node interprets itself.

---

# 9. C++ Example

Let's interpret:

```text
5 + 10
```

```cpp
#include <iostream>
#include <memory>

// Abstract Expression
class Expression
{
public:
    virtual ~Expression() = default;
    virtual int interpret() const = 0;
};

// Terminal Expression
class Number : public Expression
{
public:
    explicit Number(int value)
        : m_value(value)
    {}

    int interpret() const override
    {
        return m_value;
    }

private:
    int m_value;
};

// Non-Terminal Expression
class AddExpression : public Expression
{
public:
    AddExpression(std::shared_ptr<Expression> left,
                  std::shared_ptr<Expression> right)
        : m_left(std::move(left)),
          m_right(std::move(right))
    {}

    int interpret() const override
    {
        return m_left->interpret() +
               m_right->interpret();
    }

private:
    std::shared_ptr<Expression> m_left;
    std::shared_ptr<Expression> m_right;
};

int main()
{
    auto five = std::make_shared<Number>(5);
    auto ten = std::make_shared<Number>(10);

    AddExpression expr(five, ten);

    std::cout << expr.interpret();
}
```

---

## Design Focus

Notice:

The expression itself becomes an object.

Instead of:

```cpp
5 + 10
```

we have:

```text
AddExpression

↓

Number

↓

Number
```

The expression tree performs the calculation.

---

# 10. Qt Example

Suppose your College ERP allows users to filter students.

User enters:

```text
Department == "CSE"

AND

CGPA > 8.5

AND

Attendance > 75
```

Instead of writing nested `if` statements,

parse the filter into an expression tree:

```text
          AND
         /   \
      AND    Attendance
     /   \
Department CGPA
```

Each node evaluates one part of the condition.

The UI remains independent of the evaluation logic.

---

# 11. Medical Software Example

Imagine a **Treatment Planning System**.

Physicists define optimization constraints:

```text
PTV.D95 > 95%

AND

Heart.MeanDose < 20

AND

SpinalCord.MaxDose < 45
```

Expression Tree:

```text
             AND
           /     \
        AND      SpinalCord
       /   \
    PTV    Heart
```

Each node:

* retrieves the relevant clinical value,
* evaluates its own condition,
* returns `true` or `false`.

The optimizer then evaluates the root expression.

Benefits:

* New operators (`OR`, `NOT`) can be added by creating new expression classes.
* Existing expressions remain unchanged.

---

# 12. Advantages

### Extensibility

Add new grammar rules by creating new expression classes.

### Open/Closed Principle

New operators don't require modifying existing ones.

### Readability

The expression tree mirrors the grammar.

### Reusability

Expression nodes can be reused in many trees.

---

# 13. Disadvantages

### Many Classes

Every grammar rule often becomes a class.

### Performance

Large expression trees can be slower than compiled code.

### Complexity

Not suitable for large programming languages.

### When NOT to Use

Avoid the GoF Interpreter Pattern when:

* the grammar is very large,
* performance is critical,
* a mature parser or expression library already exists.

For large languages, architects usually choose parser generators or AST-based compilers.

---

# 14. Best Practices

* Keep expression classes immutable.
* Separate parsing from evaluation.
* Keep context independent of expressions.
* Use Composite to represent the expression tree.
* Use Visitor if many different operations must traverse the tree.

---

# 15. Common Mistakes

### Mistake 1

Using Interpreter for a full programming language.

The GoF pattern is intended for relatively small grammars.

---

### Mistake 2

Mixing parsing and interpretation.

A common architecture is:

```text
Text

↓

Parser

↓

Expression Tree

↓

Interpreter
```

---

### Mistake 3

Putting application logic inside expression nodes.

Expression nodes should focus on evaluating grammar constructs.

---

### Mistake 4

Ignoring operator precedence.

Expressions such as:

```text
2 + 3 × 4
```

must be parsed correctly before interpretation.

---

# 16. Pattern Variations

## 1. Mathematical Interpreter

Supports:

* *
* *
* *
* /

---

## 2. Boolean Interpreter

Supports:

* AND
* OR
* NOT

---

## 3. Rule Engine

Supports:

```text
Age > 18

AND

Country == India
```

---

## 4. DSL Interpreter

Supports domain-specific languages.

Examples:

* Medical protocols
* Workflow rules
* CAD constraints

---

# 17. Related Patterns

| Pattern     | Difference                                               |
| ----------- | -------------------------------------------------------- |
| Interpreter | Represents grammar as objects.                           |
| Composite   | Represents the expression tree structure.                |
| Visitor     | Performs additional operations on the tree.              |
| Command     | Encapsulates actions, not grammar.                       |
| Strategy    | Selects algorithms rather than interpreting expressions. |

---

## Interpreter + Composite

This is one of the most common combinations.

The expression tree is naturally a **Composite**:

```text
Expression
     |
   Add
  /   \
 5     Multiply
      /       \
     2         3
```

Composite provides the tree structure.

Interpreter provides the evaluation behavior.

---

# 18. Industry Usage

The classic GoF Interpreter Pattern is less common than other patterns, but its underlying ideas appear in many systems.

* **Microsoft:** Spreadsheet formulas, rule engines, and expression evaluators.
* **Google:** Search query parsing and filtering systems.
* **Adobe:** Filter expressions and automation rules.
* **Qt:** Qt itself doesn't provide a general-purpose Interpreter framework, but applications built with Qt often implement custom DSLs and expression evaluators.
* **ZEISS / Siemens / Philips:** Medical rule engines, protocol definitions, and optimization constraints.
* **Autodesk:** CAD constraint expressions and scripting.
* **Databases:** SQL parsers typically build ASTs and use more advanced techniques than the classic GoF Interpreter.

The architectural goal is **representing a language as collaborating objects**.

---

# 19. Interview Questions

## Beginner

1. What problem does the Interpreter Pattern solve?
2. What is a Terminal Expression?
3. What is a Non-Terminal Expression?

---

## Intermediate

1. Why is Composite commonly used with Interpreter?
2. What is the role of the Context object?
3. When should you avoid the Interpreter Pattern?

---

## Advanced

1. How would you design a DSL for treatment planning constraints?
2. How would you support operator precedence?
3. How would you optimize evaluation of very large expression trees?

---

## Scenario-Based

Your TPS allows users to define constraints such as:

```text
PTV.D95 > 95%

AND

Heart.MeanDose < 20 Gy
```

Design an Interpreter-based architecture to evaluate these expressions.

---

## Architecture

Design an Interpreter for a **College ERP** search language.

Supported operators:

* `AND`
* `OR`
* `NOT`
* `=`
* `>`
* `<`

Supported fields:

* Department
* Year
* CGPA
* Attendance

Explain:

* the expression hierarchy,
* parsing,
* evaluation,
* and context management.

---

# 20. Practice Exercises

### Beginner Exercise

Design an arithmetic interpreter supporting:

* Numbers
* Addition
* Multiplication

Draw the expression tree for:

```text
(3 + 4) × 5
```

---

### Intermediate Exercise

Design a Qt search filter language supporting:

```text
Department = "ECE"

AND

CGPA > 8.0
```

Represent the filter as an expression tree.

---

### Advanced Exercise

Design an **Interpreter architecture** for a **Treatment Planning System** that supports constraint expressions such as:

```text
PTV.D95 >= 95%

AND

Heart.MeanDose < 20 Gy

OR

SpinalCord.MaxDose < 45 Gy
```

Requirements:

* Support `AND`, `OR`, and `NOT`.
* Allow future operators without modifying existing expression classes.
* Separate parsing from evaluation.
* Explain how Composite and Visitor could be integrated into the design.

**Do not implement the solution yet.** Focus on architecture, grammar representation, and extensibility.

---

# Key Architectural Takeaway

The **Interpreter Pattern** is about **representing the rules of a small language as objects**.

A junior developer thinks:

> "I'll hardcode every possible expression with `if-else` statements."

A software architect thinks:

> **"The language will evolve. I'll model each grammar rule as an object so new operators and expressions can be added without rewriting the evaluator."**

For large languages, modern architectures typically extend beyond the classic GoF Interpreter Pattern with parsers, ASTs, Visitors, and compiler techniques—but the underlying object-oriented principles remain valuable.

---

## ⭐ Architect's Insight

In real-world systems, you'll often see this combination:

```text
Text Expression
        ↓
      Parser
        ↓
   Expression Tree (Composite)
        ↓
 Interpreter / Evaluator
        ↓
      Result
```

This layered architecture is used in many rule engines, spreadsheet formula evaluators, search filters, and domain-specific languages.


---
[⬅️ Command Pattern ](/CommandPattern.md)        |  [Iterator Pattern  ➡️](/IteratorPattern.md) 
---
## **License**
This project is licensed under the MIT License.

---

Happy Coding!
