---
type: foundation
title: OOP I — Classes, Objects, Constructors, this, and Object Lifecycle
tags: [foundations, java, oop, classes, objects, constructors, this, final, packages, access-modifiers, pre-phase-0]
created: 2026-09-29
status: seen
source: "Kunal Kushwaha · Community Classroom — lecture 17 OOP (introduction, properties, packages, access)"
related: ["[[Methods]]", "[[Introduction to Programming]]", "[[Classes and Objects]]", "[[OOP II]]"]
---

# 18 · OOP I — Classes, Objects & Constructors

> 📁 Part 18 of 28 in [Ytube dsa/](README.md) · **Prev:** [17 — Math & Bitwise](17-Math_and_Bitwise_Notes.md) · **Next:** [19 — OOP II](19-OOP_Inheritance_Notes.md)

## Introduction
Note 01 defined an object as **code + data**. This is where that becomes concrete.

A **class** is a blueprint — it describes what data an object holds and what it can do. An **object** is one instance built from that blueprint, living on the heap. Everything in Java is organised this way, which is why `main` has to sit inside a class at all.

For DSA specifically, you need exactly enough OOP to write a `Node`, a `TreeNode`, and a "design this structure" class. That's this note.

## Characteristics
- **Class = blueprint, object = instance.** One class, unlimited objects.
- **Objects live on the heap**, and variables hold references to them (note 01).
- **Constructors initialise**, they don't create — `new` allocates, the constructor fills in.
- **`this` disambiguates** between a field and a parameter of the same name.

---

## 1. Class and Object

```java
public class Box {
    int length;              // instance variables — one copy PER OBJECT
    int breadth;
    int height;

    int volume() {           // instance method — operates on THIS object's data
        return length * breadth * height;
    }
}

Box b1 = new Box();          // b1 gets its own length/breadth/height
Box b2 = new Box();          // b2 has a completely separate set
```

`new Box()` does three things: allocates memory on the heap, runs the constructor, and returns a reference. `b1` and `b2` hold different addresses, so changing `b1.length` cannot affect `b2`.

> 💡 **Fields get defaults, locals don't.** An uninitialised `int` field is `0` and an object field is `null` (note 07's array defaults, same rule). An uninitialised *local variable* is a compile error (note 06 §6). Fields are zeroed during class loading (note 03 §5); locals are not.

## 2. Constructors

A constructor has **the same name as the class** and **no return type** — not even `void`.

```java
public class Box {
    int l, b, h;

    Box() {                              // no-arg constructor
        this(1, 1, 1);                   // delegate to the other one
    }

    Box(int l, int b, int h) {           // parameterised — overloading (note 06)
        this.l = l;                      // this.l = the FIELD; l = the PARAMETER
        this.b = b;
        this.h = h;
    }
}
```

**Verified execution order:**
```
  [static block] runs ONCE when the class loads
  [instance block] runs before EVERY constructor body
  Box(l,b,h)
  Box() no-arg, delegated via this()
```

Four things that output proves:

1. **Static block runs once**, at class load — before any object exists.
2. **Instance block runs before every constructor body**, on every `new`.
3. **`this(...)` delegates** to another constructor, and the delegate's body finishes *first*.
4. If you write **no** constructor, `javac` generates a default no-arg one (note 04 §15's phantom constructor). Write *any* constructor and that freebie disappears — which is why adding `Box(int,int,int)` breaks existing `new Box()` calls unless you add the no-arg one back.

> 💡 **`this` is a reference to the current object.** Its two uses: disambiguating `this.l = l`, and `this(...)` for constructor chaining. `this(...)` must be the **first statement** in the constructor.

## 3. `final`

```java
final int MAX = 100;            // a constant — cannot be reassigned
final Box b = new Box();
b.length = 5;                   // ALLOWED — the object is mutable
b = new Box();                  // ERROR — the reference is final
```

`final` freezes the **reference**, not the object. This is the same reference/object distinction from note 06 §4, and it trips up everyone once.

Applied to a class it means "cannot be extended" (`String` is `final`); to a method, "cannot be overridden".

## 4. Static — Class-Level, Not Object-Level

```java
public class Human {
    int age;                     // one per OBJECT
    static long population;      // ONE for the whole CLASS
}
```

`static` members belong to the class itself. Every object shares one copy, and you access them via the class name (`Human.population`) rather than an instance.

**The rule you already met in note 06 §10:** a static method cannot touch instance state, because it runs without an object:

```
error: non-static method helper() cannot be referenced from a static context
```

| | Instance | Static |
|---|---|---|
| Copies | one per object | one per class |
| Access | `obj.field` | `ClassName.field` |
| Can use `this` | ✅ | ❌ — there is no current object |
| Can call instance methods directly | ✅ | ❌ |
| Typical use | per-object data | counters, constants, utilities (`Math.max`) |

**Static blocks** initialise static state once, at class load:
```java
static { System.out.println("runs once when the class loads"); }
```

## 5. Access Modifiers

| Modifier | Same class | Same package | Subclass (other pkg) | Everywhere |
|---|---|---|---|---|
| `private` | ✅ | ❌ | ❌ | ❌ |
| *(default)* | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

**Encapsulation** is the practice: make fields `private`, expose `public` getters/setters, and you keep control over how state changes.

```java
public class Account {
    private double balance;                  // nobody can set this directly

    public double getBalance() { return balance; }

    public void deposit(double amount) {
        if (amount > 0) balance += amount;   // the class enforces its own rules
    }
}
```

Without `private`, any code could write `account.balance = -500` and no validation could stop it.

> 💡 In DSA you'll often see `public` fields on a `Node` class. That's deliberate — a `Node` is a plain data holder with no invariant to protect, and getters would just add noise. Encapsulation is a tool, not a ritual.

## 6. Packages

A **package** is a folder and a namespace (note 04 §4). Two classes with the same name can coexist in different packages:

```java
package com.kunal.a;
public class Greeting { }

package com.kunal.b;
public class Greeting { }        // completely different type
```

Import one, fully-qualify the other:
```java
import com.kunal.a.Greeting;
...
com.kunal.b.Greeting g2 = new com.kunal.b.Greeting();
```

## 7. Object Lifecycle and Garbage Collection

```
new Box()  ->  memory allocated on the heap
           ->  instance blocks run
           ->  constructor runs
           ->  reference returned
    ...used...
    no references remain  ->  eligible for garbage collection
```

From note 01: an object with no reachable reference is destroyed automatically. You never free memory.

> ⚠️ **`finalize()` is dead.** Older material (including some of this lecture's notes) shows overriding `finalize()` for cleanup. It was **deprecated in Java 9 and removed in Java 18** — it was unreliable, unpredictably timed, and could resurrect objects. Use try-with-resources or `Cleaner` instead. Don't learn `finalize`.

## 8. Wrapper Classes

Every primitive has an object counterpart, needed wherever objects are required — generics, collections (note 07 §9):

| Primitive | Wrapper |
|---|---|
| `int` | `Integer` |
| `char` | `Character` |
| `double` | `Double` |
| `boolean` | `Boolean` |

**Autoboxing** converts automatically:
```java
Integer a = 5;            // autoboxing: int -> Integer
int b = a;                // unboxing: Integer -> int
List<Integer> list = new ArrayList<>();
list.add(5);              // autoboxed
```

> ⚠️ **The `Integer` caching trap:**
> ```java
> Integer a = 127, b = 127;   // a == b  ->  true
> Integer c = 128, d = 128;   // c == d  ->  FALSE
> ```
> Java caches boxed `Integer` objects from −128 to 127, so small values share instances and `==` accidentally works. Above 127, new objects are created. **Always use `.equals()`** for wrappers — the same lesson as Strings in note 11, for the same reason.

---

## ⚠️ Common Misunderstandings
**1. A class is an object.**
❌ same thing · ✅ a class is the **blueprint**; an object is one **instance** with its own copy of the fields.

**2. Constructors create the object.**
❌ create · ✅ `new` allocates; the constructor **initialises**. It has no return type, not even `void`.

**3. You always get a free no-arg constructor.**
❌ always · ✅ only if you write **none**. Adding any constructor removes the default.

**4. `this(...)` can go anywhere in the constructor.**
❌ anywhere · ✅ it must be the **first statement**.

**5. `final` makes an object immutable.**
❌ immutable · ✅ it freezes the **reference**. `final Box b` still allows `b.length = 5`.

**6. Static methods can use instance fields.**
❌ can · ✅ `error: non-static ... cannot be referenced from a static context`. There is no `this`.

**7. `private` fields are hidden from the same class.**
❌ hidden · ✅ `private` is per **class**, not per object — one `Box` can read another `Box`'s privates.

**8. Use `finalize()` for cleanup.**
❌ use it · ✅ deprecated in Java 9, **removed in 18**. Use try-with-resources.

**9. `==` works on `Integer`.**
❌ works · ✅ only by accident for −128…127 (the cache). Use `.equals()`.

## JavaScript Comparison
| | Java | JavaScript |
|---|---|---|
| Class | required for all code | optional — objects can be literals |
| Fields | declared with types, get defaults | assigned in the constructor, no declaration needed |
| Constructor | same name as class, overloadable | `constructor()`, **one only** |
| Overloaded constructors | ✅ | ❌ — use default/rest parameters |
| `this` | the current instance, fixed | depends on **call site** (a notorious trap) |
| Private | `private` keyword | `#field` (ES2022) or closure convention |
| Static | `static` | `static` |
| Inheritance model | classes | prototypes (with `class` syntax over them) |

> 💡 The biggest adjustment: **Java's `this` is never ambiguous.** In JS, `this` depends on how a function is called, which is why you see `.bind(this)` and arrow functions everywhere. In Java `this` always means "the object this method was called on", full stop.

## Interview Angles
- **"What is a class vs an object?"** — Blueprint vs instance. Expect it as a warm-up.
- **"What are the four OOP pillars?"** — Encapsulation, inheritance, polymorphism, abstraction. (The last three are note 19.)
- **"What is encapsulation and why?"** — Private fields + public methods, so the class controls how its state changes and can validate.
- **"Static vs instance?"** — One copy per class vs per object; statics have no `this`.
- **"Can a constructor be private?"** — Yes — that's how singletons and factory methods work (note 20).
- **"What does `final` mean in each position?"** — Variable: can't reassign. Method: can't override. Class: can't extend.
- **"Why is `Integer a = 128; Integer b = 128; a == b` false?"** — The −128…127 cache. A favourite trick question.

## Related · Next
- **Related:** [[Methods]] (06 — static, `this`, stack/heap) · [[Introduction to Programming]] (01 — objects as code+data) · [[Classes and Objects]] (0.7) · [[OOP II]] (19)
- **Practice:** write a `BankAccount` with a private balance, a validating `deposit`, and two constructors chained with `this(...)`. Then write the `Node` class you'll need for linked lists — deliberately with public fields, and be ready to justify it.
- **Next:** [19 — OOP II: Inheritance, Polymorphism & Interfaces](19-OOP_Inheritance_Notes.md)

---

## 🔁 Rapid Revision (self-test)
<details><summary>1. Class vs object, and where does each live?</summary>

A class is the blueprint; an object is one instance. Objects live on the **heap**; the variable holds a reference on the stack.
</details>

<details><summary>2. What does `new Box()` actually do?</summary>

Allocates heap memory, runs instance initialiser blocks, runs the constructor, returns a reference.
</details>

<details><summary>3. The verified order of static block, instance block, and constructor?</summary>

Static block **once** at class load; then per object: instance block, then the constructor body. With `this(...)`, the delegated constructor's body completes first.
</details>

<details><summary>4. When do you lose the free no-arg constructor?</summary>

As soon as you declare **any** constructor yourself.
</details>

<details><summary>5. What exactly does `final` freeze on an object reference?</summary>

The **reference**. `final Box b` still allows `b.length = 5`; it only forbids `b = new Box()`.
</details>

<details><summary>6. Why can't a static method use instance fields?</summary>

It runs without any object, so there is no `this` to read them from — `error: non-static method cannot be referenced from a static context`.
</details>

<details><summary>7. The four access modifiers, weakest to strongest?</summary>

`private` (class only) → default (package) → `protected` (package + subclasses) → `public` (everywhere).
</details>

<details><summary>8. Why is `finalize()` not worth learning?</summary>

Deprecated in Java 9, **removed in Java 18** — unreliable and unpredictably timed. Use try-with-resources or `Cleaner`.
</details>

<details><summary>9. Why does `Integer a = 128, b = 128; a == b` return false?</summary>

Java caches boxed Integers from −128 to 127. Inside that range `==` accidentally works; outside it, separate objects. Use `.equals()`.
</details>
