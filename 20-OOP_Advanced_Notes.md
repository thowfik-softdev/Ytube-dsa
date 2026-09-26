---
type: foundation
title: OOP III — Generics, Exceptions, Object Methods, Singleton & Enums
tags: [foundations, java, oop, generics, exceptions, equals, hashcode, singleton, enum, collections, pre-phase-0]
created: 2026-09-29
status: seen
source: "Kunal Kushwaha · Community Classroom — lecture 17 OOP (generics, exceptions, singleton, enums, collections)"
related: ["[[OOP II]]", "[[Arrays]]", "[[Hashing]]", "[[OOP I]]"]
---

# 20 · OOP III — Generics, Exceptions & Object Methods

> 📁 Part 20 of 28 in [Ytube dsa/](README.md) · **Prev:** [19 — OOP II](19-OOP_Inheritance_Notes.md) · **Next:** [21 — Linked List](21-LinkedList_Notes.md)

## Introduction
The parts of OOP you'll actually touch while doing DSA: **generics** (so your data structures work with any type), **exceptions** (so they fail informatively), and the **`Object` methods** — `equals`, `hashCode`, `toString` — which decide whether your objects work correctly inside collections.

---

## 1. Generics — Type-Safe Containers

You met `ArrayList<Integer>` in note 07. Writing your own generic class is the same idea from the other side.

```java
public class CustomGenArrayList<T> {
    private Object[] data = new Object[10];
    private int size = 0;

    public void add(T value) {
        data[size++] = value;
    }

    @SuppressWarnings("unchecked")
    public T get(int index) {
        return (T) data[index];       // the cast the compiler inserts for you
    }
}
```

`<T>` is a **type parameter** — a placeholder filled in at use:
```java
CustomGenArrayList<Integer> list = new CustomGenArrayList<>();
list.add(5);
int x = list.get(0);        // no cast needed — the compiler knows it's an Integer
```

**Why generics exist:** before them, collections held `Object` and every read needed a cast that could fail at runtime. Generics move that failure to **compile time**.

**Bounded types** constrain what `T` can be:
```java
public class NumberBox<T extends Number> { }     // T must be Number or a subclass
```

**Wildcards** describe flexibility at the use site:
```java
void print(List<?> list) { }                     // any type
void sum(List<? extends Number> list) { }        // Number or subtype — read-only
void fill(List<? super Integer> list) { }        // Integer or supertype — write-safe
```

> ⚠️ **Type erasure.** Generics are a *compile-time* feature — at runtime the type parameter is erased to `Object`. So `List<String>` and `List<Integer>` are the **same class** at runtime, you cannot write `new T[10]`, and `instanceof List<String>` won't compile. This is also why generics avoid array covariance's runtime hole (note 07 §12): errors surface during compilation instead.

## 2. Exceptions

An exception is an object describing something that went wrong.

```java
try {
    int[] arr = new int[2];
    arr[5] = 1;
} catch (ArrayIndexOutOfBoundsException ex) {
    System.out.println("caught: " + ex.getMessage());
} finally {
    System.out.println("finally ALWAYS runs");
}
```
**Output**
```
  caught: Index 5 out of bounds for length 2
  finally ALWAYS runs
```

### The hierarchy

```
Throwable
├── Error              — JVM-level, don't catch (StackOverflowError, OutOfMemoryError)
└── Exception
    ├── RuntimeException   — UNCHECKED (NullPointer, ArrayIndexOutOfBounds, …)
    └── everything else    — CHECKED (IOException, …) — must be declared or caught
```

**Checked** exceptions must be handled or declared with `throws`; the compiler enforces it. **Unchecked** (anything under `RuntimeException`) need not be — they usually signal bugs rather than recoverable conditions.

`StackOverflowError` from note 14 is an **`Error`**, which is why you don't catch it in real code.

### Custom exceptions

The stack lecture (note 22) defines one:
```java
public class StackException extends Exception {
    public StackException(String message) { super(message); }
}
```
and throws it:
```java
public int pop() throws StackException {
    if (isEmpty()) {
        throw new StackException("Cannot pop from an empty stack!!");
    }
    return data[ptr--];
}
```
Extending `Exception` makes it **checked** — callers are forced to deal with an empty stack. Extending `RuntimeException` would make it unchecked. Choosing between them is a real design decision: checked for conditions a caller can reasonably recover from, unchecked for programming errors.

> ⚠️ **`finally` runs, but it can't change an already-computed return value.** Verified:
> ```java
> static int tricky() { int x = 1; try { return x; } finally { x = 99; } }
> ```
> ```
> 1  <- finally ran, but return value was already fixed
> ```
> `return x` evaluates `x` (1) *before* `finally` runs. Assigning to `x` afterwards changes the variable, not the already-captured result. A `return` **inside** `finally` would override it — which is exactly why that's considered bad practice.

**try-with-resources** closes things automatically:
```java
try (Scanner sc = new Scanner(new File("input.txt"))) {
    // use sc
}   // sc.close() runs automatically, even on exception
```
This is the modern replacement for `finally { x.close(); }` — and for `finalize()` (note 18 §7).

## 3. The `Object` Methods

Every class silently extends `Object`, inheriting `toString`, `equals`, `hashCode` and others. The defaults are rarely what you want.

```java
class Point {
    int x, y;
    @Override public boolean equals(Object o) {
        if (this == o) return true;                    // same object — fast path
        if (!(o instanceof Point)) return false;       // null-safe type check
        Point p = (Point) o;
        return x == p.x && y == p.y;                   // compare the STATE
    }
    @Override public int hashCode() { return Objects.hash(x, y); }
    @Override public String toString() { return "Point(" + x + "," + y + ")"; }
}
```
**Output**
```
  p1 == p2      : false
  p1.equals(p2) : true
  HashSet size after adding both: 1  (1 because equals+hashCode agree)
  toString: Point(1,2)
```

Three lessons in that output:

- **`==` is reference equality** — the same rule as Strings (note 11) and arrays (note 07).
- **`equals` compares state** once you override it. Without the override you'd get `Object`'s version, which is just `==`.
- **The `HashSet` collapsed two equal points into one entry** — *because* `hashCode` was overridden to match.

> ⚠️ **The `equals`/`hashCode` contract is the single most important rule here:** if two objects are `equals`, they **must** have the same `hashCode`. Override one without the other and `HashMap`/`HashSet` break silently — equal objects land in different buckets, so lookups miss and duplicates appear. The set would have shown size **2** instead of 1.

`toString()` is called automatically by `System.out.println(obj)` and by string concatenation (note 11 §3's `"Kunal" + new ArrayList<>()` → `Kunal[]`). Overriding it makes debugging vastly easier.

## 4. Singleton

A class with exactly one instance:

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() { }                     // private ctor — nobody else can `new` it

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

The **private constructor** is the mechanism — it's the answer to note 18's "can a constructor be private?". A private constructor also means the class **cannot be subclassed**, since a subclass constructor couldn't call `super()`.

> ⚠️ This version is **not thread-safe** — two threads can both see `instance == null` and create two objects. Real fixes: an `enum` singleton (simplest and serialisation-safe), a static holder class, or double-checked locking with `volatile`.

## 5. Enums

A fixed set of named constants — and in Java, a full class.

```java
enum Light { RED, AMBER, GREEN }
```

Each constant is a `public static final` instance of the enum type. Because the set is closed, the compiler knows every possibility — which is why a `switch` over an enum needs no `default` (note 05 §7, verified).

Enums can carry state and behaviour:
```java
enum Planet {
    MERCURY(3.303e+23), EARTH(5.976e+24);
    private final double mass;
    Planet(double mass) { this.mass = mass; }
    double getMass() { return mass; }
}
```

Useful methods: `values()` (all constants), `ordinal()` (position), `name()`, and `valueOf(String)`.

## 6. The Collections Framework — the Map

You've used `ArrayList` (note 07). The wider hierarchy:

```
Collection
├── List      — ordered, duplicates OK       → ArrayList, LinkedList
├── Set       — no duplicates                → HashSet, LinkedHashSet, TreeSet
└── Queue     — FIFO                         → ArrayDeque, PriorityQueue

Map (separate)  — key → value                → HashMap, LinkedHashMap, TreeMap
```

| Need | Use | Lookup |
|---|---|---|
| Indexed, growable list | `ArrayList` | O(1) by index |
| Frequent insert/delete at ends | `ArrayDeque` / `LinkedList` | O(1) at ends |
| Uniqueness, order irrelevant | `HashSet` | O(1) avg |
| Uniqueness, sorted | `TreeSet` | O(log n) |
| Key → value | `HashMap` | O(1) avg |
| Key → value, sorted by key | `TreeMap` | O(log n) |
| Always pull the min/max | `PriorityQueue` | O(log n) |

**These are the DSA workhorses.** `HashMap` and `HashSet` turn O(n²) brute force into O(n) constantly — and both depend entirely on `equals`/`hashCode` being correct (§3).

---

## ⚠️ Common Misunderstandings
**1. Generics exist at runtime.**
❌ exist · ✅ **erased** to `Object`. `List<String>` and `List<Integer>` are the same class at runtime; `new T[10]` is illegal.

**2. You can catch any problem.**
❌ any · ✅ `Error` (StackOverflow, OutOfMemory) signals JVM-level failure — don't catch it in real code.

**3. Checked vs unchecked is a style choice.**
❌ style · ✅ checked (extends `Exception`) is **compiler-enforced**; unchecked (extends `RuntimeException`) is not.

**4. `finally` can change the returned value.**
❌ can · ✅ verified `1`, not `99` — the return value is captured before `finally` runs. Only a `return` inside `finally` overrides it, which you shouldn't write.

**5. Overriding `equals` is enough.**
❌ enough · ✅ you **must** override `hashCode` too, or hash-based collections break silently. The verified `HashSet` size of 1 depends on both.

**6. `==` compares object contents.**
❌ contents · ✅ references — same as Strings and arrays. Use `.equals()`.

**7. The basic singleton is thread-safe.**
❌ safe · ✅ two threads can both pass the null check. Use an enum, a holder class, or double-checked locking.

**8. An enum is just a list of constants.**
❌ just constants · ✅ a full class — it can have fields, constructors and methods, and it makes `switch` exhaustive without `default`.

## JavaScript Comparison
| | Java | JavaScript |
|---|---|---|
| Generics | compile-time, erased | none (TypeScript has them) |
| Checked exceptions | ✅ compiler-enforced | ❌ — all are unchecked |
| `finally` | ✅ same semantics | ✅ |
| Custom errors | `extends Exception` | `extends Error` |
| Value equality | `equals()` you write | `===` deep-compares nothing; you write it too |
| Hash-based collections | `HashMap` needs `hashCode` | `Map` uses reference identity for objects |
| Enum | `enum` | frozen object or union type (TS) |

> 💡 JS `Map` keyed by an object uses **reference identity** with no way to customise it — two structurally equal objects are always different keys. Java lets you define equality via `equals`/`hashCode`, which is more power and more responsibility.

## Interview Angles
- **"Why generics?"** — Compile-time type safety and no casts. Follow-up: type erasure, and what it prevents (`new T[]`).
- **"Checked vs unchecked exceptions?"** — Compiler-enforced vs not; recoverable conditions vs programming errors.
- **"The equals/hashCode contract?"** — Equal objects must have equal hash codes. **Asked constantly**, because breaking it corrupts `HashMap` silently.
- **"What happens if you override `equals` but not `hashCode`?"** — Equal objects land in different buckets; lookups fail and duplicates appear in a `HashSet`.
- **"Implement a singleton."** — Private constructor + static instance. Then: "the naive version isn't thread-safe" — say it before they ask.
- **"When would you use a `TreeMap` over a `HashMap`?"** — When you need keys in sorted order or range queries; O(log n) instead of O(1).
- **"Does `finally` always run?"** — Yes, except on `System.exit` or JVM crash. And it cannot alter an already-computed return value.

## Related · Next
- **Related:** [[OOP II]] (19) · [[Arrays]] (07 — `ArrayList`, covariance) · [[Hashing]] (3.4) · [[OOP I]] (18)
- **Practice:** write a generic `Pair<K, V>`, then a `Point` with correct `equals`/`hashCode` and prove it by putting duplicates in a `HashSet`. Then deliberately delete `hashCode` and watch the set size change.
- **Next:** [21 — Linked List](21-LinkedList_Notes.md)

---

## 🔁 Rapid Revision (self-test)
<details><summary>1. What problem do generics solve, and what is type erasure?</summary>

They move container type errors from runtime to compile time and remove casts. Erasure means the type parameter is gone at runtime — `List<String>` and `List<Integer>` are the same class, and `new T[10]` is illegal.
</details>

<details><summary>2. Checked vs unchecked exceptions?</summary>

Checked (extends `Exception`) must be caught or declared — compiler-enforced. Unchecked (extends `RuntimeException`) need not be; they usually indicate bugs.
</details>

<details><summary>3. Why shouldn't you catch `Error`?</summary>

`Error` is JVM-level failure — `StackOverflowError`, `OutOfMemoryError`. Not recoverable in normal code.
</details>

<details><summary>4. Does `finally` change the return value?</summary>

No. Verified: `try { return x; } finally { x = 99; }` returns **1** — the value was captured before `finally` ran.
</details>

<details><summary>5. The equals/hashCode contract, and what breaks without it?</summary>

Equal objects must have equal hash codes. Override `equals` alone and hash collections break silently — equal objects land in different buckets, so the verified `HashSet` would hold 2 entries instead of 1.
</details>

<details><summary>6. What makes a singleton work, and what's wrong with the simple version?</summary>

A **private constructor** (so nobody else can `new` it) plus a static accessor. The naive version isn't thread-safe — two threads can both pass the null check.
</details>

<details><summary>7. Why does a switch over an enum not need `default`?</summary>

The set of constants is closed, so the compiler can verify every case is covered — exhaustiveness without a catch-all.
</details>

<details><summary>8. Which collection for: uniqueness · key→value sorted · always pull the smallest?</summary>

`HashSet` · `TreeMap` · `PriorityQueue`.
</details>
