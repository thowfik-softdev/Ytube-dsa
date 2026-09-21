---
type: foundation
title: Linear Search — The Baseline Algorithm, Return-Value Design, and Searching 2-D & Strings
tags: [foundations, java, linear-search, searching, arrays, complexity, pre-phase-0]
created: 2026-09-29
status: seen
source: "Kunal Kushwaha · Community Classroom — lecture 09 Linear Search"
related: ["[[Arrays]]", "[[Binary Search]]", "[[Big-O Intuition]]"]
---

# 8 · Linear Search

> 📁 Part 8 of 28 in [Ytube dsa/](README.md) · **Prev:** [07 — Arrays & ArrayList](07-Arrays_Notes.md) · **Next:** [09 — Binary Search](09-Binary_Search_Notes.md)

## Introduction
**Linear search** walks an array from one end to the other, comparing each element to the target. It is the simplest possible algorithm — and the baseline every other search is measured against.

It is worth taking seriously for two reasons. First, it is the **only** option when data is unsorted. Second, this lecture is really about a design question that outlives the algorithm: **what should a search return when it finds nothing?**

## Characteristics
- **Works on anything:** no sorting, no structure, no preprocessing required.
- **O(n) time, O(1) space:** in the worst case you touch every element once.
- **Early exit:** `return` the moment you find the target — don't finish the loop.
- **Order-independent:** the array can be in any order, contain duplicates, be any type.

---

## 1. The Three Return Shapes

The lecture writes the same search three ways, and the difference matters more than the code.

```java
// Returns the INDEX, or -1 if not found.  <-- the useful one
static int linearSearch(int[] arr, int target) {
    if (arr.length == 0) {
        return -1;
    }
    for (int index = 0; index < arr.length; index++) {
        if (arr[index] == target) {
            return index;              // found — stop immediately
        }
    }
    return -1;                         // only reached if nothing matched
}

// Returns the ELEMENT, or Integer.MAX_VALUE if not found.  <-- avoid
static int linearSearch2(int[] arr, int target) { /* ... */ return Integer.MAX_VALUE; }

// Returns true / false.  <-- fine when you only need existence
static boolean linearSearch3(int[] arr, int target) {
    for (int element : arr) {
        if (element == target) {
            return true;
        }
    }
    return false;
}
```
```java
int[] nums = {23, 45, 1, 2, 8, 19, -3, 16, -11, 28};
System.out.println(linearSearch3(nums, 19));
```
**Output**
```
true
```

| Return shape | Use when | Problem |
|---|---|---|
| **index**, `-1` if absent | you need the position | `-1` is a safe sentinel: never a valid index |
| **element** | almost never | returning the target tells you nothing you didn't pass in |
| **boolean** | you only need "does it exist" | loses the position |

> 💡 **Why returning the element is pointless:** you already *have* the target — you passed it in. The only new information a search can give you is *where* it is. `linearSearch2` also has to invent `Integer.MAX_VALUE` as a "not found" marker, which is a legitimate value an array could contain. Return the **index**.

> 💡 `-1` works as "not found" because indexes are never negative. This is the one case where a sentinel is genuinely safe — unlike `max()` returning `-1` in note 07, where `-1` was valid data.

## 2. Searching a Range

```java
static int linearSearch(int[] arr, int target, int start, int end) {
    if (arr.length == 0) {
        return -1;
    }
    for (int index = start; index <= end; index++) {
        if (arr[index] == target) {
            return index;
        }
    }
    return -1;
}
```
```java
int[] arr = {18, 12, -7, 3, 14, 28};
System.out.println(linearSearch(arr, 3456, 1, 4));
```
**Output**
```
-1
```

Note `index <= end` — this is an **inclusive** range, so `end` must be a valid index. The caller passing `arr.length` instead of `arr.length - 1` would get `ArrayIndexOutOfBoundsException`. Deciding inclusive-vs-exclusive and staying consistent is half of getting loop bounds right.

## 3. Minimum — Best-So-Far Again

```java
static int min(int[] arr) {
    int ans = arr[0];                      // seed with a REAL element
    for (int i = 1; i < arr.length; i++) { // start at 1 — 0 is already the seed
        if (arr[i] < ans) {
            ans = arr[i];
        }
    }
    return ans;
}
```
```java
int[] arr = {18, 12, 7, 3, 14, 28};
System.out.println(min(arr));
```
**Output**
```
3
```

Same shape as `max` in note 07. Seeding with `arr[0]` (not `0`, not `Integer.MAX_VALUE`) is what makes it correct for every input — and starting the loop at `i = 1` avoids comparing the seed against itself.

## 4. Searching a 2-D Array

```java
static int[] search(int[][] arr, int target) {
    for (int row = 0; row < arr.length; row++) {
        for (int col = 0; col < arr[row].length; col++) {
            if (arr[row][col] == target) {
                return new int[]{row, col};        // return BOTH coordinates
            }
        }
    }
    return new int[]{-1, -1};
}
```
```java
int[][] arr = {
    {23, 4, 1},
    {18, 12, 3, 9},
    {78, 99, 34, 56},
    {18, 12}
};
System.out.println(Arrays.toString(search(arr, 56)));
System.out.println(max(arr));
```
**Output**
```
[2, 3]
99
```

Two techniques here:

- **Returning two values** — Java methods return one thing, so pack the pair into an `int[]{row, col}`. (Alternatives: a small class, or `int[]` of fixed meaning. The array is the DSA convention.)
- **`arr[row].length` again** — the rows in this example are jagged (3, 4, 4, 2 elements). A fixed column bound would crash.

For the 2-D maximum, seed with `Integer.MIN_VALUE` — here there is no single "first element" to seed from, and `MIN_VALUE` is smaller than any possible `int`:

```java
static int max(int[][] arr) {
    int max = Integer.MIN_VALUE;
    for (int[] ints : arr) {
        for (int element : ints) {
            if (element > max) max = element;
        }
    }
    return max;
}
```
`Integer.MIN_VALUE` is `-2147483648` — the same number that broke `Math.abs` in note 05.

## 5. Searching a String

A `String` is a sequence of `char`, so the same loop works two ways:

```java
static boolean search(String str, char target) {
    for (int i = 0; i < str.length(); i++) {     // by index
        if (target == str.charAt(i)) return true;
    }
    return false;
}

static boolean search2(String str, char target) {
    for (char ch : str.toCharArray()) {          // as a char array
        if (ch == target) return true;
    }
    return false;
}
```
```java
System.out.println(Arrays.toString("Kunal".toCharArray()));
```
**Output**
```
[K, u, n, a, l]
```

`charAt(i)` reads in place; `toCharArray()` **copies** the whole string into a new `char[]` — O(n) extra space. For a single pass, `charAt` is the better habit.

## 6. Counting Digits — Two Ways

```java
static int digits(int num) {              // loop: peel digits
    if (num < 0) num = num * -1;
    if (num == 0) return 1;               // 0 has one digit, but the loop would return 0
    int count = 0;
    while (num > 0) {
        count++;
        num = num / 10;
    }
    return count;
}

static int digits2(int num) {             // math: log10
    if (num < 0) num = num * -1;
    return (int)(Math.log10(num)) + 1;
}
```
```java
System.out.println(digits2(-345678));
```
**Output**
```
6
```

Both are **O(log₁₀ n)** — well, `digits2` is O(1) arithmetic. But note the asymmetry: `digits` explicitly handles `num == 0`, and `digits2` **does not**. `Math.log10(0)` is `-Infinity`, and casting that to `int` gives `Integer.MIN_VALUE`, so `digits2(0)` returns `-2147483647` instead of `1`. The loop version is slower but safer — a good reminder that the "clever" one-liner needs the same edge-case care.

## 7. Complexity

| | Time | Space |
|---|---|---|
| Best case (target first) | **O(1)** | O(1) |
| Average | **O(n/2) → O(n)** | O(1) |
| Worst (last, or absent) | **O(n)** | O(1) |
| 2-D search | **O(rows × cols)** | O(1) |

Linear search is O(n) **and cannot be improved** on unsorted data — you cannot rule out any element without looking at it. The only way to do better is to *add structure*: sort it (→ binary search, O(log n)) or hash it (→ `HashSet`, O(1) average). That trade — pay once to organise, then search fast forever — is the idea note 09 is built on.

---

## ⚠️ Common Misunderstandings
**1. Finishing the loop after finding the target.**
❌ set a flag and keep looping · ✅ `return` immediately. Anything after the match is wasted work.

**2. Returning the element instead of the index.**
❌ tells the caller nothing new · ✅ return the index, `-1` if absent.

**3. Using a data value as the "not found" marker.**
❌ `Integer.MAX_VALUE` or `0` · ✅ `-1` for an index search, because indexes are never negative.

**4. Seeding `min`/`max` with `0`.**
❌ breaks on all-negative (or all-positive) data · ✅ seed with `arr[0]`, or `Integer.MIN_VALUE`/`MAX_VALUE` when there is no first element.

**5. Off-by-one in a range search.**
❌ passing `arr.length` as `end` with `index <= end` · ✅ pass `arr.length - 1` for an inclusive range.

**6. `digits2(0)`.**
❌ returns `1` · ✅ `Math.log10(0)` is `-Infinity` → casting gives a huge negative. The loop version guards `0` explicitly; the log version doesn't.

**7. `toCharArray()` is free.**
❌ free · ✅ it allocates and copies a new array — O(n) space. Use `charAt(i)` for a single pass.

## JavaScript Comparison
| | Java | JavaScript |
|---|---|---|
| Find index | hand-written loop, or `Arrays.asList(a).indexOf(x)` | `arr.indexOf(x)` — built in |
| Find existence | hand-written loop | `arr.includes(x)` |
| Find by predicate | loop, or streams | `arr.find(fn)` / `arr.findIndex(fn)` |
| Not-found marker | `-1` (your convention) | `-1` for `indexOf`, `undefined` for `find` |
| Char at index | `str.charAt(i)` | `str[i]` or `str.charAt(i)` |
| String → chars | `str.toCharArray()` (copies) | `[...str]` or `str.split("")` |

> 💡 JS hands you `indexOf`/`includes`, so you rarely write this loop. In Java you write it constantly — and in interviews you write it in **both** languages, because the point is the algorithm, not the library.

## Interview Angles
- **"Search an unsorted array."** — Linear search, O(n), and say *why* it can't be beaten: you cannot exclude an element without examining it.
- **"How would you speed up repeated searches?"** — Sort once (O(n log n)) then binary search each query (O(log n)), or build a `HashSet` (O(n) once, O(1) per lookup). Choose by how many queries you expect.
- **"What do you return when it's not found?"** — `-1` for an index. Be ready to explain why a *value* sentinel is unsafe.
- **"Find the max/min."** — Single pass, seed with `arr[0]`. Follow-up: second-largest in one pass (track two variables, don't sort).
- **"Search a 2-D array."** — Nested loops, O(rows × cols), return `{row, col}`. Follow-up: *"what if each row is sorted?"* → that's note 09.

## Related · Next
- **Related:** [[Arrays]] (07 — the structure being searched) · [[Binary Search]] (09 — what sorting buys you) · [[Big-O Intuition]]
- **Practice:** write `count(int[], int)` (how many times a target appears — no early exit here, you must see every element), and `indexOfLast` (search backwards).
- **Next:** [09 — Binary Search](09-Binary_Search_Notes.md)

---

## 🔁 Rapid Revision (self-test)
<details><summary>1. Complexity of linear search, and why it can't be improved on unsorted data?</summary>

O(n) time, O(1) space. On unsorted data you cannot rule out any element without looking at it, so every element must be examined in the worst case.
</details>

<details><summary>2. Which of the three return shapes is best, and why?</summary>

Return the **index** (`-1` if absent). The boolean loses position; returning the element gives back what the caller already passed in, and needs an unsafe sentinel for "not found".
</details>

<details><summary>3. Why is `-1` a safe "not found" marker here but not in `max()`?</summary>

Indexes are never negative, so `-1` can't collide with a real answer. A *maximum* of `-1` is perfectly possible, so there it is ambiguous.
</details>

<details><summary>4. How do you return two values (row and col) from a Java method?</summary>

Pack them: `return new int[]{row, col};`. Java returns a single value, so an array (or a small class) carries the pair.
</details>

<details><summary>5. Why seed a 2-D max with `Integer.MIN_VALUE` instead of `arr[0]`?</summary>

There is no single first element to seed from in a nested traversal, and `MIN_VALUE` (-2147483648) is ≤ every possible int.
</details>

<details><summary>6. `charAt(i)` vs `toCharArray()`?</summary>

`charAt` reads in place, O(1) and no allocation. `toCharArray` copies the whole string into a new array — O(n) extra space.
</details>

<details><summary>7. What does `digits2(0)` return, and why?</summary>

Not `1`. `Math.log10(0)` is `-Infinity`; casting to `int` clamps to a large negative. The loop version guards `num == 0` explicitly.
</details>
