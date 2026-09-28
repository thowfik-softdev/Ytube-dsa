---
type: foundation
title: Trees II — AVL Self-Balancing and Segment Trees
tags: [foundations, java, trees, avl, self-balancing, rotations, segment-tree, range-query, pre-phase-0]
created: 2026-09-29
status: seen
source: "Kunal Kushwaha · Community Classroom — lecture 20 Trees (AVL, Segment trees)"
related: ["[[Trees I]]", "[[Complexity Analysis]]", "[[Tree]]", "[[Binary Search]]"]
---

# 24 · Trees II — AVL & Segment Trees

> 📁 Part 24 of 28 in [Ytube dsa/](README.md) · **Prev:** [23 — Trees I](23-Trees_Basics_Notes.md) · **Next:** [25 — Heaps & HashMaps](25-Heaps_and_HashMaps_Notes.md)

## Introduction
Note 23 ended on a problem: a plain BST degenerates into a linked list when fed sorted data, and all its O(log n) guarantees evaporate.

Two structures answer two different needs:

- **AVL trees** keep a BST balanced automatically, restoring the O(log n) guarantee.
- **Segment trees** answer *range* questions ("sum from index 3 to 9") in O(log n) instead of O(n).

Both are a step up in difficulty, and both are rarer in interviews than notes 21–23 — but the *ideas* (rotation, tree-over-a-range) are worth having.

---

# Part A — AVL Trees

## 1. The Balance Factor

An **AVL tree** is a BST that keeps itself balanced. The invariant:

> **For every node, the heights of its left and right subtrees differ by at most 1.**

```java
int balanceFactor(Node node) {
    return height(node.left) - height(node.right);
}
```

- `0`, `1`, `-1` → balanced
- `> 1` → **left-heavy**, needs fixing
- `< -1` → **right-heavy**, needs fixing

Each node caches its own height so the check is O(1) rather than a subtree walk:

```java
class Node {
    int value;
    Node left, right;
    int height;                       // cached, updated on the way back up
}
```

## 2. Rotations — the Repair Operation

A **rotation** rearranges three nodes to reduce height while **preserving the BST ordering**. That last part is what makes it safe: an inorder traversal produces the identical sequence before and after.

**Right rotation** (fixes a left-heavy chain):

```
      z                  y
     / \               /   \
    y   T4   ---->    x     z
   / \               / \   / \
  x   T3            T1 T2 T3 T4
 / \
T1  T2
```

```java
Node rightRotate(Node z) {
    Node y = z.left;
    Node T3 = y.right;

    y.right = z;          // y becomes the new root
    z.left = T3;          // z adopts y's old right subtree

    z.height = 1 + Math.max(height(z.left), height(z.right));   // update BOTTOM-UP
    y.height = 1 + Math.max(height(y.left), height(y.right));

    return y;             // the new subtree root
}
```

**Left rotation** is the mirror image.

> 💡 Note the height updates: **`z` before `y`**, because `z` is now the child. Updating in the wrong order gives wrong heights and the tree silently stops balancing.

## 3. The Four Cases

After inserting, walk back up and fix the first unbalanced node. Which rotation depends on *where* the new node went:

| Case | Condition | Fix |
|---|---|---|
| **Left-Left** | bf > 1 and value < node.left.value | right rotate |
| **Right-Right** | bf < −1 and value > node.right.value | left rotate |
| **Left-Right** | bf > 1 and value > node.left.value | left rotate the child, **then** right rotate |
| **Right-Left** | bf < −1 and value < node.right.value | right rotate the child, **then** left rotate |

```java
Node insert(Node node, int value) {
    if (node == null) return new Node(value);
    if (value < node.value) node.left  = insert(node.left, value);
    else                    node.right = insert(node.right, value);

    node.height = 1 + Math.max(height(node.left), height(node.right));
    return rotate(node, value);            // rebalance on the way back up
}
```

The **recurse-and-reassign** shape from note 23 is what makes this work — each call returns its (possibly rotated) subtree root, and the parent just reassigns.

> 💡 **The zig-zag cases need two rotations.** A Left-Right shape is a "bend"; one rotation straightens it into Left-Left, and the second fixes the height. Trying to fix a bend with a single rotation just moves it.

## 4. Why It Matters

| | Plain BST | AVL |
|---|---|---|
| Search | O(log n) … or O(n) | **O(log n) guaranteed** |
| Insert | O(log n) … or O(n) | **O(log n)** (+ O(1) rotations) |
| Sorted input | degenerates to a list | stays balanced |
| Extra cost | none | a height field + rotations |

**Inserting 1,2,3,4,5 into a plain BST gives a 5-deep chain; into an AVL tree it gives a 3-level balanced tree.** That's the entire point.

> 💡 **Java's `TreeMap` and `TreeSet` are red-black trees**, not AVL. Red-black trees balance more loosely (height up to 2·log n), so they rotate less on insert — better for write-heavy use. AVL is more rigidly balanced, so lookups are slightly faster. Both guarantee O(log n); knowing *why* two variants exist is the interesting part.

---

# Part B — Segment Trees

## 5. The Problem

Given an array, answer many queries like *"what's the sum of elements 3 through 9?"* and also *"update element 5 to a new value"*.

| Approach | Query | Update |
|---|---|---|
| Plain array | O(n) | O(1) |
| Prefix-sum array | **O(1)** | O(n) — rebuild everything |
| **Segment tree** | **O(log n)** | **O(log n)** |

Prefix sums are perfect for a **static** array. The moment updates are involved, they collapse. A segment tree is the structure for **both** operations being frequent.

## 6. The Structure

Each node stores the answer for a **range**; leaves are single elements, and each internal node combines its two children.

```
                 [0-6] sum=42
                /            \
        [0-3] sum=18       [4-6] sum=24
        /       \           /        \
   [0-1]=7   [2-3]=11   [4-5]=15   [6-6]=9
```

```java
class Node {
    int data;
    int startInterval, endInterval;
    Node left, right;
}

Node construct(int[] arr, int start, int end) {
    if (start == end) {                              // leaf: one element
        Node leaf = new Node(start, end);
        leaf.data = arr[start];
        return leaf;
    }
    Node node = new Node(start, end);
    int mid = (start + end) / 2;
    node.left  = construct(arr, start, mid);
    node.right = construct(arr, mid + 1, end);
    node.data  = node.left.data + node.right.data;   // combine children
    return node;
}
```

Build is **O(n)**; the tree holds about **2n** nodes (n leaves plus n−1 internal), so space is O(n).

## 7. Query — Three Cases

```java
int query(Node node, int qs, int qe) {
    if (node.startInterval >= qs && node.endInterval <= qe) {
        return node.data;                            // 1. FULLY inside — take it
    }
    if (node.startInterval > qe || node.endInterval < qs) {
        return 0;                                    // 2. NO overlap — identity value
    }
    return query(node.left, qs, qe) + query(node.right, qs, qe);   // 3. PARTIAL — split
}
```

Every segment-tree query is these three cases:

1. **Total overlap** → return the cached answer, don't descend
2. **No overlap** → return the operation's identity (0 for sum, `MAX_VALUE` for min)
3. **Partial overlap** → recurse into both children and combine

Case 1 is where the O(log n) comes from — a whole subtree is answered without visiting it.

> 💡 **The identity value must match the operation.** Sum → `0`. Min → `Integer.MAX_VALUE`. Max → `Integer.MIN_VALUE`. Product → `1`. Returning `0` from a *min* query would silently make every answer 0 — a genuinely nasty bug.

## 8. Update

```java
void update(Node node, int index, int value) {
    if (node.startInterval == node.endInterval) {
        node.data = value;                           // leaf: set it
        return;
    }
    int mid = (node.startInterval + node.endInterval) / 2;
    if (index <= mid) update(node.left, index, value);
    else               update(node.right, index, value);
    node.data = node.left.data + node.right.data;    // recompute on the way back UP
}
```

Only the path from root to leaf changes — **O(log n)** nodes. Recomputing on the way back up is postorder (note 23 §3): you need the children's new values before you can fix the parent.

**Lazy propagation** extends this to *range* updates ("add 5 to everything from 3 to 9") by marking a node as "pending" instead of pushing the change to every leaf. Worth knowing the name; rarely needed outside competitive programming.

## 9. Complexity

| | Build | Query | Update | Space |
|---|---|---|---|---|
| AVL tree | O(n log n) | O(log n) | O(log n) | O(n) |
| Segment tree | **O(n)** | **O(log n)** | **O(log n)** | O(n) (≈2n nodes) |

---

## ⚠️ Common Misunderstandings
**1. Rotations can break the BST ordering.**
❌ can break · ✅ they **preserve** it — inorder output is identical before and after. That's what makes them safe.

**2. One rotation fixes any imbalance.**
❌ any · ✅ the zig-zag cases (Left-Right, Right-Left) need **two** — straighten the bend, then rotate.

**3. Update heights in any order.**
❌ any · ✅ **bottom-up** — the new child before the new root, or the heights are wrong and balancing silently fails.

**4. Java's `TreeMap` is an AVL tree.**
❌ AVL · ✅ **red-black** — looser balance, fewer rotations on insert, better for write-heavy workloads.

**5. Prefix sums are always enough for range queries.**
❌ always · ✅ only for **static** arrays. One update costs O(n) to rebuild; a segment tree does both in O(log n).

**6. A segment tree query always walks to the leaves.**
❌ always · ✅ a **fully covered** node returns immediately without descending. That's the whole optimisation.

**7. Returning 0 for "no overlap" always works.**
❌ always · ✅ only for sum. Min needs `Integer.MAX_VALUE`, max needs `MIN_VALUE`, product needs `1`.

**8. A segment tree needs exactly n nodes.**
❌ n · ✅ about **2n** — n leaves plus n−1 internal nodes. Array-backed implementations usually allocate 4n to be safe.

## Interview Angles
- **"Why do we need self-balancing trees?"** — Plain BSTs degenerate to O(n) on sorted input. AVL/red-black restore the guarantee.
- **"What's a rotation?"** — A local rearrangement that reduces height while preserving inorder order. Be able to draw the right-rotation diagram.
- **"AVL vs red-black?"** — AVL is more strictly balanced (faster lookups); red-black rotates less (faster inserts). Java uses red-black for `TreeMap`.
- **"Range sum with updates?"** — Segment tree, O(log n) both. If updates never happen, say **prefix sums, O(1)** — recognising when the simpler tool suffices is the better answer.
- **"What's lazy propagation?"** — Defer range updates by marking nodes pending; push down only when needed.
- **Be honest about depth here.** AVL and segment trees are uncommon in standard interviews. Knowing *why they exist* matters more than reciting the rotation code.

## Related · Next
- **Related:** [[Trees I]] (23) · [[Complexity Analysis]] (13) · [[Tree]] (4.1) · [[Binary Search]] (09 — the same halving idea)
- **Practice:** insert 1,2,3,4,5 into a plain BST and draw the chain; then do it with AVL rotations and draw the balanced result. Build a segment tree for `{1,3,5,7,9,11}` and hand-trace a query for range 1–3.
- **Next:** [25 — Heaps & HashMaps](25-Heaps_and_HashMaps_Notes.md)

---

## 🔁 Rapid Revision (self-test)
<details><summary>1. What is the AVL invariant, and what is a balance factor?</summary>

Every node's left and right subtree heights differ by at most 1. Balance factor = height(left) − height(right); outside {−1, 0, 1} means rebalance.
</details>

<details><summary>2. What does a rotation preserve, and why does that matter?</summary>

The **BST ordering** — inorder output is unchanged. That's what makes rearranging nodes safe.
</details>

<details><summary>3. Which cases need two rotations, and why?</summary>

Left-Right and Right-Left — the zig-zags. The first rotation straightens the bend into a straight line; the second fixes the height.
</details>

<details><summary>4. In what order do you update heights during a rotation?</summary>

Bottom-up — the node that became the child first, then the new subtree root.
</details>

<details><summary>5. AVL vs red-black, and which does Java use?</summary>

AVL is strictly balanced (faster lookups); red-black balances loosely with fewer rotations (faster inserts). Java's `TreeMap`/`TreeSet` use **red-black**.
</details>

<details><summary>6. When do prefix sums beat a segment tree, and when do they fail?</summary>

Static arrays — O(1) queries. They fail with updates, which cost O(n) to rebuild; a segment tree does both in O(log n).
</details>

<details><summary>7. The three cases of a segment-tree query?</summary>

Total overlap → return the node's cached value. No overlap → return the operation's identity. Partial → recurse both children and combine.
</details>

<details><summary>8. What identity value does each operation need for "no overlap"?</summary>

Sum → 0 · Min → `Integer.MAX_VALUE` · Max → `Integer.MIN_VALUE` · Product → 1. Using 0 for a min query silently returns 0 everywhere.
</details>
