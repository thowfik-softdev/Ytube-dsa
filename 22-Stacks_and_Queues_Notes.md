---
type: foundation
title: Stacks & Queues — LIFO, FIFO, and the Problems Only They Solve
tags: [foundations, java, stack, queue, deque, lifo, fifo, circular-queue, monotonic-stack, pre-phase-0]
created: 2026-09-29
status: seen
source: "Kunal Kushwaha · Community Classroom — lecture 19 Stacks & Queues"
related: ["[[Linked List]]", "[[Arrays]]", "[[Stack]]", "[[Queue]]", "[[Recursion I]]"]
---

# 22 · Stacks & Queues

> 📁 Part 22 of 28 in [Ytube dsa/](README.md) · **Prev:** [21 — Linked List](21-LinkedList_Notes.md) · **Next:** [23 — Trees I](23-Trees_Basics_Notes.md)

## Introduction
Both are **restricted** data structures — deliberately crippled versions of a list, where the restriction *is* the feature.

- A **stack** allows access at one end only: **LIFO** (last in, first out). A pile of plates.
- A **queue** allows insertion at one end and removal at the other: **FIFO** (first in, first out). A line at a counter.

Restricting access sounds like a loss. It isn't: it makes certain problems trivial that are awkward otherwise, and it gives you O(1) guarantees on every operation.

---

## 1. Stack — the Operations

| Operation | Does | Cost |
|---|---|---|
| `push(x)` | add to the top | O(1) |
| `pop()` | remove **and return** the top | O(1) |
| `peek()` | look at the top without removing | O(1) |
| `isEmpty()` | any elements? | O(1) |
| `isFull()` | array-backed only | O(1) |

**Verified:**
```
after push 1,2,3 -> peek=3  pop=3  pop=2
```
Last in, first out — `3` went in last and comes out first.

## 2. Building a Stack on an Array

```java
public class CustomStack {
    protected int[] data;
    private static final int DEFAULT_SIZE = 10;
    int ptr = -1;                                  // -1 means EMPTY

    public CustomStack() { this(DEFAULT_SIZE); }   // constructor chaining (note 18)
    public CustomStack(int size) { this.data = new int[size]; }

    public boolean push(int item) {
        if (isFull()) { System.out.println("Stack is full!!"); return false; }
        ptr++;
        data[ptr] = item;
        return true;
    }

    public int pop() throws StackException {
        if (isEmpty()) throw new StackException("Cannot pop from an empty stack!!");
        return data[ptr--];                        // return data[ptr], THEN decrement
    }

    public int peek() throws StackException {
        if (isEmpty()) throw new StackException("Cannot peek from an empty stack!!");
        return data[ptr];
    }

    public boolean isFull()  { return ptr == data.length - 1; }
    public boolean isEmpty() { return ptr == -1; }
}
```

The entire structure is **an array plus one integer**. `ptr` tracks the top; `-1` is the natural "empty" sentinel because it's one below the first valid index.

> 💡 **`return data[ptr--]` is post-decrement:** it returns `data[ptr]` *then* decrements. Writing `data[--ptr]` would decrement first and return the wrong element. The commented-out three-line version in the lecture code does the same thing more readably — prefer that when the terse form isn't obviously right.

> 💡 The popped slot isn't cleared. For `int[]` that's fine, but for an **object** stack you should null it out — otherwise the array keeps a reference and the object can't be garbage collected (note 01). This is a real memory-leak pattern, and `java.util.Stack` does clear the slot.

**Dynamic stack** — subclass it and grow on overflow (note 19's inheritance):
```java
public class DynamicStack extends CustomStack {
    @Override
    public boolean push(int item) {
        if (this.isFull()) {
            int[] temp = new int[data.length * 2];       // double, like ArrayList
            System.arraycopy(data, 0, temp, 0, data.length);
            data = temp;
        }
        return super.push(item);                          // then the normal push
    }
}
```
That's the exact growth strategy from note 07 §9 — and the reason `push` stays **O(1) amortised**.

## 3. Queue — and Why the Naive Version Fails

| Operation | Does | Cost |
|---|---|---|
| `offer(x)` / `add` | add at the rear | O(1) |
| `poll()` / `remove` | remove from the front | O(1) |
| `peek()` | look at the front | O(1) |

**Verified:**
```
after offer 1,2,3 -> peek=1  poll=1  poll=2
```
First in, first out — the opposite of the stack.

A naive array queue removes from the front by **shifting everything left** — O(n) per removal. Worse, if you just advance a `front` index instead, the space before it is wasted forever and the queue "fills up" despite being mostly empty.

**The circular queue** fixes both by wrapping indices with modulo:

```java
public boolean insert(int item) {
    if (isFull()) return false;
    data[end++] = item;
    end = end % data.length;         // wrap the rear around
    size++;
    return true;
}

public int remove() {
    int removed = data[front++];
    front = front % data.length;     // wrap the front around
    size--;
    return removed;
}
```

The `% data.length` is the whole idea: when an index runs off the end it comes back to `0`, so freed space at the front is reused. **O(1) for both operations, no shifting, no wasted space.** This is why `ArrayDeque` beats `LinkedList` as a queue — it's a circular array with no per-node allocation.

## 4. Java's Built-ins — Use `ArrayDeque`

```java
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1); stack.pop(); stack.peek();

Queue<Integer> queue = new ArrayDeque<>();
queue.offer(1); queue.poll(); queue.peek();
```

| Class | Use as | Verdict |
|---|---|---|
| **`ArrayDeque`** | stack **and** queue | ✅ **the default** — fast, circular array |
| `java.util.Stack` | stack | ❌ legacy, synchronised, extends `Vector` |
| `LinkedList` | queue / deque | ⚠️ works, but allocates a node per element |
| `PriorityQueue` | ordered removal | different structure — note 25 |

> ⚠️ **Don't use `java.util.Stack`.** It extends `Vector`, so it's synchronised (slow) and — worse — it exposes indexed access, letting you reach into the middle and break the abstraction. Even Java's own docs steer you to `ArrayDeque`. Every method is one letter different, so it's an easy habit to fix.

> 💡 **Prefer `offer`/`poll`/`peek` over `add`/`remove`/`element`.** The first set returns `false`/`null` on failure; the second throws. For queue algorithms, the non-throwing versions read better.

## 5. Valid Parentheses — the Canonical Stack Problem

```java
static boolean valid(String s) {
    Deque<Character> st = new ArrayDeque<>();
    for (char ch : s.toCharArray()) {
        if (ch == '(' || ch == '{' || ch == '[') {
            st.push(ch);                                 // opening: remember it
        } else {
            if (st.isEmpty()) return false;              // closing with nothing open
            char open = st.pop();
            if ((ch == ')' && open != '(') ||
                (ch == '}' && open != '{') ||
                (ch == ']' && open != '[')) return false;   // mismatched pair
        }
    }
    return st.isEmpty();                                 // anything left open = invalid
}
```
**Verified:**
```
"{[()]}" -> true   "{[(])}" -> false
```

**Why a stack is the right tool:** the most recently opened bracket must be the first one closed — that *is* LIFO. Note the final `st.isEmpty()`: `"((("` never fails a match but leaves three unclosed brackets, so the emptiness check is what catches it.

Same shape solves: expression evaluation, undo/redo, browser history, and **the JVM's own call stack** (note 06 §12).

## 6. The Other Classic Problems

**Stack from two queues / queue from two stacks** — a pure exercise in the two disciplines. The queue-from-stacks trick: push onto `in`; to remove, if `out` is empty, drain `in` into `out` (which reverses the order), then pop from `out`. **O(1) amortised** — each element moves between stacks at most once.

**Largest rectangle in a histogram** — the hardest thing in this lecture, and the introduction to the **monotonic stack**: keep a stack whose values only increase, and when a smaller bar arrives, pop and compute areas. O(n) instead of O(n²).

**Next greater element** — the same monotonic idea, and the pattern behind a whole family of problems.

> 💡 The monotonic stack is the real payoff of this topic. The rule of thumb: *if a problem asks for the next/previous greater/smaller element, a stack turns O(n²) into O(n).*

## 7. Complexity

| | Stack (array) | Stack (linked) | Queue (circular) | Queue (linked) |
|---|---|---|---|---|
| push / offer | O(1)* | O(1) | O(1) | O(1) |
| pop / poll | O(1) | O(1) | O(1) | O(1) |
| peek | O(1) | O(1) | O(1) | O(1) |
| Search | O(n) | O(n) | O(n) | O(n) |
| Space | O(n) | O(n) + pointers | O(n) | O(n) + pointers |

\* amortised, if it doubles on overflow.

**Every core operation is O(1).** That's the point of restricting access — there's no searching, no shifting, no decisions.

---

## ⚠️ Common Misunderstandings
**1. A stack is a special list you can index.**
❌ index it · ✅ access is **top-only**. If you're reaching into the middle, you want a list.

**2. `java.util.Stack` is the right stack.**
❌ right · ✅ legacy and synchronised, and it leaks indexed access. Use `ArrayDeque`.

**3. `data[--ptr]` and `data[ptr--]` are the same.**
❌ same · ✅ post-decrement returns the current top then moves; pre-decrement moves first and returns the wrong element.

**4. A simple front/rear array queue is fine.**
❌ fine · ✅ without wrapping, freed front space is lost forever and the queue "fills" while half empty. Use modulo — a circular queue.

**5. `LinkedList` is the natural queue in Java.**
❌ natural · ✅ `ArrayDeque` is faster — a circular array with no per-node allocation.

**6. Valid-parentheses only needs matching on close.**
❌ only · ✅ you also need `isEmpty()` at the **end** — `"((("` matches nothing but is still invalid.

**7. Popping an object stack frees the object.**
❌ frees · ✅ the array slot still references it. Null it out, or you leak.

**8. Stacks and queues are interchangeable.**
❌ interchangeable · ✅ LIFO vs FIFO produces completely different traversals — DFS vs BFS on a tree (note 23).

## JavaScript Comparison
| | Java | JavaScript |
|---|---|---|
| Stack | `ArrayDeque` — `push`/`pop`/`peek` | array — `push`/`pop` |
| Queue | `ArrayDeque` — `offer`/`poll` | array — `push`/`shift` (⚠️ `shift` is O(n)) |
| Dedicated class | ✅ | ❌ — arrays do both |
| Empty check | `isEmpty()` | `arr.length === 0` |
| Peek | `peek()` | `arr[arr.length - 1]` / `arr[0]` |

> 💡 JS's `shift()` is **O(n)** because it re-indexes the whole array, so a naive JS queue is quadratic in a loop. That's the exact problem Java's `ArrayDeque` solves with a circular buffer — worth knowing if you write queue-based algorithms in JS.

## Interview Angles
- **"Valid parentheses."** (LeetCode 20) — The canonical stack question. Don't forget the final empty check.
- **"Implement a queue using stacks."** (232) — Two stacks, drain on demand, O(1) amortised. Explain the amortisation.
- **"Min stack."** (155) — Return the minimum in O(1): keep a second stack of minima.
- **"Next greater element."** (496/503) — Monotonic stack, O(n).
- **"Largest rectangle in histogram."** (84) — The hard monotonic-stack problem.
- **"Why is `ArrayDeque` preferred?"** — Circular array, no node allocation, not synchronised. Naming `java.util.Stack` as legacy is a good signal.
- **"Where do stacks appear implicitly?"** — The JVM call stack, recursion, DFS, expression parsing, undo.

## Related · Next
- **Related:** [[Linked List]] (21) · [[Arrays]] (07) · [[Stack]] (2.2) · [[Queue]] (2.3) · [[Recursion I]] (14 — the call stack)
- **Practice:** implement both on arrays (with the circular fix), then solve Valid Parentheses (20), Min Stack (155) and Implement Queue using Stacks (232).
- **Next:** [23 — Trees I](23-Trees_Basics_Notes.md)

---

## 🔁 Rapid Revision (self-test)
<details><summary>1. LIFO vs FIFO, with the verified evidence?</summary>

Stack: push 1,2,3 → peek 3, pop 3, pop 2. Queue: offer 1,2,3 → peek 1, poll 1, poll 2. Last-in-first-out vs first-in-first-out.
</details>

<details><summary>2. What is an array-backed stack, minimally?</summary>

An array plus one integer `ptr` marking the top, starting at `-1` for empty.
</details>

<details><summary>3. Why `data[ptr--]` and not `data[--ptr]`?</summary>

Post-decrement returns the current top *then* moves the pointer. Pre-decrement moves first and returns the wrong element.
</details>

<details><summary>4. Why does a naive array queue break, and what fixes it?</summary>

Removing from the front either shifts everything (O(n)) or permanently wastes the freed space. A **circular queue** wraps indices with `% data.length`, making both operations O(1) with no waste.
</details>

<details><summary>5. Why avoid `java.util.Stack`?</summary>

Legacy: it extends `Vector`, so it's synchronised (slow) and exposes indexed access, breaking the abstraction. Use `ArrayDeque`.
</details>

<details><summary>6. In valid parentheses, why check `isEmpty()` at the end?</summary>

`"((("` never hits a mismatch but leaves brackets unclosed. The final emptiness check is what rejects it.
</details>

<details><summary>7. How do you build a queue from two stacks?</summary>

Push onto `in`. To remove, if `out` is empty, drain `in` into `out` (reversing the order), then pop `out`. O(1) amortised — each element moves at most once.
</details>

<details><summary>8. What is a monotonic stack for?</summary>

Next/previous greater-or-smaller element problems — keeping the stack ordered turns O(n²) scans into O(n).
</details>
