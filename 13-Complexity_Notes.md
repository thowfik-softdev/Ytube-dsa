---
type: foundation
title: Complexity Analysis — Big-O, Recurrences, and Measuring What Actually Matters
tags: [foundations, java, complexity, big-o, time-complexity, space-complexity, recurrence, masters-theorem, pre-phase-0]
created: 2026-09-29
status: seen
source: "Kunal Kushwaha · Community Classroom — lecture 15 Complexity Analysis"
related: ["[[Big-O Intuition]]", "[[Sorting]]", "[[Binary Search]]", "[[Recursion]]"]
---

# 13 · Complexity Analysis

> 📁 Part 13 of 28 in [Ytube dsa/](README.md) · **Prev:** [12 — Patterns](12-Patterns_Notes.md) · **Next:** [14 — Recursion I](14-Recursion_Basics_Notes.md)

## Introduction
**Complexity analysis answers one question: how does the work grow as the input grows?** Not "how many milliseconds" — that depends on your laptop, the JIT, and what else is running. Instead we count **operations as a function of n**, and keep only the part that dominates.

This note comes *before* recursion deliberately: recursive algorithms are analysed with **recurrence relations**, and you cannot reason about merge sort or the naive Fibonacci without them.

## Characteristics
- **Machine-independent:** counts operations, not seconds.
- **Asymptotic:** describes behaviour as n → ∞, so constants and lower-order terms drop.
- **Worst-case by default:** "O(n)" means "never worse than proportional to n".
- **Two dimensions:** time (operations) and space (extra memory).

---

## 1. Why Not Just Time It?

```
naive fib(10) = 55         calls=177          time=     0 ms
naive fib(20) = 6765       calls=21891        time=     0 ms
naive fib(30) = 832040     calls=2692537      time=     6 ms
naive fib(35) = 9227465    calls=29860703     time=    61 ms
naive fib(40) = 102334155  calls=331160281    time=   677 ms
iterative fib(40) = 102334155  time=0 ms
```

Measured on JDK 21. Two lessons:

- **The time column is unreliable** — `fib(20)` and `fib(10)` both read `0 ms`. Timing hides everything until the input is large.
- **The call count is not** — 177 → 21,891 → 2.7M → 331M. Each +10 on n multiplies calls by ~120. That is **exponential**, and it is visible in the counts long before the clock notices.

Complexity analysis is counting the second column *without running anything*.

## 2. The Notations

| Notation | Means | Everyday use |
|---|---|---|
| **O(f)** — Big-O | **upper** bound: never worse than f | the one you'll use 95% of the time |
| **Ω(f)** — Big-Omega | **lower** bound: never better than f | "sorting by comparison is Ω(n log n)" |
| **Θ(f)** — Big-Theta | **tight** bound: both at once | when best and worst match |
| **o(f)** — little-o | strictly less than f | rare, theoretical |
| **ω(f)** — little-omega | strictly greater than f | rare, theoretical |

Strictly, saying "quicksort is O(n²)" is *true* (it's an upper bound) but unhelpful; people usually mean Θ. In interviews, "O" with best/average/worst stated explicitly is what's expected.

## 3. The Rules

**1. Drop constants.** O(2n) → **O(n)**. Doubling the work doesn't change how it *scales*.

**2. Drop lower-order terms.** O(n² + n + 500) → **O(n²)**. At n = 1,000,000, n² utterly dwarfs the rest.

**3. Different inputs, different variables.** Two loops over two arrays is **O(a + b)**, not O(n). Nested is **O(a × b)**.

**4. Sequential = add, nested = multiply.**
```java
for (int i = 0; i < n; i++) { }     // O(n)
for (int j = 0; j < n; j++) { }     // O(n)   -> total O(n)

for (int i = 0; i < n; i++) {
    for (int j = 0; j < n; j++) { } // O(n) * O(n) = O(n²)
}
```

## 4. The Common Classes

| Complexity | Name | Example | n = 1,000,000 |
|---|---|---|---|
| **O(1)** | constant | array index, HashMap get | 1 |
| **O(log n)** | logarithmic | binary search | 20 |
| **O(n)** | linear | linear search, single loop | 1,000,000 |
| **O(n log n)** | linearithmic | merge sort, `Arrays.sort` | 20,000,000 |
| **O(n²)** | quadratic | bubble/selection/insertion, nested loops | 10¹² |
| **O(2ⁿ)** | exponential | naive Fibonacci, subsets | astronomically large |
| **O(n!)** | factorial | permutations, brute-force TSP | beyond astronomical |

> 💡 **The practical dividing line:** up to O(n log n) scales to millions. O(n²) is fine to ~10,000 and painful beyond. O(2ⁿ) is unusable past ~40 — exactly what `fib(40)` showed at 331 million calls.

## 5. Reading Complexity Off Code

**The rule of thumb:** count the nesting depth of loops over the input — but *what the loop does to the counter* matters more than the nesting.

```java
for (int i = 0; i < n; i++) { }         // O(n)   — counter grows by +1
for (int i = 1; i < n; i *= 2) { }      // O(log n) — counter DOUBLES
for (int i = n; i > 0; i /= 2) { }      // O(log n) — counter halves
```

Anything that multiplies or divides the counter is logarithmic — that's why binary search, the doubling in `ArrayList`, and the unbounded-array search in note 09 are all O(log n).

**Two loops that look nested but aren't quadratic:**
```java
int i = 0;
while (i < arr.length) {
    if (...) { swap(...); }    // no i++ here
    else     { i++; }
}
```
This is the cyclic sort from note 10 — **O(n)**, not O(n²), because each swap permanently places one element, capping total swaps at n. **Counting iterations is not the same as counting nesting.**

## 6. Space Complexity

How much **extra** memory the algorithm needs — not counting the input.

```java
static void reverse(int[] arr) {       // O(1) — a few index variables
    int start = 0, end = arr.length - 1;
    ...
}

static int[] reversed(int[] arr) {     // O(n) — a whole new array
    int[] out = new int[arr.length];
    ...
}
```

**Recursion always costs stack space.** Each pending call keeps a frame (note 06 §12), so depth-d recursion is **O(d)** space — which is why this happens:

```
StackOverflowError caught at depth ~19754
```

Around 19,700 frames on a default JVM stack. Recursion of depth n is O(n) space *and* has a hard ceiling; an iterative loop of the same depth is O(1) and has none.

## 7. Recurrence Relations — Analysing Recursion

A recursive function's cost is defined in terms of itself.

**Linear recursion** — `fact(n)` makes one call on `n-1`:
```
T(n) = T(n-1) + O(1)   →   O(n)
```

**Binary search** — one call on half the input:
```
T(n) = T(n/2) + O(1)   →   O(log n)
```

**Merge sort** — two calls on half, plus a linear merge:
```
T(n) = 2T(n/2) + O(n)  →   O(n log n)
```

**Naive Fibonacci** — two calls on nearly the whole input:
```
T(n) = T(n-1) + T(n-2) + O(1)   →   O(2ⁿ)
```
That last one is the 331 million calls. Each level roughly doubles the number of calls, and there are n levels.

### Master's Theorem

For `T(n) = a·T(n/b) + O(n^d)` — *a* subproblems, each 1/*b* the size, plus O(n^d) to combine:

| Compare | Result | Example |
|---|---|---|
| d > log_b(a) | **O(n^d)** | the combining dominates |
| d = log_b(a) | **O(n^d log n)** | merge sort: a=2, b=2, d=1 → O(n log n) |
| d < log_b(a) | **O(n^log_b(a))** | the recursion dominates |

Merge sort: a = 2, b = 2, d = 1. log₂2 = 1 = d → **O(n log n)**. ✓
Binary search: a = 1, b = 2, d = 0. log₂1 = 0 = d → **O(n⁰ log n) = O(log n)**. ✓

**Akra–Bazzi** generalises this to uneven splits (`T(n) = T(n/3) + T(2n/3) + O(n)`), which Master's can't handle. Worth knowing the name; rarely needed in interviews.

## 8. Best, Average, Worst — They Differ

From the measured sorting note (10):

| Input | Bubble | Selection | Insertion |
|---|---|---|---|
| `{5,3,4,1,2}` | 10 comparisons | 15 | 10 |
| `{1,2,3,4,5}` (sorted) | **4** | **15** | **4** |

Bubble and insertion detect sorted input and exit early — **O(n) best case**. Selection cannot — it is Θ(n²) in every case. Same Big-O bucket on paper, very different in practice.

**Amortised** is a fourth flavour: a single operation is occasionally expensive, but *averaged over many*, it's cheap. `ArrayList.add` is **O(1) amortised** — usually a single write, occasionally an O(n) resize-and-copy, rare enough that the average is constant.

---

## ⚠️ Common Misunderstandings
**1. O(n) is always faster than O(n²).**
❌ always · ✅ **for large n**. Constants matter for small inputs — which is why library sorts switch to insertion sort for small subarrays.

**2. Big-O tells you how long something takes.**
❌ seconds · ✅ how the work **grows**. `fib(10)` and `fib(20)` both timed 0 ms, but one did 123× more work.

**3. Two sequential loops are O(n²).**
❌ multiply · ✅ **add** — O(n) + O(n) = O(2n) = O(n). Only *nested* loops multiply.

**4. Nested loops always mean O(n²).**
❌ always · ✅ cyclic sort has a conditional `i++` inside a `while` and is **O(n)**, because each swap permanently places an element.

**5. Recursion is free of space cost.**
❌ free · ✅ every pending call holds a stack frame — O(depth) space, and `StackOverflowError` at ~19,700 frames.

**6. Dropping constants means constants don't matter.**
❌ never matter · ✅ they don't affect *scaling*, but 2n vs 100n is real. Big-O is a tool, not the whole truth.

**7. O(log n) needs a specific base.**
❌ base matters · ✅ bases differ by a constant factor (`log₂n = log₁₀n / log₁₀2`), which Big-O drops. Everyone writes log n.

**8. Best case is what you quote.**
❌ best · ✅ **worst case** unless stated otherwise. Quote best/average/worst separately when they differ.

## JavaScript Comparison
Complexity is language-independent — an O(n²) algorithm is O(n²) everywhere. What differs is the **constant factor** and what the built-ins cost:

| Operation | Java | JavaScript |
|---|---|---|
| `arr.length` / `.length` | O(1) | O(1) |
| Append | `ArrayList.add` O(1) amortised | `push` O(1) amortised |
| Insert at front | `ArrayList.add(0, x)` O(n) | `unshift` O(n) |
| Map/Set lookup | `HashMap.get` O(1) avg | `Map.get` O(1) avg |
| Sort | O(n log n) | O(n log n) |
| `str + str` in a loop | O(n²) | engine-optimised, often O(n) |
| Stack depth limit | ~19,700 frames | ~10,000 frames typically |

## Interview Angles
- **"What's the complexity?"** — asked after *every* solution. Give time **and** space, and say which case.
- **"Can you do better?"** — usually yes, and usually by adding a data structure: a HashMap to trade O(n) space for O(1) lookup, or sorting first to enable binary search.
- **"Why is naive Fibonacci so slow?"** — `T(n) = T(n-1) + T(n-2)` → O(2ⁿ); it recomputes the same subproblems. Memoising drops it to O(n). *This is the doorway to dynamic programming.*
- **"Analyse merge sort."** — `T(n) = 2T(n/2) + O(n)`, then Master's → O(n log n). Space O(n) for the merge buffer.
- **"O(1) space" constraints** — means you may use a fixed number of variables, no array proportional to n. Recursion usually breaks it.
- **Know the lower bound:** comparison sorting is **Ω(n log n)**. Anything claiming faster must not be comparing (counting sort, radix sort, cyclic sort).

## Related · Next
- **Related:** [[Big-O Intuition]] (1.1) · [[Sorting]] (10 — the measured counts) · [[Binary Search]] (09 — O(log n)) · [[Recursion]] (14)
- **Practice:** state time *and* space for every algorithm in notes 08–12 without looking. Then derive the recurrence for merge sort and solve it with Master's theorem.
- **Next:** [14 — Recursion I](14-Recursion_Basics_Notes.md)

---

## 🔁 Rapid Revision (self-test)
<details><summary>1. What does Big-O actually measure?</summary>

How the number of operations **grows** with input size — machine-independent, asymptotic, worst-case by default. Not milliseconds.
</details>

<details><summary>2. The two simplification rules?</summary>

Drop constants (O(2n) → O(n)) and drop lower-order terms (O(n² + n) → O(n²)).
</details>

<details><summary>3. Sequential vs nested loops?</summary>

Sequential **add** (O(n) + O(n) = O(n)); nested **multiply** (O(n) × O(n) = O(n²)).
</details>

<details><summary>4. Why is `for (i = 1; i < n; i *= 2)` O(log n)?</summary>

The counter multiplies rather than increments, so it reaches n in log₂n steps. Anything that repeatedly multiplies or halves is logarithmic.
</details>

<details><summary>5. Why is cyclic sort O(n) despite a conditional `i++` in a while loop?</summary>

Each swap permanently places at least one element, capping swaps at n. Counting iterations ≠ counting nesting.
</details>

<details><summary>6. What is the space cost of recursion, and what's the hard limit?</summary>

O(depth) for the stack frames. Measured: `StackOverflowError` at ~19,754 frames on a default JVM stack.
</details>

<details><summary>7. Recurrences for binary search, merge sort, and naive Fibonacci?</summary>

`T(n) = T(n/2) + O(1)` → O(log n) · `T(n) = 2T(n/2) + O(n)` → O(n log n) · `T(n) = T(n-1) + T(n-2) + O(1)` → O(2ⁿ).
</details>

<details><summary>8. Master's theorem on merge sort?</summary>

`T(n) = a·T(n/b) + O(n^d)` with a=2, b=2, d=1. log₂2 = 1 = d, so the middle case applies: **O(n^d log n) = O(n log n)**.
</details>

<details><summary>9. Why does selection sort have the same best and worst case?</summary>

It must scan the whole unsorted region to find the maximum regardless of order — no early exit. 15 comparisons sorted or not, vs bubble's 4 vs 10.
</details>

<details><summary>10. What does "amortised O(1)" mean?</summary>

A single operation is occasionally expensive but cheap on average over many. `ArrayList.add` is usually one write, occasionally an O(n) resize.
</details>
