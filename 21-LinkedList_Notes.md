---
type: foundation
title: Linked List — Singly, Doubly, Circular, and the Fast/Slow Pointer
tags: [foundations, java, linked-list, nodes, pointers, fast-slow, references, pre-phase-0]
created: 2026-09-29
status: seen
source: "Kunal Kushwaha · Community Classroom — lecture 18 LinkedList"
related: ["[[OOP I]]", "[[Arrays]]", "[[Linked List]]", "[[Two Pointers]]"]
---

# 21 · Linked List

> 📁 Part 21 of 28 in [Ytube dsa/](README.md) · **Prev:** [20 — OOP III](20-OOP_Advanced_Notes.md) · **Next:** [22 — Stacks & Queues](22-Stacks_and_Queues_Notes.md)

## Introduction
An array stores elements **contiguously** — one block, indexable by arithmetic. A **linked list** stores them **scattered**, each node holding a value plus a reference to the next.

That single change flips every trade-off: insertion becomes O(1) instead of O(n), but random access becomes O(n) instead of O(1). This is also the first structure you *build* rather than use — and it's built entirely out of note 18's classes and note 06's references.

## Characteristics
- **Node-based:** each node holds a value and a pointer to the next.
- **No random access:** reaching index *k* means walking *k* links.
- **O(1) insert/delete at a known position** — just rewire references.
- **Grows without limit:** no resizing, no capacity.
- **Extra memory per element** for the pointer.

---

## 1. The Node

```java
public class LL {
    private Node head;
    private Node tail;
    private int size;

    private class Node {
        int value;
        Node next;
        Node(int value) { this.value = value; }
        Node(int value, Node next) { this.value = value; this.next = next; }
    }
}
```

That's it — a value and a reference to another object of the same class. **A node pointing at a node is the entire idea.**

`head`, `tail` and `size` are bookkeeping: `head` is the entry point, `tail` makes appending O(1) instead of O(n), and `size` makes `size()` O(1).

> 💡 `Node` is an **inner class** here (note 18) — it has no meaning outside the list, so it doesn't belong at the top level. In DSA problems you'll usually see a plain `class ListNode { int val; ListNode next; }` with public fields, because there's no invariant to protect.

## 2. Insertion

```java
public void insertFirst(int val) {
    Node node = new Node(val);
    node.next = head;          // 1. point the new node at the old head
    head = node;               // 2. move head to the new node
    if (tail == null) tail = head;    // first ever insert: tail is this too
    size += 1;
}

public void insertLast(int val) {
    if (tail == null) { insertFirst(val); return; }
    Node node = new Node(val);
    tail.next = node;          // old tail points at the new node
    tail = node;               // tail moves
    size++;
}
```

**Order matters enormously in step 1–2.** Write `head = node` first and you lose the reference to the rest of the list — it becomes garbage (note 01) and is unrecoverable. **Always attach the new node's `next` before moving `head`.**

```java
public void insert(int val, int index) {
    if (index == 0)    { insertFirst(val); return; }
    if (index == size) { insertLast(val);  return; }

    Node temp = head;
    for (int i = 1; i < index; i++) {      // walk to the node BEFORE the target
        temp = temp.next;
    }
    Node node = new Node(val, temp.next);   // new node points at the rest
    temp.next = node;                       // previous node points at the new one
    size++;
}
```

Inserting in the middle is **O(n) to find the spot, O(1) to do the insert**. That distinction matters: if you already hold the node, insertion is genuinely constant.

**Verified:**
```
built: 1 -> 2 -> 3 -> END
```

## 3. Deletion

```java
public int deleteLast() {
    if (size <= 1) return deleteFirst();
    Node secondLast = get(size - 2);
    int val = tail.value;
    tail = secondLast;
    tail.next = null;          // cut the link — the old tail becomes garbage
    size--;
    return val;
}
```

> ⚠️ **Deleting the last element of a singly linked list is O(n)**, even with a `tail` pointer — you need the *second*-to-last node, and the only way to find it is to walk from the head. This is precisely the weakness a **doubly** linked list fixes.

## 4. Reversal — the Classic

```java
static Node reverse(Node head) {
    Node prev = null, cur = head;
    while (cur != null) {
        Node next = cur.next;      // 1. SAVE the rest before you destroy the link
        cur.next = prev;           // 2. flip this node's pointer backwards
        prev = cur;                // 3. advance prev
        cur = next;                // 4. advance cur
    }
    return prev;                   // prev is the new head
}
```
**Verified:**
```
built:    1 -> 2 -> 3 -> END
reversed: 3 -> 2 -> 1 -> END
```

Three pointers, four lines, and **step 1 is the whole trick** — `cur.next = prev` destroys the forward link, so you must save `next` before overwriting it. Skipping that line loses the rest of the list instantly.

**O(n) time, O(1) space.** Write this out until it's automatic; it's one of the most-asked linked-list questions.

## 5. Fast & Slow Pointers (Floyd's Algorithm)

Move one pointer one step and another two steps. Their relationship reveals structure.

**Cycle detection:**
```java
static boolean hasCycle(Node head) {
    Node slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;       // they met — there's a loop
    }
    return false;                             // fast hit the end — no loop
}
```
**Verified:**
```
hasCycle (no cycle): false
hasCycle (cycle made): true
```

If there's a cycle, the fast pointer laps the slow one and they collide. If there isn't, `fast` runs off the end. **O(n) time, O(1) space** — the `HashSet` alternative costs O(n) space.

The same idea solves:
- **Find the middle** — when `fast` reaches the end, `slow` is at the midpoint
- **Nth node from the end** — start `fast` n steps ahead, then move both
- **Cycle start** — after they meet, reset one to head and advance both one step at a time

Note `fast != null && fast.next != null` — checking **both** avoids a `NullPointerException` on `fast.next.next` for even-length lists. Order matters here too: `fast != null` must be tested first (short-circuit `&&`).

## 6. Doubly and Circular Lists

**Doubly linked** — each node also points backwards:
```java
class Node { int value; Node next; Node prev; }
```
Costs one more reference per node, buys backward traversal and **O(1) deletion of a known node** (you can reach its predecessor). This is what `java.util.LinkedList` actually is, and what `LinkedHashMap` uses to preserve insertion order.

**Circular** — the last node points back to the head instead of `null`. Traversal has no natural end, so you loop `do { ... } while (node != head)`. Used for round-robin scheduling and circular buffers.

| | Singly | Doubly | Circular |
|---|---|---|---|
| Pointers per node | 1 | 2 | 1 (or 2) |
| Backward traversal | ❌ | ✅ | ❌ (singly) |
| Delete a known node | O(n) | **O(1)** | O(n) |
| Memory overhead | lowest | highest | lowest |

## 7. Array vs Linked List

| Operation | Array / ArrayList | Linked List |
|---|---|---|
| Access by index | **O(1)** | O(n) |
| Search | O(n) | O(n) |
| Insert at front | O(n) — shift everything | **O(1)** |
| Insert at end | O(1) amortised | **O(1)** with a tail |
| Insert in middle | O(n) shift | O(n) find + O(1) rewire |
| Delete at front | O(n) | **O(1)** |
| Memory per element | just the value | value + pointer(s) |
| Cache friendliness | **excellent** (contiguous) | poor (scattered) |

> 💡 **In practice, `ArrayList` usually wins** even where the table suggests otherwise. Its contiguous memory is cache-friendly, and modern CPUs fetch entire cache lines — so walking an array is dramatically faster per element than chasing pointers across the heap. Reach for a linked list when you genuinely need O(1) insert/delete at a position you already hold, or when you're building a structure on top of it (stack, queue, LRU cache, adjacency list).

## 8. Where This Leads

The node-and-pointer idea is the foundation of almost everything left:

- **Stack / Queue** (note 22) — a linked list with restricted access points
- **Tree** (note 23) — a node with *two* `next` pointers, called `left` and `right`
- **Graph** — nodes with a list of neighbours
- **LRU cache** — a doubly linked list plus a HashMap

Getting comfortable rewiring pointers here is what makes trees feel easy later.

---

## ⚠️ Common Misunderstandings
**1. `head = node` then `node.next = head`.**
❌ that order · ✅ **attach first, then move `head`** — otherwise the rest of the list is orphaned and unrecoverable.

**2. A `tail` pointer makes `deleteLast` O(1).**
❌ O(1) · ✅ still **O(n)** in a singly linked list — you need the second-to-last node. Only a doubly linked list fixes it.

**3. Reversal doesn't need a temp variable.**
❌ doesn't · ✅ you must save `cur.next` **before** overwriting it, or you lose the remainder of the list.

**4. `while (fast.next != null)` is enough.**
❌ enough · ✅ check `fast != null && fast.next != null`, in that order, or even-length lists throw `NullPointerException`.

**5. Cycle detection needs a HashSet.**
❌ needs · ✅ fast/slow pointers do it in **O(1) space**.

**6. Linked lists are faster than arrays for inserts.**
❌ generally · ✅ only when you already hold the position. Finding it is O(n), and cache locality usually makes `ArrayList` win anyway.

**7. `LinkedList` is a good default in Java.**
❌ good default · ✅ `ArrayList` is nearly always better. `LinkedList` earns its place mainly as a `Deque`.

## JavaScript Comparison
| | Java | JavaScript |
|---|---|---|
| Node | `class Node { int value; Node next; }` | `{ value, next }` object literal |
| Null terminator | `null` | `null` |
| Built-in | `java.util.LinkedList` (doubly) | none — arrays only |
| Typical use | rarely; `ArrayList` preferred | rarely; arrays are optimised |

> 💡 JS has no linked list at all — arrays are so heavily optimised that one is almost never worth it. You build it in both languages for the same reason: interviews and understanding.

## Interview Angles
- **"Reverse a linked list."** (LeetCode 206) — Near-guaranteed. Three pointers, save-flip-advance. Be ready for the recursive version too.
- **"Detect a cycle."** (141) — Floyd's fast/slow. Then: "find where the cycle starts" (142) — reset one pointer to head.
- **"Find the middle."** (876) — Fast/slow; when fast ends, slow is the middle.
- **"Merge two sorted lists."** (21) — Same merge step as merge sort (note 16), on pointers.
- **"Remove the nth node from the end."** (19) — Two pointers n apart, one pass.
- **"Array vs linked list?"** — Give the table, then the cache-locality point. That last part is what separates a textbook answer from an experienced one.
- **Always ask about edge cases:** empty list, single node, deleting the head.

## Related · Next
- **Related:** [[OOP I]] (18 — the class this is built from) · [[Arrays]] (07 — the contrast) · [[Linked List]] (2.4) · [[Two Pointers]] (3.1)
- **Practice:** build the list from scratch with `insertFirst`, `insertLast`, `delete` and `reverse`. Then solve 206, 141, 876 and 21 — all four are the same pointer discipline.
- **Next:** [22 — Stacks & Queues](22-Stacks_and_Queues_Notes.md)

---

## 🔁 Rapid Revision (self-test)
<details><summary>1. What is a node, and what makes a linked list?</summary>

A value plus a reference to the next node. A node pointing at a node, repeated, is the whole structure.
</details>

<details><summary>2. Why must `node.next = head` come before `head = node`?</summary>

Moving `head` first orphans the rest of the list — nothing references it any more and it's garbage collected. Attach, then move.
</details>

<details><summary>3. Why is `deleteLast` O(n) even with a tail pointer?</summary>

You need the second-to-last node to become the new tail, and a singly linked list can only reach it by walking from the head.
</details>

<details><summary>4. The three pointers in reversal, and the critical step?</summary>

`prev`, `cur`, `next`. Critical: **save `cur.next` before overwriting it** with `prev`, or the rest of the list is lost. O(n) time, O(1) space.
</details>

<details><summary>5. How does cycle detection work, and why check both conditions?</summary>

Fast/slow pointers — with a cycle the fast one laps the slow one and they meet. Check `fast != null && fast.next != null` in that order, or even-length lists throw NPE.
</details>

<details><summary>6. Three problems fast/slow pointers solve?</summary>

Cycle detection, finding the middle, and the nth node from the end — all O(n) time, O(1) space.
</details>

<details><summary>7. What does a doubly linked list buy, and at what cost?</summary>

Backward traversal and O(1) deletion of a known node; costs one extra reference per node. It's what `java.util.LinkedList` is.
</details>

<details><summary>8. Why is `ArrayList` usually faster in practice despite the table?</summary>

Contiguous memory is cache-friendly — CPUs fetch whole cache lines, so walking an array beats chasing scattered pointers.
</details>
