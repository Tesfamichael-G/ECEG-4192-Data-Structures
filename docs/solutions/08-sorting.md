# Solutions to Chapter 8 — Sorting

[:octicons-arrow-left-24: Back to Chapter 8](../chapters/08-sorting.md)

---

## 1. Bubble sort — `[5, 3, 1, 4, 2]`

**Pass 1:**
- (5,3) swap → `[3,5,1,4,2]`
- (5,1) swap → `[3,1,5,4,2]`
- (5,4) swap → `[3,1,4,5,2]`
- (5,2) swap → `[3,1,4,2,5]` (5 at end)

**Pass 2:**
- (3,1) swap → `[1,3,4,2,5]`
- (3,4) keep
- (4,2) swap → `[1,3,2,4,5]`

**Pass 3:**
- (1,3) keep
- (3,2) swap → `[1,2,3,4,5]`

**Pass 4:** no swaps → done.

**Total passes: 4. Total comparisons: 4+3+2+1 = 10. Swaps: 6.**

---

## 2. Insertion sort — `[5, 3, 1, 4, 2]`

| i | key | shifts | Array |
|:---:|:---:|:---:|---|
| 1 | 3 | 1 (5) | [3, 5, 1, 4, 2] |
| 2 | 1 | 2 (5, 3) | [1, 3, 5, 4, 2] |
| 3 | 4 | 1 (5) | [1, 3, 4, 5, 2] |
| 4 | 2 | 3 (5, 4, 3) | [1, 2, 3, 4, 5] |

**Total comparisons: 7. Total shifts: 7.**

---

## 3. Selection sort — `[5, 3, 1, 4, 2]`

| i | Min in suffix | Swap |
|:---:|:---:|---|
| 0 | 1 (idx 2) | swap 5, 1 → [1, 3, 5, 4, 2] |
| 1 | 2 (idx 4) | swap 3, 2 → [1, 2, 5, 4, 3] |
| 2 | 3 (idx 4) | swap 5, 3 → [1, 2, 3, 4, 5] |
| 3 | 4 (idx 3) | no swap |

**Total swaps: 3. Comparisons: 4+3+2+1 = 10.**

---

## 4. Merge sort — `[5, 3, 1, 4, 2]`

Divide: `[5,3,1]` | `[4,2]`.

Recursively: `[5]` + `[3,1]` → `[5]` + `[1,3]` → `[1,3,5]`.

`[4]` + `[2]` → `[2,4]`.

Merge `[1,3,5]` + `[2,4]`:
- 1 vs 2 → 1
- 3 vs 2 → 2
- 3 vs 4 → 3
- 5 vs 4 → 4
- 5 → 5

**Result:** `[1, 2, 3, 4, 5]` ✓

**Comparisons:** level 1 (~2), level 2 (~2), merge (~4). **Total ≈ 8.** O(n log n).

---

## 5. Quicksort — `[5, 3, 1, 4, 2]`, last-element pivot

Pivot = 2 (index 4).

Partition: `i=-1`. Scan j=0..3:
- j=0, a[0]=5 > 2, no swap.
- j=1, a[1]=3 > 2, no swap.
- j=2, a[2]=1 < 2, i=0, swap a[0]↔a[2] → `[1, 3, 5, 4, 2]`.
- j=3, a[3]=4 > 2, no swap.

Final: swap a[1]↔a[4] → `[1, 2, 5, 4, 3]`. Pivot at index 1.

Left `[1]` — sorted.

Right `[5, 4, 3]` (indices 2-4), pivot=3 (index 4):
- Partition: j=2, a=5 > 3 no; j=3, a=4 > 3 no. Final swap a[2]↔a[4] → `[1, 2, 3, 4, 5]`. Pivot at index 2.

Left `[3]` sorted. Right `[4, 5]`, pivot=5:
- Partition: j=3, a=4 < 5 → i=3, no swap (already at i). Final swap a[4]↔a[4] → no change. Pivot at index 4.

Left `[4]` sorted. Done.

**Result:** `[1, 2, 3, 4, 5]`. **Total comparisons ≈ 8.**

---

## 6. Quicksort worst case — `[1, 2, 3, 4, 5, 6]`

Pivot = last element each time.

- Partition 1 (pivot=6): 5 comparisons, splits off 6.
- Partition 2 (pivot=5): 4 comparisons, splits off 5.
- Partition 3 (pivot=4): 3 comparisons.
- Partition 4 (pivot=3): 2 comparisons.
- Partition 5 (pivot=2): 1 comparison.

**Total comparisons: 15** = n(n−1)/2 = **O(n²)**.

Compare with already-sorted vs random — for random, expected is O(n log n) ≈ 12–15 comparisons. **Sorted input hits the worst case.** Fix: random pivot or median-of-three.

---

## 7. Heapsort — `[5, 3, 1, 4, 2]`

**Build max-heap:**

- i=1 (value 3): children indices 3, 4 (4, 2). Max child 4. 3 < 4 → swap. `[5, 4, 1, 3, 2]`.
- i=0 (value 5): children 4, 1. Max child 4. 5 > 4 → no swap.

**Max-heap:** `[5, 4, 1, 3, 2]`.

**Extract-max:**

| Step | Array before | After swap & sift |
|:---:|---|---|
| 1 | [5, 4, 1, 3, 2] | [4, 3, 1, 2, 5] |
| 2 | [4, 3, 1, 2, 5] | [3, 2, 1, 4, 5] |
| 3 | [3, 2, 1, 4, 5] | [2, 1, 3, 4, 5] |
| 4 | [2, 1, 3, 4, 5] | [1, 2, 3, 4, 5] |

**Sorted:** `[1, 2, 3, 4, 5]` ✓

---

## 8. Stability test `[(3,A), (1,B), (2,C), (3,D), (1,E)]`

**Merge sort (stable):** → `[(1,B), (1,E), (2,C), (3,A), (3,D)]`. **B before E, A before D.** ✓

**Quicksort (unstable):** could produce `[(1,E), (1,B), (2,C), (3,D), (3,A)]` — **E before B, D before A.** ✗

**Merge sort preserves relative order of equal keys.**

---

## 9. Nearly-sorted (90% sorted) analysis

**Insertion sort** — its O(n) best case extends to nearly-sorted: if only k elements are out of place, insertion sort does at most O(n + k·inversions) work. For 90% sorted with ~10% out of place, inversions ≈ 0.1n × avg distance ≈ O(n).

**Comparisons:** ~1.1n. **Best choice.**

---

## 10. Comparison counts for n = 1000

| Sort | Best | Average | Worst |
|---|:---:|:---:|:---:|
| Bubble | 999 | ~500k | ~500k |
| Insertion | 999 | ~250k | ~500k |
| Selection | ~500k | ~500k | ~500k |
| Merge | ~10k | ~10k | ~10k |
| Quicksort | ~10k | ~10k | ~500k |
| Heapsort | ~10k | ~10k | ~10k |

---

## 11. External sort — 10 GB on 1 GB RAM

**External merge sort:**
1. Read 1 GB chunks; sort each in memory; write back to disk as a **run**. → 10 sorted runs of 1 GB each.
2. **K-way merge:** open all 10 runs, keep one block from each in memory, merge using a heap. Write output sequentially.

**Total disk I/O:** 2 passes (read all, write all) + merge passes. **O(n log(n/m))** where m = number of runs.

**Feasible on any disk** — the RAM constraint is respected throughout.

---

## 12. Counting sort for 10M integers in [0, 1000]

**Counting sort:**
1. Create a count array of size 1001. O(1001) space.
2. Count occurrences of each value. O(n).
3. Reconstruct sorted array. O(n + 1001).

**Total O(n + k)** = O(10M + 1001) ≈ **10M operations** — **20× faster than O(n log n)** ≈ 230M comparisons.

**Requires:** keys in a known bounded range (0..1000). ✓