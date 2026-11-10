---
revealjs:
  mode: diapo
  highlight:
    light: vs
    dark:  vs2015
  autoAnimate:  true
  autoAnimateDuration: 1.0
  transition: slide
  transitionSpeed: default
  js: assets/lecture-08.js
---

# ECEG-4192

## Chapter 8 — Sorting Algorithms

Comparison sorts · Divide & conquer · Complexity

Semester VII · 2025/26

<aside class="notes">
Sorting ties together everything from Chapter 1: Big-O analysis, recursion, heaps, and divide-and-conquer.
</aside>

---

## Learning outcomes

By the end of this chapter you should be able to:

1. Explain why sorting matters.

2. Implement **bubble, selection, insertion, merge, quick, heap** sort.

3. Analyse **best, average, worst** case complexity.

4. Distinguish **stable** vs **unstable**, **in-place** vs **out-of-place**.

5. Prove a lower bound of **$\Omega(n\log n)$** for comparison sorts.

6. Choose the right sort for a given input.

---

## 8.1 — Why sorting?

<div class="fragment">

- Enables **binary search** — O(log n) instead of O(n).
- Enables **deduplication** — adjacent duplicates.
- Enables **range queries** — sorted order.
- Foundation for **Kruskal's MST**, **median finding**, **grouping**.

</div>

<div class="fragment">

**Measures of a sort:**

- **Time** — best, average, worst.
- **Space** — auxiliary memory.
- **Stability** — equal keys preserve relative order.
- **In-place** — uses O(1) auxiliary space.

</div>

---

## 8.2 — Bubble sort

```python
def bubble_sort(a):
    n = len(a)
    for i in range(n):
        for j in range(n - i - 1):
            if a[j] > a[j+1]:
                a[j], a[j+1] = a[j+1], a[j]
```

<div class="fragment">

**Idea:** repeatedly swap adjacent out-of-order pairs. Largest bubble to the end each pass.

| Property | Value |
|---|---|
| Time (best) | O(n) — with early-exit flag |
| Time (avg/worst) | **O(n²)** |
| Space | **O(1)** |
| Stable? | ✅ |
| In-place? | ✅ |

</div>

<div class="fragment">

**Rarely used in practice** — educational only.

</div>

---

## 8.3 — Selection sort

```python
def selection_sort(a):
    n = len(a)
    for i in range(n):
        m = i
        for j in range(i+1, n):
            if a[j] < a[m]:
                m = j
        a[i], a[m] = a[m], a[i]
```

<div class="fragment">

**Idea:** find minimum of remaining, swap into place.

| Property | Value |
|---|---|
| Time | **O(n²)** always |
| Space | **O(1)** |
| Stable? | ❌ |
| In-place? | ✅ |

</div>

<div class="fragment">

**Minimises swaps** — useful when writes are expensive (e.g., flash memory).

</div>

---

## 8.4 — Insertion sort

```python
def insertion_sort(a):
    for i in range(1, len(a)):
        key = a[i]
        j = i - 1
        while j >= 0 and a[j] > key:
            a[j+1] = a[j]
            j -= 1
        a[j+1] = key
```

<div class="fragment">

**Idea:** build sorted prefix one element at a time.

| Property | Value |
|---|---|
| Time (best) | **O(n)** — already sorted |
| Time (avg/worst) | **O(n²)** |
| Space | **O(1)** |
| Stable? | ✅ |
| In-place? | ✅ |

</div>

<div class="fragment">

**Best for small / nearly-sorted inputs.** Python's Timsort uses insertion sort for blocks of $\le 64$ elements.

</div>

---

## Insertion sort trace

Sort `[5, 2, 9, 1, 7]`:

| Pass | Key | Array after |
|:---:|:---:|---|
| — | — | 5 2 9 1 7 |
| 1 | 2 | **2 5** 9 1 7 |
| 2 | 9 | **2 5 9** 1 7 |
| 3 | 1 | **1 2 5 9** 7 |
| 4 | 7 | **1 2 5 7 9** |

<div class="fragment">

**Each pass:** shift larger elements right, insert key.

Total shifts in worst case: $1 + 2 + \cdots + (n-1) = n(n-1)/2 = O(n^2)$.

</div>

---

## 8.5 — Merge sort

```python
def merge_sort(a):
    if len(a) <= 1:
        return a
    mid = len(a) // 2
    L = merge_sort(a[:mid])
    R = merge_sort(a[mid:])
    return merge(L, R)

def merge(L, R):
    out, i, j = [], 0, 0
    while i < len(L) and j < len(R):
        if L[i] <= R[j]:
            out.append(L[i]); i += 1
        else:
            out.append(R[j]); j += 1
    return out + L[i:] + R[j:]
```

<div class="fragment">

**Idea:** split, sort recursively, merge two sorted halves.

| Property | Value |
|---|---|
| Time | **O(n log n)** — all cases |
| Space | **O(n)** auxiliary |
| Stable? | ✅ |
| In-place? | ❌ |

</div>

---

## Merge sort — the recurrence

$$ T(n) = 2\,T(n/2) + O(n) $$

<div class="fragment">

**Recursion tree:**

- Depth $d$ has $2^d$ subproblems of size $n/2^d$.
- Work at depth $d$: $2^d \cdot O(n/2^d) = O(n)$.
- Depth: $\log_2 n$.

**Total: $O(n \log n)$.**

</div>

<div class="fragment">

**Advantages over quicksort:** guaranteed worst case; **stable**; good for **linked lists** and **external sorting**.

**Disadvantage:** $O(n)$ auxiliary space.

</div>

---

## Merge sort — merge step

Merge `[2, 5]` and `[1, 3]`:

| i, j | Compare | Output |
|:---:|---|---|
| 0, 0 | 2 vs 1 | 1 |
| 0, 1 | 2 vs 3 | 2 |
| 1, 1 | 5 vs 3 | 3 |
| 1, 2 | (L exhausted) | append 5 |

<div class="fragment">

Result: `[1, 2, 3, 5]`.

**O(m + n)** to merge two lists of sizes $m, n$.

</div>

---

## 8.6 — Quick sort

```python
def quick_sort(a, lo=0, hi=None):
    if hi is None: hi = len(a) - 1
    if lo < hi:
        p = partition(a, lo, hi)
        quick_sort(a, lo, p-1)
        quick_sort(a, p+1, hi)

def partition(a, lo, hi):
    pivot = a[hi]
    i = lo - 1
    for j in range(lo, hi):
        if a[j] <= pivot:
            i += 1
            a[i], a[j] = a[j], a[i]
    a[i+1], a[hi] = a[hi], a[i+1]
    return i + 1
```

<div class="fragment">

**Idea:** pick pivot, partition into $\le$ pivot and $>$ pivot, recurse.

| Property | Value |
|---|---|
| Time (best/avg) | **O(n log n)** |
| Time (worst) | **O(n²)** — bad pivots |
| Space (aux) | **O(log n)** — recursion |
| Stable? | ❌ |
| In-place? | ✅ |

</div>

---

## Quick sort — worst case

**Worst case:** pivot always minimum or maximum → $T(n) = T(n-1) + O(n) = O(n^2)$.

<div class="fragment">

**Happens when:** input is already sorted (with last-element pivot) — common in practice!

</div>

<div class="fragment">

**Fix:** **randomised quicksort** — pick pivot uniformly at random.

Then expected time $O(n\log n)$, with **high probability**.

**Fix 2:** **median-of-three** — pick median of first, middle, last. Harder for adversarial inputs.

</div>

---

## Partition trace

Sort `[3, 6, 1, 8, 2]`, pivot = last element (2):

lo = 0, pivot = 2, i = -1.

| j | a[j] | a[j] ≤ 2? | Action |
|:---:|:---:|:---:|---|
| 0 | 3 | ❌ | — |
| 1 | 6 | ❌ | — |
| 2 | 1 | ✅ | i=0, swap a[0]↔a[2] → `[1, 6, 3, 8, 2]` |
| 3 | 8 | ❌ | — |

<div class="fragment">

Final: swap a[i+1] = a[1] ↔ pivot a[4] → `[1, 2, 3, 8, 6]`.

Pivot at index 1. Recurse on `[1]` and `[3, 8, 6]`.

</div>

---

## 8.7 — Heap sort

```python
def heap_sort(a):
    n = len(a)
    for i in range(n//2 - 1, -1, -1):
        heapify(a, n, i)                # build max-heap — O(n)
    for i in range(n-1, 0, -1):
        a[0], a[i] = a[i], a[0]         # move max to end
        heapify(a, i, 0)                # restore heap

def heapify(a, n, i):
    l, r, m = 2*i+1, 2*i+2, i
    if l < n and a[l] > a[m]: m = l
    if r < n and a[r] > a[m]: m = r
    if m != i:
        a[i], a[m] = a[m], a[i]
        heapify(a, n, m)
```

<div class="fragment">

| Property | Value |
|---|---|
| Time | **O(n log n)** — all cases |
| Space | **O(1)** |
| Stable? | ❌ |
| In-place? | ✅ |

**Guaranteed worst case** — good when predictability matters.

</div>

---

## 8.8 — The comparison lower bound

???+ note "Theorem"
    Any comparison-based sorting algorithm has worst-case time $\Omega(n \log n)$.

<div class="fragment">

**Proof sketch.** A decision tree for $n$ items has $n!$ leaves (one per permutation).

A binary tree of height $h$ has $\le 2^h$ leaves. So:

$$ 2^h \ge n! \Rightarrow h \ge \log_2(n!) = \Theta(n\log n). $$

</div>

<div class="fragment">

**Consequence:** merge sort and heap sort are **asymptotically optimal** for comparison sorts.

**Beating it requires non-comparison sorts** — counting sort, radix sort, bucket sort — but those assume restricted key types.

</div>

---

## 8.9 — Comparison table

| Algorithm | Best | Average | Worst | Space | Stable |
|---|:---:|:---:|:---:|:---:|:---:|
| Bubble | O(n) | O(n²) | O(n²) | O(1) | ✅ |
| Selection | O(n²) | O(n²) | O(n²) | O(1) | ❌ |
| Insertion | **O(n)** | O(n²) | O(n²) | O(1) | ✅ |
| Merge | O(n log n) | O(n log n) | **O(n log n)** | O(n) | ✅ |
| Quick | O(n log n) | O(n log n) | **O(n²)** | O(log n) | ❌ |
| Heap | O(n log n) | O(n log n) | **O(n log n)** | **O(1)** | ❌ |

---

## Which sort should I use?

| Situation | Best sort |
|---|---|
| Small $n$ ($\le 20$) | **Insertion** |
| Nearly sorted | **Insertion** |
| Stable required | **Merge** or **Insertion** |
| Guaranteed worst case | **Merge** or **Heap** |
| Space-constrained | **Heap** (O(1)) |
| General-purpose, in practice | **Quicksort** (small constant) |
| Linked list | **Merge sort** |
| Library call | Whatever `sorted()` uses (usually Timsort) |

<div class="fragment">

**Modern languages use hybrid sorts** — Timsort (Python, Java) combines merge + insertion.

</div>

---

## Chapter summary

1. **Simple sorts (O(n²))**: bubble, selection, insertion — good for small or nearly sorted.

2. **Merge sort**: **stable**, O(n log n), O(n) space.

3. **Quick sort**: **fast average**, O(n²) worst — mitigate with random pivot.

4. **Heap sort**: **guaranteed O(n log n)**, O(1) space, not stable.

5. **Lower bound:** $\Omega(n\log n)$ for comparison sorts — proven via decision tree.

6. **Hybrids** (Timsort) dominate in real libraries.

---

## Review questions

1. Define stability and in-place sorting. Give one sort for each combination.

2. Trace insertion sort on `[5, 2, 9, 1, 7]`.

3. Why is merge sort O(n log n) in all cases?

4. When does quicksort degrade to O(n²)? How to mitigate?

5. Show how `buildHeap` achieves O(n), not O(n log n).

6. Prove that comparison sorting is $\Omega(n \log n)$.

7. Compare merge sort and quicksort for **linked lists**.

> Answers: **Solutions → Chapter 8**.

---

## References

- **Weiss**, Ch. 7 — sorting algorithms.
- **Lafore**, Ch. 6 (simple sorts), Ch. 7 (advanced sorts).
- **CLRS**, Ch. 2 (insertion), Ch. 8 (lower bound), Ch. 6 (heapsort), Ch. 7 (quicksort).
- **Sedgewick & Wayne**, Ch. 2 — sorting.