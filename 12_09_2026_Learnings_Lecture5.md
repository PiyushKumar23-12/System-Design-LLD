# SOLID Principles

SOLID is a set of five principles that help us write code that is:

* Maintainable
* Readable
* Flexible
* Less tightly coupled
* Easier to extend
* Easier to test

The five principles are:

* **S** — Single Responsibility Principle
* **O** — Open/Closed Principle
* **L** — Liskov Substitution Principle
* **I** — Interface Segregation Principle
* **D** — Dependency Inversion Principle

---

# S — Single Responsibility Principle (SRP)

### Definition

> **A class should have one responsibility and therefore only one reason to change.**

In simple words:

> **One class should focus on one responsibility.**

### Important

SRP does **not** mean:

> A class should contain only one method.

A class can have multiple methods, as long as those methods belong to the **same responsibility**.

---

## ❌ Without SRP

Suppose we have a `Cart` class:

```cpp
class Cart {
public:

    double calculateTotal() {
        // calculate cart total
    }

    void saveToDatabase() {
        // save cart to database
    }

    void generateInvoice() {
        // generate invoice
    }
};
```

Here `Cart` has multiple responsibilities:

1. Calculate the cart total
2. Save data to the database
3. Generate an invoice

Therefore, the class has multiple reasons to change.

For example:

* Pricing logic changes → `Cart` changes
* Database changes → `Cart` changes
* Invoice format changes → `Cart` changes

This violates **SRP**.

---

## ✅ Applying SRP

Separate the responsibilities into different classes:

```cpp
class Cart {
public:
    double calculateTotal() {
        // calculate cart total
    }
};
```

```cpp
class CartRepository {
public:
    void saveToDatabase(const Cart& cart) {
        // save cart to database
    }
};
```

```cpp
class CartInvoiceGenerator {
public:
    void generateInvoice(const Cart& cart) {
        // generate invoice
    }
};
```

Now each class has a clear responsibility.

```text
Cart
 └── Calculate Total

CartRepository
 └── Save to Database

CartInvoiceGenerator
 └── Generate Invoice
```

---

## Why SRP?

SRP helps us reduce:

* Tight coupling
* Unnecessary changes
* Code complexity
* Risk of breaking existing functionality
* Difficulty in testing

---

## Interview Definition

> **Single Responsibility Principle says that a class should have one responsibility and one reason to change. It does not mean one method per class; rather, the methods of a class should belong to the same responsibility.**

---

# O — Open/Closed Principle (OCP)

### Definition

> **Software entities should be open for extension but closed for modification.**

In simple words:

> **We should be able to add new functionality without modifying existing, tested code.**

---

## What does Open for Extension mean?

We should be able to **add new behavior**.

For example, suppose our application supports:

```text
MySQL
MongoDB
File Storage
```

Later we want to add:

```text
Redis
```

We should be able to add Redis support without changing the existing storage implementations.

---

## What does Closed for Modification mean?

Once existing code is working and tested, we should avoid modifying it every time a new requirement comes.

Instead, we should **extend** the system.

---

## ❌ Without OCP

Suppose we write:

```cpp
class SaveData {
public:

    void save(string type) {

        if (type == "SQL") {
            // save to SQL
        }
        else if (type == "MongoDB") {
            // save to MongoDB
        }
        else if (type == "File") {
            // save to File
        }
    }
};
```

Now suppose we want to add Redis:

```cpp
else if (type == "Redis") {
    // save to Redis
}
```

We have to modify the existing class.

As more storage types are added, this class keeps growing.

This is a violation of OCP.

---

## ✅ Applying OCP

Create an abstraction:

```cpp
class SaveData {
public:
    virtual void save() = 0;
};
```

Now different implementations can extend it:

```cpp
class SaveSQL : public SaveData {
public:
    void save() override {
        // save to SQL
    }
};
```

```cpp
class SaveMongoDB : public SaveData {
public:
    void save() override {
        // save to MongoDB
    }
};
```

```cpp
class SaveFile : public SaveData {
public:
    void save() override {
        // save to File
    }
};
```

Now if we want Redis:

```cpp
class SaveRedis : public SaveData {
public:
    void save() override {
        // save to Redis
    }
};
```

We **extended** the system instead of modifying existing classes.

---

## Main Idea

```text
             SaveData
                │
       ┌────────┼────────┐
       ↓        ↓        ↓
     SQL      Mongo     File
                         │
                       Redis
```

The base abstraction remains unchanged.

New functionality is added through new implementations.

---

## Why OCP?

OCP helps us:

* Add new functionality safely
* Avoid modifying stable code
* Reduce chances of introducing bugs
* Make the system easier to extend
* Reduce tight coupling

---

## Interview Definition

> **Open/Closed Principle says that software entities should be open for extension but closed for modification. When new functionality is required, we should preferably extend the existing design rather than modify already working code.**

---

# L — Liskov Substitution Principle (LSP)

### Definition

> **Objects of a subclass should be replaceable with objects of their superclass without breaking the correctness of the program.**

In simple words:

> **If `B` is a subclass of `A`, then wherever `A` is expected, we should be able to use `B` without breaking the program.**

---

## Example — Banking System

Suppose we have:

```text
Account
   │
   ├── SavingsAccount
   ├── CurrentAccount
   └── FixedDepositAccount
```

We might initially define:

```cpp
class Account {
public:

    virtual void deposit() = 0;

    virtual void withdraw() = 0;
};
```

Savings and Current accounts support both operations:

```cpp
class SavingsAccount : public Account {
public:

    void deposit() override {
        // deposit money
    }

    void withdraw() override {
        // withdraw money
    }
};
```

```cpp
class CurrentAccount : public Account {
public:

    void deposit() override {
        // deposit money
    }

    void withdraw() override {
        // withdraw money
    }
};
```

But a Fixed Deposit account may not allow withdrawal before maturity.

```cpp
class FixedDepositAccount : public Account {
public:

    void deposit() override {
        // deposit money
    }

    void withdraw() override {
        // cannot withdraw before maturity
    }
};
```

Now we have a problem.

If some code expects a generic `Account`:

```cpp
void withdrawMoney(Account* account) {
    account->withdraw();
}
```

We cannot safely pass a `FixedDepositAccount` because its `withdraw()` behavior does not satisfy the expected contract.

Therefore, `FixedDepositAccount` is not a proper substitute for `Account`.

This violates **LSP**.

---

# ❌ Bad Solution

We might try to fix it using type checking:

```cpp
void withdrawMoney(Account* account) {

    if (account->getType() == "FixedDeposit") {
        // don't withdraw
    }
    else {
        account->withdraw();
    }
}
```

But this creates problems.

The client now needs to know about specific subclasses.

If we add another special account type, we may need to modify this code again.

This leads to:

* Tight coupling
* More `if-else` conditions
* Difficult maintenance
* Violation of the idea behind OCP

---

# ✅ Better Design

Instead of forcing every account to support withdrawal, separate the capabilities.

### Deposit Account

```cpp
class NonWithdrawableAccount {
public:

    virtual void deposit() = 0;
};
```

### Withdrawable Account

```cpp
class WithdrawableAccount {
public:

    virtual void deposit() = 0;

    virtual void withdraw() = 0;
};
```

Now:

```cpp
class SavingsAccount : public WithdrawableAccount {
public:

    void deposit() override {
        // deposit
    }

    void withdraw() override {
        // withdraw
    }
};
```

```cpp
class CurrentAccount : public WithdrawableAccount {
public:

    void deposit() override {
        // deposit
    }

    void withdraw() override {
        // withdraw
    }
};
```

And:

```cpp
class FixedDepositAccount : public NonWithdrawableAccount {
public:

    void deposit() override {
        // deposit
    }
};
```

Now each type follows the correct contract.

```text
             Account Capabilities
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
   Withdrawable          Non-Withdrawable
      Account                 Account
          │                      │
     ┌────┴────┐                 │
     ↓         ↓                 ↓
 Savings     Current        Fixed Deposit
```

---

## Core Idea of LSP

The important thing is **behavior**, not just inheritance.

Just because:

```text
B inherits A
```

does not automatically mean:

```text
B is a valid substitute for A
```

The subclass must honor the **contract/behavior expected from the parent**.

---

## Signs of LSP Violation

LSP may be violated when:

* A subclass throws unexpected exceptions
* A subclass refuses to perform a behavior promised by the parent
* Client code needs `if-else` or type checks for specific subclasses
* A subclass changes the expected behavior of the parent
* A subclass cannot safely be used wherever the parent is expected

---

## Interview Definition

> **Liskov Substitution Principle says that a subclass should be substitutable for its superclass without changing the correctness or expected behavior of the program. If a subclass cannot honor the contract of its parent, then inheritance is probably not appropriate.**

---

# S vs O vs L — Quick Revision

| Principle | Main Idea                                                       |
| --------- | --------------------------------------------------------------- |
| **SRP**   | One class → one responsibility → one reason to change           |
| **OCP**   | Add new behavior by extension rather than modifying stable code |
| **LSP**   | Subclass should safely substitute its parent                    |

---

# Easy Way to Remember

```text
S → One Responsibility

O → Extend, Don't Modify

L → Child Should Behave Like Parent
```

### In one line:

> **SRP separates responsibilities, OCP makes the system extensible, and LSP ensures inheritance is behaviorally correct.**
