# Google Docs / Notepad — LLD Design Evolution

## Top-Down vs Bottom-Up Approach

While designing a system, we can approach it in two ways:

### Top-Down Approach

Start with the overall system and break it into smaller components.

```text
Document Editor
      ↓
Break into responsibilities
      ↓
Document / Renderer / Persistence / Elements
```

### Bottom-Up Approach

Start by designing smaller reusable components and then combine them to build the complete system.

---

# Initial Document Editor Design

Suppose we are building a simple **Notepad / Document Editor**.

Currently, we support only two types of input:

* Text
* Image

For images, we will store the **image path**.

Initially, we might think of storing them separately:

```text
textList
imageList
```

But this creates a problem.

Suppose the user enters:

```text
Text
Image
Text
Image
```

If we store them in separate lists, we lose the original order.

Therefore, we need a **single list** that maintains the order in which elements were added.

---

# Initial Architecture

We could start with:

```cpp
class DocumentEditor {
    vector<string> elements;

public:
    void addText();
    void addImage();
    void renderDocument();
    void saveToFile();
};
```

Here, everything is handled by one class.

---

# Problems With Initial Architecture

## 1. SRP Violation

`DocumentEditor` has multiple responsibilities:

```text
DocumentEditor
 ├── Add text
 ├── Add image
 ├── Render document
 └── Save document
```

So it has multiple reasons to change.

For example:

* Rendering logic changes
* Storage mechanism changes
* New document element is added
* Document structure changes

All of these can force changes to `DocumentEditor`.

Therefore, it violates **Single Responsibility Principle**.

---

# 2. OCP Violation

Initially:

```text
elements
 ├── Text
 └── Image
```

But later we may want to support:

```text
Video
Table
Audio
Link
Code Block
```

If everything is stored as `string`, `DocumentEditor` will need more and more logic:

```cpp
if (type == "text") {
    // ...
}
else if (type == "image") {
    // ...
}
else if (type == "video") {
    // ...
}
```

Every new element requires modifying existing code.

This goes against the **Open/Closed Principle**.

---

# Introducing Polymorphism

Instead of treating every element as a `string`, we create an abstraction.

```cpp
class DocumentElement {
public:
    virtual void render() = 0;
};
```

Now we can create different types of elements.

```cpp
class TextElement : public DocumentElement {
public:
    void render() override {
        // render text
    }
};
```

```cpp
class ImageElement : public DocumentElement {
public:
    void render() override {
        // render image
    }
};
```

Now:

```text
             DocumentElement
             <<abstract>>
                  |
          ┌───────┴───────┐
          ↓               ↓
    TextElement      ImageElement
```

Later we can add:

```cpp
class VideoElement : public DocumentElement {
public:
    void render() override {
        // render video
    }
};
```

without modifying `TextElement` or `ImageElement`.

This gives us extensibility through **polymorphism** and supports OCP.

---

# Introducing Document

Now the actual document can store all elements in their original order.

```cpp
class Document {
    vector<DocumentElement*> elements;

public:
    void addElement(DocumentElement* element);
};
```

For example:

```text
Document
   |
   ├── TextElement
   ├── ImageElement
   ├── TextElement
   └── ImageElement
```

Because everything is stored through the base type:

```cpp
vector<DocumentElement*> elements;
```

we can store:

```cpp
TextElement*
ImageElement*
VideoElement*
TableElement*
```

as long as they inherit from `DocumentElement`.

---

# Introducing Persistence

Saving the document is another separate responsibility.

Instead of making `DocumentEditor` directly save to a file, we create an abstraction:

```cpp
class Persistence {
public:
    virtual void save(string data) = 0;
};
```

Different implementations can provide different storage mechanisms.

```cpp
class SaveToFile : public Persistence {
public:
    void save(string data) override {
        // save to file
    }
};
```

```cpp
class SaveToDB : public Persistence {
public:
    void save(string data) override {
        // save to database
    }
};
```

Later we can add:

```text
SaveToSQL
SaveToMongo
SaveToCloud
SaveToS3
```

without changing the abstraction.

---

# Improved Document Editor

Now `DocumentEditor` becomes:

```cpp
class DocumentEditor {
    Document* doc;
    Persistence* persistence;

public:
    void addText();
    void addImage();
    void renderDoc();
    void save();
};
```

The important change is:

> `DocumentEditor` still exposes methods such as `addText()`, `addImage()`, `renderDoc()` and `save()`, but it no longer performs all the actual work itself.

It delegates the work to the appropriate objects.

For example:

```text
DocumentEditor
      |
      ├── addText()
      |      ↓
      |   Document
      |
      ├── addImage()
      |      ↓
      |   Document
      |
      └── save()
             ↓
        Persistence
```

---

# Current Design

```text
DocumentEditor
    |
    ├── Document
    |
    └── Persistence
            |
       ┌────┴────┐
       ↓         ↓
 SaveToFile   SaveToDB
```

And:

```text
Document
    |
    ↓
DocumentElement
    |
    ├── TextElement
    └── ImageElement
```

---

# LSP in This Design

`Document` stores:

```cpp
vector<DocumentElement*> elements;
```

Suppose:

```cpp
TextElement text;
ImageElement image;
```

Both can be passed where `DocumentElement*` is expected:

```cpp
elements.push_back(&text);
elements.push_back(&image);
```

The code works through the common interface:

```cpp
DocumentElement* element;
element->render();
```

The actual implementation is selected at runtime.

```text
DocumentElement*
       |
   ┌───┴────┐
   ↓        ↓
 Text     Image
   ↓        ↓
render()  render()
```

This demonstrates **polymorphism**, and provided each subtype correctly follows the `DocumentElement` contract, it also satisfies **LSP**.

> A `TextElement` or `ImageElement` should be safely substitutable wherever a `DocumentElement` is expected.

---

# DIP in This Design

`DocumentEditor` should not directly depend on:

```text
SaveToFile
SaveToDB
```

Instead:

```text
DocumentEditor
       ↓
 Persistence
       ↑
 ┌─────┴─────┐
 ↓           ↓
File         DB
```

`Persistence` is the abstraction.

Therefore:

```cpp
class DocumentEditor {
    Persistence* persistence;
};
```

`DocumentEditor` doesn't care how saving is implemented.

---

# Dependency Injection

The actual `Persistence` implementation can be provided from outside.

For example:

```cpp
SaveToFile file;

DocumentEditor editor(&document, &file);
```

or:

```cpp
SaveToDB db;

DocumentEditor editor(&document, &db);
```

The `DocumentEditor` does not need to change.

```text
Client
   |
   | creates dependency
   ↓
SaveToFile / SaveToDB
   |
   | injects
   ↓
DocumentEditor
```

---

# Another Problem — DocumentEditor Still Knows Too Much

At this stage, `DocumentEditor` has methods like:

```text
addText()
addImage()
renderDoc()
save()
```

Even though it delegates the actual work, it still needs to know:

```text
Document → what methods does it provide?
Persistence → what methods does it provide?
```

For example, if `Document` is responsible for rendering:

```text
Document
 ├── addElement()
 ├── getElements()
 └── render()
```

then `DocumentEditor` needs to know that `Document` has a `render()` method.

If later we remove `render()` from `Document`, we may also need to modify `DocumentEditor`.

Similarly, changes to the persistence interface can force changes in `DocumentEditor`.

This creates **unnecessary reasons for `DocumentEditor` to change**.

---

# Separate Rendering Responsibility

Rendering is its own responsibility.

So we introduce:

```cpp
class DocumentRenderer {
public:
    void render(Document* document);
};
```

Now:

```text
DocumentRenderer
       |
       ↓
    Document
       |
       ↓
DocumentElement
   /          \
  ↓            ↓
Text         Image
```

`Document` no longer needs to know how to render itself.

Therefore:

```text
Document
→ stores document data

DocumentRenderer
→ renders document
```

---

# Final Architecture

```text
                              Client
                         /       |       \
                        ↓        ↓        ↓
               DocumentEditor  Persistence  DocumentRenderer
                    |              |              |
                    ↓              ↓              ↓
                 Document     ┌────┴────┐       Document
                    |         ↓         ↓          |
                    |     SaveToFile  SaveToDB      |
                    |                              |
                    ↓                              ↓
             DocumentElement                 DocumentElement
                /       \                       /       \
               ↓         ↓                     ↓         ↓
        TextElement  ImageElement       TextElement  ImageElement
```

More simply:

```text
                         Client
                    /       |       \
                   ↓        ↓        ↓
          DocumentEditor  Persistence  DocumentRenderer
                |            |              |
                ↓            ↓              ↓
            Document     File / DB       Document
                |
                ↓
        DocumentElement
          /         \
         ↓           ↓
      Text          Image
```

---

# Principle of Least Knowledge

Also called the **Law of Demeter**.

> **A class should talk only to its immediate friends and should have minimal knowledge about the internal structure of other objects.**

In simple words:

> **Talk to your immediate friend, not your friend's friend.**

---

## In Our Design

`DocumentRenderer` directly knows about:

```text
Document
```

So:

```text
DocumentRenderer
       ↓
   Document
```

It asks the document for its elements:

```cpp
auto elements = document->getElements();
```

Then it interacts with those elements:

```text
DocumentRenderer
       ↓
   Document
       ↓
DocumentElement
```

The renderer doesn't need to know how `Document` internally stores its elements.

It shouldn't directly manipulate:

```cpp
document->elements;
```

Instead, it should use a public operation such as:

```cpp
document->getElements();
```

This reduces coupling.

---

# Final Responsibilities

| Class              | Responsibility                           |
| ------------------ | ---------------------------------------- |
| `Client`           | Creates and wires objects                |
| `DocumentEditor`   | Provides editing operations              |
| `Document`         | Stores document data/elements            |
| `DocumentElement`  | Common abstraction for document elements |
| `TextElement`      | Represents text                          |
| `ImageElement`     | Represents image                         |
| `DocumentRenderer` | Renders document                         |
| `Persistence`      | Saving abstraction                       |
| `SaveToFile`       | File storage                             |
| `SaveToDB`         | Database storage                         |

---

# SOLID Principles in This Design

### SRP

Responsibilities are separated:

```text
Document          → Data
DocumentEditor    → Editing
DocumentRenderer  → Rendering
Persistence       → Saving abstraction
SaveToFile        → File saving
SaveToDB          → DB saving
```

### OCP

We can add:

```text
VideoElement
TableElement
AudioElement

SaveToSQL
SaveToMongo
SaveToCloud
```

without modifying the existing abstraction.

### LSP

Any valid `DocumentElement` subtype can be used through:

```cpp
DocumentElement*
```

while preserving the expected `render()` behavior.

### ISP

ISP is about keeping interfaces **small and client-specific**. In this particular design, the main abstractions are already focused (`DocumentElement` and `Persistence`), but the diagram itself is not a strong example of ISP unless we have a larger interface that needs to be split.

### DIP

High-level modules depend on abstractions:

```text
DocumentEditor
      ↓
Persistence
      ↑
SaveToFile / SaveToDB
```

---

# Final Mental Model

```text
DocumentEditor
      ↓
   "I want to edit"

Document
      ↓
   "I store data"

DocumentRenderer
      ↓
   "I render"

Persistence
      ↓
   "I define saving"

SaveToFile / SaveToDB
      ↓
   "I actually save"
```

### Most Important Takeaway

> **Instead of creating one giant `DocumentEditor` that knows everything, we keep each responsibility separate and connect the components through abstractions.**

This gives us:

```text
SRP → Separate responsibilities
OCP → Easy extension
LSP → Safe substitution
DIP → Depend on abstractions
DI  → Inject concrete implementations
Law of Demeter → Minimize unnecessary knowledge
```
