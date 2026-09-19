---
type: foundation
title: First Java Program — Structure, Primitives, Input, Type Conversion & Inside the .class File
tags: [foundations, java, main-method, packages, primitives, scanner, type-casting, type-promotion, bytecode, javap, pre-phase-0]
created: 2026-09-09
status: seen
source: "Kunal Kushwaha · Community Classroom — First Java Program"
related: ["[[How Java Works]]", "[[Introduction to Programming]]", "[[Running a Java Program]]", "[[Variables and Types]]", "[[Conditionals]]", "[[Loops]]"]
---

# 4 · Your First Java Program

> 📁 Part 4 of 6 in [Ytube dsa/](README.md) · **Prev:** [03 — Introduction to Java](03-Introduction_to_Java_Notes.md) · **Next:** [05 — Conditionals, Switch & Loops](05-Conditionals_and_Loops_Notes.md)

## Introduction
[03](03-Introduction_to_Java_Notes.md) explained the machinery — `javac`, bytecode, the JVM. This is the first program that actually *uses* it: the rules a `.java` file must obey, every word of `public static void main(String[] args)`, the primitive types, reading input, how Java silently converts between number types — and, at the end, **what the `.class` file the compiler produced actually contains**.

Worked code lives in the course project at `Ytube java dsa/src/com/thowfik/firstjavaprogram/` *(the snippets below show `package com.thowfik;` — the files have since moved into the `com.thowfik.firstjavaprogram` package)*: `Main.java`, `Primitives.java`, `Inputs.java`, `Sum.java`, `TypeCasting.java`. **Every output below is that code actually compiled and run on JDK 21** — including the two places where the lecture's own programs are wrong.

## Characteristics of a Java File
- **Everything lives in a class:** there is no code outside one. A `.java` file *is* a class.
- **Filename = public class name:** `public class Main` must live in `Main.java`.
- **One fixed entry point:** execution starts at `public static void main(String[] args)`.
- **Compiled, then run:** `javac Main.java` → `Main.class` (bytecode) → `java Main`.
- **Package = folder:** `package com.thowfik;` means the file sits in `com/thowfik/`.

---

## 1. Structure of a `.java` File

Source code you write is saved with the **`.java`** extension. The rules:

- **Everything written in a `.java` file must be inside a class.** There is no "top level" like JavaScript.
- **A class with the same name as the file must be present.** `Main.java` must contain `class Main`.
- **That class must be `public`.**
- **A `main` method must be present in that public class** — it is where the program starts.

> 💡 **Naming convention:** the first letter of a class name should be **uppercase** (`Main`, `BankAccount`). This is convention, not a compiler rule — `class main` compiles — but every Java codebase follows it, so breaking it reads as a mistake.

## 2. Compiling and Running

```bash
javac Main.java     # .java  ->  Main.class  (bytecode)
java Main           # run it  (class NAME, no extension)
```

`javac` takes the **file name** (with `.java`). `java` takes the **class name** — no `.java`, no `.class`.

> ⚠️ The lecture PDF writes the command as `Javac Main.java` with a capital J. Commands are **case-sensitive** — it is `javac`, lowercase. A capital `J` gives "command not found".

## 3. Hello World — Every Word Explained

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```
**Output**
```
Hello World
```

| # | Word | What it means |
|---|---|---|
| 1 | **`public`** (line 1) | Access modifier — allows this **class** to be accessed from anywhere. |
| 2 | **`class`** | A named group of properties (data) and functions (behaviour). |
| 3 | **`Main`** | The class's name — must match the file name. |
| 4 | **`public`** (line 2) | Lets the **method** be called from anywhere. The JVM calls `main` from *outside* your class, so it must be public. |
| 5 | **`static`** | Lets `main` run **without creating an object**. The JVM calls it before any object of your class exists. |
| 6 | **`void`** | Return type — this method hands nothing back. |
| 7 | **`main`** | The method's name — the exact name the JVM searches for. |
| 8 | **`String[] args`** | Command-line arguments, as a `String` array. `java Main hello world` → `args = {"hello", "world"}`. |
| 9 | **`System`** | A `final` class in the `java.lang` package. |
| 10 | **`out`** | A `public static` field of `System`, of type `PrintStream`. |
| 11 | **`println`** | A `PrintStream` method — prints its argument **and adds a newline**. `print` does the same **without** the newline. |

**Change any one of pieces 4–8 and the program stops being runnable:**

| Remove / change | What happens |
|---|---|
| drop `public` | compiles fine, but at launch: `Error: Main method not found in class Main` |
| drop `static` | same error — the JVM looks specifically for a **static** `main` and will not build an object to find one |
| `void` → `int` | Java demands a `return` on every path, and the JVM no longer recognises it as an entry point |
| rename `main` → `start` | no entry point found |
| drop `String[] args` | the signature no longer matches; not an entry point |

> 💡 The JVM does not run "any method" — it hunts for **this exact signature**. Note that it compiles either way: `javac` checks your code is legal Java; the *launcher* is what needs the signature.

**The version in the project** (`com/thowfik/Main.java`) — same skeleton, doing something real:

```java
package com.thowfik;

import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);

        System.out.print("Hey, what's your name? ");
        String name = input.nextLine();

        System.out.println("Hey " + name + ", how are you!");
    }
}
```
**Output** *(typing `Thowfik` at the prompt)*
```
Hey, what's your name? Thowfik
Hey Thowfik, how are you!
```

Note `print` for the prompt (no newline, so you type on the same line) and `println` for the answer. This is the exact class disassembled in §15.

## 4. What Is a Package?

A **package is just a folder** that Java files live in — and it is also a namespace.

```java
package com.thowfik;
```

The file must sit at `…/src/com/thowfik/Main.java`, and the class's true name becomes `com.thowfik.Main` — which is what you run:

```bash
javac -d out src/com/thowfik/Main.java
java -cp out com.thowfik.Main        # <- FULL name, dots not slashes
```

> 💡 Packages are why two libraries can each define a `Main` without colliding. The reversed-domain convention (`com.yourname`) makes clashes essentially impossible.

## 5. Comments

Text in the source that the **compiler ignores**.

```java
// single-line comment

/* multi-line
   comment */

/** Javadoc comment — used to document classes and methods */
```

## 6. Primitive Data Types

A **primitive** is a type that **cannot be broken down further**. Contrast with `String`, which is not primitive because it decomposes into `char`s:

```
"Kunal"  ->  'K' 'u' 'n' 'a' 'l'
```

You cannot break an `int` or a `char` into anything smaller. A primitive is **not an object** — it holds the raw value directly, with no heap allocation and no reference ([01](01-Intro_to_Programming_Notes.md)).

Java has exactly **8**:

| Type | Size | Range (approx) | Default | Example |
|---|---|---|---|---|
| `byte` | 1 byte | −128 to 127 | `0` | `byte age = 25;` |
| `short` | 2 bytes | −32,768 to 32,767 | `0` | `short year = 2026;` |
| `int` | 4 bytes | ±2.1 billion | `0` | `int rollno = 64;` |
| `long` | 8 bytes | ±9.2 quintillion | `0L` | `long big = 3262343252433212L;` |
| `float` | 4 bytes | ~7 decimal digits | `0.0f` | `float marks = 98.6f;` |
| `double` | 8 bytes | ~15 decimal digits | `0.0d` | `double d = 46254253.43542;` |
| `char` | 2 bytes | 0 to 65,535 (Unicode) | `'\u0000'` | `char letter = 'r';` |
| `boolean` | JVM-dependent | `true` / `false` | `false` | `boolean check = true;` |

> ⚠️ **The lecture lists only 6**, omitting `byte` and `short`. There are **8**. Interviewers ask for the count.

**The suffix rules that actually bite:**
- A whole number with no suffix is an **`int`** → too big? add **`L`** (capital — lowercase `l` looks like `1`).
- A decimal with no suffix is a **`double`** → want a float? add **`f`**, or it will not compile.
- `'r'` in **single** quotes is a `char`; `"r"` in double quotes is a `String`.
- Daily use: `int`, `double`, `boolean`, `char`. `short` is almost never worth it.

**Literals and identifiers** — two words that get used constantly:

```java
int a = 10;
//  ^      ^
//  |      literal   (the raw value written in source)
//  identifier       (the name you chose)
```
An **identifier** is any name you pick: variables, methods, classes, packages. A **literal** is a value written directly in the code — `10`, `'A'`, `true`, `"hello"`.

**Output: `Primitives.java`**
```
int     rollno       = 64
char    letter       = r
float   marks        = 98.6
double  largeDecimal = 4.625425343542E7
long    largeInteger = 3262343252433212
boolean check        = true
byte    age          = 25
short   year         = 2026
The letter 'r' as a number = 114
```

> ⚠️ **Two things in that output.**
> **(a)** `46254253.43542` printed as **`4.625425343542E7`** — Java switches to scientific notation for doubles ≥ 10⁷. The value is unchanged, only the display. `System.out.printf("%.5f%n", d)` forces plain digits.
> **(b)** `'r'` printed as **`114`**. A `char` *is* a number — its Unicode code point — which is why `('a' + 1)` is legal arithmetic. This underpins most character-counting tricks in DSA.

## 7. Reading Input — `Scanner`

`Scanner` lives in `java.util`. Three steps: **import** it, **create** one, **use** it.

```java
import java.util.Scanner;               // 1. import

Scanner input = new Scanner(System.in); // 2. create
int rollno = input.nextInt();           // 3. use
```

| Word | Meaning |
|---|---|
| `Scanner` | the class that reads input, from `java.util` |
| `input` | the **object** you created (name is yours to choose) |
| `new` | the keyword that creates an object |
| `System.in` | `System` is a class, `in` is its field for the **standard input stream** (the keyboard) |

| Method | Reads |
|---|---|
| `nextInt()` / `nextFloat()` / `nextDouble()` / `nextLong()` / `nextBoolean()` | one token, converted to that type |
| `next()` | one **word**, stopping at the first space |
| `nextLine()` | the **whole line** including spaces, up to Enter |

Typing `Hey kunal`: `next()` returns `Hey`; `nextLine()` returns `Hey kunal`.

**Output: `Inputs.java`** *(typing `64`, then `98.6`)*
```
Please enter your roll number: 64
Please enter your marks: 98.6
Your roll number is 64
Your marks are 98.6
```

> ⚠️ **Never print the Scanner object itself.** `System.out.println(input)` calls its `toString()` and dumps internal state:
> ```
> java.util.Scanner[delimiters=\p{javaWhitespace}+][position=7][match valid=true]…
> ```
> Print the **variable you read into** (`rollno`, `marks`), never the reader.

## 8. Program — Sum of Two Numbers

```java
package com.thowfik;

import java.util.Scanner;

public class Sum {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        System.out.print("Enter the first value = ");
        int num1 = input.nextInt();
        System.out.print("Enter the second value = ");
        int num2 = input.nextInt();
        int sum = num1 + num2;
        System.out.println("Sum =  " + sum);
    }
}
```
**Output** *(typing `70`, then `80`)*
```
Enter the first value = 70
Enter the second value = 80
Sum =  150
```

## 9. Type Conversion (automatic / widening)

When one type is assigned to another, Java performs an **automatic conversion** — but only under two conditions:

1. The two types are **compatible**.
2. The destination type is **larger** than the source type.

```java
int i = 100;
long l = i;      // OK — int (4 bytes) fits inside long (8 bytes)
double d = l;    // OK — long fits inside double
```

This is **widening**: no data can be lost, so Java does it silently. Going the other way needs your explicit permission.

## 10. Type Casting (explicit / narrowing)

Converting a **larger** type into a **smaller** one. Java refuses to do it silently, so you write the target type in parentheses:

```java
int num = (int) (76.5f);
System.out.println(num);
```
```
76
```

The `.5` is **truncated, not rounded** — casting a decimal to an integer chops the fractional part.

**What casting can silently destroy:**

```java
int a = 257;
byte b = (byte) (a);
System.out.println(b);
```
```
1
```

`byte` holds −128…127. 257 does not fit, so the extra bits are discarded and you get `1` (257 = 256 + 1, and it wraps). **No warning, no error** — this is why narrowing requires an explicit cast: the cast *is* you accepting the risk.

## 11. Automatic Type Promotion in Expressions

While evaluating an expression, intermediate values can exceed the operands' range — so Java **promotes** them:

1. Every `byte`, `short` or `char` operand is promoted to **`int`** when an expression is evaluated.
2. If any operand is `long`, `float` or `double`, the **whole expression** is promoted to that type.

```java
byte a = 40;
byte b = 50;
byte c = 100;
int d = (a * b) / c;
System.out.println(d);
```
```
20
```
`a * b` is 2000 — far outside `byte`'s range. It works because both were promoted to `int` first.

**The trap this creates:**

```java
byte b = 50;
b = b * 2;      // does NOT compile
```
```
error: incompatible types: possible lossy conversion from int to byte
```
`b * 2` promotes to `int`, and an `int` cannot be assigned back into a `byte` without a cast. But:

```java
byte b = 50;
b *= 2;         // compiles fine -> 100
```
**Compound assignment operators (`*=`, `+=`, …) contain a hidden implicit cast.** `b *= 2` is not identical to `b = b * 2` — a classic interview question.

**The full promotion example** — this is `com/thowfik/TypeCasting.java`:

```java
byte b = 43;
char c = 'a';
short s = 1024;
int i = 50000;
float f = 5.67f;
double d = 0.1234;
double result = (f * b) + (i / c) - (d - s);
//  float + int - double = double
System.out.println((f * b) + "---" + (i / c) + "---" + (d - s));
System.out.println(result);
```
```
243.81---515----1023.8766
1782.6865975585938
```

Reading it term by term:
- `f * b` → `float × byte` → `byte` promotes to `int`, then the whole term to **float** = `243.81`
- `i / c` → `int / char` → `char` (`'a'` = 97) promotes to `int` → **integer division**: `50000 / 97` = **`515`**, not 515.46 — the remainder is discarded
- `d - s` → `double − short` → **double** = `-1023.8766` (0.1234 − 1024)
- the expression contains a `double`, so `result` is a **double**

> 💡 **Two things hiding in that output.**
> **(a)** `515----1023.8766` looks like four dashes but is three: the separator `"---"` followed by the **minus sign** of a negative number.
> **(b)** The final result is `1782.68…`, *larger* than any term — because `- (d - s)` subtracts a **negative**, so it adds `1023.8766`. Worth tracing by hand: `243.81 + 515 + 1023.8766`.

> 💡 `i / c` is the one to remember. Two ints divide as integers and silently truncate. Off-by-one bugs in DSA very often trace back to exactly this.

### The rest of the `TypeCasting.java` experiments

Each of these is a commented-out line in that file — all verified:

```java
int number = 'A';
System.out.println(number);                 // 65   -- char IS its Unicode code point

System.out.println(3 * 5.6235234);          // 16.8705702   (double, ~15 digits)
System.out.println(3 * 5.6235234f);         // 16.87057     (float,  ~7 digits)
```

The last pair is the clearest demonstration of why `double` is the default: the same arithmetic in `float` **loses precision at the 7th digit** (`16.87057` vs `16.8705702`). Use `float` only when memory genuinely matters; otherwise `double`.

## 12. Program — Prime Number

```java
import java.util.Scanner;

public class Prime {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        System.out.print("Please enter a number: ");
        int n = in.nextInt();

        if (n <= 1) {
            System.out.println("Neither prime nor composite");
            return;
        }
        for (int c = 2; c * c <= n; c++) {
            if (n % c == 0) {
                System.out.println("Not Prime");
                return;
            }
        }
        System.out.println("Prime");
    }
}
```
**Output** *(one run per line)*
```
n=1   -> Neither prime nor composite
n=2   -> Prime
n=4   -> Not Prime
n=9   -> Not Prime
n=17  -> Prime
n=25  -> Not Prime
n=49  -> Not Prime
n=97  -> Prime
```

> ⚠️ **The lecture's version has a real bug — I ran it to confirm.** It uses `while (c*c < n)` and then `if (c*c > n) print("Prime")`. For a **perfect square of a prime**, `c*c` ends up exactly **equal** to `n`, so both conditions are false and the program **prints nothing at all**:
>
> ```
> n=9   -> (NOTHING PRINTED)
> n=25  -> (NOTHING PRINTED)
> n=49  -> (NOTHING PRINTED)
> ```
>
> The `if (n == 4)` special case in the lecture code is a patch for this very bug — 4 is just the smallest case where it shows. The fix is the boundary: **`c * c <= n`**, which removes the need for any special case. This is the same √n reasoning from [02](02-Flow_Of_Program_Notes.md); getting `<` vs `<=` right *is* the problem.

## 13. Control Flow — Quick Examples

**`if`** — the body runs only when the condition is true.
```java
int a = 10;
if (a == 10) {
    System.out.println("Hello");
}
```
```
Hello
```

**`while`** — repeats until the condition becomes false.
```java
int count = 1;
while (count != 5) {
    System.out.println("count");
    count++;
}
```
```
count
count
count
count
```
> 💡 It prints the **word** `count` four times, not `1 2 3 4` — `"count"` in quotes is a string literal, not the variable. Drop the quotes to print the value. Also note `!= 5` runs 1,2,3,4 — four iterations, not five.

**`for`** — the same loop with init, condition and update on one line.
```java
for (int count = 1; count != 5; count++) {
    System.out.println(count);
}
```
```
1
2
3
4
```

> ⚠️ Prefer `<` over `!=` as a loop condition. `count != 5` never terminates if the counter ever skips past 5 (e.g. `count += 2`); `count < 5` is safe either way.

## 14. Program — Celsius to Fahrenheit

```java
import java.util.Scanner;

public class CelsiusToFahrenheit {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        float tempC = in.nextFloat();
        float tempF = (tempC * 9 / 5) + 32;
        System.out.println(tempF);
    }
}
```
**Output** *(typing `45`)*
```
113.0
```

> 💡 This works **only because `tempC` is a `float`**. Written as `(tempC * 9) / 5` with an `int` temperature it still works, but `tempC * (9 / 5)` would compute `9/5` as **integer division = 1** and give you the wrong answer for every input. Same integer-division trap as `i / c` in §11.

---

## 15. What the `.class` File Actually Looks Like

Compile `Main.java` and you get `Main.class`. Open it in an editor and you get garbage, because it is **binary**. Point IntelliJ at it and its FernFlower decompiler reconstructs Java-looking source — but that is a *reconstruction*, not the file.

### The raw bytes
```
00000000: cafe babe 0000 0041 0040 0a00 0200 0307  .......A.@......
00000010: 0004 0c00 0500 0601 0010 6a61 7661 2f6c  ..........java/l
00000020: 616e 672f 4f62 6a65 6374 0100 063c 696e  ang/Object...<in
00000030: 6974 3e01 0003 2829 5607 0008 0100 116a  it>...()V......j
00000040: 6176 612f 7574 696c 2f53 6361 6e6e 6572  ava/util/Scanner
```

| Bytes | Meaning |
|---|---|
| `cafe babe` | **magic number** — every `.class` begins with it. The JVM checks it before parsing anything else |
| `0000` | minor version |
| `0041` | major version = **65** → compiled for Java 21 |
| `0040` | the constant pool has **64** entries |

Then `01 00 10 6a 61 76 61…` reads as: tag `01` = a UTF-8 string, length `0x0010` = 16, contents `java/lang/Object`. That is why `java/util/Scanner` and `<init>` are plainly readable — **names and string literals sit in the file as text**.

For the tiny Hello World, the class file is **413 bytes** against 116 bytes of source: compiling made it *bigger*, because every name is spelled out in full.

### The readable form — `javap`

You don't need a decompiler; the JDK ships a disassembler:

```bash
javap -c -p -cp out com.thowfik.Main
```
```java
public class com.thowfik.Main {
  public com.thowfik.Main();
    Code:
       0: aload_0
       1: invokespecial #1    // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: new           #7    // class java/util/Scanner
       3: dup
       4: getstatic     #9    // Field java/lang/System.in:Ljava/io/InputStream;
       7: invokespecial #15   // Method java/util/Scanner."<init>":(Ljava/io/InputStream;)V
      10: astore_1
      11: getstatic     #18   // Field java/lang/System.out:Ljava/io/PrintStream;
      14: ldc           #22   // String Hey, what\'s your name?
      16: invokevirtual #24   // Method java/io/PrintStream.print:(Ljava/lang/String;)V
      19: aload_1
      20: invokevirtual #30   // Method java/util/Scanner.nextLine:()Ljava/lang/String;
      23: astore_2
      24: getstatic     #18   // Field java/lang/System.out:Ljava/io/PrintStream;
      27: aload_2
      28: invokedynamic #34,0 // InvokeDynamic #0:makeConcatWithConstants:(…)…
      33: invokevirtual #38   // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      36: return
}
```

### Reading it — the JVM is a *stack machine*

There are no registers. Every instruction pushes to or pops from an **operand stack**. `Scanner input = new Scanner(System.in);` becomes five instructions:

1. **`new #7`** — allocate a `Scanner` **on the heap**, push the reference
2. **`dup`** — duplicate the reference (one copy for the constructor to consume, one to store)
3. **`getstatic #9`** — push `System.in`
4. **`invokespecial #15`** — pop both, run `Scanner`'s constructor
5. **`astore_1`** — pop the reference, store it in local slot 1

That is the stack/heap split from [01](01-Intro_to_Programming_Notes.md) made literal: `new` puts the **object on the heap**, `astore_1` puts the **reference in a local slot**.

### Three things this reveals

**(a) A constructor you never wrote.** `public com.thowfik.Main();` is in the file even though `Main.java` declares none. **`javac` generates a default no-arg constructor** when a class declares no constructor, and its body is `aload_0; invokespecial Object.<init>; return` — push `this`, call `super()`. Decompile any class of your own and you will see this phantom constructor. It is real, and the compiler added it.

**(b) String `+` is not what you think.** `"Hey " + name + ", how are you!"` compiles to a single **`invokedynamic makeConcatWithConstants`**. Since Java 9 the compiler emits a call the JVM wires up at runtime, rather than the old `StringBuilder` chain. Worth remembering at [[Strings]] (1.4): the "`+` in a loop is a trap" rule is still true, but for a different reason than most tutorials give.

**(c) `#7`, `#18`, `#22` are constant-pool indexes.** Instructions never carry names inline; they carry an index into a table at the top of the file:

```
   #7 = Class      #8    // java/util/Scanner
  #18 = Fieldref   …     // java/lang/System.out:Ljava/io/PrintStream;
  #22 = String     #23   // Hey, what's your name?
```

That indirection is exactly the **"replace symbolic references with direct references"** step of Linking in [03](03-Introduction_to_Java_Notes.md).

### Type descriptors
The strings after a `:` are **descriptors**:

| Descriptor | Means |
|---|---|
| `()V` | takes nothing, returns `void` |
| `(Ljava/lang/String;)V` | takes a `String`, returns `void` |
| `[Ljava/lang/String;` | an **array** of `String` — that's `String[] args` |
| `I` `J` `D` `F` `C` `Z` `B` `S` | `int` `long` `double` `float` `char` `boolean` `byte` `short` |

So `main`'s full descriptor is `([Ljava/lang/String;)V`.

### Why it decompiles so cleanly
A `.class` keeps full class, method and field **names** plus types, because the JVM needs them at runtime to link things together — so FernFlower can rebuild near-original source. Compiled C keeps none of that, which is why it decompiles into a mess. It is also why **obfuscators** exist: ProGuard renames everything to `a`, `b`, `c` precisely to destroy this readability.

### The headline
**`aload_0` and `invokevirtual` are not machine code.** No CPU executes them. They are instructions for the *virtual* machine — which is exactly why the same bytes run on Windows, macOS and Linux ([03](03-Introduction_to_Java_Notes.md)).

> 💡 Try it on anything you've compiled: `javap -c -p ClassName` for instructions, `javap -v ClassName` for the header and constant pool.

---

## ⚠️ Common Misunderstandings
**1. "It compiled, so it will run."**
❌ compiling means runnable · ✅ `javac` checks legal Java; the **launcher** separately needs `public static void main(String[])`.
*Why:* drop `static` and you get a clean compile followed by `Error: Main method not found`.

**2. `float marks = 98.6;`**
❌ works · ✅ needs `98.6f`.
*Why:* an unsuffixed decimal literal is a `double`, and Java will not narrow it silently.

**3. `b = b * 2;` where `b` is a `byte`.**
❌ same as `b *= 2` · ✅ the first is a **compile error**, the second compiles and gives `100`.
*Why:* operands promote to `int`, and `int` → `byte` needs a cast. Compound operators carry a **hidden implicit cast**; plain assignment does not.

**4. `50000 / 'a'` gives 515.46.**
❌ decimal result · ✅ **`515`** — `char` promotes to `int`, and int ÷ int **truncates**.
*Why:* integer division discards the remainder. Cast one side to `double` if you want a real quotient.

**5. `(int) 76.5` rounds to 77.**
❌ rounds · ✅ **truncates to 76**.
*Why:* casting a decimal to an integer chops the fraction. Use `Math.round()` to round.

**6. "A cast is always safe because I asked for it."**
❌ safe · ✅ `(byte) 257` silently yields **`1`**.
*Why:* narrowing discards the bits that don't fit. The cast is you *accepting* the loss, not preventing it.

**7. `while (count != 5)` is a fine loop condition.**
❌ safe · ✅ prefer `count < 5`.
*Why:* `!=` never terminates if the counter steps past the target (`count += 2`). `<` is safe either way.

**8. "The `.class` file is what the decompiler shows me."**
❌ that's the file · ✅ it's FernFlower's **reconstruction** from bytecode; the file is binary.
*Why:* `javap` shows the truth. The decompiler shows a plausible Java rendering of it.

**9. "The empty constructor in the decompiled output is a bug."**
❌ invented by the tool · ✅ **`javac` generated it**, because the class declared none.

**10. Java has 6 primitive types.**
❌ six · ✅ **eight** — the lecture's table omits `byte` and `short`.

## JavaScript Comparison
| | Java | JavaScript |
|---|---|---|
| Entry point | `public static void main(String[] args)` — exact signature | top of the file; `node file.js` |
| Command-line args | `args` parameter | `process.argv` |
| Namespacing | `package com.thowfik;` = a real folder | ES modules / file paths |
| Number types | 8 primitives; size and suffix matter | one `number` (plus `BigInt`) |
| `50000 / 97` | **`515`** (int division truncates) | `515.46…` (always float division) |
| Single character | `char c = 'r'` — a distinct type | just a 1-char string |
| Narrowing | explicit cast required: `(byte) x` | no concept — numbers are numbers |
| Reading input | `new Scanner(System.in)` | `readline` / `prompt()` |
| Compiled artifact | `.class` bytecode you can disassemble | none — the source *is* the artifact |

> 💡 The integer-division row is the one that bites JS developers hardest. In JS, `/` always gives you a decimal. In Java, `int / int` throws the remainder away — silently, with no warning.

## Interview Angles
- **"Why is `main` static?"** — The JVM calls it before any object of your class exists; a non-static method would need an instance to be called on.
- **"Can you overload `main`?"** — Yes, `main(int)` compiles fine, but the JVM only ever launches `main(String[])`. Tests whether you separate compiling from launching.
- **"Difference between `b = b * 2` and `b *= 2` for a byte?"** — The first fails to compile; compound assignment has an implicit narrowing cast. A very common trick question.
- **"What does `(byte) 257` give and why?"** — `1`. Narrowing discards high-order bits. Follow-up: why Java forces an explicit cast at all.
- **"What are the 8 primitives, and which are the defaults for literals?"** — Whole numbers default to `int`, decimals to `double`.
- **"How would you check if a number is prime?"** — Naive **O(n)**, then **O(√n)** with the factor-pair reasoning. Get the `<=` boundary right; `<` breaks on perfect squares, exactly as the lecture code does.
- **"Have you looked at bytecode?"** — Rare but a strong signal. Being able to say *"String `+` compiles to `invokedynamic makeConcatWithConstants` since Java 9"* shows you understand the compiler, not just the syntax.

## Related · Next
- **Related:** [[How Java Works]] (the machinery this runs on) · [[Variables and Types]] (0.2) · [[Conditionals]] (0.3) · [[Loops]] (0.4) · [[Strings]] (1.4)
- **Practice:** run `javap -c -p` on `Primitives` and `TypeCasting`. Find where each type is stored (`istore`, `fstore`, `lstore`, …) and where the promotions from §11 appear as explicit `i2f` / `i2d` conversion instructions — proof the JVM tracks types all the way down.
- **Next:** continue the playlist; new lectures become `05-…`

---

## 🔁 Rapid Revision (self-test)
Answer out loud **before** expanding.

<details><summary>1. Four rules a .java file must follow?</summary>

Everything must be inside a class · a class matching the file name must exist · that class must be `public` · it must contain `main`.
</details>

<details><summary>2. Why must `main` be both `public` and `static`?</summary>

`public` because the JVM calls it from outside your class; `static` because the JVM calls it before any object exists, so there is nothing to call it on. Either one missing compiles fine but fails at launch.
</details>

<details><summary>3. What are `System`, `out` and `println`?</summary>

`System` is a final class in `java.lang`; `out` is a `public static` field of type `PrintStream`; `println` is a `PrintStream` method that prints its argument **and** a newline (`print` omits the newline).
</details>

<details><summary>4. Literal vs identifier?</summary>

In `int a = 10;` — `10` is the **literal** (a value written directly in source), `a` is the **identifier** (a name you chose). Identifiers also name methods, classes and packages.
</details>

<details><summary>5. Name all 8 primitives and the default type of `42` and `4.2`.</summary>

`byte short int long float double char boolean`. `42` is an **`int`**; `4.2` is a **`double`**.
</details>

<details><summary>6. Type conversion vs type casting?</summary>

**Conversion** is automatic (widening) — types compatible and destination larger, e.g. `int` → `long`. **Casting** is explicit (narrowing) — `(int) 76.5f` → `76`, and it can silently lose data.
</details>

<details><summary>7. Why does `byte b = 50; b = b * 2;` fail, but `b *= 2` work?</summary>

`b * 2` promotes to `int`, and `int` cannot be assigned to `byte` without a cast. Compound assignment operators contain a **hidden implicit cast**, so `b *= 2` compiles and gives `100`.
</details>

<details><summary>8. In `(f*b) + (i/c) - (d*s)`, why is `i/c` equal to 515?</summary>

`c` is `'a'` = 97, promoted to `int`. `50000 / 97` is **integer division**, so the remainder is discarded: 515, not 515.46.
</details>

<details><summary>9. What does `(byte) 257` print, and why?</summary>

`1`. A `byte` holds −128…127, so the bits that don't fit are discarded and the value wraps (257 = 256 + 1). No warning — the cast is you accepting the loss.
</details>

<details><summary>10. What's wrong with the lecture's prime program?</summary>

It uses `while (c*c < n)` then `if (c*c > n)`. For a perfect square of a prime (9, 25, 49) `c*c` ends up exactly equal to `n`, so neither branch fires and it **prints nothing**. The `if (n == 4)` special case is a patch for the same bug. Fix the boundary: `c * c <= n`.
</details>

<details><summary>11. First four bytes of every .class file, and why?</summary>

`cafe babe` — the magic number. The JVM checks it before parsing to confirm the file really is a class file.
</details>

<details><summary>12. Where did the empty constructor in the decompiled output come from?</summary>

`javac` generated it. A class declaring no constructor gets a default no-arg one whose body is just `super()` — `aload_0; invokespecial Object.<init>; return`.
</details>

<details><summary>13. What does `new` / `dup` / `invokespecial` / `astore_1` tell you about memory?</summary>

`new` allocates the object **on the heap** and pushes its reference; `astore_1` stores that **reference** in a local variable slot. The stack/heap split, made literal in bytecode.
</details>

<details><summary>14. What is `([Ljava/lang/String;)V`, and why isn't it machine code?</summary>

The **descriptor** for `main`: takes a `String[]`, returns `void`. Instructions like `aload_0` target the *virtual* machine — no CPU runs them; the JVM converts them to real machine code at runtime, which is what makes the file portable.
</details>
