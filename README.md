# Ytube dsa/ — 📺 Notes from the YouTube DSA playlist

**Why this folder exists:** written-up notes for the lectures I watch in the [Kunal Kushwaha Java + DSA playlist](https://www.youtube.com/playlist?list=PL9gnSGHSqcnr_DxHsP7AW9ftq0AtAyYqJ), one file per lecture PDF, numbered in watch order. This is the layer *underneath* the curriculum: `01-concepts/` starts at item **0.1 — Running a Java Program** and teaches you to *write* Java, while these answer **why** programming languages exist at all, **how** you plan a program before typing it, and **what actually happens** between `javac` and your output.

Kept **separate from `01-concepts/`** on purpose: those notes map 1:1 to `ROADMAP.md` items and are written session-by-session as you learn. These follow the *playlist*, not the roadmap — mixing them in would break that mapping.

## Read in this order

| # | File | Source | What it covers |
|---|------|--------|----------------|
| **01** | [01-Intro_to_Programming_Notes.md](01-Intro_to_Programming_Notes.md) | `Intro_to_Programming_Notes.pdf` | Why languages exist · procedural vs functional vs OOP · static vs dynamic typing · stack & heap · garbage collection |
| **02** | [02-Flow_Of_Program_Notes.md](02-Flow_Of_Program_Notes.md) | `Flow_Of_Program_Notes.pdf` | Flowchart symbols · pseudocode · the prime problem · why √n is enough |
| **03** | [03-Introduction_to_Java_Notes.md](03-Introduction_to_Java_Notes.md) | `Introduction_to_Java_Notes.pdf` | `.java` → `.class` → machine code · platform independence · JDK/JRE/JVM · class loader · interpreter vs JIT |
| **04** | [04-First_Java_Program_Notes.md](04-First_Java_Program_Notes.md) | `First_Java_Program_Notes.pdf` | Every word of `main` · the 8 primitives · `Scanner` · conversion, casting & promotion · **inside the `.class` file** |
| **05** | [05-Conditionals_and_Loops_Notes.md](05-Conditionals_and_Loops_Notes.md) | `— *(project code)*` | `if`/ladders · every `switch` form → pattern matching · all loop forms · `break`/`continue`/labels · digit peeling · what they compile to |
| **06** | [06-Methods_Notes.md](06-Methods_Notes.md) | `Methods.pdf` | Return rules · **pass-by-value** · scope & shadowing · overloading resolution · varargs · the call stack |
| **07** | [07-Arrays_Notes.md](07-Arrays_Notes.md) | `Arrays.pdf` | Arrays as heap objects · defaults & bounds · two-pointer reverse · 2-D & jagged · `ArrayList` capacity vs size |
| **08** | [08-Linear_Search_Notes.md](08-Linear_Search_Notes.md) | `Linear Search.pdf` | The three return shapes · range search · 2-D search · why `-1` is a safe sentinel here |
| **09** | [09-Binary_Search_Notes.md](09-Binary_Search_Notes.md) | `Binary Search.pdf` | The template · the overflow bug · ceiling/floor from crossed pointers · rotated & mountain arrays · **binary search on the answer** |
| **10** | [10-Sorting_Notes.md](10-Sorting_Notes.md) | `Bubble/Selection/Insertion/Cyclic Sort.pdf` | Measured comparison counts · which sorts are adaptive · stability · **cyclic sort** and its six O(n) problems |
| **11** | [11-Strings_Notes.md](11-Strings_Notes.md) | `Strings.pdf · StringBuffer.pdf` | Immutability · the String pool · `'a'+'b'=195` · the O(n²) concatenation trap · `StringBuilder` |
| **12** | [12-Patterns_Notes.md](12-Patterns_Notes.md) | `Patterns.pdf` | The universal four-question method · grow-then-shrink in one ternary · the distance-from-edge formula |
| **13** | [13-Complexity_Notes.md](13-Complexity_Notes.md) | `Complexity Analysis.pdf` | Why timing lies · the rules · reading Big-O off loop counters · recurrences · Master's theorem |
| **14** | [14-Recursion_Basics_Notes.md](14-Recursion_Basics_Notes.md) | `Recursion - 1.pdf` | Base cases · the `fact(5)` trace · tail recursion (Java doesn't optimise it) · **331M calls for fib(40)** · memoization |
| **15** | [15-Recursion_Subsets_Notes.md](15-Recursion_Subsets_Notes.md) | `Recursion - Subsets & Strings.pdf` | The processed/unprocessed pattern · 2ⁿ subsets · n! permutations · dice pruning · phone pad |
| **16** | [16-Recursion_Sorting_Backtracking_Notes.md](16-Recursion_Sorting_Backtracking_Notes.md) | `Merge Sort.pdf · Quick Sort.pdf · Backtracking.pdf` | Merge & quick sort · why Java ships both · the place/recurse/**undo** template · N-Queens · Sudoku |
| **17** | [17-Math_and_Bitwise_Notes.md](17-Math_and_Bitwise_Notes.md) | `Maths.pdf · Bitwise.pdf` | √n primality · Euclid · the sieve · XOR tricks · `n & (n-1)` · fast exponentiation |
| **18** | [18-OOP_Basics_Notes.md](18-OOP_Basics_Notes.md) | `OOP-1.pdf` | Classes & objects · constructors and `this` · `final` · `static` · access modifiers · the `Integer` cache trap |
| **19** | [19-OOP_Inheritance_Notes.md](19-OOP_Inheritance_Notes.md) | `OOP - 2/3/4.pdf` | Inheritance · **dynamic dispatch** · overloading vs overriding · abstract classes vs interfaces |
| **20** | [20-OOP_Advanced_Notes.md](20-OOP_Advanced_Notes.md) | `OOP - 6/7/8.pdf` | Generics & type erasure · exceptions · **equals/hashCode contract** · singleton · enums · the collections map |
| **21** | [21-LinkedList_Notes.md](21-LinkedList_Notes.md) | `LinkedList.pdf` | Nodes & rewiring · reversal · **fast/slow pointers** · doubly & circular · why `ArrayList` usually still wins |
| **22** | [22-Stacks_and_Queues_Notes.md](22-Stacks_and_Queues_Notes.md) | `Stacks and Queues.pdf` | LIFO vs FIFO · array-backed stack · the circular queue fix · valid parentheses · monotonic stack |
| **23** | [23-Trees_Basics_Notes.md](23-Trees_Basics_Notes.md) | `Trees - 1.pdf` | Binary trees & BSTs · the four traversals · why inorder gives sorted output · why balance matters |
| **24** | [24-Trees_Advanced_Notes.md](24-Trees_Advanced_Notes.md) | `Trees - 2 (AVL).pdf · Trees - 3 (Segment).pdf` | Balance factors & rotations · the four cases · segment trees & the three query cases |
| **25** | [25-Heaps_and_HashMaps_Notes.md](25-Heaps_and_HashMaps_Notes.md) | `Heaps - 1.pdf · Hashmaps introduction.pdf` | Heaps as arrays · `PriorityQueue` · **min-heap for top-K** · hashing, collisions, load factor · Karp–Rabin |
| **26** | [26-Advanced_Sorting_Notes.md](26-Advanced_Sorting_Notes.md) | `CountSort.pdf · Radix sort.pdf · HuffmanEncoding.pdf` | Beating O(n log n) by not comparing · why radix needs stability · Huffman · sqrt decomposition |
| **27** | [27-LargeNumbers_and_FileIO_Notes.md](27-LargeNumbers_and_FileIO_Notes.md) | `LargeNumbers.pdf · File handling.pdf` | `BigInteger` & `BigDecimal` · why `0.1+0.2` isn't `0.3` · Scanner vs BufferedReader · try-with-resources |
| **28** | [28-Git_and_GitHub_Notes.md](28-Git_and_GitHub_Notes.md) | `Git GitHub Notes.pdf` | The four areas · branching & conflicts · **reset vs revert** · `reflog` as the safety net · `.gitignore` gotchas |

> Each file keeps its **source PDF's name** with a `NN-` prefix, so the numbers give the watch order and the name points back to the original notes.

**Then continue to** `01-concepts/0.1-running-java.md` in the main study vault — where the curriculum proper begins.

## 📦 Coverage

All **28 notes are complete** — the full [`DSA-Bootcamp-Java`](https://github.com/kunal-kushwaha/DSA-Bootcamp-Java) lecture series (28 lecture folders, 89 PDFs, 256 Java files) written up end to end.

Recursion, OOP and trees are split across several notes each, because one note per lecture folder would be unreadable at that size. Complexity (`13`) deliberately comes **before** recursion, since the recursion notes lean on recurrence relations.

## How to revise with these
1. First pass: read them top-to-bottom, in order.
2. Every pass after that: skip to the **🔁 Rapid Revision** self-test at the bottom of each file, answer out loud *before* expanding, and re-open only the section you fumbled.
3. Files 1 and 3 are the interview-heavy ones (memory model, JVM internals). File 2 is a *thinking habit* — flowchart → pseudocode → code — that pays off on every DSA problem. Files 4 and 5 hold the Java-specific traps — integer division, silent overflow, switch fall-through.

## Conventions
- Same note shape as `06-templates/concept.md`: intuition → runnable Java → JS comparison → misconceptions → interview angles.
- Every Java snippet was **compiled and run on JDK 21** — the documented outputs are real console output, not assumed.
- Flowcharts are **mermaid** code blocks, so they render as diagrams in Obsidian and on GitHub.
- Where the source PDFs are wrong, the note says so in a **⚠️ Correction to the lecture notes** callout rather than repeating the error. Where a note is written from the project code instead of a PDF (file 5), bugs found while running it are called out the same way.
