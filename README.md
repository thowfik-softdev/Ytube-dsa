# Ytube dsa/ — 📺 Notes from the YouTube DSA playlist

**Why this folder exists:** written-up notes for the lectures I watch in the [Kunal Kushwaha Java + DSA playlist](https://www.youtube.com/playlist?list=PL9gnSGHSqcnr_DxHsP7AW9ftq0AtAyYqJ), one file per lecture PDF, numbered in watch order. This is the layer *underneath* the curriculum: [01-concepts/](../01-concepts/) starts at item **0.1 — Running a Java Program** and teaches you to *write* Java, while these answer **why** programming languages exist at all, **how** you plan a program before typing it, and **what actually happens** between `javac` and your output.

Kept **separate from `01-concepts/`** on purpose: those notes map 1:1 to [ROADMAP.md](../ROADMAP.md) items and are written session-by-session as you learn. These follow the *playlist*, not the roadmap — mixing them in would break that mapping.

## Read in this order
| # | File | Source PDF | What it answers |
|---|------|------------|-----------------|
| **1** | [01-Intro_to_Programming_Notes.md](01-Intro_to_Programming_Notes.md) | `Intro_to_Programming_Notes.pdf` | Why do languages exist? · procedural vs functional vs OOP · static vs dynamic typing · stack & heap · garbage collection |
| **2** | [02-Flow_Of_Program_Notes.md](02-Flow_Of_Program_Notes.md) | `Flow_Of_Program_Notes.pdf` | Flowchart symbols · pseudocode · the prime-number problem · why √n is enough (O(n) → O(√n)) |
| **3** | [03-Introduction_to_Java_Notes.md](03-Introduction_to_Java_Notes.md) | `Introduction_to_Java_Notes.pdf` | `.java` → `.class` → machine code · why Java is platform independent · JDK/JRE/JVM · class loader · interpreter vs JIT |
| **4** | [04-First_Java_Program_Notes.md](04-First_Java_Program_Notes.md) | `First_Java_Program_Notes.pdf` | Structure of a `.java` file · every word of `main` · the 8 primitives · `Scanner` · type conversion, casting & promotion · what the `.class` file contains |
| **5** | [05-Conditionals_and_Loops_Notes.md](05-Conditionals_and_Loops_Notes.md) | — *(written from the project code)* | `if` / `else if` ladders · every `switch` form, classic → pattern matching · `for` / `while` / `do-while` / for-each · `break`, `continue`, labels · digit peeling & Fibonacci · what `switch` and loops compile to |

> Each file keeps its **source PDF's name** with a `NN-` prefix, so the numbers give the watch order and the name still points back to the original notes. New lectures continue the sequence: `06-…`, `07-…`

**Then continue to** [01-concepts/0.1-running-java.md](../01-concepts/0.1-running-java.md) — where the curriculum proper begins.

## How to revise with these
1. First pass: read them top-to-bottom, in order.
2. Every pass after that: skip to the **🔁 Rapid Revision** self-test at the bottom of each file, answer out loud *before* expanding, and re-open only the section you fumbled.
3. Files 1 and 3 are the interview-heavy ones (memory model, JVM internals). File 2 is a *thinking habit* — flowchart → pseudocode → code — that pays off on every DSA problem. Files 4 and 5 hold the Java-specific traps — integer division, silent overflow, switch fall-through.

## Conventions
- Same note shape as [06-templates/concept.md](../06-templates/concept.md): intuition → runnable Java → JS comparison → misconceptions → interview angles.
- Every Java snippet was **compiled and run on JDK 21** — the documented outputs are real console output, not assumed.
- Flowcharts are **mermaid** code blocks, so they render as diagrams in Obsidian and on GitHub.
- Where the source PDFs are wrong, the note says so in a **⚠️ Correction to the lecture notes** callout rather than repeating the error. Where a note is written from the project code instead of a PDF (file 5), bugs found while running it are called out the same way.
