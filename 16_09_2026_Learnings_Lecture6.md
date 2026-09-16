# SOLID Principles

SOLID is a set of five principles that help us design code that is:

* Maintainable
* Flexible
* Extensible
* Loosely coupled
* Easier to test

The five principles are:

* **S** — Single Responsibility Principle
* **O** — Open/Closed Principle
* **L** — Liskov Substitution Principle
* **I** — Interface Segregation Principle
* **D** — Dependency Inversion Principle

---

# L — Liskov Substitution Principle (LSP)

### Definition

> **Objects of a child class should be replaceable for objects of the parent class without breaking the correctness of the program.**

Suppose:

```text
A → Parent Class
B → Child Class
```

The client expects an object of `A`:

```text
Client → A
```

If we give the client an object of `B` instead:

```text
Client → B
```

the program should still work correctly.

### Simple Idea

```text
Parent → broader/general class
Child  → narrower/specific class
```

If the child cannot behave correctly wherever the parent is expected, then the inheritance relationship is probably wrong.

---

# LSP Guidelines

There are several rules that help us determine whether a child class properly follows LSP.

---

## 1. Signature Rule

A child method overriding a parent method should have a compatible method signature.

### Method Argument Rule

The child should accept the **same or broader** range of arguments than the parent contract allows.

```cpp
class Parent {
public:
    virtual void solve(string s) {
        // ...
    }
};

class Child : public Parent {
public:
    void solve(string s) override {
        // ...
    }
};
```

The child must not make the input requirements stricter.

### Example

Suppose the parent accepts:

```text
string
```

The child should not suddenly require:

```text
only strings of length > 10
```

because code that was valid for the parent may now fail for the child.

> **Child should not strengthen the input requirements of the parent.**

---

## 2. Return Type Rule

An overridden method in the child should return the **same type or a narrower/more specific type**, where the language supports covariant return types.

For example:

```text
Parent → Animal
Child  → Animal
```

is valid.

A more specific return type can also be valid:

```text
Parent → Animal
Child  → Dog
```

because:

```text
Dog IS-A Animal
```

This is called **Covariance**.

### Example

```cpp
class Animal {
};

class Dog : public Animal {
};
```

Conceptually:

```cpp
class Parent {
public:
    virtual Animal* solve(string s) {
        return new Animal();
    }
};

class Child : public Parent {
public:
    Dog* solve(string s) override {
        return new Dog();
    }
};
```

Here:

```text
Parent return type → Animal*
Child return type  → Dog*
```

`Dog*` is a narrower/more specific type than `Animal*`.

### Important

In C++, covariant return types are supported for certain **pointer/reference** return types. You cannot generally change a value return type from `Animal` to `Dog` in an override.

---

# 3. Exception Rule

A child should not introduce **broader or unexpected exceptions** that violate the parent's contract.

Suppose:

```text
Parent::m1()
    → can throw R1
```

The child implementation should not introduce an exception that is broader than what the client is prepared to handle.

Conceptually:

```text
Parent
  └── throws R1

Child
  └── throws R1
      OR a narrower/more specific exception
```

### Example

If the parent contract says that a method may produce a specific category of error, the child should not suddenly introduce a completely different, broader failure contract.

The key idea is:

> **A child should not make error handling more difficult for code written against the parent.**

> **Note:** C++ does not use Java-style checked exceptions, so this rule is mainly a conceptual LSP guideline in C++.

---

# 4. Property Rule

A child class should preserve the important properties/invariants established by the parent.

### What is an Invariant?

An **invariant** is a condition that should always remain true for an object.

For example:

```text
Account balance >= 0
```

If this is an invariant of the parent `Account` class, child classes should also preserve it.

```text
Parent Account
    balance >= 0

        ↓

Child Account
    balance >= 0
```

The child should not introduce behavior that breaks the parent's invariant.

---

# 5. History Constraint

A child should not violate constraints or assumptions established by the parent during the object's lifetime.

For example:

```text
Parent → Account
```

Suppose the parent's contract says:

```text
Withdrawal is always allowed.
```

Now we create:

```text
Child → FixedDepositAccount
```

But a fixed deposit may not allow withdrawal before maturity.

Therefore:

```text
Account
   ↓
withdraw() is expected

FixedDepositAccount
   ↓
withdraw() is not always allowed
```

The child cannot satisfy the parent's behavioral contract.

This indicates an **LSP violation**.

### Better Design

Instead of making every account a withdrawable account:

```text
Account
 ├── SavingsAccount
 ├── CurrentAccount
 └── FixedDepositAccount
```

we can separate capabilities:

```text
                 Account
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
   Withdrawable          Non-Withdrawable
      Account                 Account
          │                      │
     ┌────┴────┐                 │
     ↓         ↓                 ↓
  Savings    Current       Fixed Deposit
```

Now `FixedDepositAccount` is not forced to provide a behavior it cannot support.

---

# 6. Method Rule

The method rules are mainly concerned with:

* **Preconditions**
* **Postconditions**

---

## Preconditions

A **precondition** is a condition that must be satisfied **before** a method executes.

Suppose the parent method requires:

```text
0 <= num <= 5
```

Then the child should not make the precondition stronger.

### ❌ Stronger Precondition

Parent:

```text
0 <= num <= 5
```

Child:

```text
0 <= num <= 3
```

This is wrong because an input such as:

```text
num = 4
```

is valid for the parent but invalid for the child.

### ✅ Weaker Precondition

Parent:

```text
0 <= num <= 5
```

Child:

```text
0 <= num <= 10
```

This is acceptable because every input valid for the parent is still valid for the child.

### Rule

> **Child should have the same or weaker preconditions than the parent.**

In other words:

```text
Parent accepts: 0 <= num <= 5

Child accepts:  0 <= num <= 10
```

Good.

But:

```text
Parent accepts: 0 <= num <= 5

Child accepts:  0 <= num <= 3
```

Bad.

---

# Postconditions

A **postcondition** is a condition that must be satisfied **after** a method executes.

The child should preserve the parent's postcondition or provide a **stronger guarantee**.

### Example

Suppose:

```text
Parent → Car

brake()
```

Parent's postcondition:

```text
Car should slow down.
```

An electric car can implement:

```text
ElectricCar → brake()

→ speed decreases
→ battery may also be charged
```

The child still satisfies:

```text
speed decreases
```

and provides an additional guarantee:

```text
battery gets charged
```

So the child has a stronger postcondition while still satisfying the parent's contract.

### Rule

> **Child should have the same or stronger postconditions than the parent.**

---

# LSP Summary

| Rule                          | Child should                                                 |
| ----------------------------- | ------------------------------------------------------------ |
| **Arguments / Preconditions** | Keep same or make weaker                                     |
| **Return Type**               | Same or narrower/more specific where covariance is supported |
| **Exceptions**                | Not introduce broader/unexpected failures                    |
| **Invariants**                | Preserve parent's invariants                                 |
| **History Constraints**       | Preserve parent's behavioral constraints                     |
| **Postconditions**            | Keep same or make stronger                                   |

### Easy Memory Trick

```text
Input  → Weaken
Output → Strengthen
```

The child should:

```text
Accept MORE
Promise MORE
```

not:

```text
Accept LESS
Promise LESS
```

---

# I — Interface Segregation Principle (ISP)

### Definition

> **Clients should not be forced to depend on methods they do not use.**

Another common definition:

> **Many client-specific interfaces are better than one general-purpose interface.**

The main idea is:

> **Don't create a huge interface containing unrelated methods.**

---

## ❌ Without ISP

Suppose we have:

```cpp
class Shape {
public:
    virtual double area() = 0;
    virtual double volume() = 0;
};
```

Now consider:

```text
Square
Rectangle
Cube
```

A square needs:

```text
area()
```

but does not need:

```text
volume()
```

Similarly, a rectangle needs:

```text
area()
```

but does not need:

```text
volume()
```

Only a 3D shape such as a cube needs both.

So forcing every shape to implement `volume()` is unnecessary.

---

## ✅ Applying ISP

Separate the interfaces based on their responsibilities.

```cpp
class Shape2D {
public:
    virtual double area() = 0;
};
```

```cpp
class Shape3D {
public:
    virtual double area() = 0;
    virtual double volume() = 0;
};
```

Now:

```text
Shape2D
   ├── Square
   └── Rectangle

Shape3D
   └── Cube
```

`Square` and `Rectangle` don't need to implement `volume()`.

---

## Why ISP?

ISP helps us:

* Avoid unnecessary dependencies
* Keep interfaces small
* Reduce coupling
* Make implementations simpler
* Improve maintainability

### Easy Memory

> **Don't force a class to implement something it doesn't need.**

---

# D — Dependency Inversion Principle (DIP)

### Definition

> **High-level modules should not depend directly on low-level modules. Both should depend on abstractions.**

More formally:

> **High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions.**

---

# High-Level vs Low-Level Modules

### High-Level Module

Deals with:

```text
Business Logic
```

Examples:

```text
OrderService
PaymentService
UserService
```

### Low-Level Module

Deals with implementation details such as:

```text
Database
API
File System
External Services
```

Examples:

```text
MySQL
MongoDB
FileStorage
PaymentAPI
```

---

# ❌ Without DIP

Suppose:

```cpp
class OrderService {
    MySQLDatabase db;

public:
    void placeOrder() {
        // business logic

        db.save();
    }
};
```

Here:

```text
OrderService
      ↓
MySQLDatabase
```

The high-level business logic is directly dependent on a low-level implementation.

If we change:

```text
MySQL → MongoDB
```

we have to modify `OrderService`.

This creates tight coupling.

---

# ✅ Applying DIP

Introduce an abstraction:

```cpp
class Database {
public:
    virtual void save() = 0;
};
```

Now the low-level implementations depend on this abstraction:

```cpp
class MySQLDatabase : public Database {
public:
    void save() override {
        // save to MySQL
    }
};
```

```cpp
class MongoDatabase : public Database {
public:
    void save() override {
        // save to MongoDB
    }
};
```

And the high-level module also depends on the abstraction:

```cpp
class OrderService {
    Database* db;

public:

    OrderService(Database* db) {
        this->db = db;
    }

    void placeOrder() {
        // business logic

        db->save();
    }
};
```

Now the dependency becomes:

```text
High-Level Module
       │
       ↓
   Abstraction
       ↑
       │
Low-Level Module
```

More concretely:

```text
             Database
            /        \
           /          \
          ↓            ↓
    MySQLDatabase   MongoDatabase
          ↑
          │
     OrderService
```

`OrderService` does not care whether the database is MySQL or MongoDB.

---

# Dependency Injection

**Dependency Injection (DI)** is a technique used to provide a dependency to a class from outside rather than creating the dependency directly inside the class.

For example:

```cpp
class OrderService {
    Database* db;

public:
    OrderService(Database* db) {
        this->db = db;
    }
};
```

Here the `Database` object is passed into `OrderService`.

```cpp
MySQLDatabase mysql;

OrderService orderService(&mysql);
```

The dependency is **injected** from outside.

---

# DIP vs Dependency Injection

These two terms are related but not identical.

### DIP

A **design principle**:

```text
Depend on abstractions, not concrete implementations.
```

### Dependency Injection

A **technique** for supplying dependencies:

```text
Pass the dependency from outside.
```

So:

```text
DIP → Principle
DI  → Technique
```

---

# OCP and DIP Relationship

OCP says:

> **Software should be open for extension but closed for modification.**

DIP helps us achieve this by making high-level code depend on abstractions rather than concrete implementations.

For example:

```text
             Database Interface
              /             \
             ↓               ↓
          MySQL            MongoDB
```

We can add:

```text
PostgreSQL
Redis
FileDatabase
```

without changing the high-level business logic.

Therefore:

> **DIP is one of the important techniques/principles that helps us design systems that are easier to extend and follow OCP.**

---

# SOLID Quick Revision

| Principle   | Main Idea                                                      |
| ----------- | -------------------------------------------------------------- |
| **S — SRP** | One responsibility and one reason to change                    |
| **O — OCP** | Open for extension, closed for modification                    |
| **L — LSP** | Child should be safely substitutable for parent                |
| **I — ISP** | Don't force clients to depend on unused methods                |
| **D — DIP** | High-level and low-level modules should depend on abstractions |

---

# One-Line Memory Trick

```text
S → Single Responsibility
O → Open for Extension
L → Child behaves like Parent
I → Interfaces should be small
D → Depend on Abstractions
```

### Overall Goal of SOLID

```text
Less Coupling
     ↓
Better Maintainability
     ↓
Easier Extension
     ↓
Safer Changes
     ↓
Better Testability
```
