---
type: foundation
title: Trees I — Binary Trees, BSTs, and the Four Traversals
tags: [foundations, java, trees, binary-tree, bst, traversal, dfs, bfs, recursion, pre-phase-0]
created: 2026-09-29
status: seen
source: "Kunal Kushwaha · Community Classroom — lecture 20 Trees (introduction, BST, questions)"
related: ["[[Linked List]]", "[[Recursion I]]", "[[Stacks and Queues]]", "[[Tree]]"]
---

# 23 · Trees I — Binary Trees & BST

> 📁 Part 23 of 28 in [Ytube dsa/](README.md) · **Prev:** [22 — Stacks & Queues](22-Stacks_and_Queues_Notes.md) · **Next:** [24 — Trees II](24-Trees_Advanced_Notes.md)

## Introduction
A **tree** is a linked list where each node points at *two* others. That's genuinely all that changes structurally — and it changes everything about what the structure can do.

Where a list is linear (O(n) to search), a **balanced** tree is logarithmic (O(log n)), because each step discards half the remaining nodes. Same idea as binary search (note 09), now baked into the data structure itself.

Trees are also where recursion stops being an exercise and becomes the natural way to write code. Almost every tree algorithm is three lines: *do something with the node, recurse left, recurse right.*

## Vocabulary

```
            50          <- root (no parent)
           /  \
         30    70       <- internal nodes
        /  \   / \
      20   40 60  80    <- leaves (no children)
```

| Term | Meaning |
|---|---|
| **Root** | the top node, the only one with no parent |
| **Leaf** | a node with no children |
| **Height** | longest path from a node down to a leaf (in edges or nodes — *state which*) |
| **Depth** | distance from the root down to a node |
| **Subtree** | any node plus everything below it — itself a valid tree |
| **Balanced** | left and right heights differ by at most 1 at every node |

**"Every subtree is itself a tree"** is the property that makes recursion work. When you recurse into `node.left`, you're handing the same function a smaller instance of the identical problem.

---

## 1. The Node

```java
class Node {
    int value;
    Node left;
    Node right;
    Node(int value) { this.value = value; }
}
```

Compare with note 21's linked-list node — **one extra reference**. That's the whole difference.

## 2. Binary Tree vs Binary Search Tree

A **binary tree** is any tree where each node has at most two children.

A **binary search tree** adds an ordering invariant:

> **Everything in the left subtree is smaller than the node; everything in the right subtree is larger.**

```java
static Node insert(Node root, int value) {
    if (root == null) return new Node(value);        // base: found the empty spot
    if (value < root.value) root.left  = insert(root.left, value);
    else                   root.right = insert(root.right, value);
    return root;                                      // return the (possibly new) subtree
}
```

The `root.left = insert(root.left, ...)` idiom is worth pausing on: each call returns the subtree it was given (or a brand-new node), and the parent **reassigns** it. That pattern — *recurse and reassign* — handles insertion and deletion without any special-casing of the parent link.

Searching follows the same invariant and discards half the tree at each step — **O(log n)** in a balanced tree.

## 3. The Four Traversals

**Verified on the tree built by inserting 50, 30, 70, 20, 40, 60, 80:**

```
inorder   (sorted!):     20 30 40 50 60 70 80
preorder  (root first):  50 30 20 40 70 60 80
postorder (root last):   20 40 30 60 80 70 50
BFS level order:         50 30 70 20 40 60 80
height: 3
```

### The three depth-first traversals

They differ **only** in where you touch the node relative to the recursive calls:

```java
static void inorder(Node n) {                 // LEFT, node, RIGHT
    if (n == null) return;
    inorder(n.left);
    System.out.print(n.value + " ");
    inorder(n.right);
}

static void preorder(Node n) {                // node, LEFT, RIGHT
    if (n == null) return;
    System.out.print(n.value + " ");
    preorder(n.left);
    preorder(n.right);
}

static void postorder(Node n) {               // LEFT, RIGHT, node
    if (n == null) return;
    postorder(n.left);
    postorder(n.right);
    System.out.print(n.value + " ");
}
```

**One line moves. That's it.** The name tells you where the *root* goes: **pre**order = root first, **in**order = root in the middle, **post**order = root last.

| Traversal | Order | Use it for |
|---|---|---|
| **Inorder** | L, N, R | **Sorted output from a BST** — the defining property |
| **Preorder** | N, L, R | Copying a tree, serialising it, prefix expressions |
| **Postorder** | L, R, N | Deleting a tree (children before parent), computing sizes/heights |

> 💡 **Inorder on a BST gives sorted order.** That's the single most useful fact about BSTs, and the verified output shows it: `20 30 40 50 60 70 80`. It's also the standard way to *validate* a BST — do an inorder walk and check the values only increase.

> 💡 **Postorder is the one for anything that needs its children's results first** — height, size, "is this subtree balanced". If your recursion needs answers from below before it can decide, it's postorder-shaped.

### Breadth-first (level order)

DFS uses the **call stack**; BFS needs an explicit **queue** (note 22):

```java
static void bfs(Node root) {
    Queue<Node> q = new LinkedList<>();
    q.add(root);
    while (!q.isEmpty()) {
        Node n = q.poll();
        System.out.print(n.value + " ");
        if (n.left != null)  q.add(n.left);
        if (n.right != null) q.add(n.right);
    }
}
```
```
50 30 70 20 40 60 80
```

Level by level, left to right. **The data structure decides the traversal:** a **queue** gives you breadth-first, and swapping it for a **stack** gives you depth-first. That's the clearest illustration of why note 22's LIFO/FIFO distinction matters.

## 4. Height and the Recursive Shape

```java
static int height(Node n) {
    if (n == null) return 0;
    return 1 + Math.max(height(n.left), height(n.right));
}
```
```
height: 3
```

Three lines, and it's the template for nearly every tree problem:

1. **Base case:** `null` → return the identity value (0 here)
2. **Recurse** into both subtrees
3. **Combine** their results with something (`max`, `+`, `&&`)

Swap the combine step and you get a different algorithm:

```java
size(n)       ->  1 + size(left) + size(right)
sum(n)        ->  n.value + sum(left) + sum(right)
max(n)        ->  max(n.value, max(left), max(right))
isBalanced(n) ->  |height(left) - height(right)| <= 1  &&  both subtrees balanced
```

> ⚠️ **State whether height counts edges or nodes.** This version returns `3` for a 3-level tree — it counts **nodes** on the longest path. The edge-based definition returns `2`, using `return -1` for `null`. Both are common; interviewers accept either if you say which you're using.

## 5. Why Balance Matters

```
Balanced (height 3)         Degenerate (height 7)
      50                     20
     /  \                      \
   30    70                     30
  / \   /  \                      \
 20 40 60  80                      40  ...
```

Inserting **sorted** data into a naive BST produces the right-hand shape — every node goes right, and the tree degenerates into a linked list.

| | Balanced BST | Degenerate BST |
|---|---|---|
| Search | **O(log n)** | O(n) |
| Insert | **O(log n)** | O(n) |
| Delete | **O(log n)** | O(n) |

**All of a BST's advantage depends on balance**, and nothing in the basic insert maintains it. That's exactly the problem **AVL trees** solve (note 24) — and why Java's `TreeMap` is a red-black tree rather than a plain BST.

## 6. Complexity

| Operation | Balanced | Worst (degenerate) |
|---|---|---|
| Search / insert / delete | O(log n) | O(n) |
| Traversal (any) | **O(n)** — every node visited | O(n) |
| Space (recursion) | O(log n) stack | **O(n) stack** |
| Space (BFS queue) | O(width) ≈ O(n/2) | O(1) |

> ⚠️ The recursion depth follows the tree's **height**, so a degenerate tree of a million nodes will hit note 14's ~19,700-frame `StackOverflowError`. A balanced million-node tree is only ~20 frames deep.

---

## ⚠️ Common Misunderstandings
**1. A binary tree is a binary search tree.**
❌ same · ✅ a BST adds the ordering invariant: left < node < right, at **every** node.

**2. The BST property only applies to direct children.**
❌ children · ✅ the **entire** left subtree must be smaller. A node can satisfy its parent and still break the invariant higher up — the classic "validate a BST" trap.

**3. The three DFS traversals are structurally different.**
❌ different · ✅ **one line moves.** The name says where the root is visited.

**4. BFS is just recursion done differently.**
❌ recursion · ✅ it needs an explicit **queue**. DFS gets its stack for free from the call stack; BFS has no equivalent.

**5. A BST is always O(log n).**
❌ always · ✅ only when **balanced**. Sorted input degenerates it into a linked list — O(n).

**6. Height is unambiguous.**
❌ unambiguous · ✅ edges or nodes? Our version returns 3 (nodes) for a 3-level tree; the edge version returns 2. Say which.

**7. Recursion on trees can't overflow.**
❌ can't · ✅ depth follows height — fine for balanced trees (~20 frames per million nodes), fatal for degenerate ones.

**8. Inorder gives sorted output for any binary tree.**
❌ any · ✅ only for a **BST**. On an arbitrary binary tree it's just an ordering.

## JavaScript Comparison
| | Java | JavaScript |
|---|---|---|
| Node | `class Node { int value; Node left, right; }` | `{ value, left, right }` |
| Built-in tree | `TreeMap` / `TreeSet` (red-black) | **none** — `Map` is a hash table |
| Sorted map | `TreeMap`, O(log n) | no equivalent; sort keys manually |
| Queue for BFS | `ArrayDeque` | array with `shift()` (⚠️ O(n)) |

> 💡 JS has no sorted-map structure at all, so `TreeMap`-style range queries have no direct equivalent. Worth knowing when someone claims "JS has everything Java has".

## Interview Angles
- **"Traverse a tree."** — Know all four cold. Inorder-on-BST-gives-sorted is the fact to volunteer.
- **"Validate a BST."** (LeetCode 98) — The trap is checking only immediate children. Correct approaches: inorder must be strictly increasing, or pass down (min, max) bounds.
- **"Maximum depth."** (104) — The `1 + max(left, right)` template.
- **"Level order traversal."** (102) — BFS with a queue; track level size to group per level.
- **"Lowest common ancestor."** (235 BST / 236 general) — The BST version uses the ordering to pick a direction; the general version is postorder.
- **"Invert a binary tree."** (226) — Swap children, recurse. Famous for being the question that got Max Howell rejected.
- **"Why do we need AVL/red-black trees?"** — Because plain BSTs degenerate on sorted input. That's note 24.
- **Default to recursion** for trees — then be ready to give the iterative version with an explicit stack.

## Related · Next
- **Related:** [[Linked List]] (21 — one pointer fewer) · [[Recursion I]] (14 — the shape all of this uses) · [[Stacks and Queues]] (22 — DFS vs BFS) · [[Tree]] (4.1)
- **Practice:** build a BST by insertion, print all four traversals, and compute height and size. Then solve 104, 226, 102 and 98 — the last one is where most people slip.
- **Next:** [24 — Trees II: AVL & Segment Trees](24-Trees_Advanced_Notes.md)

---

## 🔁 Rapid Revision (self-test)
<details><summary>1. What distinguishes a BST from a binary tree?</summary>

The ordering invariant: everything in the left **subtree** is smaller and everything in the right is larger — at every node, not just direct children.
</details>

<details><summary>2. The three DFS traversals and what separates them?</summary>

Preorder (N,L,R), inorder (L,N,R), postorder (L,R,N). **Only the position of the visit line changes** — the name says where the root goes.
</details>

<details><summary>3. What does inorder give you on a BST, and what's it used for?</summary>

**Sorted order** — verified `20 30 40 50 60 70 80`. It's the standard way to validate a BST.
</details>

<details><summary>4. Which traversal for computing height or size, and why?</summary>

**Postorder** — you need both children's results before you can combine them.
</details>

<details><summary>5. How does BFS differ from DFS structurally?</summary>

BFS needs an explicit **queue**; DFS uses the call stack implicitly. Swapping queue for stack switches between them.
</details>

<details><summary>6. The three-step template for tree recursion?</summary>

Base case on `null` returning an identity, recurse into both subtrees, combine the results. Changing only the combine step gives height, size, sum or balance.
</details>

<details><summary>7. What happens if you insert sorted data into a plain BST?</summary>

It degenerates into a linked list — height n, and every operation becomes O(n). This is why self-balancing trees exist.
</details>

<details><summary>8. When does tree recursion risk StackOverflowError?</summary>

Depth follows height. A balanced million-node tree is ~20 frames; a degenerate one is a million and overflows at ~19,700.
</details>
