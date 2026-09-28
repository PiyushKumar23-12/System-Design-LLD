# Singleton Design Pattern

## 1. What is Singleton?

**Singleton** is a design pattern where a class is allowed to have **only one object/instance throughout the application**.

### Normal Class

A normal class can create multiple objects:

```cpp
A* a1 = new A();
A* a2 = new A();
A* a3 = new A();
```

Here, each `new A()` creates a **new object on the heap**.

The pointers `a1`, `a2`, `a3` are separate pointer variables.

```text
Stack                 Heap

a1 ────────────────→  A object 1

a2 ────────────────→  A object 2

a3 ────────────────→  A object 3
```

So:

```cpp
a1 != a2
```

---

# 2. Singleton Goal

For a Singleton:

```cpp
Singleton* s1 = Singleton::getInstance();
Singleton* s2 = Singleton::getInstance();
```

Both should point to the **same object**.

```text
Stack                 Heap

s1 ────────┐
           ├────────→ Singleton object
s2 ────────┘
```

Therefore:

```cpp
s1 == s2   // true
```

The basic idea is:

> **First call → create the object**
> **Every later call → return the same object**

---

# 3. How do we achieve Singleton?

We mainly use **encapsulation**.

There are three important steps:

1. Make the constructor `private`
2. Create a `static` instance pointer
3. Create a `static` `getInstance()` method

Why private constructor?

Because otherwise anyone could do:

```cpp
Singleton* s = new Singleton();
```

and create multiple objects.

---

# 4. Lazy Initialization

The object is created **only when it is requested for the first time**.

```cpp
class Singleton {

private:
    static Singleton* instance;

    Singleton() {
    }

public:
    static Singleton* getInstance() {

        if (instance == nullptr) {
            instance = new Singleton();
        }

        return instance;
    }
};
```

Define the static variable outside the class:

```cpp
Singleton* Singleton::instance = nullptr;
```

Now:

```cpp
Singleton* s1 = Singleton::getInstance();
Singleton* s2 = Singleton::getInstance();

cout << (s1 == s2);
```

Output:

```text
1
```

Because both pointers point to the same object.

---

# 5. Why `static`?

We make `instance` static because we want **only one instance variable for the entire class**, not one copy per object.

```cpp
static Singleton* instance;
```

Similarly:

```cpp
static Singleton* getInstance();
```

means `getInstance()` belongs to the **class**, so we can call:

```cpp
Singleton::getInstance();
```

without creating a Singleton object first.

---

# 6. Multithreading Problem

The previous implementation has a problem when multiple threads execute simultaneously.

Suppose:

```cpp
if (instance == nullptr) {
    instance = new Singleton();
}
```

Two threads can reach this code at almost the same time.

```text
Thread 1                  Thread 2

instance == nullptr?     instance == nullptr?
       ↓                         ↓
      true                      true
       ↓                         ↓
 create object             create object
```

Now two objects may be created.

That violates the Singleton requirement.

---

# 7. Mutex

A **critical section** is a section of code where multiple threads could access shared data and cause a race condition.

A **mutex** helps ensure that only one thread enters the critical section at a time.

```cpp
static mutex mtx;
```

Then:

```cpp
lock_guard<mutex> lock(mtx);
```

automatically locks the mutex when entering the scope and unlocks it when leaving.

---

# 8. Double-Checked Locking

We don't want to lock every time because locking/unlocking has overhead.

So we check twice:

```cpp
class Singleton {

private:
    static Singleton* instance;
    static mutex mtx;

    Singleton() {
    }

public:
    static Singleton* getInstance() {

        if (instance == nullptr) {

            lock_guard<mutex> lock(mtx);

            if (instance == nullptr) {
                instance = new Singleton();
            }
        }

        return instance;
    }
};
```

Definitions:

```cpp
Singleton* Singleton::instance = nullptr;
mutex Singleton::mtx;
```

### Why check twice?

First check:

```cpp
if (instance == nullptr)
```

avoids locking when the object already exists.

Then after acquiring the lock, we check again:

```cpp
if (instance == nullptr)
```

because another thread might have created the object while this thread was waiting for the lock.

### Flow

```text
                getInstance()
                     |
                     ↓
          instance == nullptr?
             /            \
           No              Yes
           |                |
           ↓                ↓
      return instance     lock mutex
                            |
                            ↓
                   instance == nullptr?
                       /          \
                     No            Yes
                     |              |
                     ↓              ↓
               return instance   create object
                                      |
                                      ↓
                               return instance
```

> In production C++, hand-written double-checked locking has memory-ordering subtleties; `std::call_once` or a function-local static is generally safer.

---

# 9. Eager Initialization

Another approach is to create the Singleton **when the program/class initialization happens**, rather than waiting for the first call.

```cpp
Singleton* Singleton::instance = new Singleton();
```

Now the object already exists.

Every call:

```cpp
Singleton::getInstance();
```

returns that same object.

### Advantages

* No `nullptr` check for creation
* No locking needed for lazy creation
* Simple approach

### Disadvantage

The object is created even if the application **never uses it**.

Suppose the Singleton is:

* expensive in memory
* expensive to initialize
* requires resources

Then eager initialization can waste resources.

---

# 10. Lazy vs Eager Initialization

|                 | Lazy Initialization         | Eager Initialization      |
| --------------- | --------------------------- | ------------------------- |
| Object creation | First use                   | During initialization     |
| Resource usage  | Only when needed            | Even if unused            |
| Thread safety   | Needs consideration         | Initialization is simpler |
| Complexity      | Higher                      | Lower                     |
| Main advantage  | Avoids unnecessary creation | Simple and predictable    |

---

# 11. Singleton Structure

The basic Singleton pattern looks like:

```cpp
class Singleton {

private:
    static Singleton* instance;

    Singleton() {
    }

public:
    static Singleton* getInstance() {

        if (instance == nullptr) {
            instance = new Singleton();
        }

        return instance;
    }
};
```

The important parts are:

```text
private constructor
       ↓
prevent direct object creation

static instance
       ↓
one shared instance

static getInstance()
       ↓
global access point to that instance
```

---

# 12. Real-World Use Cases

Singleton can be useful when the application genuinely needs **one shared instance**.

### Database Connection Manager

```text
Application
     |
     ↓
DatabaseManager
     |
     └── one shared instance
```

### Logging System

```text
Multiple classes
      |
      ↓
 Logger
      |
      └── one shared instance
```

### Configuration Manager

```text
Service A ──┐
Service B ──┼──→ ConfigurationManager
Service C ──┘
```

All components can access the same configuration manager.

---

# 13. Interview Definition

> **Singleton is a creational design pattern that ensures a class has only one instance and provides a global access point to that instance.**

### How to implement?

Remember:

```text
Private Constructor
        +
Static Instance
        +
Static getInstance()
```

### Common examples

```text
Database Connection
Logger
Configuration Manager
Cache Manager
```

---

# 14. Key Points to Remember

```text
Normal Class
→ Multiple objects

Singleton
→ Only one object

Private Constructor
→ Prevents direct object creation

Static Instance
→ One shared instance per class

Static getInstance()
→ Access the shared instance

Lazy Initialization
→ Create when first needed

Mutex
→ Protect shared initialization in multithreaded code

Double-Checked Locking
→ Avoid locking on every access

Eager Initialization
→ Create instance beforehand
```

### One-line memory trick

> **Singleton = Private Constructor + Static Instance + Static Access Method**
