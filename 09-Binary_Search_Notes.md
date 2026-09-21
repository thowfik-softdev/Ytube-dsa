---
type: foundation
title: Binary Search — The Template, the Overflow Bug, and Every Variant That Matters
tags: [foundations, java, binary-search, searching, sorted-arrays, complexity, interview, pre-phase-0]
created: 2026-09-29
status: seen
source: "Kunal Kushwaha · Community Classroom — lecture 10 Binary Search"
related: ["[[Linear Search]]", "[[Arrays]]", "[[Big-O Intuition]]", "[[Binary Search]]"]
---

# 9 · Binary Search

> 📁 Part 9 of 28 in [Ytube dsa/](README.md) · **Prev:** [08 — Linear Search](08-Linear_Search_Notes.md) · **Next:** [10 — Sorting](10-Sorting_Notes.md)

## Introduction
**Binary search** finds a target in a **sorted** array by repeatedly halving the search space. Look at the middle: if the target is smaller, throw away the right half; if larger, throw away the left. Each comparison eliminates half of what's left.

That is the whole idea, and it turns **O(n)** into **O(log n)**. For a million elements, linear search may take a million comparisons; binary search takes **20**.

This is the single most-asked algorithm in interviews — not because it is hard, but because it is *easy to get subtly wrong*. Boundaries, overflow, and the "return something other than the index" variants are where candidates fail.

## Characteristics
- **Requires sorted data.** Non-negotiable; the whole method rests on it.
- **O(log n) time, O(1) space** (iterative).
- **Three pointers:** `start`, `end`, and `mid` recomputed every pass.
- **The search space always shrinks.** If it ever doesn't, you have an infinite loop.

---

## 1. The Template

```java
static int binarySearch(int[] arr, int target) {
    int start = 0;
    int end = arr.length - 1;

    while (start <= end) {
        int mid = start + (end - start) / 2;

        if (target < arr[mid]) {
            end = mid - 1;              // discard mid and everything right
        } else if (target > arr[mid]) {
            start = mid + 1;            // discard mid and everything left
        } else {
            return mid;                 // found
        }
    }
    return -1;                          // start crossed end — not present
}
```
```java
int[] arr = {-18, -12, -4, 0, 2, 3, 4, 15, 16, 18, 22, 45, 89};
System.out.println(binarySearch(arr, 22));
```
**Output**
```
10
```

Four details that are the whole algorithm:

| Detail | Why |
|---|---|
| `while (start <= end)` | `<=`, not `<`. With one element left, `start == end` and that element still needs checking. |
| `mid - 1` / `mid + 1` | `mid` has *already* been compared. Leaving it in the range is the classic infinite loop. |
| `end = arr.length - 1` | inclusive upper bound, matching `<=`. |
| `return -1` | the loop exits when `start > end` — the space is empty. |

## 2. The Overflow Bug — `start + (end - start) / 2`

The lecture leaves the naive version commented out, and it is worth understanding:

```java
int mid = (start + end) / 2;            // WRONG for large arrays
int mid = start + (end - start) / 2;    // correct
```

If `start` and `end` are both near `Integer.MAX_VALUE` (2,147,483,647), then `start + end` **overflows to a negative number** — and `arr[negative]` throws. The fix is algebraically identical (`start + (end-start)/2 == (start+end)/2`) but never forms the large intermediate sum.

This is a famous bug: it sat in the JDK's own `Arrays.binarySearch` for **nine years** before being found in 2006. It only triggers on arrays of over a billion elements, which is exactly why it survived so long — and why interviewers like it. Write the safe form always; it costs nothing.

## 3. Why It's O(log n)

Each pass halves the space: n → n/2 → n/4 → … → 1. The number of halvings to reach 1 is **log₂ n**.

| n | Linear search (worst) | Binary search (worst) |
|---|---|---|
| 10 | 10 | 4 |
| 1,000 | 1,000 | 10 |
| 1,000,000 | 1,000,000 | **20** |
| 1,000,000,000 | 1 billion | **30** |

Doubling the data adds **one** comparison. That is what logarithmic means, and it is why sorting once (O(n log n)) to enable binary search pays off the moment you have more than a handful of queries.

## 4. Order-Agnostic Binary Search

What if you don't know whether the array is ascending or descending? Detect it first, then flip the comparisons.

```java
static int orderAgnosticBS(int[] arr, int target) {
    int start = 0;
    int end = arr.length - 1;

    boolean isAsc = arr[start] < arr[end];      // detect direction once

    while (start <= end) {
        int mid = start + (end - start) / 2;

        if (arr[mid] == target) return mid;

        if (isAsc) {
            if (target < arr[mid]) end = mid - 1; else start = mid + 1;
        } else {
            if (target > arr[mid]) end = mid - 1; else start = mid + 1;
        }
    }
    return -1;
}
```
```java
int[] arr = {99, 80, 75, 22, 11, 10, 5, 2, -3};   // DESCENDING
System.out.println(orderAgnosticBS(arr, 22));
```
**Output**
```
3
```

Note the equality check moved **above** the direction branch — otherwise you'd duplicate it in both arms.

> ⚠️ `arr[start] < arr[end]` misreads a **single-element** array (both are the same element, so `isAsc` is `false`) — harmless, because the first `mid` check finds or rejects it immediately. With all-equal elements it is also `false`, and still correct for the same reason.

## 5. Ceiling and Floor — When the Target Isn't There

This is where binary search gets genuinely useful, and where most people stumble.

**Ceiling** — the smallest element **≥** target:

```java
static int ceiling(int[] arr, int target) {
    if (target > arr[arr.length - 1]) {
        return -1;                    // nothing is big enough
    }
    int start = 0, end = arr.length - 1;
    while (start <= end) {
        int mid = start + (end - start) / 2;
        if (target < arr[mid])      end = mid - 1;
        else if (target > arr[mid]) start = mid + 1;
        else return mid;
    }
    return start;                     // <-- the insight
}
```
```java
int[] arr = {2, 3, 5, 9, 14, 16, 18};
System.out.println(ceiling(arr, 15));
```
**Output**
```
5
```
Index 5 holds `16` — the smallest value ≥ 15. Correct.

**Floor** — the greatest element **≤** target: identical loop, but `return end`.

```java
int[] arr = {2, 3, 5, 9, 14, 16, 18};
System.out.println(floor(arr, 1));
```
**Output**
```
-1
```
Nothing is ≤ 1, and `end` has walked off the left edge to `-1` — which doubles as the "not found" answer.

> 💡 **The key insight, worth memorising:** when the loop ends without a match, `start` and `end` have **crossed**, and they straddle the gap where the target would go:
> ```
> ...  arr[end]  <  target  <  arr[start]  ...
>        floor                   ceiling
> ```
> So `start` is the **ceiling** index and `end` is the **floor** index, for free. You don't need extra tracking — you just return a different pointer.

**Smallest letter greater than target** is the same idea with a wrap-around:

```java
public char nextGreatestLetter(char[] letters, char target) {
    int start = 0, end = letters.length - 1;
    while (start <= end) {
        int mid = start + (end - start) / 2;
        if (target < letters[mid]) end = mid - 1; else start = mid + 1;
    }
    return letters[start % letters.length];    // wrap to 0 if past the end
}
```
There is no equality branch at all — even on a match it moves right, because it wants *strictly greater*. `start % length` wraps back to the first letter when the target is ≥ everything.

## 6. First and Last Position of a Target

With duplicates, plain binary search returns *some* index of the target, not the first or the last. The fix: **don't stop when you find it** — record it and keep searching the side you care about.

```java
int search(int[] nums, int target, boolean findStartIndex) {
    int ans = -1;
    int start = 0, end = nums.length - 1;
    while (start <= end) {
        int mid = start + (end - start) / 2;
        if (target < nums[mid])      end = mid - 1;
        else if (target > nums[mid]) start = mid + 1;
        else {
            ans = mid;                      // potential answer — remember it
            if (findStartIndex) end = mid - 1;   // keep looking LEFT
            else                start = mid + 1; // keep looking RIGHT
        }
    }
    return ans;
}
```
Two O(log n) passes → **O(log n)** overall. The pattern — *record a candidate, then keep shrinking* — reappears constantly in binary-search problems.

## 7. Peak of a Mountain Array

A "mountain" rises then falls. Find the peak in O(log n):

```java
public int peakIndexInMountainArray(int[] arr) {
    int start = 0, end = arr.length - 1;
    while (start < end) {                 // note: < not <=
        int mid = start + (end - start) / 2;
        if (arr[mid] > arr[mid + 1]) {
            end = mid;                    // descending side — mid might BE the peak
        } else {
            start = mid + 1;              // ascending side — mid is definitely not
        }
    }
    return start;                         // start == end == peak
}
```

Two deliberate differences from the template:

- **`while (start < end)`** — the loop ends when exactly one element remains, and that element *is* the answer. With `<=` it would never terminate.
- **`end = mid`, not `mid - 1`** — on the descending side `mid` could itself be the peak, so it must stay in the range.

The invariant: both pointers always keep the best candidate inside their range, so when the range narrows to one element, that element is the peak. **Searching in a mountain array** then chains this with two order-agnostic searches: one on the ascending half, one on the descending half.

## 8. Rotated Sorted Array

A sorted array rotated at some point (`{4,5,6,7,0,1,2}`) is two sorted runs. Find the **pivot** (the largest element, where the drop happens), then binary search the correct half:

```java
static int findPivot(int[] arr) {
    int start = 0, end = arr.length - 1;
    while (start <= end) {
        int mid = start + (end - start) / 2;
        if (mid < end && arr[mid] > arr[mid + 1])  return mid;       // drop on the right
        if (mid > start && arr[mid] < arr[mid - 1]) return mid - 1;  // drop on the left
        if (arr[mid] <= arr[start]) end = mid - 1; else start = mid + 1;
    }
    return -1;                     // no pivot — the array isn't rotated
}
```
```java
int[] arr = {4, 5, 6, 7, 0, 1, 2};
System.out.println(countRotations(arr));     // pivot + 1
```
**Output**
```
4
```
The pivot is index 3 (value 7), so the array was rotated **4** times. Note the guards `mid < end` and `mid > start` — without them, `arr[mid + 1]` / `arr[mid - 1]` would run off the array.

> ⚠️ **Duplicates break this.** If `arr[start] == arr[mid] == arr[end]`, you cannot tell which side is sorted. The lecture's `findPivotWithDuplicates` handles it by checking whether `start` or `end` is itself the pivot and then skipping one from each side — which degrades to **O(n)** in the worst case (an array of all-equal elements). Binary search fundamentally cannot stay logarithmic when it can't distinguish the halves.

## 9. Binary Search on the *Answer*, Not the Array

The most powerful variant. When the answer is a **number in a range** and you can *test* a candidate quickly, binary search the range of possible answers.

**Split Array Largest Sum** — split `nums` into `m` subarrays, minimising the largest subarray sum:

```java
public int splitArray(int[] nums, int m) {
    int start = 0, end = 0;
    for (int num : nums) {
        start = Math.max(start, num);   // lower bound: the largest single element
        end += num;                     // upper bound: everything in one piece
    }

    while (start < end) {
        int mid = start + (end - start) / 2;

        int sum = 0, pieces = 1;
        for (int num : nums) {           // can we do it with max-sum = mid?
            if (sum + num > mid) { sum = num; pieces++; }
            else                 { sum += num; }
        }

        if (pieces > m) start = mid + 1;  // mid too small — needs too many pieces
        else            end = mid;        // mid works; try smaller
    }
    return end;
}
```

The array is never binary-searched. **The answer space is.** The bounds come from reasoning (the answer can't be less than the biggest element, or more than the total), and each candidate is validated by a linear check. **O(n log(sum))**.

Recognise the shape: *"minimise the maximum"* or *"maximise the minimum"* almost always means binary search on the answer.

## 10. Searching a 2-D Matrix

**Fully sorted matrix** (each row sorted, and each row starts after the previous ends) — treat it as one long sorted array, or binary search rows then columns. **O(log(m×n))**.

**Row- and column-wise sorted** (rows sorted left→right, columns sorted top→bottom, but rows don't chain) — use the **staircase** walk from the top-right corner:

```java
static int[] search(int[][] matrix, int target) {
    int r = 0, c = matrix[0].length - 1;       // start top-RIGHT
    while (r < matrix.length && c >= 0) {
        if (matrix[r][c] == target) return new int[]{r, c};
        if (matrix[r][c] < target) r++;        // too small — go DOWN
        else                       c--;        // too big  — go LEFT
    }
    return new int[]{-1, -1};
}
```
```java
int[][] arr = {{10,20,30,40},{15,25,35,45},{28,29,37,49},{33,34,38,50}};
System.out.println(Arrays.toString(search(arr, 49)));
```
**Output**
```
[2, 3]
```

The top-right corner is the only one where the two directions give **opposite** information: everything left is smaller, everything below is larger. So each step eliminates a whole row or column — **O(m + n)**, not O(log) but far better than O(m×n). (Top-left would be useless: both directions increase.)

## 11. Infinite / Unbounded Array

When you can't call `.length`, find a bounded window first by **doubling**:

```java
int start = 0, end = 1;
while (target > arr[end]) {
    int temp = end + 1;
    end = end + (end - start + 1) * 2;   // double the window size
    start = temp;
}
return binarySearch(arr, target, start, end);
```
```java
int[] arr = {3, 5, 7, 9, 10, 90, 100, 130, 140, 160, 170};
System.out.println(ans(arr, 10));
```
**Output**
```
4
```
Doubling reaches the target's neighbourhood in O(log n) steps, then the search itself is O(log n) — **O(log n)** overall. Same doubling trick that makes `ArrayList` growth amortised O(1).

---

## ⚠️ Common Misunderstandings
**1. Binary search works on any array.**
❌ any array · ✅ **sorted only**. On unsorted data it silently returns wrong answers — no error, just nonsense.

**2. `mid = (start + end) / 2`.**
❌ fine · ✅ overflows for very large indexes. Use `start + (end - start) / 2`. (Nine years undetected in the JDK.)

**3. `while (start < end)` in the standard template.**
❌ equivalent · ✅ misses the last element when `start == end`. Use `<=` — *except* in the peak/answer-space variants, where `<` is deliberate and `end = mid` keeps the candidate.

**4. `end = mid` in the standard template.**
❌ safe · ✅ infinite loop — `mid` was already compared and must leave the range: `end = mid - 1`.

**5. Returning `-1` from ceiling/floor searches.**
❌ nothing to return · ✅ when the loop ends, `start` is the **ceiling** index and `end` is the **floor** index, for free.

**6. Plain binary search finds the first occurrence.**
❌ the first · ✅ *some* occurrence. Record the candidate and keep shrinking toward the side you want.

**7. Rotated-array pivot search handles duplicates.**
❌ handles them · ✅ with `arr[start] == arr[mid] == arr[end]` you can't tell which half is sorted; the fix degrades to O(n).

**8. Bounds guards are optional.**
❌ optional · ✅ `arr[mid + 1]` / `arr[mid - 1]` need `mid < end` / `mid > start`, or they read out of bounds.

## JavaScript Comparison
| | Java | JavaScript |
|---|---|---|
| Built-in | `Arrays.binarySearch(arr, key)` | none — write it yourself |
| Return when absent | `-(insertion point) - 1` | n/a |
| Integer overflow in `mid` | real (`int` is 32-bit) | not an issue below 2⁵³, but `(s+e)/2` needs `Math.floor` |
| Integer division | `/` truncates automatically | `Math.floor((e - s) / 2)` required |

> 💡 `Arrays.binarySearch` returns `-(insertionPoint) - 1` when absent — an encoded *ceiling*. That negative value is exactly the §5 insight, packaged: `-result - 1` gives the index where the target would be inserted.

## Interview Angles
- **"Implement binary search."** — Write the template from memory. Say `start + (end - start) / 2` and explain why unprompted; it reads as experience.
- **"Why O(log n)?"** — Each comparison halves the space; log₂(1,000,000) ≈ 20.
- **"Find the first/last occurrence."** — Record the candidate, keep shrinking. Two passes, still O(log n).
- **"Search a rotated sorted array."** — Find the pivot, then search the correct half. Follow-up: *"with duplicates?"* → worst case O(n), and say why.
- **"Find the peak."** — `while (start < end)` with `end = mid`. Explain the invariant: the best candidate never leaves the range.
- **"Minimise the largest…"** — Binary search on the **answer space**, with a linear feasibility check. The highest-value pattern in this note.
- **"Search a row/column-sorted matrix."** — Staircase from the top-right, O(m + n). Explain why that corner and no other.

## Related · Next
- **Related:** [[Linear Search]] (08 — the O(n) baseline) · [[Arrays]] (07) · [[Big-O Intuition]] · [[Binary Search]] (2.6)
- **Practice:** implement `ceiling` and `floor` from scratch, then Search Insert Position (LeetCode 35 — literally the ceiling). Then Find Peak Element (162) and Search in Rotated Sorted Array (33).
- **Next:** [10 — Sorting](10-Sorting_Notes.md)

---

## 🔁 Rapid Revision (self-test)
<details><summary>1. What does binary search require, and what does it cost?</summary>

A **sorted** array. O(log n) time, O(1) space iteratively. On unsorted data it returns wrong answers silently.
</details>

<details><summary>2. Why `start + (end - start) / 2`?</summary>

`(start + end)` can overflow `int` for very large indexes, producing a negative index. The rearranged form is algebraically identical but never forms the big sum. This bug lived in the JDK for nine years.
</details>

<details><summary>3. Why `<=` in the loop, and why `mid - 1` / `mid + 1`?</summary>

`<=` because when `start == end` there is still one unchecked element. `mid ± 1` because `mid` has already been compared — leaving it in the range causes an infinite loop.
</details>

<details><summary>4. After a failed search, what are `start` and `end`?</summary>

They've crossed and straddle the gap: `start` is the **ceiling** index (smallest ≥ target), `end` is the **floor** index (largest ≤ target). Free, with no extra tracking.
</details>

<details><summary>5. How do you find the FIRST occurrence among duplicates?</summary>

On a match, store `ans = mid` and keep searching left (`end = mid - 1`) instead of returning. Mirror it for the last occurrence.
</details>

<details><summary>6. In the peak-finding loop, why `start < end` and `end = mid`?</summary>

The loop should stop with exactly one element left, which is the peak. On the descending side `mid` may itself be the peak, so it must stay in the range — `end = mid - 1` could discard the answer.
</details>

<details><summary>7. How do you search a rotated sorted array, and what breaks it?</summary>

Find the pivot (where the drop happens), then binary search the appropriate sorted half. Duplicates break it: when `arr[start] == arr[mid] == arr[end]` you can't tell which half is sorted, and the workaround is O(n).
</details>

<details><summary>8. What is "binary search on the answer"?</summary>

When the answer is a number in a known range and a candidate can be *tested* quickly, binary search the range instead of the array. Signature phrasing: "minimise the maximum" / "maximise the minimum".
</details>

<details><summary>9. How do you search a row- and column-sorted matrix, and why the top-right?</summary>

Staircase: start top-right; if too small go down, too big go left. O(m + n). Top-right is the only corner where the two directions give opposite information — from top-left both increase, so nothing can be eliminated.
</details>

<details><summary>10. How do you binary search an array with no known length?</summary>

Double the window (`end = end + (end - start + 1) * 2`) until the target is inside it, then binary search that window. O(log n) overall.
</details>
