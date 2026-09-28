---
type: foundation
title: Large Numbers & File Handling — BigInteger, BigDecimal and Java I/O
tags: [foundations, java, biginteger, bigdecimal, overflow, file-handling, io, scanner, bufferedreader, pre-phase-0]
created: 2026-09-29
status: seen
source: "Kunal Kushwaha · Community Classroom — lecture 22 Large Numbers, lecture 23 File Handling"
related: ["[[First Java Program]]", "[[Conditionals and Loops]]", "[[OOP III]]"]
---

# 27 · Large Numbers & File Handling

> 📁 Part 27 of 28 in [Ytube dsa/](README.md) · **Prev:** [26 — Advanced Sorting](26-Advanced_Sorting_Notes.md) · **Next:** [28 — Git & GitHub](28-Git_and_GitHub_Notes.md)

## Introduction
Two practical topics that have haunted earlier notes.

**Overflow** has bitten us repeatedly: `Fibonacci(47)` went negative (note 05), reversing `1534236469` produced garbage (note 05), `(start + end) / 2` was a nine-year JDK bug (note 09), and `a * b / gcd` can overflow (note 17). `BigInteger` is the answer when the numbers genuinely don't fit.

**File handling** is how a program talks to the world beyond `System.in`.

---

# Part A — Large Numbers

## 1. The Ceiling

| Type | Max value |
|---|---|
| `int` | 2,147,483,647 (~2.1 × 10⁹) |
| `long` | 9,223,372,036,854,775,807 (~9.2 × 10¹⁸) |

And Java **wraps silently** past those — no exception, just a wrong answer. Verified across earlier notes:

```
Fibonacci(47) with int  ->  -1323752223
F(93) with long         ->  -6246583658587674878
reverse(1534236469)     ->  1056389759   (true answer 9646324351)
```

`20!` already exceeds `long`. `100!` has 158 digits. Cryptography routinely uses 2048-bit numbers. For all of these, primitives are simply the wrong tool.

## 2. `BigInteger`

Arbitrary precision — limited only by memory.

```java
import java.math.BigInteger;

BigInteger a = new BigInteger("123456789012345678901234567890");
BigInteger b = BigInteger.valueOf(12345);          // from a long — preferred when it fits

BigInteger sum  = a.add(b);
BigInteger prod = a.multiply(b);
BigInteger diff = a.subtract(b);
BigInteger quot = a.divide(b);
BigInteger rem   = a.mod(b);
BigInteger p    = a.pow(3);
```

**Factorial of 100 — impossible with `long`:**
```java
static BigInteger factorial(int n) {
    BigInteger result = BigInteger.ONE;
    for (int i = 2; i <= n; i++) {
        result = result.multiply(BigInteger.valueOf(i));
    }
    return result;
}
```

Three things that trip people up:

- **No operators.** `a + b` does not compile — you must call `.add(b)`. There's no operator overloading in Java.
- **Immutable.** `a.add(b)` returns a *new* object; `a` is unchanged. Exactly like `String` (note 11 §1), and the same mistake: `a.add(b);` on its own line does nothing. You must assign.
- **Constants:** `BigInteger.ZERO`, `ONE`, `TWO`, `TEN` — use them instead of constructing.

**Comparison** uses `compareTo`, not `<`:
```java
if (a.compareTo(b) > 0) { }        // a > b
if (a.equals(b)) { }               // equality — NOT ==
```

> ⚠️ **`BigInteger` is much slower** — every operation allocates, and there's no CPU instruction for it. Use it only when values genuinely exceed `long`. For DSA, prefer restructuring to avoid overflow (modular arithmetic, `(end - start) / 2`) over reaching for `BigInteger`.

## 3. `BigDecimal` — for Money

```java
System.out.println(0.1 + 0.2);      // 0.30000000000000004
```

That's not a Java bug — it's IEEE 754 binary floating point, identical in JavaScript and almost every language. Some decimals simply have no exact binary representation.

```java
BigDecimal x = new BigDecimal("0.1");
BigDecimal y = new BigDecimal("0.2");
System.out.println(x.add(y));       // 0.3 — exact
```

> ⚠️ **Always construct `BigDecimal` from a `String`.** `new BigDecimal(0.1)` takes a `double` that is *already* imprecise and faithfully preserves the error — `0.1000000000000000055511151231257827…`. The `String` constructor is the only one that means what you wrote.

**Never use `double` for money.** Use `BigDecimal`, or store integer cents. Also use `compareTo` rather than `equals` for value comparison — `equals` compares scale too, so `2.0` and `2.00` are *not* equal.

---

# Part B — File Handling

## 4. Reading

```java
import java.io.*;
import java.util.Scanner;

// Scanner — same API as keyboard input (note 04)
try (Scanner sc = new Scanner(new File("input.txt"))) {
    while (sc.hasNextLine()) {
        System.out.println(sc.nextLine());
    }
}

// BufferedReader — faster for bulk reading
try (BufferedReader br = new BufferedReader(new FileReader("input.txt"))) {
    String line;
    while ((line = br.readLine()) != null) {      // null marks end of file
        System.out.println(line);
    }
}

// Files (Java 8+) — the modern, concise way
List<String> lines = Files.readAllLines(Path.of("input.txt"));
```

| | `Scanner` | `BufferedReader` | `Files` |
|---|---|---|---|
| Parses types | ✅ `nextInt()` etc. | ❌ strings only | ❌ |
| Speed | slower | **fast** | fast |
| Best for | mixed parsed input | large files, line-by-line | small files, one call |

> 💡 **`BufferedReader` is the competitive-programming default** — `Scanner` parses with regex, which is meaningfully slower on large inputs. If a solution times out on I/O, this is usually why.

> 💡 `while ((line = br.readLine()) != null)` assigns *and* tests in one expression. It looks odd, but it's the standard idiom — the alternative reads the first line twice.

## 5. Writing

```java
try (BufferedWriter bw = new BufferedWriter(new FileWriter("output.txt"))) {
    bw.write("Hello");
    bw.newLine();
}

try (PrintWriter pw = new PrintWriter("output.txt")) {
    pw.printf("Name: %s, Age: %d%n", "Kunal", 21);     // printf, like note 11
}

Files.write(Path.of("out.txt"), List.of("line 1", "line 2"));
```

`new FileWriter("f.txt", true)` appends instead of overwriting — the second argument is easy to miss and the difference between adding a line and destroying a file.

## 6. try-with-resources

```java
try (Scanner sc = new Scanner(new File("input.txt"))) {
    // use it
}   // sc.close() runs automatically — even if an exception is thrown
```

**Always use this for files.** Anything implementing `AutoCloseable` is closed automatically, in reverse order of opening, even on an exception path.

The old `finally { sc.close(); }` form works but is verbose and easy to get wrong. And it's the proper replacement for `finalize()` (note 18 §7), which was removed in Java 18.

> ⚠️ **Unclosed files leak OS handles.** The operating system limits how many a process may hold; leak enough and further opens fail. Unlike memory, the garbage collector will not save you here.

## 7. Checked Exceptions Return

File operations throw **checked** exceptions (note 20 §2) — the compiler forces you to handle them:

```java
public static void main(String[] args) throws IOException {   // declare it
    // ...
}

// or catch it
try {
    Files.readAllLines(Path.of("missing.txt"));
} catch (IOException e) {
    System.out.println("Could not read: " + e.getMessage());
}
```

`FileNotFoundException` extends `IOException`, so catching the latter covers both. This is the clearest example of *why* checked exceptions exist: a missing file is a perfectly foreseeable condition that a caller can genuinely recover from.

---

## ⚠️ Common Misunderstandings
**1. `long` is big enough.**
❌ enough · ✅ `20!` already overflows it, and it wraps **silently**. Verified: `F(93)` came out negative.

**2. `a + b` works on BigInteger.**
❌ works · ✅ Java has no operator overloading — use `.add()`, `.multiply()`, `.compareTo()`.

**3. `a.add(b);` modifies `a`.**
❌ modifies · ✅ `BigInteger` is **immutable** — it returns a new object. Assign the result, exactly as with `String`.

**4. `0.1 + 0.2 == 0.3`.**
❌ equal · ✅ `0.30000000000000004`. IEEE 754, not a Java quirk — JS does the same.

**5. `new BigDecimal(0.1)` is precise.**
❌ precise · ✅ it inherits the `double`'s existing error. Use the **String** constructor.

**6. `BigDecimal.equals` compares values.**
❌ values · ✅ it compares value **and scale** — `2.0` ≠ `2.00`. Use `compareTo`.

**7. `Scanner` is fine for large files.**
❌ fine · ✅ regex parsing makes it slow. Use `BufferedReader` for bulk input.

**8. The garbage collector closes files.**
❌ closes · ✅ it frees memory, not OS handles. Use try-with-resources.

**9. `new FileWriter(path)` appends.**
❌ appends · ✅ it **overwrites**. Pass `true` as the second argument to append.

## JavaScript Comparison
| | Java | JavaScript |
|---|---|---|
| Arbitrary integers | `BigInteger` | `BigInt` — with real operators (`a + b`) |
| Decimal money | `BigDecimal` | no built-in — use a library |
| `0.1 + 0.2` | `0.30000000000000004` | **identical** |
| Read a file | `Files.readAllLines` | `fs.readFileSync` / `fs.promises.readFile` |
| Auto-close | try-with-resources | no equivalent (streams are managed) |
| Checked exceptions | ✅ forced | ❌ none |

> 💡 JS's `BigInt` supports normal operators (`10n ** 100n`), which is nicer than Java's method calls — but it cannot be mixed with `Number` without an explicit conversion. Both languages share the `0.1 + 0.2` problem exactly, because both use IEEE 754.

## Interview Angles
- **"How would you handle numbers larger than `long`?"** — `BigInteger`. Mention it's immutable, method-based, and slower.
- **"Why not use `double` for money?"** — Binary floating point can't represent many decimals exactly. `BigDecimal` from a String, or integer cents.
- **"Why is `0.1 + 0.2 != 0.3`?"** — IEEE 754. A classic, and it's language-independent.
- **"`Scanner` vs `BufferedReader`?"** — Parsing convenience vs speed. The competitive-programming answer is `BufferedReader`.
- **"How do you ensure a file gets closed?"** — try-with-resources; note that GC doesn't release OS handles.
- **In DSA specifically:** overflow awareness matters far more than `BigInteger` itself. Interviewers want to hear *"this could overflow, so I'd use `long`"* — restructuring beats big-number types.

## Related · Next
- **Related:** [[First Java Program]] (04 — primitives and their limits) · [[Conditionals and Loops]] (05 — the overflow bugs) · [[OOP III]] (20 — checked exceptions)
- **Practice:** compute `100!` with `BigInteger`, then print `0.1 + 0.2` as a `double` and as a `BigDecimal` from Strings. Then read a file with `BufferedReader` inside try-with-resources.
- **Next:** [28 — Git & GitHub](28-Git_and_GitHub_Notes.md)

---

## 🔁 Rapid Revision (self-test)
<details><summary>1. What happens when an int or long overflows in Java?</summary>

It **wraps silently** — no exception. Verified: `Fibonacci(47)` → −1323752223 and `F(93)` as a long → negative.
</details>

<details><summary>2. Three things to know about BigInteger?</summary>

No operators (use `.add()`), **immutable** (assign the result), and much slower than primitives.
</details>

<details><summary>3. Why is `0.1 + 0.2` not `0.3`?</summary>

IEEE 754 binary floating point can't represent those decimals exactly. It's identical in JavaScript — not a Java bug.
</details>

<details><summary>4. Why must BigDecimal be constructed from a String?</summary>

`new BigDecimal(0.1)` receives a `double` that is already imprecise and preserves that error. The String constructor takes the literal digits.
</details>

<details><summary>5. Why use `compareTo` rather than `equals` on BigDecimal?</summary>

`equals` compares value **and scale**, so `2.0` and `2.00` are unequal. `compareTo` compares numeric value only.
</details>

<details><summary>6. Scanner vs BufferedReader for files?</summary>

`Scanner` parses types but is slow (regex). `BufferedReader` reads lines fast — the default for large input.
</details>

<details><summary>7. What does try-with-resources do, and why does it matter for files?</summary>

Closes any `AutoCloseable` automatically, even on exception. Files hold **OS handles** that the garbage collector will not release.
</details>

<details><summary>8. What does `new FileWriter("f.txt")` do to an existing file?</summary>

**Overwrites** it. Pass `true` as the second argument to append instead.
</details>
