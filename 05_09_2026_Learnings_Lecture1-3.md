# LLD & OOP Fundamentals

## 1. LLD — Low-Level Design

LLD focuses on the **internal code structure** of an application.

The main goals of a good LLD are:

* **Scalability** — The design should be able to accommodate new requirements without major changes.
* **Maintainability** — The code should be easy to understand, modify, debug, and extend.
* **Reusability** — Components should be designed so they can be reused wherever appropriate.

### LLD vs HLD

| LLD                       | HLD                                     |
| ------------------------- | --------------------------------------- |
| Focuses on code structure | Focuses on system architecture          |
| Classes and objects       | Services and components                 |
| Interfaces                | APIs                                    |
| SOLID principles          | Scalability and availability            |
| Design patterns           | Load balancing                          |
| Class relationships       | Database architecture                   |
| Detailed implementation   | Technology and infrastructure decisions |

### HLD focuses on questions like:

* Which database should we use?
* SQL or NoSQL?
* Which technologies should we use?
* How do we scale the servers?
* How do different services communicate?
* Where should caching be used?
* How do we handle millions of requests?
* How can we optimize infrastructure cost?

### LLD focuses on questions like:

* What classes should exist?
* What should each class be responsible for?
* How should classes interact?
* Should we use inheritance or composition?
* Which interfaces should we create?
* Which design pattern fits the problem?
* How can we make the code maintainable and reusable?

---

# 2. DSA vs LLD vs HLD

### DSA

**DSA is a tool for solving computational problems efficiently.**

It focuses on:

* Data structures
* Algorithms
* Time complexity
* Space complexity
* Efficient problem solving

### LLD

**LLD is a tool for designing clean and maintainable code.**

It focuses on:

* Classes
* Objects
* Interfaces
* Relationships
* SOLID principles
* Design patterns

### HLD

**HLD is a tool for designing complete systems.**

It focuses on:

* Services
* APIs
* Databases
* Caching
* Message queues
* Load balancing
* Scalability
* Reliability

---

# 3. Procedural Programming

Procedural programming organizes a program around **procedures/functions** that operate on data.

It introduced and commonly uses concepts such as:

* Loops
* Functions
* Conditional statements
* Blocks
* Variables
* Procedures

### Problems with procedural programming

As applications become large, purely procedural designs can become difficult to manage.

Some common challenges are:

### 1. Real-world modelling

Real-world entities are difficult to represent naturally using only functions and separate data.

For example:

```text
Car
 ├── Data
 │    ├── brand
 │    ├── model
 │    └── speed
 │
 └── Behaviour
      ├── start()
      ├── stop()
      └── brake()
```

Keeping the data and its related behaviour together becomes useful.

### 2. Data security

If data is freely accessible throughout the program, it becomes difficult to control how that data is modified.

### 3. Reusability

Large procedural programs can become tightly coupled, making components harder to reuse.

### 4. Maintainability

As the codebase grows, modifying one part of the program can unintentionally affect other parts.

### 5. Scalability

Adding new features can become increasingly difficult when data and functionality are spread across many unrelated functions.

This led to the popularity of **Object-Oriented Programming (OOP)** for modelling complex software.

---

# 4. Object-Oriented Programming (OOP)

OOP organizes software around **objects**.

An object combines:

> **Data + Behaviour**

## Object

An object represents an entity that has:

### Characteristics / State

The properties that describe the object.

### Behaviour

The operations/functions that the object can perform.

For example:

```text
Car

Characteristics:
- Engine
- Brand
- Model
- Wheels

Behaviour:
- Start
- Stop
- Accelerate
- Brake
- Gear Shift
```

An object therefore represents a real-world entity along with the operations associated with it.

---

# 5. Class

A **class is a blueprint/template used to create objects.**

For example:

```cpp
class Car {
private:
    int speed;

public:
    void start();
    void stop();
    void accelerate();
};
```

Objects can then be created from the class:

```cpp
Car car1;
Car car2;
```

Here:

```text
Class  → Blueprint
Object → Actual instance
```

---

# 6. Four Pillars of OOP

The four major pillars of OOP are:

```text
        OOP
         │
 ┌───────┼────────┬──────────┐
 ↓       ↓        ↓          ↓
Abstraction Encapsulation Inheritance Polymorphism
```

---

# 7. Abstraction

### Definition

**Abstraction means hiding implementation details and exposing only the necessary functionality.**

The user should know **what an object can do**, without necessarily knowing **how it does it internally**.

### Real-world example

When driving a car, we use:

```text
Accelerator
Brake
Steering
Gear
```

We don't need to know the internal implementation of the engine to drive the car.

---

## Abstraction using C++

C++ supports abstraction using mechanisms such as:

* Abstract classes
* Pure virtual functions
* Interfaces (through abstract classes)

Example:

```cpp
class Car {
public:
    virtual void accelerate() = 0;
};
```

Here:

```cpp
virtual void accelerate() = 0;
```

is a **pure virtual function**.

The class `Car` is therefore an **abstract class**.

A derived class must provide an implementation:

```cpp
class SportsCar : public Car {
public:
    void accelerate() override {
        // Implementation
    }
};
```

The user can interact with the car through the abstraction:

```cpp
Car* myCar = new SportsCar();

myCar->accelerate();
```

```cpp
Car c;              // ❌ Error
Car* c = new Car(); // ❌ Error

class SportsCar : public Car {
public:
    void accelerate() override {
        cout << "Accelerating";
    }
};

SportsCar s;              // ✅
Car* c = new SportsCar(); // ✅
```

The caller only needs to know:

```text
Car → accelerate()
```

It doesn't need to know the internal implementation used by `SportsCar`.

### Key idea

> **Abstraction focuses on what an object does rather than how it does it.**

---

# 8. Encapsulation

### Definition

**Encapsulation means bundling data and the methods that operate on that data together, while controlling access to the internal state.**

In OOP:

```text
Object
 ├── Data
 └── Methods
```

For example:

```cpp
class Car {
private:
    int speed;

public:
    int getCurrSpeed() {
        return speed;
    }

    void setSpeed(int s) {
        speed = s;
    }
};
```

Here, `speed` is private.

Therefore:

```cpp
Car* myCar = new Car();

myCar->speed = 100;  // ❌ Not allowed
```

Instead, the class provides controlled methods:

```cpp
myCar->setSpeed(100);        // ✅
int speed = myCar->getCurrSpeed(); // ✅
```

### Access Modifiers

C++ provides three main access modifiers:

```text
public
private
protected
```

### public

Members can be accessed from outside the class.

```cpp
public:
    void start();
```

### private

Members can only be directly accessed from within the class and its friends.

```cpp
private:
    int speed;
```

### protected

Members can be accessed within the class and by derived classes (subject to the inheritance mode).

```cpp
protected:
    int enginePower;
```

### Key idea

> **Encapsulation protects an object's internal state by controlling how it can be accessed and modified.**

---

# 9. Inheritance

### Definition

**Inheritance allows one class to derive properties and behaviour from another class.**

It represents an **"is-a" relationship**.

Example:

```text
             Car
              │
       ┌──────┴──────┐
       ↓             ↓
   ManualCar     ElectricCar
```

Example in C++:

```cpp
class ManualCar : public Car {
};
```

Here:

```text
Car       → Parent / Base class
ManualCar → Child / Derived class
```

---

# 10. Types of Inheritance Access in C++

The inheritance mode determines how the inherited `public` and `protected` members appear in the derived class.

Assume:

```cpp
class Car {
public:
    int wheels;

protected:
    int enginePower;

private:
    int speed;
};
```

---

## Public Inheritance

```cpp
class ManualCar : public Car {
};
```

Access becomes:

```text
Car member       ManualCar
--------------------------------
public     →     public
protected  →     protected
private    →     inaccessible directly
```

So:

```text
public    → public
protected → protected
private   → inaccessible
```

This is the most common form when modelling a genuine **is-a relationship**.

---

## Protected Inheritance

```cpp
class ManualCar : protected Car {
};
```

Access becomes:

```text
Car member       ManualCar
--------------------------------
public     →     protected
protected  →     protected
private    →     inaccessible directly
```

So:

```text
public    → protected
protected → protected
private   → inaccessible
```

---

## Private Inheritance

```cpp
class ManualCar : private Car {
};
```

Access becomes:

```text
Car member       ManualCar
--------------------------------
public     →     private
protected  →     private
private    →     inaccessible directly
```

So:

```text
public    → private
protected → private
private   → inaccessible
```

### Important

The inheritance mode changes the **accessibility of inherited members**. It does not give the derived class direct access to the base class's private members.

---

# 11. Polymorphism

### Definition

**Polymorphism means "many forms".**

It allows the same interface or function name to represent different behaviour.

There are two commonly discussed forms in C++:

```text
Polymorphism
     │
 ┌───┴──────────┐
 ↓              ↓
Compile-time   Run-time
Polymorphism   Polymorphism
```

---

# 12. Compile-Time Polymorphism

Compile-time polymorphism is resolved by the compiler.

A common example is **function/method overloading**.

### Method Overloading

Multiple functions have the same name but different parameter lists.

```cpp
class Calculator {
public:

    int add(int a, int b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }
};
```

Both functions are named:

```text
add()
```

but have different parameter lists.

The compiler determines which function to call based on the arguments.

```cpp
Calculator c;

c.add(2, 3);       // calls add(int, int)
c.add(2, 3, 4);    // calls add(int, int, int)
```

---

# 13. Run-Time Polymorphism

Run-time polymorphism is commonly achieved through **method overriding and virtual functions**.

A derived class provides its own implementation of a function defined in the base class.

Example:

```cpp
class Car {
public:
    virtual void start() {
        cout << "Car starts";
    }
};

class ElectricCar : public Car {
public:
    void start() override {
        cout << "Electric car starts";
    }
};
```

Now:

```cpp
Car* car = new ElectricCar();

car->start();
```

Because `start()` is virtual, the call is resolved at runtime and:

```text
ElectricCar::start()
```

is executed.

### Key idea

> **Method overriding + virtual functions enable runtime polymorphism in C++.**

---

# 14. Quick Revision

### LLD

> Designing the internal structure of code.

Focus:

```text
Classes
Objects
Interfaces
SOLID
Design Patterns
Relationships
Maintainability
Reusability
```

### HLD

> Designing the architecture of the complete system.

Focus:

```text
Services
APIs
Databases
Caching
Queues
Load Balancers
Scalability
Reliability
Cost
```

### OOP

> A programming paradigm that models software using objects containing data and behaviour.

### Four Pillars

```text
Abstraction
Encapsulation
Inheritance
Polymorphism
```

### Abstraction

> Hide implementation details and expose essential functionality.

### Encapsulation

> Bundle data and behaviour together and control access to internal state.

### Inheritance

> Derive a new class from an existing class.

### Polymorphism

> Allow the same interface/name to represent different behaviours.
