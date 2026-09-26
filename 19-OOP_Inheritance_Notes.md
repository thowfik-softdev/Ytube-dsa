---
type: foundation
title: OOP II — Inheritance, Polymorphism, Abstraction and Interfaces
tags: [foundations, java, oop, inheritance, polymorphism, overriding, abstract, interfaces, super, pre-phase-0]
created: 2026-09-29
status: seen
source: "Kunal Kushwaha · Community Classroom — lecture 17 OOP (inheritance, polymorphism, abstract, interfaces)"
related: ["[[OOP I]]", "[[Methods]]", "[[OOP III]]", "[[Classes and Objects]]"]
---

# 19 · OOP II — Inheritance, Polymorphism & Interfaces

> 📁 Part 19 of 28 in [Ytube dsa/](README.md) · **Prev:** [18 — OOP I](18-OOP_Basics_Notes.md) · **Next:** [20 — OOP III](20-OOP_Advanced_Notes.md)

## Introduction
Three of the four OOP pillars, and they build on each other: **inheritance** lets one class reuse another; **polymorphism** lets one reference behave as many types; **abstraction** lets you define *what* without *how*.

The practical payoff is that you can write code against a general type (`Shape`) and have it work with any specific type (`Circle`, `Square`) — including ones written later.

---

## 1. Inheritance — `extends`

```java
class Box {
    int l, b, h;
}

class BoxWeight extends Box {       // BoxWeight IS-A Box
    double weight;                  // plus its own field
}
```

The subclass gets every non-private member of the parent, and adds its own. The test is **"is-a"**: a `BoxWeight` *is a* `Box`. If you can't say that sentence truthfully, inheritance is the wrong tool.

**`super` does two jobs:**

```java
class BoxWeight extends Box {
    double weight;
    BoxWeight(int l, int b, int h, double weight) {
        super(l, b, h);             // 1. call the PARENT constructor — must be FIRST
        this.weight = weight;
    }
    void show() {
        super.show();               // 2. call the parent's version of a method
    }
}
```

**Verified: the parent constructor always runs first**, whether you call `super(...)` explicitly or not:
```
  Circle ctor - super() ran first, implicitly
```
If you don't write `super(...)`, Java inserts a call to the parent's **no-arg** constructor. If the parent has none, that's a compile error — which is the real reason "always provide a no-arg constructor" is common advice.

**Types of inheritance Java supports:** single (A → B), multilevel (A → B → C), hierarchical (A → B, A → C).

> ⚠️ **Java has no multiple inheritance of classes** — `class C extends A, B` will not compile. The reason is the **diamond problem**: if both A and B define `show()`, which does C inherit? Java sidesteps it by allowing multiple **interfaces** instead (§5), where there's no state to conflict.

## 2. Polymorphism — One Reference, Many Forms

```java
Shape s = new Circle();      // parent reference, child object
s.area();
```
**Output**
```
  Circle.area (child) - dynamic dispatch
```

The **variable's type is `Shape`**, but the **object is a `Circle`**, and Java calls `Circle`'s version. The decision happens at **runtime**, based on the actual object — this is **dynamic dispatch** (or runtime polymorphism).

That's what makes this possible:
```java
Shape[] shapes = { new Circle(), new Square(), new Triangle() };
for (Shape shape : shapes) {
    shape.area();            // each calls its OWN version
}
```
One loop, three behaviours, and a fourth shape added later needs no change here.

**Overloading vs overriding** — the interview classic:

| | Overloading | Overriding |
|---|---|---|
| Where | same class | subclass |
| Signature | **different** parameters | **identical** signature |
| Resolved | **compile** time (note 06 §11 bytecode) | **runtime** |
| Return type | may differ | must match (or be a subtype) |
| Also called | static/compile-time polymorphism | dynamic/runtime polymorphism |

> 💡 **Always write `@Override`.** It's optional, but it makes the compiler verify that you really are overriding something. Misspell the method name or get a parameter type wrong and you've silently created an *overload* that never gets called — `@Override` turns that into a compile error.

**Rules for overriding:** the signature must match; access cannot be narrower (`public` can't become `private`); `final` and `static` methods can't be overridden; `private` methods aren't inherited so they can't be either.

## 3. Upcasting and Downcasting

```java
Shape s = new Circle();              // UPCAST — always safe, implicit
Circle c = (Circle) s;               // DOWNCAST — explicit, can fail
```

An upcast is safe because a `Circle` really *is* a `Shape`. A downcast asserts something the compiler can't verify, and a wrong one throws `ClassCastException` at runtime. Guard it:

```java
if (s instanceof Circle c) {         // pattern matching (Java 16+)
    c.area();                        // c is already typed — no cast needed
}
```
This is the same `instanceof` pattern from note 05 §9.

## 4. Abstraction — `abstract`

An **abstract class** defines *what* without *how*. It cannot be instantiated; subclasses must fill in the blanks.

```java
abstract class Animal {
    Animal() { System.out.println("Animal ctor runs even though abstract"); }
    abstract void sound();                       // no body — subclasses MUST implement
    void breathe() { System.out.println("concrete method in abstract class"); }
}

class Dog extends Animal {
    void sound() { System.out.println("Woof"); }
}
```
**Output**
```
  Animal ctor runs even though abstract
  Woof
  concrete method in abstract class
```

Three things that proves:

- **`new Animal()` is illegal**, but `Animal`'s constructor still runs as part of building a `Dog`. Abstract classes have constructors — they just can't be called directly.
- **Abstract classes can hold concrete methods** (`breathe`) and state, which is exactly what distinguishes them from interfaces.
- **A subclass must implement every abstract method**, or be declared abstract itself.

## 5. Interfaces — Pure Contract

```java
interface Engine {
    int CAP = 100;                               // implicitly public static final
    void start();                                // implicitly public abstract
    default void stop() {                        // default method (Java 8+)
        System.out.println("default method in interface");
    }
}

class Petrol implements Engine {
    public void start() { System.out.println("Petrol.start"); }
}
```
**Output**
```
  Petrol.start
  default method in interface
  interface field CAP = 100
```

Everything in an interface is implicitly `public`, methods are implicitly `abstract`, and fields are implicitly `public static final` — constants, not state.

**A class can implement many interfaces**, which is Java's answer to multiple inheritance:
```java
class NiceCar implements Engine, Media, Brake { }
```
Safe because interfaces (classically) carry **no state** — there's nothing to conflict, only method contracts.

**Java 8+ additions:**
- **`default` methods** — a body in an interface, so you can add methods to an interface without breaking every implementer.
- **`static` methods** — utilities attached to the interface.
- **`private` methods** (Java 9+) — shared helpers for default methods.

### Abstract class vs interface

| | Abstract class | Interface |
|---|---|---|
| Instantiable | ❌ | ❌ |
| Fields / state | ✅ any | only `public static final` constants |
| Constructors | ✅ | ❌ |
| Concrete methods | ✅ | only `default` / `static` |
| How many per class | **one** | **many** |
| Relationship | **is-a** | **can-do** |

**Choose by intent:** an abstract class for a genuine family sharing state and behaviour (`Animal` → `Dog`); an interface for a capability that unrelated types can have (`Comparable`, `Runnable`).

> 💡 If two interfaces provide the *same* default method, the implementing class is forced to override it and pick — so even with defaults, the diamond problem is resolved explicitly rather than silently.

## 6. Where This Shows Up in DSA

Less than you'd expect, but sharply:

- **`Comparable`** — implement `compareTo` so `Arrays.sort` can order your objects.
- **`Comparator`** — a separate ordering, passed to the sort. This is the one you'll use constantly for custom sorts and priority queues.
- **Polymorphic node types** — a generic `Node<T>` used across list, tree and graph code.
- **`toString` / `equals` / `hashCode`** — overriding `Object`'s methods (note 20).

```java
class Student implements Comparable<Student> {
    int marks;
    @Override public int compareTo(Student o) {
        return this.marks - o.marks;        // negative / zero / positive
    }
}
Arrays.sort(students);                       // now this works
```

> ⚠️ `this.marks - o.marks` can **overflow** for extreme values (note 05's silent wrap). `Integer.compare(this.marks, o.marks)` is the safe form and reads better.

---

## ⚠️ Common Misunderstandings
**1. Java supports multiple inheritance of classes.**
❌ supports · ✅ single only. Multiple **interfaces** are allowed, because they carry no state — the diamond problem.

**2. You must call `super(...)` explicitly.**
❌ must · ✅ if you don't, Java inserts a call to the parent's **no-arg** constructor. It's an error only when the parent has none.

**3. Overloading and overriding are similar.**
❌ similar · ✅ overloading = same class, different parameters, **compile-time**. Overriding = subclass, identical signature, **runtime**.

**4. `@Override` is decoration.**
❌ decoration · ✅ it makes the compiler verify you're actually overriding. Without it, a typo silently becomes an overload that never runs.

**5. Abstract classes can't have constructors.**
❌ can't · ✅ they can, and they run when a subclass is built. Verified: `Animal ctor runs even though abstract`.

**6. Interfaces can't have method bodies.**
❌ can't · ✅ since Java 8, `default` and `static` methods can. Fields are still constants only.

**7. Downcasting is always safe.**
❌ safe · ✅ `ClassCastException` at runtime if wrong. Guard with `instanceof`.

**8. The variable's type decides which method runs.**
❌ the variable · ✅ the **object** does. `Shape s = new Circle(); s.area()` calls `Circle.area` — dynamic dispatch.

**9. `compareTo` should return `a - b`.**
❌ always · ✅ that overflows for extreme values. Use `Integer.compare(a, b)`.

## JavaScript Comparison
| | Java | JavaScript |
|---|---|---|
| Inheritance | `extends`, single, class-based | `extends`, single, prototype-based |
| Parent call | `super(...)` first in constructor | `super(...)` before using `this` |
| Interfaces | ✅ built in | ❌ — TypeScript adds them |
| Abstract classes | ✅ `abstract` | ❌ — convention (throw in the base method) |
| Method resolution | dynamic dispatch on the object | prototype chain lookup |
| Multiple inheritance | interfaces only | mixins via `Object.assign` |
| Type checking | compile-time | none at runtime — duck typing |

> 💡 JS gets polymorphism free through duck typing — if it has the method, you can call it. Java requires the type relationship to be declared, which is more ceremony but catches mismatches at compile time.

## Interview Angles
- **"The four pillars?"** — Encapsulation (18), inheritance, polymorphism, abstraction. Define each in one sentence.
- **"Overloading vs overriding?"** — Nearly guaranteed. Same class/different params/compile-time vs subclass/same signature/runtime.
- **"Abstract class vs interface?"** — State and constructors vs pure contract; one vs many; is-a vs can-do. Then: "since Java 8 interfaces can have default methods, so the gap narrowed."
- **"Why no multiple inheritance?"** — The diamond problem; interfaces avoid it by carrying no state.
- **"What is dynamic dispatch?"** — The object, not the reference type, decides which override runs — resolved at runtime.
- **"Can you override a static method?"** — No. Declaring the same signature in a subclass **hides** it, and which one runs depends on the reference type, not the object. A good trap question.
- **"How do you sort custom objects?"** — `Comparable` for a natural order, `Comparator` for alternatives. Mention `Integer.compare` over subtraction.

## Related · Next
- **Related:** [[OOP I]] (18) · [[Methods]] (06 — overloading resolution) · [[OOP III]] (20) · [[Classes and Objects]] (0.7)
- **Practice:** build `Shape` (abstract) with `Circle`/`Square`/`Triangle`, loop over a `Shape[]` calling `area()`, and confirm each subclass's version runs. Then make `Student implements Comparable` and sort an array of them.
- **Next:** [20 — OOP III: Statics, Generics & Exceptions](20-OOP_Advanced_Notes.md)

---

## 🔁 Rapid Revision (self-test)
<details><summary>1. What's the test for whether inheritance is appropriate?</summary>

The **is-a** test: a `BoxWeight` *is a* `Box`. If that sentence isn't true, use composition instead.
</details>

<details><summary>2. What happens if you don't write `super(...)`?</summary>

Java inserts a call to the parent's **no-arg** constructor. The parent constructor always runs first — verified. It's a compile error only if the parent has no no-arg constructor.
</details>

<details><summary>3. Overloading vs overriding — four differences?</summary>

Same class vs subclass · different vs identical parameters · compile-time vs runtime · static vs dynamic polymorphism.
</details>

<details><summary>4. Why write `@Override`?</summary>

The compiler verifies you're actually overriding. Without it, a typo silently creates an overload that never gets called.
</details>

<details><summary>5. `Shape s = new Circle(); s.area();` — which runs and why?</summary>

`Circle.area()`. The **object**, not the reference type, decides — dynamic dispatch at runtime.
</details>

<details><summary>6. Can an abstract class have a constructor?</summary>

Yes, and it runs when a subclass is instantiated — verified. You just can't call `new Animal()` directly.
</details>

<details><summary>7. Three differences between abstract classes and interfaces?</summary>

State + constructors vs constants only · one vs many per class · is-a vs can-do. Since Java 8 interfaces also allow `default`/`static` bodies.
</details>

<details><summary>8. Why does Java forbid multiple class inheritance?</summary>

The diamond problem — ambiguity when two parents define the same member. Interfaces are allowed because they classically carry no state.
</details>

<details><summary>9. Can you override a static method?</summary>

No — you **hide** it. Which version runs depends on the reference type, not the object, so there's no dynamic dispatch.
</details>
