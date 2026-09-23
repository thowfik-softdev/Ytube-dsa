---
type: foundation
title: Patterns — The Universal Method for Nested-Loop Printing Problems
tags: [foundations, java, patterns, nested-loops, loops, pre-phase-0]
created: 2026-09-29
status: seen
source: "Kunal Kushwaha · Community Classroom — lecture 13 Patterns"
related: ["[[Conditionals and Loops]]", "[[Arrays]]", "[[Matrix]]"]
---

# 12 · Patterns

> 📁 Part 12 of 28 in [Ytube dsa/](README.md) · **Prev:** [11 — Strings & StringBuilder](11-Strings_Notes.md) · **Next:** [13 — Complexity Analysis](13-Complexity_Notes.md)

## Introduction
Pattern printing looks like busywork — stars and numbers arranged into triangles. It isn't. It is **nested-loop fluency training**, and it is the cheapest possible way to build the row/column reasoning that every 2-D array, matrix and grid problem needs.

The lecture's real contribution is a **universal method** that turns any pattern into the same four questions. Learn the method and you never memorise a pattern again.

## The Universal Method

For any pattern, answer these in order:

1. **How many rows?** → that's the outer loop.
2. **How many columns in *this* row?** → that's the inner loop's bound, usually a function of `row`.
3. **What do I print at each position?** → the inner loop's body.
4. **What's the relationship between row number and what appears?** → the formula.

Then: **outer loop = rows, inner loop = columns, `println()` after each row.**

That last line is the one beginners forget. Without it everything prints on a single line.

---

## 1. The Square — the Baseline

```java
static void pattern1(int n) {
    for (int row = 1; row <= n; row++) {
        for (int col = 1; col <= n; col++) {
            System.out.print("* ");
        }
        System.out.println();          // end of row
    }
}
```
**Output** — `pattern1(4)`
```
* * * * 
* * * * 
* * * * 
* * * * 
```

Every row has the same number of columns (`n`), so the inner bound doesn't depend on `row`. Note `print` inside, `println` outside.

## 2. The Triangle — Columns Depend on the Row

```java
for (int row = 1; row <= n; row++) {
    for (int col = 1; col <= row; col++) {     // <-- bound is `row`
        System.out.print("* ");
    }
    System.out.println();
}
```
**Output** — `pattern2(4)`
```
* 
* * 
* * * 
* * * * 
```

This is the entire idea: **`col <= row`**. Row 1 prints 1 star, row 4 prints 4. Everything else is a variation on what you write in that bound.

## 3. The Inverted Triangle — Count Down

```java
for (int col = 1; col <= n - row + 1; col++) {
    System.out.print("* ");
}
```
**Output** — `pattern3(4)`
```
* * * * 
* * * 
* * 
* 
```

`n - row + 1`: row 1 → 4 stars, row 4 → 1 star. When you want a decreasing count, subtract the row from `n`. Sanity-check by plugging in the first and last row.

## 4. Printing the Column Number

```java
for (int col = 1; col <= row; col++) {
    System.out.print(col + " ");       // print the counter, not a star
}
```
**Output** — `pattern4(4)`
```
1 
1 2 
1 2 3 
1 2 3 4 
```

Step 3 of the method: the *shape* is unchanged from §2 — only the character changed. Shape and content are independent, which is why the method separates them.

## 5. Two Halves — the Diamond

The trick for any pattern that grows then shrinks: run the outer loop to `2n` and compute the real count with a **ternary**.

```java
static void pattern5(int n) {
    for (int row = 0; row < 2 * n; row++) {
        int totalColsInRow = row > n ? 2 * n - row : row;     // grow, then shrink
        for (int col = 0; col < totalColsInRow; col++) {
            System.out.print("* ");
        }
        System.out.println();
    }
}
```
**Output** — `pattern5(4)`
```

* 
* * 
* * * 
* * * * 
* * * 
* * 
* 
```

`totalColsInRow` rises `0,1,2,3,4` then falls `3,2,1`. The first row prints nothing because `row` starts at `0` — a small wart, but it shows the formula honestly.

**This single ternary handles every "grows then shrinks" pattern.** Don't write two separate loops.

## 6. Adding Spaces — Centring

To centre a shape, print `n - count` spaces *before* the content:

```java
static void pattern28(int n) {
    for (int row = 0; row < 2 * n; row++) {
        int totalColsInRow = row > n ? 2 * n - row : row;

        int noOfSpaces = n - totalColsInRow;
        for (int s = 0; s < noOfSpaces; s++) {
            System.out.print(" ");
        }
        for (int col = 0; col < totalColsInRow; col++) {
            System.out.print("* ");
        }
        System.out.println();
    }
}
```
**Output** — `pattern28(4)`
```
    
   * 
  * * 
 * * * 
* * * * 
 * * * 
  * * 
   *
```

**Two inner loops, one row:** spaces first, then content. Almost every pyramid, diamond and arrow is exactly this — spaces, then stars, sometimes spaces again.

## 7. The Hard One — Distance From the Edge

```java
static void pattern31(int n) {
    int originalN = n;
    n = 2 * n;
    for (int row = 0; row <= n; row++) {
        for (int col = 0; col <= n; col++) {
            int atEveryIndex = originalN
                - Math.min(Math.min(row, col), Math.min(n - row, n - col));
            System.out.print(atEveryIndex + " ");
        }
        System.out.println();
    }
}
```
**Output** — `pattern31(4)`
```
4 4 4 4 4 4 4 4 4 
4 3 3 3 3 3 3 3 4 
4 3 2 2 2 2 2 3 4 
4 3 2 1 1 1 2 3 4 
4 3 2 1 0 1 2 3 4 
4 3 2 1 1 1 2 3 4 
4 3 2 2 2 2 2 3 4 
4 3 3 3 3 3 3 3 4 
4 4 4 4 4 4 4 4 4 
```

Concentric rings. The insight is that the value at any cell depends on its **distance from the nearest edge**:

- distance to the top = `row`
- distance to the left = `col`
- distance to the bottom = `n - row`
- distance to the right = `n - col`

Take the **minimum** of those four, and subtract from `n`. No conditionals, no special cases — one formula for all 81 cells.

> 💡 **This is the lesson worth keeping.** When a pattern resists `if`-chains, stop thinking about *drawing* and ask: *what formula maps `(row, col)` to the value?* That shift — from procedure to coordinate formula — is the same one that later makes matrix problems (spiral traversal, rotation, diagonals) tractable.

## 8. Complexity

Every pattern here is **O(rows × cols)** — for an n×n pattern, **O(n²)** — with **O(1)** space.

That is unavoidable: printing n² characters takes n² work. The nested loop isn't inefficiency, it's the output size. Worth being able to say, because "nested loop = O(n²)" is a reflex that needs the *reason* attached.

> 💡 If you build the output into a `StringBuilder` instead of calling `print` per character, you get the same O(n²) work but far fewer I/O calls — noticeably faster for large patterns (note 11 §6).

---

## ⚠️ Common Misunderstandings
**1. Forgetting `System.out.println()` after the inner loop.**
❌ everything on one line · ✅ `println()` at the end of each outer iteration ends the row.

**2. Using `println` inside the inner loop.**
❌ one character per line · ✅ `print` inside, `println` outside.

**3. Mixing up which loop is which.**
❌ inner loop = rows · ✅ **outer = rows, inner = columns**. The inner loop runs fully for each single outer pass.

**4. Writing two loops for a grow-then-shrink pattern.**
❌ two separate halves · ✅ one loop to `2n` with `row > n ? 2*n - row : row`.

**5. Guessing the bound instead of checking it.**
❌ hoping `n - row` is right · ✅ plug in the **first and last row** and verify the counts.

**6. Reaching for `if` chains on hard patterns.**
❌ special-casing every region · ✅ find the `(row, col)` formula — §7 does 81 cells with one expression.

**7. Assuming an off-by-one is wrong.**
❌ · ✅ `pattern5` genuinely prints an empty first row because `row` starts at `0`. Decide whether the formula or the loop start should change — don't paper over it.

## JavaScript Comparison
| | Java | JavaScript |
|---|---|---|
| Print without newline | `System.out.print(x)` | `process.stdout.write(x)` — `console.log` always adds one |
| Print with newline | `System.out.println(x)` | `console.log(x)` |
| Repeat a character | loop, or `" ".repeat(n)` | `" ".repeat(n)` |
| Build a row | `StringBuilder` | template literal or `arr.join("")` |

> 💡 In JS you'd usually build each row as a string and `console.log` it once, because there is no convenient `print`. Java has both — and `String.repeat(n)` (Java 11+) can replace the space loop entirely: `System.out.print(" ".repeat(noOfSpaces))`.

## Interview Angles
- Pattern printing is **rarely asked directly** in senior interviews — it's a learning exercise, not a real problem.
- What it *builds* is asked constantly: nested-loop reasoning for **matrix traversal**, spiral order, rotating an image, diagonal traversal.
- The transferable skill is §7: expressing a cell's value as a **function of `(row, col)`** rather than a procedure.
- If it does come up, narrate the four questions (rows? columns per row? what to print? relationship?) — the method is more impressive than the answer.

## Related · Next
- **Related:** [[Conditionals and Loops]] (05 — nested loops) · [[Arrays]] (07 — 2-D traversal) · [[Matrix]] (1.3)
- **Practice:** print a hollow square (border only — `row` or `col` at an edge), Pascal's triangle (each value is the sum of the two above), and a right-aligned triangle. Each is one bound change plus one formula.
- **Next:** [13 — Complexity Analysis](13-Complexity_Notes.md)

---

## 🔁 Rapid Revision (self-test)
<details><summary>1. The four questions of the universal method?</summary>

How many rows? How many columns in this row? What do I print at each position? What's the relationship between the row number and what appears?
</details>

<details><summary>2. Which loop is rows and where does `println()` go?</summary>

Outer loop = rows, inner = columns. `print` inside the inner loop, `println()` after it — at the end of each outer pass.
</details>

<details><summary>3. How do you make a triangle grow vs shrink?</summary>

Grow: `col <= row`. Shrink: `col <= n - row + 1`. Always verify by plugging in the first and last rows.
</details>

<details><summary>4. How do you handle a pattern that grows then shrinks?</summary>

One outer loop to `2n` with a ternary: `int count = row > n ? 2*n - row : row;` — never two separate loops.
</details>

<details><summary>5. How do you centre a pattern?</summary>

Print `n - count` spaces before the content — two inner loops per row: spaces, then stars.
</details>

<details><summary>6. What's the trick behind the concentric-rings pattern?</summary>

The value is a function of the cell's **distance from the nearest edge**: `n - min(row, col, n-row, n-col)`. One formula, no conditionals.
</details>

<details><summary>7. Complexity of pattern printing?</summary>

O(rows × cols) — O(n²) for an n×n pattern, O(1) space. That's the size of the output, not inefficiency.
</details>
