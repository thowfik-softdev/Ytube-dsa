---
type: foundation
title: Sorting — Bubble, Selection, Insertion & Cyclic Sort, and the Problems Each One Solves
tags: [foundations, java, sorting, bubble-sort, selection-sort, insertion-sort, cyclic-sort, stability, complexity, pre-phase-0]
created: 2026-09-29
status: seen
source: "Kunal Kushwaha · Community Classroom — lecture 11 Sorting"
related: ["[[Arrays]]", "[[Binary Search]]", "[[Big-O Intuition]]", "[[Cyclic Sort]]", "[[Sorting]]"]
---

# 10 · Sorting

> 📁 Part 10 of 28 in [Ytube dsa/](README.md) · **Prev:** [09 — Binary Search](09-Binary_Search_Notes.md) · **Next:** [11 — Strings & StringBuilder](11-Strings_Notes.md)

## Introduction
Four sorting algorithms, all **O(n²)** except the last — and that is exactly why they're worth learning. You will never ship bubble sort; you *will* be asked to write it, and more importantly, the fourth one (**cyclic sort**) is a genuine interview weapon that solves a whole family of problems in O(n).

The useful framing: each algorithm has a different **invariant** — a statement that is true after every pass. Learn the invariant and the code writes itself.

## The Four at a Glance

| Algorithm | Invariant after pass *i* | Best | Average | Worst | Stable | Swaps |
|---|---|---|---|---|---|---|
| **Bubble** | the largest *i* elements are in place at the end | **O(n)** | O(n²) | O(n²) | ✅ | O(n²) |
| **Selection** | the largest *i* elements are in place at the end | O(n²) | O(n²) | O(n²) | ❌ | **O(n)** |
| **Insertion** | the first *i* elements are sorted among themselves | **O(n)** | O(n²) | O(n²) | ✅ | O(n²) |
| **Cyclic** | every value seen so far sits at its own index | **O(n)** | O(n) | O(n) | ❌ | O(n) |

Cyclic sort beats the O(n log n) lower bound only because it doesn't compare — it **exploits knowing the values are 1…n**.

---

## 1. Bubble Sort — Swap Adjacent Pairs

Repeatedly walk the array swapping any neighbours that are out of order. Each pass "bubbles" the largest remaining element to the end.

```java
static void bubble(int[] arr) {
    boolean swapped;
    for (int i = 0; i < arr.length; i++) {
        swapped = false;
        for (int j = 1; j < arr.length - i; j++) {     // -i: the tail is already sorted
            if (arr[j] < arr[j - 1]) {
                int temp = arr[j];
                arr[j] = arr[j - 1];
                arr[j - 1] = temp;
                swapped = true;
            }
        }
        if (!swapped) {          // a full clean pass = already sorted
            break;
        }
    }
}
```
**Output** — `{5, 3, 4, 1, 2}`
```
[1, 2, 3, 4, 5]  comparisons=10
```

Two optimisations, both in that code:

- **`arr.length - i`** — after pass *i*, the last *i* elements are final. Not re-checking them halves the work.
- **The `swapped` flag** — if a whole pass makes no swaps, the array is sorted and you stop. This is what gives bubble sort its **O(n) best case**:

```
bubble on ALREADY SORTED: comparisons=4  (early exit)
```
Four comparisons for a sorted 5-element array — one clean pass, then out. Without the flag it would do all 10 regardless.

## 2. Selection Sort — Pick the Extreme, Put It in Place

Find the largest element in the unsorted region, swap it into the last unsorted slot. Repeat.

```java
static void selection(int[] arr) {
    for (int i = 0; i < arr.length; i++) {
        int last = arr.length - i - 1;
        int maxIndex = getMaxIndex(arr, 0, last);
        swap(arr, maxIndex, last);
    }
}

static int getMaxIndex(int[] arr, int start, int end) {
    int max = start;
    for (int i = start; i <= end; i++) {
        if (arr[max] < arr[i]) {
            max = i;
        }
    }
    return max;
}
```
**Output** — `{5, 3, 4, 1, 2}`
```
[1, 2, 3, 4, 5]  comparisons=15
```

Note it takes **15** comparisons where bubble took 10 — and on already-sorted input:

```
selection on ALREADY SORTED: comparisons=15  (no early exit)
```

**Selection sort cannot detect a sorted array.** It always scans the entire unsorted region to find the maximum, so it is O(n²) in every case — best, average, worst.

Its one real advantage: it performs **at most n swaps** (one per pass), where bubble can do O(n²). If writes are expensive (flash memory, say), that matters. Otherwise insertion sort beats it.

> ⚠️ Selection sort is **not stable** — swapping a far-away maximum into place jumps it over equal elements, changing their relative order.

## 3. Insertion Sort — Insert Into the Sorted Prefix

Treat the left portion as sorted. Take the next element and walk it left until it lands in the right spot.

```java
static void insertion(int[] arr) {
    for (int i = 0; i < arr.length - 1; i++) {
        for (int j = i + 1; j > 0; j--) {
            if (arr[j] < arr[j - 1]) {
                swap(arr, j, j - 1);
            } else {
                break;                    // <-- the whole optimisation
            }
        }
    }
}
```
**Output** — `{5, 3, 4, 1, 2}`
```
[1, 2, 3, 4, 5]  comparisons=10
```
```
insertion on ALREADY SORTED: comparisons=4
```

That `break` is why insertion sort is **O(n) on nearly-sorted data**: the moment an element is ≥ its left neighbour, it's in position and the inner loop stops immediately.

**This is the one to prefer among the three.** It is stable, adaptive (fast on nearly-sorted input), and works *online* — it can sort data as it arrives, since the prefix is always sorted. Real libraries use it as the base case of larger sorts: Java's `Arrays.sort` switches to insertion sort for small subarrays (under ~47 elements) because its low overhead beats the recursion.

## 4. Cyclic Sort — the Interview Pattern

**When the array contains the numbers 1…n (or 0…n) in some order**, you don't need comparisons. Every value knows exactly where it belongs: value `v` belongs at index `v - 1`.

```java
static void sort(int[] arr) {
    int i = 0;
    while (i < arr.length) {
        int correct = arr[i] - 1;          // where arr[i] SHOULD live
        if (arr[i] != arr[correct]) {
            swap(arr, i, correct);          // send it home; don't advance i
        } else {
            i++;                            // already correct — move on
        }
    }
}
```
**Output** — `{5, 4, 3, 2, 1}`
```
[1, 2, 3, 4, 5]
```

**Why it's O(n)** even though `i` doesn't always advance: every swap puts at least one value permanently into its correct slot. There are n values, so there are at most n swaps, plus at most n increments of `i` — **O(2n) = O(n)**.

> 💡 **Why compare `arr[i] != arr[correct]` instead of `arr[i] != i + 1`?** Comparing *values at the two positions* handles duplicates gracefully — if the target slot already holds the same value, swapping would loop forever. This is the detail that makes the duplicate problems below work.

### The whole family it unlocks

Sort first with the cyclic loop, then a single scan finds the anomaly:

```java
// Missing number — array holds 0..n with one missing
for (int index = 0; index < arr.length; index++) {
    if (arr[index] != index) return index;
}
return arr.length;          // the missing one is n itself
```
```java
int[] arr = {4, 0, 2, 1};
System.out.println(missingNumber(arr));
```
**Output**
```
3
```

| Problem | After cyclic sort, look for… |
|---|---|
| **Missing Number** | the first index where `arr[i] != i` |
| **Find All Missing** | every index where `arr[i] != i + 1` → collect `i + 1` |
| **Find Duplicate** | the value that wants a slot already holding the same value |
| **Find All Duplicates** | every index where `arr[i] != i + 1` → collect `arr[i]` |
| **Set Mismatch** | that index gives both the duplicate (`arr[i]`) and the missing (`i + 1`) |
| **First Missing Positive** | same, but skip values outside `1..n` |

All **O(n) time, O(1) space** — which is what makes them interview favourites. The naive alternatives (a `HashSet`, or sorting) cost O(n) space or O(n log n) time.

Note the guards that differ per problem. `MissingNumber` uses `arr[i] < arr.length` because the range is `0..n`; `MissingPositive` uses `arr[i] > 0 && arr[i] <= arr.length` to ignore negatives and out-of-range values. Get the guard wrong and you get `ArrayIndexOutOfBoundsException`.

## 5. Stability — and Why It Matters

A sort is **stable** if equal elements keep their original relative order.

```
Sort [(Bob,25), (Amy,25), (Cal,20)] by age:
  stable   -> (Cal,20), (Bob,25), (Amy,25)     Bob still before Amy
  unstable -> (Cal,20), (Amy,25), (Bob,25)     order lost
```

It matters when you sort by one key and want a previous sort preserved — sort by name, then by department, and a stable sort keeps names alphabetical within each department.

**Bubble and insertion are stable** (they only swap strictly-out-of-order neighbours, never equal ones). **Selection and cyclic are not** (they swap across long distances).

This is why Java uses **two** sorting algorithms:

| `Arrays.sort(…)` on | Algorithm | Stable? |
|---|---|---|
| primitives (`int[]`) | dual-pivot quicksort | ❌ — irrelevant, equal ints are indistinguishable |
| objects (`Integer[]`, `String[]`) | TimSort (merge + insertion) | ✅ — required by the spec |

```java
int[] st = {3, 1, 2};
Arrays.sort(st);
```
```
[1, 2, 3]
```
In real code, use `Arrays.sort` / `Collections.sort` — **O(n log n)**, heavily optimised. Write these four by hand only for interviews and for the understanding.

---

## ⚠️ Common Misunderstandings
**1. Bubble sort is always O(n²).**
❌ always · ✅ **O(n) best case** *with* the `swapped` flag. Without the flag it really is always O(n²).

**2. Selection sort also gets faster on sorted input.**
❌ it does · ✅ it can't — it must scan the whole unsorted region to find the max. 15 comparisons either way.

**3. Selection sort does fewer comparisons than bubble.**
❌ fewer · ✅ **more** on our data (15 vs 10). Its advantage is fewer **swaps** (O(n) vs O(n²)).

**4. These sorts are interchangeable.**
❌ pick any · ✅ insertion is the best of the three: stable, adaptive, online, and used inside real library sorts.

**5. Cyclic sort works on any array.**
❌ any · ✅ only when values are a known contiguous range (`1..n` or `0..n`). Otherwise `arr[i] - 1` isn't a valid index.

**6. `while (i < arr.length)` with conditional `i++` is O(n²).**
❌ quadratic · ✅ **O(n)** — each swap permanently places at least one value, so there are at most n swaps total.

**7. Comparing `arr[i] != i + 1` is equivalent to `arr[i] != arr[correct]`.**
❌ equivalent · ✅ with duplicates the first form loops forever. Compare the *values at both positions*.

**8. All sorts are stable.**
❌ all · ✅ bubble and insertion are; selection and cyclic are not. It matters whenever you sort by a second key.

## JavaScript Comparison
| | Java | JavaScript |
|---|---|---|
| Built-in | `Arrays.sort(arr)` | `arr.sort()` |
| Default order | numeric for `int[]` | **lexicographic!** `[10,9,1].sort()` → `[1,10,9]` |
| Numeric sort | automatic | `arr.sort((a, b) => a - b)` — required |
| Stability | guaranteed for objects (TimSort) | guaranteed since ES2019 |
| Sorts in place | yes | yes (and returns the array) |

> 💡 JS's default string comparison is the classic bug — `[10, 9, 1].sort()` gives `[1, 10, 9]` because it compares `"10" < "9"`. Java has no such trap for `int[]`.

## Interview Angles
- **"Write bubble sort."** — Include the `swapped` flag and the `- i` bound unprompted, then state best O(n) / worst O(n²).
- **"Which of these would you actually use, and why?"** — Insertion: stable, adaptive, online, and the base case inside real library sorts.
- **"Selection vs bubble?"** — Selection does fewer swaps (O(n)), bubble fewer comparisons on partly-sorted data and can early-exit. Selection is never adaptive.
- **"What is a stable sort and when do you need it?"** — Equal elements keep their order; needed when sorting by a secondary key.
- **"Array of 1..n with one missing / one duplicate."** — Cyclic sort. O(n) time, O(1) space. This single pattern covers six LeetCode problems.
- **"Can you sort faster than O(n log n)?"** — Not by comparison — that's the proven lower bound. But *non-comparison* sorts (cyclic, counting, radix) can, by exploiting the structure of the values.

## Related · Next
- **Related:** [[Arrays]] (07) · [[Binary Search]] (09 — what sorting enables) · [[Cyclic Sort]] (2.7) · [[Sorting]] (2.5 — merge & quick sort arrive with recursion)
- **Practice:** implement all four from memory, then solve Missing Number (LeetCode 268), Find All Numbers Disappeared (448), Find the Duplicate (287), and First Missing Positive (41) — all with the same cyclic loop and a different final scan.
- **Next:** [11 — Strings & StringBuilder](11-Strings_Notes.md)

---

## 🔁 Rapid Revision (self-test)
<details><summary>1. Bubble sort's best case, and what makes it possible?</summary>

**O(n)** — but only with the `swapped` flag. A pass with no swaps means the array is sorted, so you break out. Verified: 4 comparisons on a sorted 5-element array vs 10 unsorted.
</details>

<details><summary>2. Why can't selection sort be adaptive?</summary>

It must scan the entire unsorted region to find the maximum, regardless of order. 15 comparisons on our array whether sorted or not.
</details>

<details><summary>3. What is selection sort actually good at?</summary>

Minimising **swaps** — at most n of them, one per pass. Useful when writes are expensive.
</details>

<details><summary>4. Why is insertion sort the best of the three?</summary>

Stable, adaptive (O(n) on nearly-sorted data thanks to the `break`), and online. Real library sorts use it for small subarrays.
</details>

<details><summary>5. When does cyclic sort apply, and why is it O(n)?</summary>

When values are a known contiguous range (1..n or 0..n), so value `v` belongs at index `v-1`. Every swap places at least one value permanently, so at most n swaps — O(n).
</details>

<details><summary>6. Why compare `arr[i] != arr[correct]` rather than `arr[i] != i + 1`?</summary>

With duplicates, the target slot may already hold the same value; swapping would loop forever. Comparing the values at both positions detects that.
</details>

<details><summary>7. What is a stable sort, and which of the four are stable?</summary>

Equal elements keep their relative order. Bubble and insertion are stable; selection and cyclic are not (they swap across long distances).
</details>

<details><summary>8. What does Java's `Arrays.sort` actually use?</summary>

Dual-pivot quicksort for primitives (stability is meaningless there) and TimSort for objects (stability is guaranteed by the spec).
</details>

<details><summary>9. Can anything beat O(n log n)?</summary>

Not by comparison — that's a proven lower bound. Non-comparison sorts (cyclic, counting, radix) can, by exploiting the range of the values.
</details>
