# Factory Design Pattern

## What is Factory?

A **Factory** is a class/object whose responsibility is to **create objects**.

The main idea is:

> **The client should not need to know how an object is created. It should only ask for the object it needs.**

Instead of:

```text
Client
   ↓
creates concrete object directly
```

we want:

```text
Client
   ↓
Factory
   ↓
Concrete Object
```

The Factory handles the object creation logic.

---

# Burger Shop Example

Suppose we have a burger shop.

We have an abstract `Burger` class:

```cpp
class Burger {
public:
    virtual void prepare() = 0;
};
```

`Burger` is our **Product**.

---

## Concrete Products

We have different types of burgers:

```cpp
class BasicBurger : public Burger {
public:
    void prepare() override {
        // prepare basic burger
    }
};
```

```cpp
class StandardBurger : public Burger {
public:
    void prepare() override {
        // prepare standard burger
    }
};
```

```cpp
class PremiumBurger : public Burger {
public:
    void prepare() override {
        // prepare premium burger
    }
};
```

So:

```text
                 Burger
               <<abstract>>
                    |
          ┌─────────┼─────────┐
          ↓         ↓         ↓
       Basic     Standard   Premium
       Burger      Burger     Burger
```

---

# Problem With Direct Object Creation

Suppose the client directly creates a burger:

```cpp
Burger* burger = new StandardBurger();
```

The problem is that the client now needs to know:

* Which concrete class to create
* How that object is created
* Which class corresponds to which requirement

Suppose the client says:

```text
"I want a Premium Burger"
```

The client shouldn't have to know:

```cpp
new PremiumBurger();
```

The creation logic should be handled somewhere else.

---

# Factory

We introduce:

```text
BurgerFactory
```

The client tells the factory what it wants:

```text
"basic"
"standard"
"premium"
```

The factory creates the appropriate object.

---

# Simple Factory

A **Simple Factory** is a factory class that decides which concrete class to instantiate.

```cpp
class BurgerFactory {
public:

    Burger* createBurger(string type) {

        if (type == "basic") {
            return new BasicBurger();
        }
        else if (type == "standard") {
            return new StandardBurger();
        }
        else if (type == "premium") {
            return new PremiumBurger();
        }

        return nullptr;
    }
};
```

Now the client doesn't directly create the concrete burger.

```cpp
string type = "standard";

BurgerFactory* factory = new BurgerFactory();

Burger* burger = factory->createBurger(type);

burger->prepare();
```

The flow is:

```text
Client
  |
  | "standard"
  ↓
BurgerFactory
  |
  | creates
  ↓
StandardBurger
```

---

# Why Simple Factory?

The client only needs to know:

```text
"I need a standard burger."
```

It does not need to know:

```cpp
new StandardBurger();
```

The factory hides the object creation logic.

### Main Idea

> **Simple Factory centralizes object creation in one place.**

---

# Simple Factory Structure

```text
                 <<abstract>>
                    Burger
                       ↑
          ┌────────────┼────────────┐
          │            │            │
          ↓            ↓            ↓
       Basic        Standard      Premium
       Burger         Burger        Burger

                       ↑
                       |
                  creates
                       |
                BurgerFactory
                       ↑
                       |
                    Client
```

---

# Factory Method Pattern

Simple Factory works well when we have one factory responsible for creating different products.

But suppose our application grows.

Now we have **two different burger shops**.

```text
SinghFactory
KingFactory
```

Both produce burgers, but they produce different variants.

---

# Singh Factory

Singh Factory produces:

```text
BasicBurger
StandardBurger
PremiumBurger
```

# King Factory

King Factory produces:

```text
BasicWheatBurger
StandardWheatBurger
PremiumWheatBurger
```

Now we have:

```text
Burger
 ├── BasicBurger
 ├── StandardBurger
 ├── PremiumBurger
 │
 ├── BasicWheatBurger
 ├── StandardWheatBurger
 └── PremiumWheatBurger
```

And:

```text
BurgerFactory
 ├── SinghFactory
 └── KingFactory
```

---

# Problem

In Simple Factory, we had:

```text
BurgerFactory
```

which contained all the creation logic.

But now we want different factories to decide **which concrete products they create**.

This is where **Factory Method** comes in.

---

# Factory Method

### Definition

> **Factory Method defines an interface for creating an object, but lets subclasses decide which concrete class to instantiate.**

In simple words:

> **The parent factory defines the creation method, while child factories decide what object to create.**

---

# Abstract Factory

```cpp
class BurgerFactory {
public:
    virtual Burger* createBurger(string type) = 0;
};
```

Now we create concrete factories.

---

## SinghFactory

```cpp
class SinghFactory : public BurgerFactory {
public:

    Burger* createBurger(string type) override {

        if (type == "basic")
            return new BasicBurger();

        if (type == "standard")
            return new StandardBurger();

        if (type == "premium")
            return new PremiumBurger();

        return nullptr;
    }
};
```

---

## KingFactory

```cpp
class KingFactory : public BurgerFactory {
public:

    Burger* createBurger(string type) override {

        if (type == "basic")
            return new BasicWheatBurger();

        if (type == "standard")
            return new StandardWheatBurger();

        if (type == "premium")
            return new PremiumWheatBurger();

        return nullptr;
    }
};
```

Now the client can work with the abstract factory:

```cpp
string type = "standard";

BurgerFactory* factory = new KingFactory();

Burger* burger = factory->createBurger(type);

burger->prepare();
```

The client only knows:

```text
BurgerFactory
```

It doesn't need to know the concrete creation logic.

---

# Factory Method Structure

```text
                         <<abstract>>
                         BurgerFactory
                              |
                    createBurger()
                              |
                    ┌─────────┴─────────┐
                    ↓                   ↓
             SinghFactory          KingFactory
                    |                   |
                    | creates           | creates
                    ↓                   ↓
          Basic/Standard/Premium   Wheat Burgers
```

The important relationship is:

```text
Concrete Factory
       |
       | creates
       ↓
Concrete Product
```

---

# Simple Factory vs Factory Method

| Simple Factory                         | Factory Method                             |
| -------------------------------------- | ------------------------------------------ |
| Usually one factory class              | Multiple factory implementations           |
| Factory decides which object to create | Subclasses decide which object to create   |
| Creation logic centralized             | Creation logic distributed among factories |
| Not a GoF pattern by itself            | Formal GoF design pattern                  |
| Simple to implement                    | More extensible                            |

---

# Abstract Factory Pattern

Now suppose the burger shop doesn't only sell burgers.

It also sells:

```text
Burger
Garlic Bread
```

And different factories produce different **families of products**.

For example:

```text
Singh Factory
 ├── Singh Burger
 └── Singh Garlic Bread

King Factory
 ├── King Burger
 └── King Garlic Bread
```

Now one factory creates **multiple related products**.

This is where the **Abstract Factory Pattern** is useful.

---

# Definition

> **Abstract Factory provides an interface for creating families of related objects without specifying their concrete classes.**

The important word is:

> **Family**

---

# Example

Suppose we have:

```text
              Abstract Factory
                    |
             ┌──────┴──────┐
             ↓             ↓
       SinghFactory     KingFactory
             |             |
       ┌─────┴─────┐ ┌─────┴─────┐
       ↓           ↓ ↓           ↓
    Burger     GarlicBurger   Burger    GarlicBread
```

A factory can create multiple related products.

---

# Abstract Factory Interfaces

We can have:

```cpp
class Burger {
public:
    virtual void prepare() = 0;
};
```

```cpp
class GarlicBread {
public:
    virtual void prepare() = 0;
};
```

Then the factory:

```cpp
class RestaurantFactory {
public:
    virtual Burger* createBurger() = 0;
    virtual GarlicBread* createGarlicBread() = 0;
};
```

Concrete factory:

```cpp
class SinghFactory : public RestaurantFactory {
public:

    Burger* createBurger() override {
        return new SinghBurger();
    }

    GarlicBread* createGarlicBread() override {
        return new SinghGarlicBread();
    }
};
```

Another factory:

```cpp
class KingFactory : public RestaurantFactory {
public:

    Burger* createBurger() override {
        return new KingBurger();
    }

    GarlicBread* createGarlicBread() override {
        return new KingGarlicBread();
    }
};
```

---

# Simple Factory → Factory Method → Abstract Factory

Think of the progression:

```text
Simple Factory
      ↓
One factory decides what to create

Factory Method
      ↓
Multiple factories decide what to create

Abstract Factory
      ↓
Multiple factories create families of related products
```

---

# Abstract Factory Example

Suppose we have notification systems.

We may have:

```text
SMS Notification
Push Notification
Email Notification
```

A factory can decide which notification object to create based on the requested type.

For example:

```text
type = "SMS"
      ↓
NotificationFactory
      ↓
SMSNotification
```

Similarly:

```text
type = "EMAIL"
      ↓
NotificationFactory
      ↓
EmailNotification
```

The client doesn't need to directly create:

```cpp
new SMSNotification();
new EmailNotification();
```

It simply asks the factory for the required object.

---

# Factory Pattern — Main Idea

The core problem Factory solves is:

> **Object creation should be separated from object usage.**

Instead of:

```text
Client
  ↓
creates concrete object
  ↓
uses object
```

we have:

```text
Client
  ↓
Factory
  ↓
Concrete Object
  ↓
Client uses abstraction
```

---

# Why Factory?

Factory is useful when:

* Object creation is complex
* The exact concrete class is decided at runtime
* We want to hide creation logic
* We want to reduce coupling between client and concrete classes
* We expect new product types to be added

---

# Important Relationships

### Simple Factory

```text
Client
  ↓
Factory
  ↓
Product
```

### Factory Method

```text
Client
  ↓
Abstract Factory
  ↓
Concrete Factory
  ↓
Concrete Product
```

### Abstract Factory

```text
Client
  ↓
Abstract Factory
  ↓
Concrete Factory
  ├── Product A
  └── Product B
```

---

# Strategy vs Factory

These patterns are often confused.

### Strategy

Deals with:

> **Which behavior/algorithm should I use?**

Example:

```text
Payment
 ├── UPI
 ├── Credit Card
 └── Net Banking
```

The strategies perform different **algorithms/behaviors**.

### Factory

Deals with:

> **Which object should I create?**

Example:

```text
BurgerFactory
 ├── BasicBurger
 ├── StandardBurger
 └── PremiumBurger
```

The factory handles **object creation**.

```text
Strategy → Choose behavior

Factory  → Choose/create object
```

---

# Quick Revision

| Pattern              | Main Purpose                                  |
| -------------------- | --------------------------------------------- |
| **Simple Factory**   | Centralize object creation                    |
| **Factory Method**   | Let subclasses decide which product to create |
| **Abstract Factory** | Create families of related objects            |

### Easy Memory Trick

```text
Simple Factory
→ One Factory → Many Products

Factory Method
→ Many Factories → Products

Abstract Factory
→ One Factory → Family of Products
```

---

# Final Takeaway

> **Factory Pattern separates object creation from object usage. The client asks for what it needs instead of directly deciding how the concrete object should be created.**

The three levels are:

```text
Simple Factory
      ↓
Hide creation logic

Factory Method
      ↓
Delegate creation to subclasses

Abstract Factory
      ↓
Create families of related objects
```
