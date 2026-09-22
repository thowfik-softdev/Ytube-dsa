---
type: foundation
title: Strings & StringBuilder — Immutability, the String Pool, Concatenation Costs & the Methods That Matter
tags: [foundations, java, strings, immutability, string-pool, stringbuilder, stringbuffer, performance, pre-phase-0]
created: 2026-09-29
status: seen
source: "Kunal Kushwaha · Community Classroom — lecture 12 Strings, lecture 21 StringBuffer"
related: ["[[Arrays]]", "[[Conditionals and Loops]]", "[[Strings]]", "[[Two Pointers]]"]
---

# 11 · Strings & StringBuilder

> 📁 Part 11 of 28 in [Ytube dsa/](README.md) · **Prev:** [10 — Sorting](10-Sorting_Notes.md) · **Next:** [12 — Patterns](12-Patterns_Notes.md)

## Introduction
A `String` in Java is an **object**, and — unlike almost everything else — it is **immutable**: once created, its characters can never change. Every "modification" silently creates a *new* String.

That single fact explains the String pool, why `==` sometimes works and sometimes doesn't, why `+` in a loop is a performance trap, and why `StringBuilder` exists. Get immutability and the rest follows.

## Characteristics
- **Immutable:** no method changes a String; they all return a new one.
- **Pooled:** identical *literals* share one object in the String pool.
- **An object, not a primitive:** but with literal syntax and `+` support, which makes it feel primitive.
- **Indexed like an array:** `charAt(i)`, `length()` — but you cannot assign to a position.

---

## 1. Immutability — the Foundation

```java
String name = "Kunal Kushwaha Hello World";
System.out.println(name.toLowerCase());
System.out.println(name);
```
**Output**
```
kunal kushwaha hello world
Kunal Kushwaha Hello World
```

`toLowerCase()` did **not** change `name` — it returned a new String, which we printed and discarded. The original is untouched. Every String method behaves this way; to keep a result you must assign it:

```java
name = name.toLowerCase();     // reassign the variable to the new object
```

Reassignment is not mutation — it re-points the variable while the old object stays as it was (and becomes garbage if nothing else references it):

```java
String a = "Kunal";
System.out.println(a);
a = "Kushwaha";
System.out.println(a);
```
```
Kunal
Kushwaha
```

**Why immutable?** Safety: Strings are used as `HashMap` keys, in security checks, and shared across threads. If one holder could mutate a String, every other holder's value would change under them. Immutability also makes the pool possible — sharing is only safe when nothing can change.

## 2. The String Pool and `==`

```java
String name1 = new String("Kunal");
String name2 = new String("Kunal");
System.out.println(name1 == name2);
System.out.println(name1.equals(name2));
```
**Output**
```
false
true
```

But with **literals**, `==` returns `true` (verified in note 05):

```java
String a = "Kunal";
String b = "Kunal";        //  a == b  ->  true
```

```mermaid
flowchart TD
    subgraph POOL["String pool (inside the heap)"]
        P["\"Kunal\""]
    end
    subgraph HEAP["heap (regular objects)"]
        O1["new String(\"Kunal\")"]
        O2["new String(\"Kunal\")"]
    end
    A["a"] --> P
    B["b"] --> P
    N1["name1"] --> O1
    N2["name2"] --> O2
```

Java keeps one copy of each distinct **literal** in the **String pool** and points every occurrence at it — safe precisely because Strings can't change. `new String(...)` explicitly bypasses the pool and forces a fresh object.

> ⚠️ **Never use `==` on Strings.** It compares references, so it accidentally works for literals and fails for anything built at runtime — from `Scanner`, concatenation, or a file. Always `.equals()` (or `.equalsIgnoreCase()`). This is the single most common Java bug for newcomers.

## 3. `+` on Strings — Not What You Expect

```java
System.out.println('a' + 'b');
System.out.println("a" + "b");
System.out.println((char)('a' + 3));
System.out.println("a" + 1);
System.out.println("Kunal" + new ArrayList<>());
System.out.println("Kunal" + new Integer(56));
System.out.println("a" + 'b');
```
**Output**
```
195
ab
d
a1
Kunal[]
Kunal56
ab
```

Line by line:

| Expression | Result | Why |
|---|---|---|
| `'a' + 'b'` | `195` | **Two chars are numbers.** 97 + 98. No String involved at all. |
| `"a" + "b"` | `ab` | String + String → concatenation |
| `(char)('a' + 3)` | `d` | arithmetic on the code point, cast back to char |
| `"a" + 1` | `a1` | the `int` is converted to its text form |
| `"Kunal" + new ArrayList<>()` | `Kunal[]` | any object's `toString()` is called |
| `"a" + 'b'` | `ab` | once one side is a String, the char is appended as text |

**The rule:** if **either** operand is a `String`, `+` means concatenation and the other side is converted via `toString()`. If **neither** is, `+` is arithmetic — which is why `'a' + 'b'` is `195`, the trap that catches everyone.

> 💡 `null` concatenates as the text `"null"` rather than throwing — `"x" + null` is `"xnull"`. Convenient, and occasionally the reason a mysterious `"null"` shows up in output.

## 4. The Methods You'll Actually Use

```java
String name = "Kunal Kushwaha Hello World";

name.toCharArray()        // [K, u, n, a, l,  , K, u, s, ...]
name.toLowerCase()        // kunal kushwaha hello world
name.indexOf('a')         // 3
"     Kunal   ".strip()   // "Kunal"
name.split(" ")           // [Kunal, Kushwaha, Hello, World]
```
**Output**
```
[K, u, n, a, l,  , K, u, s, h, w, a, h, a,  , H, e, l, l, o,  , W, o, r, l, d]
kunal kushwaha hello world
Kunal Kushwaha Hello World
3
Kunal
[Kunal, Kushwaha, Hello, World]
```

| Method | Returns | Note |
|---|---|---|
| `length()` | `int` | a **method** — unlike `arr.length` |
| `charAt(i)` | `char` | O(1); throws `StringIndexOutOfBoundsException` |
| `indexOf(x)` | `int` | `-1` if absent — the same sentinel convention as note 08 |
| `substring(a, b)` | `String` | `a` inclusive, `b` **exclusive** |
| `split(regex)` | `String[]` | takes a **regex**, not a literal — `split(".")` matches everything |
| `strip()` / `trim()` | `String` | `strip()` (Java 11+) is Unicode-aware; prefer it |
| `equals` / `equalsIgnoreCase` | `boolean` | never `==` |
| `compareTo` | `int` | negative / 0 / positive — used for sorting |
| `contains`, `startsWith`, `endsWith` | `boolean` | |
| `replace(a, b)` | `String` | returns a new String |

`printf` for formatted output:
```java
System.out.printf("Hello my name is %s and I am %s", "Kunal", "Cool");
```
```
Hello my name is Kunal and I am Cool
```
`%s` string · `%d` integer · `%.2f` two decimals · `%n` newline.

## 5. The Concatenation Trap

```java
String series = "";
for (int i = 0; i < 26; i++) {
    char ch = (char)('a' + i);
    series = series + ch;
}
System.out.println(series);
```
```
abcdefghijklmnopqrstuvwxyz
```

Correct output — but it built **26 separate String objects**, each a full copy of the last plus one character. Copying 1 + 2 + 3 + … + n characters is **O(n²)**.

For 26 characters nobody notices. For 100,000 it is the difference between instant and minutes.

## 6. `StringBuilder` — the Fix

```java
StringBuilder builder = new StringBuilder();
for (int i = 0; i < 26; i++) {
    char ch = (char)('a' + i);
    builder.append(ch);
}
System.out.println(builder.toString());
builder.reverse();
System.out.println(builder);
```
**Output**
```
abcdefghijklmnopqrstuvwxyz
zyxwvutsrqponmlkjihgfedcba
```

`StringBuilder` is a **mutable** character buffer — internally a `char[]` that doubles when full, exactly like `ArrayList` (note 07). `append` is **O(1) amortised**, so the whole loop is **O(n)** instead of O(n²).

| Method | Does |
|---|---|
| `append(x)` | add to the end — takes any type |
| `insert(i, x)` | insert at a position |
| `deleteCharAt(i)` / `delete(a, b)` | remove |
| `reverse()` | reverse **in place** |
| `setCharAt(i, c)` | change one character — impossible on a String |
| `toString()` | produce the final immutable String |

> 💡 **`reverse()` mutates and returns the same object.** Note the output above: after `builder.reverse()`, printing `builder` shows the reversed text — there is no second object. That is the whole point of a builder.

**`StringBuilder` vs `StringBuffer`:** identical APIs. `StringBuffer` is older and **synchronized** (thread-safe); `StringBuilder` is not, and is therefore faster. Single-threaded code — which is all DSA — should use **`StringBuilder`**. Mentioning that distinction is a common interview point.

> 💡 A single `"a" + b + "c"` is **fine** — since Java 9 the compiler turns it into one `invokedynamic makeConcatWithConstants` call (note 04 §15). The trap is concatenation **inside a loop**, where each iteration is a separate operation.

## 7. Palindrome — Strings Meet Two Pointers

```java
static boolean isPalindrome(String str) {
    if (str == null || str.length() == 0) {
        return true;
    }
    str = str.toLowerCase();
    for (int i = 0; i <= str.length() / 2; i++) {
        char start = str.charAt(i);
        char end = str.charAt(str.length() - 1 - i);
        if (start != end) {
            return false;
        }
    }
    return true;
}
```
```java
System.out.println(isPalindrome("abcba"));
```
**Output**
```
true
```

Compare index `i` with index `length - 1 - i`, walking inward — the same two-pointer idea as the array reverse in note 07.

> 💡 `str = str.toLowerCase()` is **reassignment**, not mutation — the caller's String is untouched, because `str` is a copy of the reference (note 06 §4).

> 💡 The loop runs to `<= length / 2`, one iteration more than needed — at the midpoint it compares the middle character with itself. Harmless, but `i < length / 2` is the tighter bound. Note the parallel with `while (start < end)` in the array reverse.

---

## ⚠️ Common Misunderstandings
**1. `name.toLowerCase()` changes `name`.**
❌ changes it · ✅ returns a **new** String. Assign it back: `name = name.toLowerCase()`.

**2. `==` compares String contents.**
❌ contents · ✅ references. `true` for pooled literals, `false` for `new String`/`Scanner` input. Always `.equals()`.

**3. `'a' + 'b'` is `"ab"`.**
❌ `"ab"` · ✅ **`195`** — two chars are numbers. Concatenation needs at least one String operand.

**4. `str.length` / `arr.length()`.**
❌ mixed up · ✅ `str.length()` is a method, `arr.length` is a field, `list.size()` is a method.

**5. `substring(2, 5)` includes index 5.**
❌ inclusive · ✅ end is **exclusive** — you get indexes 2, 3, 4.

**6. `split(".")` splits on a dot.**
❌ literal dot · ✅ it's a **regex**, and `.` matches any character — you get an empty array. Escape it: `split("\\.")`.

**7. `+=` in a loop is fine.**
❌ fine · ✅ **O(n²)** — each pass copies the whole String. Use `StringBuilder`.

**8. `StringBuilder.reverse()` returns a new builder.**
❌ new object · ✅ it mutates **in place** and returns the same object.

**9. Use `StringBuffer` for speed.**
❌ faster · ✅ `StringBuffer` is synchronized and **slower**. Use `StringBuilder` unless you genuinely share it across threads.

## JavaScript Comparison
| | Java | JavaScript |
|---|---|---|
| Immutable | ✅ | ✅ — same model |
| Equality | `.equals()`; `==` is reference | `===` compares text (strings are primitives) |
| Length | `str.length()` | `str.length` (property) |
| Char at | `str.charAt(i)` | `str[i]` |
| Concatenate in a loop | O(n²) — use `StringBuilder` | engines optimise it; usually fine |
| Build efficiently | `StringBuilder` | `arr.push(...)` then `arr.join("")` |
| Substring | `substring(a, b)` — b exclusive | `slice(a, b)` — b exclusive |
| Split | `split(regex)` | `split(string or regex)` |
| `'a' + 'b'` | `195` (chars are numbers) | `"ab"` (no char type) |

> 💡 The two real gotchas moving from JS: **`==` doesn't compare text**, and **`'a'` is a number**. JS has no `char` type, so both surprises are Java-only.

## Interview Angles
- **"Why are Strings immutable?"** — Safety as `HashMap` keys and across threads, plus it makes the String pool possible. Expect this one.
- **"What's the String pool?"** — Identical literals share one object; `new String()` bypasses it. Explains why `==` sometimes "works".
- **"Difference between String, StringBuilder, StringBuffer?"** — Immutable / mutable-unsynchronized / mutable-synchronized. The classic three-way question.
- **"What's wrong with `+=` in a loop?"** — O(n²) copying; `StringBuilder` makes it O(n).
- **"Reverse a string / check a palindrome."** — Two pointers, or `StringBuilder.reverse()`. Say the O(n) and mention that `charAt` avoids the `toCharArray` copy.
- **"Anagram check?"** — Sort both (O(n log n)) or count characters in an `int[26]` (O(n)). Offer the frequency-array version.

## Related · Next
- **Related:** [[Arrays]] (07) · [[Conditionals and Loops]] (05 — `char` arithmetic) · [[Strings]] (1.4) · [[Two Pointers]] (3.1)
- **Practice:** reverse words in a sentence (`split` + `StringBuilder`), check an anagram with an `int[26]` frequency array, and compress `"aaabb"` → `"a3b2"` — all three are `StringBuilder` exercises.
- **Next:** [12 — Patterns](12-Patterns_Notes.md)

---

## 🔁 Rapid Revision (self-test)
<details><summary>1. What does `name.toLowerCase()` do to `name`?</summary>

Nothing. Strings are immutable — it returns a new String. Verified: printing `name` afterwards still shows the original casing.
</details>

<details><summary>2. Why is `new String("Kunal") == new String("Kunal")` false but `"Kunal" == "Kunal"` true?</summary>

`==` compares references. Identical literals share one pooled object; `new String` deliberately creates a separate object. Never use `==` on Strings.
</details>

<details><summary>3. What is `'a' + 'b'`, and why?</summary>

`195`. Two `char`s are numbers (97 + 98) — with no String operand, `+` is arithmetic.
</details>

<details><summary>4. Why is `series += ch` in a loop O(n²)?</summary>

Strings are immutable, so each pass allocates a new String and copies everything so far: 1 + 2 + … + n characters.
</details>

<details><summary>5. What makes StringBuilder O(n)?</summary>

It's a mutable `char[]` that doubles when full (like ArrayList), so `append` is O(1) amortised — no copying per character.
</details>

<details><summary>6. StringBuilder vs StringBuffer?</summary>

Same API. `StringBuffer` is synchronized (thread-safe, slower); `StringBuilder` is not and is faster. Use `StringBuilder` in single-threaded code.
</details>

<details><summary>7. Why does `split(".")` return nothing useful?</summary>

The argument is a **regex**, and `.` matches any character. Escape it: `split("\\.")`.
</details>

<details><summary>8. How do you check a palindrome?</summary>

Two pointers: compare `charAt(i)` with `charAt(length - 1 - i)`, walking inward. O(n) time, O(1) space.
</details>
