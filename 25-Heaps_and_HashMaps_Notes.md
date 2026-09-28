---
type: foundation
title: Heaps & HashMaps — Priority Queues, Hashing, Collisions and Karp-Rabin
tags: [foundations, java, heap, priority-queue, heapsort, hashmap, hashing, collisions, karp-rabin, pre-phase-0]
created: 2026-09-29
status: seen
source: "Kunal Kushwaha · Community Classroom — lecture 24 Heaps, lecture 25 Hashmaps"
related: ["[[Trees I]]", "[[OOP III]]", "[[Heap]]", "[[Hashing]]"]
---

# 25 · Heaps & HashMaps

> 📁 Part 25 of 28 in [Ytube dsa/](README.md) · **Prev:** [24 — Trees II](24-Trees_Advanced_Notes.md) · **Next:** [26 — Advanced Sorting](26-Advanced_Sorting_Notes.md)

## Introduction
Two structures that together solve an enormous share of interview problems.

A **heap** always knows its smallest (or largest) element — perfect for "top K" and scheduling. A **HashMap** finds anything by key in O(1) — the single most common way to turn an O(n²) brute force into O(n).

---

# Part A — Heaps

## 1. What a Heap Is

A **complete binary tree** (every level filled left-to-right except possibly the last) with the **heap property**:

- **Min-heap:** every parent ≤ its children → the **minimum** is at the root
- **Max-heap:** every parent ≥ its children → the **maximum** is at the root

> ⚠️ **A heap is not sorted.** It only guarantees the *root* is the extreme. The rest is partially ordered — that weaker promise is exactly why insertion is O(log n) rather than O(n).

## 2. Stored in an Array

Because the tree is *complete*, it fits perfectly into an array with no pointers:

```
index:   0   1   2   3   4   5   6
value:  10  20  15  40  50  100 25

For node at index i:
    left child  = 2i + 1
    right child = 2i + 2
    parent      = (i - 1) / 2
```

That arithmetic replaces every pointer — a heap is the one tree that needs no `Node` class. Contiguous storage also makes it cache-friendly (note 21 §7).

## 3. The Two Operations

**Insert — "up-heapify":** add at the end, then swim upward while smaller than the parent.

```java
public void insert(int value) {
    list.add(value);
    upheap(list.size() - 1);
}

private void upheap(int index) {
    if (index == 0) return;
    int parent = (index - 1) / 2;
    if (list.get(index) < list.get(parent)) {
        swap(index, parent);
        upheap(parent);                // keep swimming
    }
}
```

**Remove — "down-heapify":** take the root (the answer), move the last element to the root, then sink it while larger than a child.

```java
public int remove() {
    int temp = list.get(0);                       // the answer
    list.set(0, list.get(list.size() - 1));       // last element to the root
    list.remove(list.size() - 1);
    downheap(0);
    return temp;
}

private void downheap(int index) {
    int min = index;
    int left = 2 * index + 1, right = 2 * index + 2;
    if (left  < list.size() && list.get(min) > list.get(left))  min = left;
    if (right < list.size() && list.get(min) > list.get(right)) min = right;
    if (min != index) { swap(min, index); downheap(min); }
}
```

Both walk one root-to-leaf path — **O(log n)**, since a complete tree of n nodes has height log₂n.

## 4. Java's `PriorityQueue`

```java
PriorityQueue<Integer> min = new PriorityQueue<>();                        // min-heap
PriorityQueue<Integer> max = new PriorityQueue<>(Collections.reverseOrder()); // max-heap
```
**Verified** — adding 5, 1, 8, 3:
```
min-heap polls: 1 3
max-heap polls: 8 5
```

| Operation | Cost |
|---|---|
| `add` / `offer` | O(log n) |
| `poll` (remove root) | O(log n) |
| `peek` | **O(1)** |
| `contains` / arbitrary remove | O(n) |

> ⚠️ **Java's `PriorityQueue` is a min-heap by default** — `poll()` gives the *smallest*. For a max-heap pass `Collections.reverseOrder()` or a comparator. Forgetting this is the most common heap bug.

> ⚠️ **Iterating a `PriorityQueue` does not give sorted order** — and neither does printing it. Only repeated `poll()` does. The internal array is heap-ordered, not sorted.

## 5. Heapsort and Top-K

**Heapsort:** build a heap, then poll everything. **O(n log n)** time, and O(1) space if done in place on the input array. Not stable, and usually slower than quicksort in practice (worse cache behaviour), but its worst case is guaranteed.

**Top-K — the real payoff.** To find the K largest of n elements, keep a **min-heap of size K**:

```java
PriorityQueue<Integer> heap = new PriorityQueue<>();      // MIN-heap for K LARGEST
for (int num : nums) {
    heap.add(num);
    if (heap.size() > K) heap.poll();                     // drop the smallest
}
```

**O(n log K)** time, **O(K)** space — versus O(n log n) to sort everything. When K is small and n is huge, that's a decisive win, and it works on a stream where sorting isn't even possible.

> 💡 **The counter-intuitive bit:** use a **min**-heap to find the **largest** elements. The heap holds the K best so far, and its root is the *weakest* of them — the one to evict when something better arrives.

---

# Part B — HashMaps

## 6. The Idea

A hash map turns a key into an array index via a **hash function**, giving O(1) average access.

```
key "apple"  →  hashCode()  →  compress (% capacity)  →  bucket index  →  store (key, value)
```

Three steps: hash the key to an integer, compress it into the array range, store the entry in that bucket. Lookup repeats the same steps and finds the entry directly — **no searching**.

## 7. Collisions

Two keys can hash to the same bucket. Two standard fixes:

**Chaining** (what Java uses) — each bucket holds a list:
```
bucket 3 → ("apple", 5) → ("grape", 9)
```
On lookup, find the bucket, then walk the short chain comparing with `equals`.

**Open addressing** — probe for the next free slot instead of chaining.

> 💡 **Java's `HashMap` upgrades a bucket's list to a red-black tree (note 24) once it holds 8+ entries**, so a pathological collision case degrades to **O(log n)** rather than O(n). That change landed in Java 8 and is a great detail to know.

**Load factor** = entries / buckets. Java resizes (doubling and rehashing everything) once it passes **0.75** — the same amortised doubling as `ArrayList` (note 07) and `StringBuilder` (note 11).

## 8. Why `equals` and `hashCode` Matter

This is where note 20 §3 becomes load-bearing:

- **`hashCode`** picks the bucket
- **`equals`** identifies the right entry within it

Break the contract — equal objects with different hash codes — and they land in **different buckets**. The map then holds duplicates and lookups miss, with no error at all.

**Verified in note 20:** a `HashSet` given two equal `Point`s held **1** entry, because both methods were overridden consistently. Remove `hashCode` and it holds 2.

## 9. Using It

```java
Map<String, Integer> m = new HashMap<>();
for (String w : "a b a c a b".split(" ")) {
    m.put(w, m.getOrDefault(w, 0) + 1);
}
System.out.println(m);
System.out.println(new TreeMap<>(m));
```
**Output**
```
frequency: {a=3, b=2, c=1}
TreeMap (sorted keys): {a=3, b=2, c=1}
```

`getOrDefault` is the frequency-counting idiom — no null check needed. (`merge(w, 1, Integer::sum)` does the same thing.)

| Implementation | Order | Lookup |
|---|---|---|
| `HashMap` | **none guaranteed** | O(1) avg |
| `LinkedHashMap` | insertion order | O(1) avg |
| `TreeMap` | **sorted by key** | O(log n) |

> ⚠️ `HashMap` ordering is **not guaranteed** and must never be relied on. It happened to print `{a=3, b=2, c=1}` here; with different keys or a different JDK it can differ. Use `LinkedHashMap` for insertion order or `TreeMap` for sorted.

## 10. The Pattern That Matters Most

**Trade O(n) space for O(1) lookup, turning O(n²) into O(n).**

```java
// Two Sum — brute force O(n²)
for (int i = 0; i < n; i++)
    for (int j = i+1; j < n; j++)
        if (nums[i] + nums[j] == target) return new int[]{i, j};

// With a HashMap — O(n)
Map<Integer, Integer> seen = new HashMap<>();
for (int i = 0; i < n; i++) {
    int complement = target - nums[i];
    if (seen.containsKey(complement)) return new int[]{seen.get(complement), i};
    seen.put(nums[i], i);
}
```

Same answer, one pass. **If a problem involves "have I seen this before?", "how many times?", or "find the pair//group", reach for a HashMap first.** Anagram grouping, duplicate detection, frequency counts, caching — all the same move.

## 11. Karp–Rabin — Rolling Hash

Substring search by **hashing** rather than comparing. Naively, checking every window costs O(n·m); Karp–Rabin computes each window's hash in **O(1)** from the previous one by removing the outgoing character and adding the incoming one.

**O(n + m) average.** Hash collisions mean a match must still be verified character by character, so the worst case remains O(n·m) — but with a good hash it effectively never happens. The rolling-hash idea also underpins plagiarism detection and `rsync`-style diffing.

---

## ⚠️ Common Misunderstandings
**1. A heap is sorted.**
❌ sorted · ✅ only the **root** is guaranteed extreme. Printing or iterating a `PriorityQueue` gives heap order, not sorted order.

**2. `PriorityQueue` is a max-heap.**
❌ max · ✅ **min-heap by default.** Pass `Collections.reverseOrder()` for a max-heap.

**3. Use a max-heap for the K largest.**
❌ max · ✅ a **min-heap of size K** — the root is the weakest of the current best, so it's the one to evict. O(n log K).

**4. A heap needs a Node class.**
❌ needs · ✅ it's a complete tree, so an **array** with `2i+1` / `2i+2` / `(i-1)/2` arithmetic is enough.

**5. HashMap operations are guaranteed O(1).**
❌ guaranteed · ✅ O(1) **average**. Heavy collisions degrade it — to O(log n) in modern Java, thanks to bucket treeification.

**6. HashMap preserves insertion order.**
❌ preserves · ✅ **no ordering guarantee at all.** Use `LinkedHashMap` or `TreeMap` when order matters.

**7. Overriding `equals` is enough for keys.**
❌ enough · ✅ `hashCode` picks the bucket. Without it, equal keys land in different buckets and lookups silently fail.

**8. A mutable object is a fine HashMap key.**
❌ fine · ✅ mutating a key after insertion changes its hash — the entry becomes unreachable, sitting in the wrong bucket forever. Use immutable keys.

## JavaScript Comparison
| | Java | JavaScript |
|---|---|---|
| Hash map | `HashMap` | `Map` (or a plain object) |
| Key equality | your `equals`/`hashCode` | **reference identity** for objects |
| Ordering | none (`HashMap`) | `Map` preserves **insertion order** |
| Sorted map | `TreeMap` | none |
| Heap / priority queue | `PriorityQueue` | **none** — implement it yourself |
| Frequency count | `getOrDefault(k, 0) + 1` | `map.set(k, (map.get(k) ?? 0) + 1)` |

> 💡 Two real gaps: JS has **no priority queue** at all (you write one for every heap problem), and JS `Map` keys use reference identity with no way to customise — so two structurally equal objects are always distinct keys.

## Interview Angles
- **"Kth largest element."** (LeetCode 215) — Min-heap of size K, O(n log K). Mention quickselect (note 16) as the O(n) average alternative.
- **"Top K frequent elements."** (347) — HashMap to count, then a heap over the counts. The two structures together.
- **"Merge K sorted lists."** (23) — A min-heap holding one node per list.
- **"Two Sum."** (1) — The canonical HashMap-beats-brute-force question.
- **"Group anagrams."** (49) — HashMap keyed by the sorted string.
- **"How does HashMap work internally?"** — Hash → compress → bucket → chaining, load factor 0.75, treeification at 8. A strong, commonly-asked answer.
- **"What breaks if hashCode is inconsistent?"** — Equal keys in different buckets; lookups miss, duplicates appear, no error.

## Related · Next
- **Related:** [[Trees I]] (23 — heaps are complete binary trees) · [[OOP III]] (20 — equals/hashCode) · [[Heap]] (4.3) · [[Hashing]] (3.4)
- **Practice:** implement a min-heap with insert and remove on an `ArrayList`. Then solve Two Sum (1), Top K Frequent (347) and Kth Largest (215).
- **Next:** [26 — Advanced Sorting](26-Advanced_Sorting_Notes.md)

---

## 🔁 Rapid Revision (self-test)
<details><summary>1. What does a heap guarantee, and what doesn't it?</summary>

The **root** is the min (or max). It does **not** guarantee the rest is sorted — it's only partially ordered, which is why insert is O(log n).
</details>

<details><summary>2. How is a heap stored without pointers?</summary>

In an array, since it's a complete tree: left = 2i+1, right = 2i+2, parent = (i−1)/2.
</details>

<details><summary>3. Up-heapify vs down-heapify?</summary>

Insert adds at the end and swims **up** while smaller than the parent. Remove takes the root, moves the last element there, and sinks it **down**. Both O(log n).
</details>

<details><summary>4. Java's PriorityQueue: min or max by default?</summary>

**Min-heap** — `poll()` returns the smallest. Use `Collections.reverseOrder()` for a max-heap. Iterating it does not give sorted order.
</details>

<details><summary>5. Which heap finds the K largest elements, and why?</summary>

A **min-heap of size K**. Its root is the weakest of the current best K, so that's what you evict when a better element arrives. O(n log K), O(K) space.
</details>

<details><summary>6. How does a HashMap find a value, and how are collisions handled?</summary>

hashCode → compress to a bucket index → search that bucket with `equals`. Java chains collisions in a list, converting to a red-black tree at 8+ entries.
</details>

<details><summary>7. What is the load factor, and what happens at 0.75?</summary>

entries / buckets. At 0.75 Java doubles the capacity and rehashes — the same amortised doubling as ArrayList.
</details>

<details><summary>8. Why must HashMap keys be immutable?</summary>

Mutating a key changes its hash code, so the entry is now in the wrong bucket and can never be found again.
</details>

<details><summary>9. The one pattern to remember from this note?</summary>

Trade O(n) space for O(1) lookup to turn O(n²) into O(n). "Have I seen this?", "how many times?", "find the pair" → HashMap.
</details>
