# UML — Low-Level Design

## 📌 Overview

**UML (Unified Modeling Language)** is a standardized way to visually represent the **structure and behaviour of a software system**.

In LLD, UML helps us understand:

* What classes exist
* What attributes and methods they have
* How classes are related
* How objects interact
* How the system behaves over time

---

# 📚 UML Categories

UML diagrams are broadly divided into two categories:

```text
                    UML
                     │
            ┌────────┴────────┐
            │                 │
       Structural        Behavioural
            │                 │
      Class Diagram     Sequence Diagram
```

There are **14 standard UML diagrams**:

* 7 Structural
* 7 Behavioural

For LLD, we mainly focus on:

* **Class Diagram** → Structural
* **Sequence Diagram** → Behavioural

---

# 🧱 Class Diagram

A **Class Diagram** represents the **static structure** of an application.

It shows:

* Classes
* Attributes
* Methods
* Access modifiers
* Relationships between classes

## Basic Class Structure

A class is generally represented as a rectangle divided into three sections:

```text
┌─────────────────────────┐
│       Class Name        │
├─────────────────────────┤
│       Attributes        │
├─────────────────────────┤
│         Methods         │
└─────────────────────────┘
```

### Example

```text
             <<abstract>>
┌────────────────────────────┐
│            Car             │
├────────────────────────────┤
│ + brand : string           │
│ - model : string           │
│ # engineCC : int           │
├────────────────────────────┤
│ + startEngine() : void     │
│ + stopEngine() : void      │
│ + accelerate() : void      │
│ + brake() : void           │
└────────────────────────────┘
```

### Three Sections

```text
1. Class Name
2. Attributes / Variables
3. Methods / Behaviour
```

### Attribute Format

```text
name : datatype
```

Example:

```text
brand : string
engineCC : int
```

### Method Format

```text
methodName(parameters) : returnType
```

Example:

```text
accelerate() : void
```

---

# 🔐 UML Access Modifiers

| Symbol | Access Modifier   |
| ------ | ----------------- |
| `+`    | Public            |
| `-`    | Private           |
| `#`    | Protected         |
| `~`    | Package / Default |

Example:

```text
+ brand : string
- model : string
# engineCC : int
```

Therefore:

```text
brand    → public
model    → private
engineCC → protected
```

---

# 🧩 Abstract vs Concrete Classes

## Abstract Class

An **abstract class cannot be instantiated directly**.

In UML, an abstract class can be represented using:

```text
<<abstract>>
```

Example:

```text
        <<abstract>>
       ┌─────────────┐
       │    Car      │
       ├─────────────┤
       │ accelerate()│
       └─────────────┘
```

In C++, an abstract class commonly contains at least one **pure virtual function**:

```cpp
class Car {
public:
    virtual void accelerate() = 0;
};
```

Therefore:

```cpp
Car car;              // ❌ Cannot create object
Car* car = new Car(); // ❌ Cannot create object
```

But a concrete derived class can be instantiated:

```cpp
class SportsCar : public Car {
public:
    void accelerate() override {
        // implementation
    }
};

SportsCar car; // ✅
```

---

# 🔗 Class Relationships

Important relationships in LLD include:

```text
Inheritance
Association
Aggregation
Composition
```

---

# 1. Inheritance — IS-A Relationship

Inheritance represents an **IS-A relationship**.

Example:

```text
        Car
         △
         │
    SportsCar
```

Meaning:

> SportsCar IS-A Car.

C++:

```cpp
class SportsCar : public Car {
};
```

### UML Representation

Inheritance is represented using:

**Solid line + hollow triangle arrowhead**

The hollow triangle points towards the **parent/base class**.

```text
SportsCar ─────────▷ Car
```

---

# 2. Association

Association represents a **general relationship or interaction** between two classes.

It does not necessarily imply ownership.

Example:

```text
┌─────────┐              ┌─────────┐
│  Arjun  │─────────────>│  House  │
└─────────┘    lives in  └─────────┘
```

Meaning:

> Arjun is associated with / interacts with a House.

Association is represented using a **line**.

---

# 3. Aggregation — Weak HAS-A

Aggregation represents a **weak HAS-A relationship**.

The whole contains other objects, but those objects can exist **independently**.

It is represented using a **hollow diamond `◇`**.

### Example

```text
Room ◇──────── Chair
```

Here:

```text
Room   → Whole / Container
Chair  → Part
◇      → Aggregation
```

Another example:

```text
             ◇
             │
            Room
          /   |   \
       Chair Sofa  Bed
```

### Important Property

If the `Room` is destroyed, the `Chair`, `Sofa`, and `Bed` can still exist independently.

Therefore:

> **Aggregation = Weak ownership**

---

# 4. Composition — Strong HAS-A

Composition represents a **strong HAS-A relationship**.

The lifetime of the contained object is strongly tied to the lifetime of the owner.

It is represented using a **filled diamond `◆`**.

### Example

```text
Car ◆──────── Engine
```

Here:

```text
Car     → Whole / Owner
Engine  → Part
◆       → Composition
```

### Important Rule

> **The diamond is always placed on the whole/owner side.**

Therefore:

```text
Car ◆──────── Engine
```

NOT:

```text
Car ────────◆ Engine  ❌
```

### C++ Example

```cpp
class A {
public:
    void method1() {
        // implementation
    }
};

class B {
private:
    A* a;

public:
    B() {
        a = new A();
    }

    void method2() {
        a->method1();
    }
};
```

Here:

```text
B ◆──────── A
```

`B` creates and owns `A`.

Therefore, `B` has a strong ownership relationship with `A`.

---

# ⚖️ Association vs Aggregation vs Composition

| Relationship | Meaning              | Ownership            | Symbol |
| ------------ | -------------------- | -------------------- | ------ |
| Association  | General relationship | No ownership implied | `────` |
| Aggregation  | Weak HAS-A           | Weak ownership       | `◇`    |
| Composition  | Strong HAS-A         | Strong ownership     | `◆`    |
| Inheritance  | IS-A                 | Parent/Child         | `▷`    |

### Easy Memory Trick

```text
IS-A        → Inheritance

HAS-A       → Aggregation / Composition

USES /      → Association
INTERACTS
```

### Diamond Rule

```text
Aggregation:
Room ◇──────── Chair

Composition:
Car ◆──────── Engine
```

> **Diamond always goes on the whole/owner side.**

---

# 🔄 Sequence Diagram

A **Sequence Diagram** represents the **interaction between objects over time**.

It answers:

> **Who calls whom, and in what order?**

Example:

```text
User → ATM → Transaction → Account
```

---

# 🧱 Components of a Sequence Diagram

## 1. Participants / Objects

First identify the objects involved.

Example:

```text
User
ATM
Transaction
Account
CashDispenser
```

---

## 2. Lifeline

A **lifeline** represents the existence of an object over time.

It is represented using a **vertical dashed line**.

```text
User              ATM
 │                 │
 ⋮                 ⋮
 │                 │
 ⋮                 ⋮
```

Time generally moves **from top to bottom**.

---

## 3. Activation Bar

An **activation bar** represents the period during which an object is actively executing an operation.

```text
 │
 █
 █
 █
 │
```

---

# 📨 Messages

Messages represent communication between objects.

## Synchronous Message

The sender waits for the receiver to complete the operation.

```text
A ─────────▶ B
```

Typically represented using a **solid line with a filled arrowhead**.

---

## Asynchronous Message

The sender does not wait for the receiver to complete the operation.

```text
A ─────────> B
```

Typically represented using a **solid line with an open arrowhead**.

---

## Return Message

Represents the response returned to the caller.

```text
A ◀ - - - - - B
```

Usually represented using a **dashed line with an open arrowhead**.

---

# 🆕 Create Message

A **Create Message** represents the creation of a new object.

```text
A ── create ──▶ B
```

The lifeline of `B` begins at the point where it is created.

---

# ❌ Destroy Message

A **Destroy Message** represents the destruction of an object.

The object's lifeline terminates at the destruction point.

---

# 📤 Lost Message

A **Lost Message** is a message that is sent but does not reach a known receiver.

```text
A ─────────> ●
```

---

# 📥 Found Message

A **Found Message** is a message whose sender is unknown but whose receiver is known.

```text
● ─────────> B
```

---

# 🔀 Sequence Diagram Control Structures

## `alt` — If / Else

Represents multiple alternative flows.

```text
alt
├── condition 1
│     ↓
│   Action A
│
└── else
      ↓
    Action B
```

Equivalent to:

```cpp
if (condition) {
    // Action A
}
else {
    // Action B
}
```

---

## `opt` — Optional / If

Represents an optional flow.

Equivalent to:

```cpp
if (condition) {
    // Action
}
```

---

## `loop` — Repetition

Represents a repeated operation.

Equivalent to:

```cpp
while (condition) {
    // Action
}
```

---

# 🏧 Example — ATM System

## Step 1 — Use Case

```text
Withdraw Cash
```

## Step 2 — Identify Objects

```text
User
ATM
Transaction
Account
CashDispenser
```

## Step 3 — Interaction

```text
User
 ↓
ATM
 ↓
Transaction
 ↓
Account
 ↓
CashDispenser
 ↓
User
```

### Example Flow

```text
User → ATM
       Enter account number + amount

ATM → Transaction
     Process withdrawal

Transaction → Account
             Verify balance

Account → Transaction
          Return result

Transaction → CashDispenser
             Dispense cash

CashDispenser → User
                Cash
```

This interaction can then be represented using a **Sequence Diagram**.

---

# 🧠 Quick Revision

## UML

```text
                    UML
                     │
            ┌────────┴────────┐
            │                 │
       Structural        Behavioural
            │                 │
      Class Diagram     Sequence Diagram
```

## Class Diagram

> **What exists in the system?**

Focus on:

```text
Classes
Attributes
Methods
Access Modifiers
Relationships
```

## Sequence Diagram

> **How do objects interact?**

Focus on:

```text
Objects
Lifelines
Activation Bars
Messages
Returns
Control Structures
```

## Relationships

```text
Inheritance  → IS-A

Association  → General relationship

Aggregation  → Weak HAS-A

Composition  → Strong HAS-A
```

## UML Access Modifiers

```text
+ → Public
- → Private
# → Protected
~ → Package / Default
```

## Sequence Controls

```text
alt  → If / Else
opt  → Optional If
loop → Repetition
```

---

# 🎯 LLD Design Process

UML helps us visualize the design **before writing the actual code**.

A typical LLD workflow is:

```text
Requirements
      ↓
Identify Classes / Objects
      ↓
Define Attributes & Methods
      ↓
Define Relationships
      ↓
Create Class Diagram
      ↓
Understand Object Interactions
      ↓
Create Sequence Diagram
      ↓
Implement the Design
```

### Final Takeaway

> **Class Diagram → Structure**

> **Sequence Diagram → Interaction**

> **UML → Visualize the design before implementation**
