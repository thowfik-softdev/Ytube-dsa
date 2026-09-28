---
type: foundation
title: Advanced Sorting & Techniques — Counting, Radix, Huffman and Sqrt Decomposition
tags: [foundations, java, counting-sort, radix-sort, huffman, greedy, sqrt-decomposition, non-comparison-sort, pre-phase-0]
created: 2026-09-29
status: seen
source: "Kunal Kushwaha · Community Classroom — lectures 26 Advanced Sorting, 27 Huffman Coding, 28 Sqrt Decomposition"
related: ["[[Sorting]]", "[[Heaps and HashMaps]]", "[[Complexity Analysis]]", "[[Trees I]]"]
---

# 26 · Advanced Sorting & Techniques

> 📁 Part 26 of 28 in [Ytube dsa/](README.md) · **Prev:** [25 — Heaps & HashMaps](25-Heaps_and_HashMaps_Notes.md) · **Next:** [27 — Large Numbers & File Handling](27-LargeNumbers_and_FileIO_Notes.md)

## Introduction
Note 13 stated a hard limit: **comparison-based sorting cannot beat O(n log n)**. That's a proven lower bound, not a lack of cleverness.

This note covers algorithms that *do* beat it — by **not comparing**. Counting sort and radix sort exploit the structure of the values themselves. Then two bonus techniques: **Huffman coding** (a greedy algorithm built on heaps and trees) and **sqrt decomposition** (a simpler alternative to segment trees).

---

## 1. Counting Sort

**When the values come from a small known range**, don't compare — count.

```java
static void countSort(int[] arr) {
    int max = Arrays.stream(arr).max().getAsInt();
    int[] count = new int[max + 1];

    for (int num : arr) {                 // 1. tally occurrences
        count[num]++;
    }

    int index = 0;                         // 2. rebuild in order
    for (int i = 0; i < count.length; i++) {
        while (count[i] > 0) {
            arr[index++] = i;
            count[i]--;
        }
    }
}
```

**O(n + k)** time and **O(k)** space, where k is the value range.

The insight: if you know values are 0–100, the *index itself* carries the ordering. Counting how many of each you saw and then walking the count array in order produces sorted output with **zero comparisons** — which is exactly how it sidesteps the O(n log n) bound.

> ⚠️ **k matters enormously.** Sorting 10 numbers that range up to 1,000,000 allocates a million-element array to sort ten values. Counting sort is only worth it when **k = O(n)** — a small, dense range. This is the trade the lower bound "forbids": you buy speed with space and a restriction on the input.

> 💡 This is the same trick as **cyclic sort** (note 10 §4): exploit knowing the values, and comparisons become unnecessary.

The version above isn't stable (it rebuilds from counts). The **stable** variant computes a prefix sum over the counts and places elements from the end backwards — necessary for radix sort.

## 2. Radix Sort

Counting sort fails for large ranges. Radix sort fixes that by sorting **one digit at a time**, least significant first, using a stable counting sort for each pass.

```
input:     170  45  75  90  802  24  2  66
by 1s:     170  90  802  2  24  45  75  66
by 10s:    802  2  24  45  66  170  75  90
by 100s:   2  24  45  66  75  90  170  802   ← sorted
```

**O(d × (n + k))** where d is the number of digits and k is the base (10). With d fixed and small, that's effectively **O(n)**.

> 💡 **Stability is not optional here.** Each pass must preserve the order established by the previous pass — that's the whole mechanism. Use an unstable sort per digit and radix sort simply doesn't work. This is the clearest practical reason stability (note 10 §5) matters.

| | Counting sort | Radix sort |
|---|---|---|
| Time | O(n + k) | O(d(n + k)) |
| Space | O(k) | O(n + k) |
| Good when | small dense range | large range, few digits |
| Comparisons | none | none |

## 3. Huffman Coding — Greedy + Heap + Tree

Compression by **variable-length codes**: frequent characters get short bit patterns, rare ones get long. Every structure from the last few notes appears at once.

**The algorithm:**

1. Count each character's frequency (**HashMap**, note 25)
2. Put every character in a **min-heap** keyed by frequency (note 25)
3. While more than one node remains: **poll the two smallest**, join them under a new node whose frequency is their sum, push it back
4. The result is a **binary tree** (note 23); left edges are `0`, right edges are `1`
5. A character's code is the path from the root to its leaf

```
freq: a=5  b=2  c=1  d=1

  combine c(1)+d(1) -> 2
  combine b(2)+[cd](2) -> 4
  combine a(5)+[bcd](4) -> 9

        (9)
       /   \
     a(5)  (4)
          /   \
        b(2)  (2)
             /   \
           c(1)  d(1)

a = 0     b = 10     c = 110     d = 111
```

`a` is the most frequent so it gets the shortest code. Fixed-width encoding would need 2 bits each = 18 bits for `aaaaabbcd`; Huffman needs 5×1 + 2×2 + 3 + 3 = **15 bits**.

**Why it's greedy:** at each step it makes the locally optimal choice (merge the two least frequent) and never reconsiders — and for this problem that provably yields the optimal prefix code.

**The prefix property** is what makes decoding unambiguous: no code is a prefix of another, because every character sits at a **leaf**. Reading bits, you can never be mid-code and also at a valid code.

**O(n log n)** — n heap operations at O(log n) each.

> 💡 Huffman is a genuinely satisfying capstone: HashMap for counting, heap for selection, tree for structure, greedy for strategy. If you want one problem that proves the pieces fit together, it's this one.

## 4. Sqrt Decomposition

A middle ground between a plain array and a segment tree (note 24) for range queries.

**The idea:** split the array into blocks of size √n and precompute each block's answer.

```
array:  [3 1 4] [1 5 9] [2 6 5] [3 5]      n=11, block ≈ 3
sums:      8       15      13      8
```

A range query takes whole blocks wholesale and handles the partial blocks at the edges element by element — at most **2√n** operations:

| | Plain array | Sqrt decomposition | Segment tree |
|---|---|---|---|
| Build | — | O(n) | O(n) |
| Query | O(n) | **O(√n)** | **O(log n)** |
| Update | O(1) | **O(1)** | O(log n) |
| Complexity to write | trivial | **easy** | fiddly |

√n is worse than log n asymptotically (for n = 1,000,000: 1,000 vs 20), but the structure is **far simpler** — an array of block sums and two loops. For moderate n, or under time pressure, it's often the better engineering choice. It also handles some queries segment trees can't express cleanly.

> 💡 The general lesson: **the asymptotically best structure isn't always the right one.** Weigh implementation cost and bug risk against the actual input size.

## 5. The Complete Sorting Picture

| Algorithm | Best | Average | Worst | Space | Stable | Compares? |
|---|---|---|---|---|---|---|
| Bubble | O(n) | O(n²) | O(n²) | O(1) | ✅ | ✅ |
| Selection | O(n²) | O(n²) | O(n²) | O(1) | ❌ | ✅ |
| Insertion | O(n) | O(n²) | O(n²) | O(1) | ✅ | ✅ |
| Merge | O(n log n) | O(n log n) | **O(n log n)** | O(n) | ✅ | ✅ |
| Quick | O(n log n) | O(n log n) | O(n²) | O(log n) | ❌ | ✅ |
| Heap | O(n log n) | O(n log n) | O(n log n) | O(1) | ❌ | ✅ |
| **Counting** | O(n+k) | O(n+k) | O(n+k) | O(k) | ✅* | ❌ |
| **Radix** | O(d(n+k)) | O(d(n+k)) | O(d(n+k)) | O(n+k) | ✅ | ❌ |
| **Cyclic** | O(n) | O(n) | O(n) | O(1) | ❌ | ❌ |

\* the prefix-sum variant.

**The bottom three beat O(n log n) only because they don't compare** — each one trades generality for speed, requiring something specific about the values.

---

## ⚠️ Common Misunderstandings
**1. Counting sort breaks the O(n log n) lower bound.**
❌ breaks it · ✅ the bound applies to **comparison** sorts. Counting sort doesn't compare, so the bound never applied.

**2. Counting sort is always faster.**
❌ always · ✅ only when **k = O(n)**. Sorting 10 values ranging to a million allocates a million-element array.

**3. Radix sort works with any per-digit sort.**
❌ any · ✅ it must be **stable**, or earlier passes' ordering is destroyed and the algorithm fails.

**4. Huffman gives every character the same code length.**
❌ same · ✅ that's *fixed-width* encoding. Huffman's whole point is **variable** length by frequency.

**5. Huffman codes could be ambiguous.**
❌ ambiguous · ✅ the **prefix property** guarantees otherwise — every character is a leaf, so no code prefixes another.

**6. Sqrt decomposition is obsolete given segment trees.**
❌ obsolete · ✅ O(√n) vs O(log n), but far simpler to write and debug. For moderate n it's often the better choice.

**7. Greedy algorithms are usually wrong.**
❌ usually · ✅ when the problem has the right structure — as Huffman does — greedy is **provably optimal**. The skill is recognising when.

## Interview Angles
- **"Can you sort faster than O(n log n)?"** — Yes, if you don't compare: counting, radix, cyclic (note 10). Name the constraint each requires.
- **"Sort ages 0–120 for a million people."** — Counting sort, O(n), because k is tiny and fixed. A great "recognise the constraint" question.
- **"Why must radix sort's inner sort be stable?"** — Each pass must preserve the previous pass's order.
- **"Explain Huffman coding."** — Greedy + min-heap + tree. Mention the prefix property and why it makes decoding unambiguous.
- **"When is greedy correct?"** — When local optimality implies global — Huffman, interval scheduling, MST. Otherwise you need DP.
- **"Segment tree or something simpler?"** — Offering sqrt decomposition shows engineering judgment, not just algorithm recall.

## Related · Next
- **Related:** [[Sorting]] (10 — the O(n²) sorts and cyclic sort) · [[Heaps and HashMaps]] (25 — Huffman's building blocks) · [[Complexity Analysis]] (13 — the lower bound) · [[Trees I]] (23)
- **Practice:** implement counting sort, then radix sort on top of it (and verify that using an unstable inner sort breaks it). Then build a Huffman tree by hand for `"aaaaabbcd"` and check you get 15 bits.
- **Next:** [27 — Large Numbers & File Handling](27-LargeNumbers_and_FileIO_Notes.md)

---

## 🔁 Rapid Revision (self-test)
<details><summary>1. How do counting and radix sort beat O(n log n)?</summary>

They don't **compare**. The lower bound applies only to comparison sorts; these exploit the structure of the values themselves.
</details>

<details><summary>2. When is counting sort actually a good idea?</summary>

When the value range k is small and dense — k = O(n). Otherwise you allocate an enormous count array for few elements.
</details>

<details><summary>3. Why must radix sort's per-digit sort be stable?</summary>

Each pass must preserve the ordering established by the previous digit. An unstable inner sort destroys it and the algorithm fails.
</details>

<details><summary>4. The four steps of Huffman coding?</summary>

Count frequencies (HashMap) → put them in a min-heap → repeatedly merge the two smallest into a new node → read codes as root-to-leaf paths (left 0, right 1).
</details>

<details><summary>5. What is the prefix property and why does it matter?</summary>

No code is a prefix of another, guaranteed because every character sits at a **leaf**. It makes decoding unambiguous with no separators.
</details>

<details><summary>6. Why is Huffman greedy, and is greedy safe here?</summary>

It always merges the two least frequent nodes and never reconsiders. For this problem greedy is **provably optimal** — which is not true in general.
</details>

<details><summary>7. Sqrt decomposition vs segment tree?</summary>

O(√n) query and O(1) update vs O(log n) for both — asymptotically worse, but far simpler to implement. Often the better engineering choice for moderate n.
</details>

<details><summary>8. Which sorts are stable, and which beat O(n log n)?</summary>

Stable: bubble, insertion, merge, radix (and counting with prefix sums). Beating O(n log n): counting, radix, cyclic — all non-comparison.
</details>
