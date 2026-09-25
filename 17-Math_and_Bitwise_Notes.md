---
type: foundation
title: Math & Bitwise — Primes, GCD, the Sieve, and Every Bit Trick Worth Knowing
tags: [foundations, java, math, number-theory, primes, sieve, gcd, bitwise, xor, bit-manipulation, pre-phase-0]
created: 2026-09-29
status: seen
source: "Kunal Kushwaha · Community Classroom — lecture 16 Maths & Bitwise Operators"
related: ["[[Flow of the Program]]", "[[Complexity Analysis]]", "[[Bit Manipulation]]", "[[Math & Number Theory]]"]
---

# 17 · Math & Bitwise Operators

> 📁 Part 17 of 28 in [Ytube dsa/](README.md) · **Prev:** [16 — Recursion III](16-Recursion_Sorting_Backtracking_Notes.md) · **Next:** [18 — OOP I](18-OOP_Basics_Notes.md)

## Introduction
Two topics that look like curiosities and are actually interview staples. **Number theory** (primes, GCD, factors) shows up as "optimise this loop" problems. **Bit manipulation** is a whole category of O(1)-space tricks that look like magic until you see the binary.

The unifying theme: both replace obvious O(n) work with something much smaller by exploiting structure.

---

# Part A — Number Theory

## 1. Primality — Back to √n

```java
static boolean isPrime(int n) {
    if (n <= 1) return false;
    int c = 2;
    while (c * c <= n) {
        if (n % c == 0) return false;
        c++;
    }
    return true;
}
```

The `c * c <= n` bound is note 02's factor-pair argument — **O(√n)**. Note this version returns `false` for `n <= 1` rather than the lecture's earlier "neither prime nor composite" message: for a boolean API, `false` is the right answer, since 0, 1 and negatives are certainly not prime.

## 2. Factors — Three Versions

```java
static void factors1(int n) {                    // O(n)
    for (int i = 1; i <= n; i++)
        if (n % i == 0) System.out.print(i + " ");
}

static void factors2(int n) {                    // O(√n)
    for (int i = 1; i <= Math.sqrt(n); i++) {
        if (n % i == 0) {
            if (n / i == i) System.out.print(i + " ");       // perfect square: print once
            else            System.out.print(i + " " + n/i + " ");
        }
    }
}
```
```java
factors3(20);
```
**Output**
```
1 2 4 5 10 20 
```

Same factor-pair insight: every factor below √n has a partner above it, so find one and you get both. `factors2` prints them out of order (1, 20, 2, 10, 4, 5); `factors3` fixes that by collecting the large partners in an `ArrayList` and printing them in reverse afterwards — **O(√n) time and O(√n) space**, the classic trade.

> ⚠️ The `n / i == i` check matters: for `n = 16`, `i = 4` has partner `4`. Without the check you'd print `4 4`.

## 3. GCD and LCM — Euclid's Algorithm

```java
static int gcd(int a, int b) {
    if (a == 0) return b;
    return gcd(b % a, a);
}

static int lcm(int a, int b) {
    return a * b / gcd(a, b);
}
```
```java
System.out.println(lcm(2, 7));
```
**Output**
```
14
```

**Euclid's insight:** `gcd(a, b) == gcd(b % a, a)`. Each step replaces the larger number with a remainder, shrinking fast — **O(log(min(a,b)))**.

The LCM identity `a × b = gcd(a,b) × lcm(a,b)` is worth memorising.

> ⚠️ **`a * b` can overflow** before the division happens. For large ints, write `a / gcd(a, b) * b` — divide first, and the intermediate stays small. Same overflow family as notes 04 and 05.

## 4. The Sieve of Eratosthenes

To find **all** primes up to n, don't test each one. Instead cross out every multiple of each prime.

```java
static void sieve(int n, boolean[] primes) {
    for (int i = 2; i * i <= n; i++) {
        if (!primes[i]) {                       // i is still unmarked -> prime
            for (int j = i * 2; j <= n; j += i) {
                primes[j] = true;               // mark all multiples as NOT prime
            }
        }
    }
    for (int i = 2; i <= n; i++) {
        if (!primes[i]) System.out.print(i + " ");
    }
}
```
```java
sieve(40, new boolean[41]);
```
**Output**
```
2 3 5 7 11 13 17 19 23 29 31 37 
```

**O(n log log n)** — effectively linear, and dramatically better than calling `isPrime` n times (O(n√n)).

Two details worth noticing:

- **`false` means prime.** The array defaults to `false` (note 07), so "unmarked" = prime for free. Inverting it would mean filling the array first.
- **Outer loop to `i * i <= n`.** Any composite ≤ n has a factor ≤ √n, so everything is already crossed out past that point. Inner loop could start at `i * i` rather than `i * 2` — smaller multiples were already marked by smaller primes.

## 5. Newton's Method for Square Roots

```java
static double sqrt(double n) {
    double x = n, root;
    while (true) {
        root = 0.5 * (x + (n / x));         // average the guess with n/guess
        if (Math.abs(root - x) < 0.5) break;
        x = root;
    }
    return root;
}
```
```java
System.out.println(sqrt(40));
```
**Output**
```
6.325023209103984
```

The true √40 is 6.32455532… so this is off by ~0.0005.

> ⚠️ **That error is the tolerance, not the method.** `Math.abs(root - x) < 0.5` stops as soon as two successive guesses are within 0.5 — far too loose. Newton's method *converges quadratically* (the correct digits roughly double each iteration), so tightening to `1e-9` costs two or three more iterations and gives full precision. If you adapt this code, change the tolerance.

---

# Part B — Bitwise Operators

## 6. The Operators

| Operator | Name | Effect |
|---|---|---|
| `&` | AND | 1 only if **both** bits are 1 |
| `\|` | OR | 1 if **either** bit is 1 |
| `^` | XOR | 1 if the bits **differ** |
| `~` | NOT | flips every bit |
| `<<` | left shift | multiply by 2 per shift |
| `>>` | right shift (signed) | divide by 2, **keeps the sign** |
| `>>>` | right shift (unsigned) | divide by 2, **fills with 0** |

```java
5 & 1 = 1     6 & 1 = 0
5 in binary = 101
5 << 1 = 10   5 >> 1 = 2
-8 >> 1 = -4  -8 >>> 1 = 2147483644
```

> 💡 **`>>` vs `>>>` is the one to remember.** `-8 >> 1` is `-4` (arithmetic shift, sign preserved); `-8 >>> 1` is `2147483644` (logical shift, the sign bit becomes a normal 1). `>>>` on a negative number almost always indicates a bug — unless you're deliberately treating the int as unsigned. Java has no `<<<`; left shift needs no sign handling.

## 7. Odd or Even

```java
private static boolean isOdd(int n) {
    return (n & 1) == 1;
}
```
```java
System.out.println(isOdd(68));
```
**Output**
```
false
```

The last bit *is* the parity: odd numbers end in 1. Marginally faster than `% 2` and — importantly — **correct for negatives**, where `n % 2` gives `-1` for odd negatives and breaks a naive `== 1` test.

## 8. XOR — the Workhorse

Two properties make XOR the most useful bit operator in interviews:

```
x ^ x = 0        (a number cancels itself)
x ^ 0 = x        (zero is the identity)
```
**Verified:** `5^5 = 0`, `5^0 = 5`.

**Find the unique number** — every element appears twice except one:

```java
private static int ans(int[] arr) {
    int unique = 0;
    for (int n : arr) {
        unique ^= n;
    }
    return unique;
}
```
```java
int[] arr = {2, 3, 3, 4, 2, 6, 4};
System.out.println(ans(arr));
```
**Output**
```
6
```

Every pair cancels to 0, leaving the singleton. **O(n) time, O(1) space** — no HashSet, no sorting. XOR is also commutative and associative, so order doesn't matter.

**Swap without a temp:**
```java
int x = 3, y = 7;
x ^= y; y ^= x; x ^= y;
```
```
x=7 y=3
```
A party trick rather than good code (it fails if both refer to the same variable, and a temp is clearer), but it demonstrates the self-cancelling property.

## 9. `n & (n-1)` — Clear the Lowest Set Bit

The most valuable single identity here. Subtracting 1 flips the lowest set bit and everything below it, so AND-ing removes exactly that bit.

**Count set bits (Brian Kernighan's algorithm):**
```java
private static int setBits(int n) {
    int count = 0;
    while (n > 0) {
        count++;
        n = n & (n - 1);          // remove one set bit per iteration
    }
    return count;
}
```
```java
System.out.println(Integer.toBinaryString(234567));
System.out.println(setBits(234567));
```
**Output**
```
111001010001000111
9
```
Count the 1s: nine. The loop runs **once per set bit**, not once per bit — O(set bits) instead of O(32).

**Power of two:** a power of two has exactly one set bit, so removing it gives zero:
```java
boolean ans = (n & (n - 1)) == 0;
```
```java
int n = 31;    // 11111
```
```
false
```
Verified: `16 -> true`, `31 -> false`.

> ⚠️ **This is buggy for `n = 0`** — and the lecture's own comment says `// note: fix for n = 0`. Verified:
> ```
> PowOfTwo n=0 : (0 & -1) == 0 -> true   <- WRONG, 0 is not a power of two
> ```
> The fix: `n > 0 && (n & (n - 1)) == 0`. Also guard negatives — `-2147483648` has one set bit and would pass.

## 10. Fast Exponentiation

```java
int ans = 1;
while (power > 0) {
    if ((power & 1) == 1) {      // this bit is set -> multiply in the current base
        ans *= base;
    }
    base *= base;                // square the base
    power = power >> 1;          // move to the next bit
}
```
```java
base = 2, power = 4;
```
**Output**
```
16
```

Exponentiation by squaring, driven by the **binary representation of the exponent**. 2¹³ = 2⁸ × 2⁴ × 2¹ because 13 is `1101`. **O(log power)** instead of O(power) — the difference between 30 multiplications and a billion.

## 11. Magic Number and Range XOR

**Magic number** — treat n's bits as which powers of 5 to include:
```java
int ans = 0, base = 5;
while (n > 0) {
    int last = n & 1;
    n = n >> 1;
    ans += last * base;
    base = base * 5;
}
```
For `n = 5` (binary `101`): 5¹ + 5³ = 5 + 125 = **130**. Verified.

**XOR of a range** — XOR from 0 to n follows a period-4 pattern:

| n % 4 | XOR of 0..n |
|---|---|
| 0 | `n` |
| 1 | `1` |
| 2 | `n + 1` |
| 3 | `0` |

```java
int ans = xor(b) ^ xor(a - 1);       // XOR of a..b
```
For a=3, b=9: **2** — and the brute-force loop agrees (both printed `2`). **O(1) instead of O(n)**, using `x ^ x = 0` to cancel everything below `a`.

---

## ⚠️ Common Misunderstandings
**1. `>>` and `>>>` are the same.**
❌ same · ✅ `-8 >> 1` is `-4`; `-8 >>> 1` is `2147483644`. `>>` keeps the sign, `>>>` fills with zeros.

**2. `(n & (n-1)) == 0` correctly detects powers of two.**
❌ correct · ✅ **`0` passes it** — the lecture flags this. Use `n > 0 && (n & (n-1)) == 0`.

**3. `n % 2 == 1` detects odd numbers.**
❌ always · ✅ fails for negatives (`-3 % 2` is `-1`). `(n & 1) == 1` is correct for both signs.

**4. `a * b / gcd(a,b)` is a safe LCM.**
❌ safe · ✅ `a * b` can overflow first. Write `a / gcd(a,b) * b`.

**5. Newton's method is inherently imprecise.**
❌ the method · ✅ the **tolerance**. `< 0.5` gives 6.325023 vs the true 6.324555. Tighten to `1e-9`.

**6. Use `isPrime` in a loop to list primes.**
❌ O(n√n) · ✅ the **sieve** is O(n log log n).

**7. XOR swap is good practice.**
❌ good · ✅ a party trick. It fails when both operands are the same variable, and a temp is clearer and just as fast.

**8. Bit tricks are premature optimisation.**
❌ always · ✅ for parity, powers of two, set-bit counts and "find the unique element", they're the *clearest* O(1)-space solution — and interviewers ask for them by name.

## JavaScript Comparison
| | Java | JavaScript |
|---|---|---|
| Bitwise ops | on 32-bit `int` | operands **coerced to 32-bit**, then back to double |
| `>>>` | unsigned right shift | same |
| Large numbers | `long` is 64-bit | bitwise silently truncates above 2³¹ |
| Binary string | `Integer.toBinaryString(n)` | `n.toString(2)` |
| Integer division | `/` truncates | `Math.floor` or `\| 0` |
| Big integers | `BigInteger` | `BigInt` (no bitwise mixing with Number) |

> 💡 JS's `|0` idiom is a bitwise OR used purely to force a 32-bit integer — the same coercion that makes bitwise ops on large JS numbers silently wrong. Java's `int` is honestly 32-bit, so the behaviour is at least predictable.

## Interview Angles
- **"Find the number that appears once."** — XOR everything. O(n) time, O(1) space. Near-guaranteed question.
- **"Count set bits."** — `n & (n-1)` in a loop; O(set bits). Mention `Integer.bitCount` exists but they want the trick.
- **"Is it a power of two?"** — `n > 0 && (n & (n-1)) == 0`. **Say the `n > 0` guard out loud** — that's the point of the question.
- **"Compute x^n efficiently."** — Exponentiation by squaring, O(log n), driven by the exponent's bits.
- **"All primes up to n."** — Sieve, O(n log log n). Explain why `false` means prime.
- **"GCD?"** — Euclid, `gcd(b % a, a)`, O(log min(a,b)). Follow-up: LCM via `a / gcd * b` to avoid overflow.
- **"Swap without a temp."** — XOR, while noting you wouldn't ship it.

## Related · Next
- **Related:** [[Flow of the Program]] (02 — the √n argument) · [[Complexity Analysis]] (13) · [[Bit Manipulation]] (5.9) · [[Math & Number Theory]] (5.10)
- **Practice:** implement the sieve, then Single Number (LeetCode 136), Number of 1 Bits (191), Power of Two (231) and Counting Bits (338). Each is three lines once the trick is clear.
- **Next:** [18 — OOP I](18-OOP_Basics_Notes.md)

---

## 🔁 Rapid Revision (self-test)
<details><summary>1. Why is `isPrime` O(√n)?</summary>

Factors pair around √n — if nothing below √n divides n, nothing above does either. Same argument as note 02.
</details>

<details><summary>2. Why does listing factors need `if (n / i == i)`?</summary>

For a perfect square the factor and its partner are the same (16 → 4×4), so without the check it prints `4 4`.
</details>

<details><summary>3. Euclid's GCD, and the LCM identity?</summary>

`gcd(a, b) = gcd(b % a, a)`, base `a == 0` → `b`. O(log min(a,b)). `a × b = gcd × lcm`, so `lcm = a / gcd * b` (divide first to avoid overflow).
</details>

<details><summary>4. How does the sieve work, and what does `false` mean?</summary>

Cross out every multiple of each prime. `false` means prime, exploiting the default value of a `boolean[]`. O(n log log n).
</details>

<details><summary>5. `-8 >> 1` vs `-8 >>> 1`?</summary>

`-4` (sign preserved) vs `2147483644` (zero-filled). `>>>` on a negative is usually a bug.
</details>

<details><summary>6. The two XOR properties, and what they buy you?</summary>

`x ^ x = 0` and `x ^ 0 = x`. XOR the whole array and duplicates cancel, leaving the unique element — O(n) time, O(1) space.
</details>

<details><summary>7. What does `n & (n-1)` do, and what two problems does it solve?</summary>

Clears the lowest set bit. Loop it to count set bits (O(set bits)); one application reaching 0 means exactly one bit was set → power of two.
</details>

<details><summary>8. What's wrong with `(n & (n-1)) == 0` as a power-of-two test?</summary>

`n = 0` passes but isn't a power of two. Use `n > 0 && (n & (n-1)) == 0`.
</details>

<details><summary>9. How does exponentiation by squaring work?</summary>

Walk the exponent's bits: multiply the running answer by the current base when a bit is set, square the base each step. O(log power).
</details>

<details><summary>10. Why is `(n & 1) == 1` better than `n % 2 == 1`?</summary>

It's correct for negatives — `-3 % 2` is `-1` in Java, so the modulo test fails.
</details>
