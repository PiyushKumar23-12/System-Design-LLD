# Design Patterns

Design Patterns are **reusable solutions to commonly occurring software design problems**.

They provide a standard way to structure classes and objects for problems that appear repeatedly in software design.

---

# Strategy Design Pattern

## Basic Idea

In many applications, there are:

```text
Part of application
        |
        ├── Part that rarely changes
        |
        └── Part that frequently changes
```

The frequently changing behavior should be **separated from the stable part**.

The **Strategy Design Pattern** helps us achieve this.

### Definition

> **Strategy Pattern defines a family of algorithms, encapsulates each algorithm in a separate class, and makes them interchangeable at runtime.**

In simple words:

> **Separate frequently changing behavior into different classes and inject the required behavior into the main class.**

---

# Example — Robot Simulation

Suppose we are building an application that simulates different robots.

Initially:

```cpp
class Robot {
public:
    void walk();
    void talk();

    virtual void projection() = 0;
};
```

Different robots can have different projections:

```cpp
class CompanionRobot : public Robot {
public:
    void projection() override {
        // projection logic
    }
};
```

```cpp
class WorkerRobot : public Robot {
public:
    void projection() override {
        // projection logic
    }
};
```

---

# New Requirement — Flying Robots

Now we introduce:

```text
Sparrow Robot
```

A Sparrow can fly:

```cpp
class SparrowRobot : public Robot {
public:
    void fly();

    void projection() override {
        // projection logic
    }
};
```

Later we may add:

```text
Crow
Pigeon
Eagle
```

All of them may need flying behavior.

---

# Problem — Code Repetition

We might end up writing:

```cpp
class SparrowRobot {
public:
    void fly() {
        // flying logic
    }
};
```

```cpp
class CrowRobot {
public:
    void fly() {
        // same flying logic
    }
};
```

```cpp
class PigeonRobot {
public:
    void fly() {
        // same flying logic
    }
};
```

This violates **DRY**:

> **Don't Repeat Yourself**

We are repeating the same behavior in multiple classes.

---

# Another Problem — Inheritance Explosion

We might try to solve this using inheritance.

But now robots can vary in many independent behaviors:

* Can talk / cannot talk
* Can walk / cannot walk
* Can fly / cannot fly

We can end up with a complicated hierarchy:

```text
                         Robot
                  /                 \
               Talk                NoTalk
              /    \              /      \
           Walk   NoWalk        Walk     NoWalk
            / \     / \          / \       / \
          Fly NoFly Fly NoFly  Fly NoFly Fly NoFly
```

As more behaviors are added, the number of combinations grows rapidly.

For example:

```text
CanTalk?
CanWalk?
CanFly?
CanSwim?
CanRun?
CanJump?
```

Inheritance becomes difficult to maintain.

---

# Problems With This Approach

### 1. Code Repetition

Common behavior may have to be copied into multiple classes.

### 2. OCP Problems

Adding or changing behavior may require modifying existing classes.

### 3. Complex Inheritance Hierarchy

Multiple independent behaviors create too many combinations.

### 4. Poor Flexibility

A robot's behavior is strongly tied to its class hierarchy.

---

# Strategy Pattern Solution

Instead of putting every behavior inside `Robot`, we separate the behaviors into their own classes.

We can identify three independent behaviors:

```text
Talk
Walk
Fly
```

Each behavior becomes a **strategy family**.

---

# Talk Strategy

Create an abstraction:

```cpp
class Talkable {
public:
    virtual void talk() = 0;
};
```

Concrete strategies:

```cpp
class NormalTalk : public Talkable {
public:
    void talk() override {
        // normal talking
    }
};
```

```cpp
class NoTalk : public Talkable {
public:
    void talk() override {
        // cannot talk
    }
};
```

So:

```text
          Talkable
          /      \
         /        \
 NormalTalk      NoTalk
```

---

# Walk Strategy

Create another strategy:

```cpp
class Walkable {
public:
    virtual void walk() = 0;
};
```

Concrete strategies:

```cpp
class NormalWalk : public Walkable {
public:
    void walk() override {
        // normal walking
    }
};
```

```cpp
class NoWalk : public Walkable {
public:
    void walk() override {
        // cannot walk
    }
};
```

```text
          Walkable
          /      \
         /        \
  NormalWalk     NoWalk
```

---

# Fly Strategy

Similarly:

```cpp
class Flyable {
public:
    virtual void fly() = 0;
};
```

Concrete strategies:

```cpp
class NormalFly : public Flyable {
public:
    void fly() override {
        // normal flying
    }
};
```

```cpp
class NoFly : public Flyable {
public:
    void fly() override {
        // cannot fly
    }
};
```

```text
           Flyable
           /     \
          /       \
    NormalFly     NoFly
```

Later we can have:

```text
Flyable
   |
   ├── NormalFly
   ├── JetFly
   └── RocketFly
```

without changing the `Robot` class.

---

# Robot Using Composition

Now instead of inheriting all these behaviors, `Robot` **has** these strategies.

```cpp
class Robot {

    Talkable* talkStrategy;
    Walkable* walkStrategy;
    Flyable* flyStrategy;

public:

    Robot(
        Talkable* talkStrategy,
        Walkable* walkStrategy,
        Flyable* flyStrategy
    ) {
        this->talkStrategy = talkStrategy;
        this->walkStrategy = walkStrategy;
        this->flyStrategy = flyStrategy;
    }
};
```

This is a **HAS-A relationship**.

```text
Robot
  |
  ├── HAS-A → Talkable
  |
  ├── HAS-A → Walkable
  |
  └── HAS-A → Flyable
```

This is **composition**.

---

# Robot Behavior

Now the robot delegates the behavior to its strategies.

```cpp
class Robot {

    Talkable* talkStrategy;
    Walkable* walkStrategy;
    Flyable* flyStrategy;

public:

    Robot(
        Talkable* t,
        Walkable* w,
        Flyable* f
    ) : talkStrategy(t),
        walkStrategy(w),
        flyStrategy(f) {}

    void talk() {
        talkStrategy->talk();
    }

    void walk() {
        walkStrategy->walk();
    }

    void fly() {
        flyStrategy->fly();
    }

    virtual void projection() = 0;
};
```

The `Robot` doesn't implement the actual talking, walking, or flying behavior.

It delegates those behaviors to the corresponding strategies.

---

# Creating a Robot

Suppose we want a Companion Robot that:

* Can talk
* Can walk
* Cannot fly

```cpp
class CompanionRobot : public Robot {
public:

    CompanionRobot(
        Talkable* t,
        Walkable* w,
        Flyable* f
    ) : Robot(t, w, f) {}

    void projection() override {
        // Companion robot projection
    }
};
```

Now:

```cpp
Robot* robo = new CompanionRobot(
    new NormalTalk(),
    new NormalWalk(),
    new NoFly()
);
```

The robot gets its behavior from the strategies passed to it.

---

# Why Composition Is Better Here

Instead of:

```text
Robot
 └── inheritance
      ├── Talk
      ├── Walk
      └── Fly
```

we use:

```text
Robot
 ├── HAS-A → Talkable
 ├── HAS-A → Walkable
 └── HAS-A → Flyable
```

Now behaviors are independent.

For example:

```text
Robot A
    NormalTalk
    NormalWalk
    NoFly

Robot B
    NoTalk
    NormalWalk
    NormalFly

Robot C
    NormalTalk
    NoWalk
    JetFly
```

We don't need a separate class for every possible combination.

---

# Runtime Behavior Change

One of the important advantages of Strategy Pattern is that the strategy can be changed.

For example:

```cpp
robot->setFlyStrategy(new JetFly());
```

Now the same robot can use a different flying algorithm.

```text
Before:

Robot → NormalFly


After:

Robot → JetFly
```

The `Robot` class itself doesn't change.

This is what is meant by:

> **Strategies are interchangeable at runtime.**

---

# Important Terms

## Context

The class that **uses the strategy**.

In our example:

```text
Robot → Context
```

The `Robot` uses the different behavior strategies.

---

## Strategy

The abstraction/interface representing a family of algorithms.

```text
Talkable
Walkable
Flyable
```

---

## Concrete Strategy

The actual implementation of the algorithm/behavior.

```text
NormalTalk
NoTalk

NormalWalk
NoWalk

NormalFly
NoFly
JetFly
```

---

# Strategy Pattern Structure

Generic structure:

```text
                  Client
                    |
                    ↓
                 Context
                    |
                  HAS-A
                    |
                    ↓
              Strategy
                    |
              ┌─────┴─────┐
              ↓           ↓
       ConcreteStrategy1  ConcreteStrategy2
```

The Client selects the required strategy.

The Context uses the strategy through the abstraction.

---

# Strategy Pattern in Payment System

Strategy Pattern is not limited to robots.

Consider a payment system.

Different payment methods have different payment algorithms:

```text
Payment
   |
   ├── UPI
   ├── Credit Card
   └── Net Banking
```

Create an abstraction:

```cpp
class PaymentStrategy {
public:
    virtual void pay() = 0;
};
```

Concrete strategies:

```cpp
class UPIPayment : public PaymentStrategy {
public:
    void pay() override {
        // UPI payment
    }
};
```

```cpp
class CreditCardPayment : public PaymentStrategy {
public:
    void pay() override {
        // Credit card payment
    }
};
```

```cpp
class NetBankingPayment : public PaymentStrategy {
public:
    void pay() override {
        // Net banking payment
    }
};
```

The payment system uses:

```cpp
class PaymentSystem {
    PaymentStrategy* strategy;

public:
    PaymentSystem(PaymentStrategy* strategy) {
        this->strategy = strategy;
    }

    void pay() {
        strategy->pay();
    }
};
```

Now the Client decides:

```cpp
UPIPayment upi;
PaymentSystem payment(&upi);
```

or:

```cpp
CreditCardPayment card;
PaymentSystem payment(&card);
```

The `PaymentSystem` does not need to know the details of UPI or Credit Card payment.

---

# Strategy Pattern — Key Idea

The main problem is usually:

```text
One class has multiple behaviors
        ↓
Those behaviors change independently
        ↓
Inheritance creates complexity
        ↓
Separate the behaviors into strategy classes
        ↓
Use composition to inject the required strategies
```

---

# Strategy vs Inheritance

### Inheritance

```text
Robot
  |
  ├── FlyingRobot
  ├── WalkingRobot
  ├── TalkingRobot
  └── ...
```

Behaviors become tightly coupled to the class hierarchy.

### Strategy

```text
                 Robot
              /    |    \
             ↓     ↓     ↓
          Talk   Walk   Fly
           |      |      |
        Strategy Strategy Strategy
```

Behaviors can vary independently.

---

# Composition Over Inheritance

Strategy Pattern is a classic example of:

> **Prefer composition over inheritance when behavior needs to vary independently.**

Instead of saying:

```text
"Robot IS-A NormalFly"
```

we say:

```text
"Robot HAS-A FlyStrategy"
```

This makes the design more flexible.

---

# SOLID Connection

Strategy Pattern naturally supports several SOLID ideas.

### SRP

Each strategy has one focused responsibility:

```text
NormalFly → Flying behavior
NoFly     → No-flying behavior
NormalTalk → Talking behavior
```

### OCP

We can add:

```text
JetFly
RocketFly
WhisperTalk
RobotWalk
```

without modifying `Robot`.

### DIP

`Robot` depends on abstractions:

```text
Robot
 ↓
Flyable
 ↓
NormalFly / NoFly / JetFly
```

rather than concrete implementations.

---

# Final Revision

```text
Strategy Pattern
        ↓
Family of algorithms/behaviors
        ↓
Put each behavior in a separate class
        ↓
Context uses strategy through abstraction
        ↓
Strategies can be changed/interchanged
        ↓
Avoid complicated inheritance
```

### Easy Memory Trick

> **"What changes frequently → separate it into a strategy."**

### Robot Example

```text
Robot
 ├── Talkable
 │    ├── NormalTalk
 │    └── NoTalk
 │
 ├── Walkable
 │    ├── NormalWalk
 │    └── NoWalk
 │
 └── Flyable
      ├── NormalFly
      ├── NoFly
      └── JetFly
```

### Payment Example

```text
PaymentSystem
      |
      ↓
PaymentStrategy
   /     |      \
  ↓      ↓       ↓
 UPI   Credit   NetBanking
```

**Core takeaway:**

> **Strategy Pattern separates frequently changing behavior from the main class, encapsulates each behavior as a separate strategy, and allows the behavior to be selected or changed without modifying the main class.**
