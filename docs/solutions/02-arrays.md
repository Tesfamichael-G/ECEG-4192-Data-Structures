# Solutions to Chapter 2 — Arrays

[:octicons-arrow-left-24: Back to Chapter 2](../chapters/02-arrays.md)

---

## 1. Prove `a[i]` is O(1)

Address = base + i*w. Computation: 1 multiply, 1 add, 1 load = 3 instructions. Independent of i, n. **O(1)**. ✓

---

## 2. Insertion cost table for `n = 100`

| Position | Shifts |
|:---:|:---:|
| 0 | 100 |
| 25 | 75 |
| 50 | 50 |
| 75 | 25 |
| 100 | 0 |

**Cheapest:** insert at the end (0 shifts) — O(1).

---

## 3. Amortised doubling, `n = 32`

Resizes at sizes 1, 2, 4, 8, 16 (copies: 1, 2, 4, 8, 16). Total copies = **31**. Amortised per insert = 31/32 < 1. ✓

---

## 4. +1 growth, `n = 32`

Resizes at every size 1..31, copying 1, 2, ..., 31. Total = $31 \cdot 32 / 2 = 496$. Ratio to doubling: 496/31 = **16×**. **O(n²).** ✓

---

## 5. Binary search on 1024

$n = 1024$, iterations = $\lceil \log_2 1024 \rceil = 10$.

- First element: 10 iterations (or fewer — depends on midpoint landing).
- Last element: ~10.
- Missing element: 10.

---

## 6. `reverse(a, i, j)`

```java
void reverse(int[] a, int i, int j) {
    while (i < j) { int t=a[i]; a[i]=a[j]; a[j]=t; i++; j--; }
}
```

**Cost:** O(j − i), in-place.

---

## 7. Rotate `[1,2,3,4,5]` by `k = 2`

Three reversals:
1. Reverse all → `[5, 4, 3, 2, 1]`
2. Reverse first `k=2` → `[4, 5, 3, 2, 1]`
3. Reverse rest → `[4, 5, 1, 2, 3]`

**Expected:** rotate right by 2 → `[4, 5, 1, 2, 3]` ✓

---

## 8. Two-pointer sum in `[1, 3, 5, 7, 9, 11]`, target 15

`left=0, right=5`: sum = 1+11=12 <15 → left=1.
`left=1, right=5`: sum = 3+11=14 <15 → left=2.
`left=2, right=5`: sum = 5+11=16 >15 → right=4.
`left=2, right=4`: sum = 5+9=14 <15 → left=3.
`left=3, right=4`: sum = 7+9=16 >15 → right=3. Exit.

**No pair sums to 15** in this array. (Closest: 5+9=14, 7+9=16.)

---

## 9. Row-major vs column-major on `int[1000][1000]`

Expected wall-clock: row-major ~5 ms, column-major ~25 ms. **Ratio ~5×** due to cache misses in column-major traversal.

Same Big-O: **O(n²)** for both.

---

## 10. Shrink thrashing

With shrink at `size < capacity / 2`: after add-then-remove alternating near the boundary, each remove triggers a resize to capacity/2, then the next add triggers a resize to capacity (since size > capacity/2 after the add). **Each pair of operations does O(n) work.** Amortised: **O(n)** per operation.

With shrink at `size < capacity / 4`: after shrinking to capacity/2, the array needs to grow again only after capacity/4 more adds. **Buffer prevents thrashing.** Amortised: **O(1)**.

---

## 11. Array vs. linked-list, `n = 100,000` random lookups

- Array (binary search): 17 lookups × 17 iterations = ~289 comparisons.
- Linked list (linear): 17 × 50,000 (average) = 850,000 comparisons.

**Array wins by ~2900×.** Plus better cache locality → actual wall-clock gap is even bigger (~10,000×).

---

## 12. Two-pointer on unsorted, with sort

Sort O(n log n) + two-pointer O(n) = **O(n log n) total**. Better than brute force O(n²).

---

## 13. Sliding window max average

Maintain a running sum. Add `a[i]`, subtract `a[i-k]`, update max. **O(n)** time, **O(1)** extra space. Naive recomputes the sum every window → O(n·k).

---

## 14. Duplicate detection

- **O(n²):** nested loops comparing every pair.
- **O(n log n):** sort, then check adjacent duplicates.
- **O(n):** hash set — insert each; check if present first. **O(n) space.**

---

## 15. Missing number

Sum formula: expected sum = $n(n+1)/2$. Actual sum = array sum. **Missing = expected − actual.**

Or XOR: XOR all `1..n` and all array elements → result is the missing number. **O(n) time, O(1) space.** ✓