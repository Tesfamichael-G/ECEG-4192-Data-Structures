# Chapter 8 — Sorting Algorithms

!!! abstract "Chapter at a glance"
    **Contact hours:** 2 lecture + 1 tutorial + 2 lab · **Home study:** ~5 h
    
    **Coverage (syllabus):** introduction to sorting algorithms.

---

## Learning outcomes

By the end of this chapter you should be able to:

1. Explain what a **stable** sort is, and why stability matters.
2. Explain what an **in-place** sort is, and its memory implications.
3. Implement and analyse: **bubble, insertion, selection, merge, quicksort, heapsort**.
4. State the best, average, and worst-case complexities of each.
5. Trace each algorithm on a small array.
6. Choose the right sort based on data properties (size, distribution, existing order, stability requirements).
7. Explain why comparison-based sorting has an Ω(n log n) lower bound.

---

## 8.1 Key concepts

### 8.1.1 Stability

A sort is **stable** if equal elements keep their **original relative order**.

**Example.** Sort `[(3, "a"), (1, "x"), (3, "b")]` by the first field:

- Stable → `[(1, "x"), (3, "a"), (3, "b")]`
- Unstable → `[(1, "x"), (3, "b"), (3, "a")]` (possible)

**Why it matters.** When you sort by one key, then by another, stability preserves the first sort as a tiebreaker. Example: sort by name, then by department — a stable sort gives alphabetically-ordered names within each department.

### 8.1.2 In-place

A sort is **in-place** if it uses O(1) auxiliary memory (beyond a few variables). Merge sort is not (needs O(n)); heapsort and quicksort are (O(1) or O(log n) for recursion).

### 8.1.3 The comparison lower bound

Any **comparison-based** sort must do at least **Ω(n log n)** comparisons in the worst case.

**Proof (information-theoretic).** There are `n!` possible permutations of input. Each comparison is a binary decision (yes/no). A decision tree that distinguishes all `n!` permutations must have height ≥ `log₂(n!) ≈ n log n − n` (Stirling). Therefore ≥ Ω(n log n) comparisons. ∎

This means: **no comparison-based sort can beat O(n log n) in the worst case.** Faster sorts exist only with additional assumptions (counting, radix).

### 8.1.4 The summary table

| Algorithm | Best | Average | Worst | Space | Stable? |
|---|:---:|:---:|:---:|:---:|:---:|
| **Bubble** | O(n) (already sorted, with flag) | O(n²) | O(n²) | O(1) | **Yes** |
| **Insertion** | O(n) (nearly sorted) | O(n²) | O(n²) | O(1) | **Yes** |
| **Selection** | O(n²) | O(n²) | O(n²) | O(1) | No |
| **Merge** | O(n log n) | O(n log n) | O(n log n) | **O(n)** | **Yes** |
| **Quicksort** | O(n log n) | O(n log n) | **O(n²)** | O(log n) | No |
| **Heapsort** | O(n log n) | O(n log n) | O(n log n) | O(1) | No |

---

## 8.2 Bubble sort

**Idea.** Repeatedly compare adjacent pairs; if out of order, swap. After each pass, the largest remaining element "bubbles" to the end.

```java
void bubbleSort(int[] a) {
    int n = a.length;
    for (int i = 0; i < n - 1; i++) {
        boolean swapped = false;
        for (int j = 0; j < n - 1 - i; j++) {
            if (a[j] > a[j + 1]) {
                int t = a[j]; a[j] = a[j + 1]; a[j + 1] = t;
                swapped = true;
            }
        }
        if (!swapped) break;   // already sorted
    }
}
```

**Cost.** Worst: O(n²) — n passes × n comparisons. Best (with `swapped` flag): O(n) — one pass, no swaps. Average: O(n²).

**Stable:** yes. **In-place:** yes.

**Practical use:** pedagogic only — never use for real data.

### Worked example 8.1 — Bubble sort trace

Sort `[5, 2, 8, 1, 9]`.

**Pass 1:**
- Compare (5, 2) → swap → `[2, 5, 8, 1, 9]`
- Compare (5, 8) → keep → `[2, 5, 8, 1, 9]`
- Compare (8, 1) → swap → `[2, 5, 1, 8, 9]`
- Compare (8, 9) → keep → `[2, 5, 1, 8, 9]`
- **After pass 1:** `[2, 5, 1, 8, 9]` — largest (9) at end ✓

**Pass 2 (n-1-i = 3 comparisons):**
- (2, 5) → keep
- (5, 1) → swap → `[2, 1, 5, 8, 9]`
- (5, 8) → keep
- **After pass 2:** `[2, 1, 5, 8, 9]`

**Pass 3:**
- (2, 1) → swap → `[1, 2, 5, 8, 9]`
- (2, 5) → keep

**Pass 4:**
- (1, 2) → keep → `[1, 2, 5, 8, 9]` — sorted, `swapped` = false → break.

**Total:** 4 passes, 10 comparisons. **O(n²).**

---

## 8.3 Insertion sort

**Idea.** Build the sorted array one element at a time — insert each new element into its correct position in the already-sorted prefix.

```java
void insertionSort(int[] a) {
    for (int i = 1; i < a.length; i++) {
        int key = a[i];
        int j = i - 1;
        while (j >= 0 && a[j] > key) {
            a[j + 1] = a[j];
            j--;
        }
        a[j + 1] = key;
    }
}
```

**Cost.** Best (nearly sorted): O(n) — one comparison per element, no shifts. Average: O(n²) — n/2 shifts per element. Worst (reverse-sorted): O(n²).

**Stable:** yes. **In-place:** yes.

**Practical use:** excellent for small arrays and **nearly-sorted data**. Java's `Arrays.sort` for primitive ints uses dual-pivot quicksort with insertion sort for subarrays < 47 elements. Python's `Timsort` uses insertion sort on small runs.

### Worked example 8.2 — Insertion sort trace

Sort `[5, 2, 8, 1, 9]`.

| `i` | `key` | `j` loop | Array after |
|:---:|:---:|---|---|
| 1 | 2 | j=0: a[0]=5 > 2, shift 5 → j=-1 | `[2, 5, 8, 1, 9]` |
| 2 | 8 | j=1: a[1]=5 ≤ 8 → stop | `[2, 5, 8, 1, 9]` |
| 3 | 1 | j=2: 8>1 shift; j=1: 5>1 shift; j=0: 2>1 shift; j=-1 | `[1, 2, 5, 8, 9]` |
| 4 | 9 | j=3: 8≤9 → stop | `[1, 2, 5, 8, 9]` |

**Total:** 4 insertions, 4 shifts total. Nearly sorted = fast.

**Best case** — array already sorted: `[1, 2, 3, 4, 5]` → 0 shifts, 4 comparisons. **O(n).**

---

## 8.4 Selection sort

**Idea.** Repeatedly find the **minimum** element in the unsorted suffix, swap it into position.

```java
void selectionSort(int[] a) {
    int n = a.length;
    for (int i = 0; i < n - 1; i++) {
        int minIdx = i;
        for (int j = i + 1; j < n; j++)
            if (a[j] < a[minIdx]) minIdx = j;
        if (minIdx != i) {
            int t = a[i]; a[i] = a[minIdx]; a[minIdx] = t;
        }
    }
}
```

**Cost.** Always **O(n²)** — n passes, each scanning the remaining suffix. Best case is also O(n²) — no early exit.

**Stable:** **No** — the swap can move equal elements past each other. **In-place:** yes.

**Practical use:** rare — its only advantage is that it does **at most n−1 swaps** (useful when writes are expensive). Otherwise insertion sort or quicksort wins.

### Worked example 8.3 — Selection sort trace

Sort `[5, 2, 8, 1, 9]`.

| `i` | Min in `a[i..n-1]` | Swap with `a[i]` | Array after |
|:---:|:---:|---|---|
| 0 | 1 (index 3) | swap a[0] ↔ a[3] | `[1, 2, 8, 5, 9]` |
| 1 | 2 (index 1) | no swap | `[1, 2, 8, 5, 9]` |
| 2 | 5 (index 3) | swap a[2] ↔ a[3] | `[1, 2, 5, 8, 9]` |
| 3 | 8 (index 3) | no swap | `[1, 2, 5, 8, 9]` |

**Total:** 2 swaps, 10 comparisons. **O(n²)** comparisons, **O(n)** swaps.

---

## 8.5 Merge sort

**Idea.** Divide the array in half, recursively sort each half, then **merge** the two sorted halves in O(n).

```java
void mergeSort(int[] a, int lo, int hi) {
    if (lo >= hi) return;
    int mid = (lo + hi) / 2;
    mergeSort(a, lo, mid);
    mergeSort(a, mid + 1, hi);
    merge(a, lo, mid, hi);
}

void merge(int[] a, int lo, int mid, int hi) {
    int[] temp = new int[hi - lo + 1];
    int i = lo, j = mid + 1, k = 0;
    while (i <= mid && j <= hi) {
        if (a[i] <= a[j]) temp[k++] = a[i++];
        else              temp[k++] = a[j++];
    }
    while (i <= mid) temp[k++] = a[i++];
    while (j <= hi)  temp[k++] = a[j++];
    System.arraycopy(temp, 0, a, lo, temp.length);
}
```

**Cost.** Recurrence: `T(n) = 2 T(n/2) + O(n)` → **O(n log n) all cases.** No adversarial input.

**Stable:** yes (with `<=` in the merge comparison). **In-place:** **No** — needs O(n) auxiliary space.

**Practical use:** excellent general-purpose sort. Java's `Arrays.sort` for objects uses Timsort (a merge-sort variant). Python's `sorted()` uses Timsort. **Use merge sort when you need guaranteed O(n log n) and stability.**

### Worked example 8.4 — Merge sort trace

Sort `[5, 2, 8, 1, 9]`.

**Divide recursively:**
```
           [5, 2, 8, 1, 9]
              /        \
        [5, 2, 8]    [1, 9]
        /      \        |
    [5, 2]   [8]     [1] [9]  (already size 1)
    /   \
  [5]   [2]
```

**Merge back:**
- `[5]` + `[2]` → `[2, 5]`
- `[2, 5]` + `[8]` → `[2, 5, 8]`
- `[1]` + `[9]` → `[1, 9]`
- `[2, 5, 8]` + `[1, 9]` → `[1, 2, 5, 8, 9]` ✓

**Total:** 5 elements, `log₂ 5 ≈ 2.32`, ~3 merge levels. **O(n log n).**

---

## 8.6 Quicksort

**Idea.** Pick a **pivot**. Partition the array so that everything < pivot is left, everything > pivot is right. Recursively sort both halves.

```java
void quickSort(int[] a, int lo, int hi) {
    if (lo >= hi) return;
    int p = partition(a, lo, hi);
    quickSort(a, lo, p - 1);
    quickSort(a, p + 1, hi);
}

int partition(int[] a, int lo, int hi) {
    int pivot = a[hi];
    int i = lo - 1;
    for (int j = lo; j < hi; j++) {
        if (a[j] <= pivot) {
            i++;
            int t = a[i]; a[i] = a[j]; a[j] = t;
        }
    }
    int t = a[i + 1]; a[i + 1] = a[hi]; a[hi] = t;
    return i + 1;
}
```

**Cost.** Best/average: **O(n log n)** (balanced partitions). Worst: **O(n²)** (already-sorted input with last-element pivot — the classic trap).

**Stable:** **No**. **In-place:** yes (except recursion stack, O(log n) average / O(n) worst).

**Practical use:** the fastest general-purpose sort in practice — small constants. Java uses dual-pivot quicksort for primitives; C++ `std::sort` is introsort (quicksort + heapsort + insertion sort). **Always randomise or median-of-three the pivot** to avoid O(n²) on sorted input.

### Worked example 8.5 — Quicksort trace

Sort `[3, 7, 8, 5, 2, 1, 9, 4]` using last-element pivot.

**Partition (pivot = 4, index 7):**

| `j` | `a[j]` | ≤ 4? | Action | Array |
|:---:|:---:|:---:|---|---|
| 0 | 3 | ✓ | i=0, swap a[0]↔a[0] | `[3, 7, 8, 5, 2, 1, 9, 4]` |
| 1 | 7 | ✗ | — | |
| 2 | 8 | ✗ | — | |
| 3 | 5 | ✗ | — | |
| 4 | 2 | ✓ | i=1, swap a[1]↔a[4] | `[3, 2, 8, 5, 7, 1, 9, 4]` |
| 5 | 1 | ✓ | i=2, swap a[2]↔a[5] | `[3, 2, 1, 5, 7, 8, 9, 4]` |
| 6 | 9 | ✗ | — | |

Final swap: `a[i+1] = a[3] ↔ a[7]` → `[3, 2, 1, 4, 7, 8, 9, 5]`. Pivot at index 3.

**Recurse left** `[3, 2, 1]` (pivot = 1): → `[1, 2, 3]`.
**Recurse right** `[7, 8, 9, 5]` (pivot = 5): → `[5, 7, 8, 9]`.

**Final:** `[1, 2, 3, 4, 5, 7, 8, 9]` ✓

### Worked example 8.6 — Quicksort worst case

**Input:** `[1, 2, 3, 4, 5, 6, 7, 8, 9]` (already sorted).

**Pivot rule: last element.**

- Pivot = 9. Partition: everything else is smaller → pivot at end (index 8). Recurse on `[1..8]`.
- Pivot = 8. Partition: pivot at end (index 7). Recurse on `[1..7]`.
- ...

**Each partition does O(n) work but only splits off one element.** Total: n + (n-1) + (n-2) + ... = **O(n²)**.

**Fix:** random pivot, median-of-three, or shuffle before sorting. **Always do one of these** — an already-sorted input is a common real-world case (e.g., re-sorting a previously-sorted list).

---

## 8.7 Heapsort (recap from Chapter 5)

**Idea.** Build a **max-heap** from the array (O(n)); repeatedly extract the max (swap root with last, sift-down).

```java
void heapSort(int[] a) {
    int n = a.length;
    for (int i = n/2 - 1; i >= 0; i--) siftDown(a, i, n);   // build max-heap
    for (int i = n - 1; i > 0; i--) {
        int t = a[0]; a[0] = a[i]; a[i] = t;
        siftDown(a, 0, i);
    }
}

void siftDown(int[] a, int i, int n) {
    while (true) {
        int left = 2*i + 1, right = 2*i + 2, largest = i;
        if (left  < n && a[left]  > a[largest]) largest = left;
        if (right < n && a[right] > a[largest]) largest = right;
        if (largest == i) break;
        int t = a[i]; a[i] = a[largest]; a[largest] = t;
        i = largest;
    }
}
```

**Cost.** Always **O(n log n)** — no adversarial input.

**Stable:** **No**. **In-place:** yes (O(1) auxiliary).

**Practical use:** when you need guaranteed O(n log n) with O(1) space. C++'s introsort falls back to heapsort when quicksort recursion gets too deep. Also: **the basis of priority queues**.

### Worked example 8.7 — Heapsort on `[5, 2, 8, 1, 9]`

**Phase 1 — Build max-heap:**

Starting array: `[5, 2, 8, 1, 9]`.

- `i = 1` (value 2): children 1 (value 1), 9 (value 9). Sift-down: 2 < 9, swap → `[5, 9, 8, 1, 2]`.
- `i = 0` (value 5): children 9, 8. Max child 9. 5 < 9, swap → `[9, 5, 8, 1, 2]`. At index 1 (value 5): children 1, 2. Max child 2. 5 > 2, no swap.

**Max-heap built:** `[9, 5, 8, 1, 2]`.

**Phase 2 — Extract max repeatedly:**

| Step | Array (before extract) | Swap root with last | Sift-down range | Array after |
|:---:|---|---|---|---|
| 1 | `[9, 5, 8, 1, 2]` | swap 9 ↔ 2 | `[2, 5, 8, 1]` → sift | `[8, 5, 2, 1, 9]` |
| 2 | `[8, 5, 2, 1, 9]` | swap 8 ↔ 1 | `[1, 5, 2]` → sift | `[5, 1, 2, 8, 9]` |
| 3 | `[5, 1, 2, 8, 9]` | swap 5 ↔ 2 | `[2, 1]` → sift | `[2, 1, 5, 8, 9]` |
| 4 | `[2, 1, 5, 8, 9]` | swap 2 ↔ 1 | `[1]` no sift | `[1, 2, 5, 8, 9]` |

**Sorted:** `[1, 2, 5, 8, 9]` ✓

**Cost.** Build: O(n). Extract n times: O(n log n). **Total: O(n log n).**

---

## 8.8 Choosing the right sort

| Situation | Best choice | Why |
|---|---|---|
| **Small array (n < 20)** | Insertion sort | Minimal constant |
| **Nearly sorted** | Insertion sort | O(n) best case |
| **Large, generic, stability needed** | Merge sort | O(n log n), stable |
| **Large, generic, speed critical** | Quicksort (randomised pivot) | Fastest in practice |
| **Large, need O(1) space** | Heapsort | O(n log n), O(1) space |
| **Integer keys in bounded range** | Counting/radix sort | O(n) time |
| **External (data > RAM)** | External merge sort | Streamable |
| **Partially sorted segments** | Timsort (Java/Python) | Detects and merges runs |

**Java's `Arrays.sort`:**
- **Primitives:** dual-pivot quicksort (fast, non-stable).
- **Objects:** Timsort (stable, merge-based).

**Python's `sorted()`:** Timsort.

**C++ `std::sort`:** introsort (quicksort + heapsort + insertion sort).

**In practice, never implement your own sort for production** — the language's built-in has 30 years of engineering.

---

## Worked example 8.8 — Stability matters

**Input:** `[(3, "Alice"), (1, "Bob"), (3, "Carol"), (2, "Dave")]`.

Sort by the first field.

**Stable sort output:** `[(1, "Bob"), (2, "Dave"), (3, "Alice"), (3, "Carol")]` — Alice before Carol.

**Unstable sort output:** `[(1, "Bob"), (2, "Dave"), (3, "Carol"), (3, "Alice")]` — Carol before Alice. Both 3s have the same key but swapped relative order.

**Impact.** If you now sort by the second field (name), the stable sort preserves Alice-before-Carol for the tie in department 3. The unstable sort loses that information.

**Rule:** use stable sorts (merge, insertion, bubble) when doing multi-key sorts. Use quicksort only when you don't care about relative order of equal elements.

---

## Chapter summary

1. **Comparison-based sorting has Ω(n log n) worst-case lower bound** — information-theoretic.
2. **Bubble sort** — O(n²) average, O(n) best (with early exit). Pedagogical only.
3. **Insertion sort** — O(n²) average, O(n) best. **Excellent for small and nearly-sorted arrays.**
4. **Selection sort** — always O(n²), O(n) swaps. Rarely preferred.
5. **Merge sort** — O(n log n) all cases, **stable**, but O(n) space.
6. **Quicksort** — O(n log n) average, **O(n²) worst** on sorted input with last-element pivot. **Randomise the pivot.**
7. **Heapsort** — O(n log n) all cases, O(1) space, not stable.
8. **Stability** preserves relative order of equal elements — critical for multi-key sorting.
9. **In-place** sorts use O(1) auxiliary memory; merge sort does not.
10. **Practical choice:** primitives → quicksort; objects → Timsort (Java/Python); O(1) space required → heapsort; small/nearly-sorted → insertion sort.

---

## Quiz

### A. Multiple choice

1. Which sort is **stable**?

   (a) Selection (b) Quicksort (c) **Merge** (d) Heapsort

2. Which sort has **O(n²) worst-case** but is fastest in practice on average?

   (a) Merge (b) **Quicksort** (c) Insertion (d) Bubble

3. Which sort uses **O(n) auxiliary space**?

   (a) Quicksort (b) Heapsort (c) **Merge** (d) Insertion

4. Insertion sort's best case is:

   (a) O(1) (b) O(log n) (c) **O(n)** (d) O(n²)

5. The comparison-sort lower bound is:

   (a) O(1) (b) O(log n) (c) O(n) (d) **Ω(n log n)**

6. Quicksort's worst case is triggered by:

   (a) Random data (b) **Already-sorted data with last-element pivot** (c) Reverse data
   (d) All-distinct values

7. Heapsort's space complexity is:

   (a) O(1) (b) **O(1)** (c) O(log n) (d) O(n)

8. Which algorithm is **never** used in production?

   (a) Insertion (b) **Bubble** (c) Quicksort (d) Merge

9. Which sort processes in-place with O(1) space?

   (a) Merge (b) **Heapsort** (c) Timsort (d) Radix

10. Java's `Arrays.sort` for **objects** uses:

    (a) Quicksort (b) **Timsort** (c) Heapsort (d) Bubble

11. Which sort has an O(n) best case?

    (a) Selection (b) **Insertion** (c) Merge (d) Heapsort

12. **Stable** sorting means:

    (a) The algorithm is deterministic (b) **Equal elements keep original relative order**
    (c) It always terminates (d) It uses O(1) space

### B. Fill in the blank

1. A sort is **\_\_\_\_\_** if it preserves the relative order of equal elements.

2. A sort is **\_\_\_\_\_** if it uses O(1) auxiliary memory.

3. Any comparison sort requires at least **Ω(\_\_\_\_\_)** comparisons in the worst case.

4. Merge sort's recurrence is **T(n) = 2 T(n/2) + O(\_\_\_\_\_)**, giving **O(n log n)**.

5. Quicksort's worst case occurs when the pivot is always the **\_\_\_\_\_** or **\_\_\_\_\_** element.

6. **\_\_\_\_\_** sort builds a max-heap, then repeatedly extracts the max.

7. **\_\_\_\_\_** sort is used by Java's `Arrays.sort` for **objects**.

8. **\_\_\_\_\_** sort is best for already-sorted input.

9. **\_\_\_\_\_** sort always does at most n−1 swaps.

10. **\_\_\_\_\_** sort has O(n log n) worst case AND O(1) space.

### C. Define / explain

1. Define **stability** and **in-place**. Give one sort of each combination.

2. Prove the **Ω(n log n)** lower bound for comparison sorts.

3. Explain why **quicksort's worst case** is O(n²). How do you prevent it?

4. Compare **merge sort** and **quicksort** on: stability, space, worst case, practical speed.

5. Trace **bubble sort** on `[4, 1, 3, 2]`. How many swaps?

6. Trace **insertion sort** on `[4, 1, 3, 2]`. How many shifts?

7. Trace **merge sort** on `[4, 1, 3, 2]`. Show all merge steps.

8. Explain why **stability matters** for multi-key sorting. Give a concrete example.

9. Why is **insertion sort** used inside quicksort for small subarrays?

10. Which sort would you choose for each scenario: (a) sorting 10 million integers in memory; (b) sorting 1 million objects needing stable order; (c) nearly-sorted data; (d) sorting in O(1) space.

---

## Answer key

??? success "A. Multiple choice"
    1. **(c)** Merge.
    2. **(b)** Quicksort.
    3. **(c)** Merge.
    4. **(c)** O(n).
    5. **(d)** Ω(n log n).
    6. **(b)** Already-sorted with last-element pivot.
    7. **(a)** O(1) (same as b).
    8. **(b)** Bubble.
    9. **(b)** Heapsort.
    10. **(b)** Timsort.
    11. **(b)** Insertion.
    12. **(b)** Equal elements keep original order.

??? success "B. Fill in the blank"
    1. stable
    2. in-place
    3. $n \log n$
    4. $n$
    5. minimum; maximum (smallest; largest)
    6. Heapsort
    7. Timsort
    8. Insertion
    9. Selection
    10. Heapsort

??? success "C. Define / explain — model answers"
    1. *Stable:* preserves original relative order of equal elements (merge, insertion, bubble — stable; quicksort, heapsort, selection — unstable). *In-place:* uses O(1) auxiliary memory (bubble, insertion, selection, heapsort, quicksort — in-place; merge — not). *Combinations:* stable+in-place = insertion; stable+not-in-place = merge; unstable+in-place = quicksort or heapsort; unstable+not-in-place = selection with extra buffer (rarely used).
    2. *Lower-bound proof:* comparison sorts distinguish `n!` possible input orderings. Each comparison yields 1 bit. A binary decision tree distinguishing `n!` possibilities must have depth ≥ log₂(n!). Stirling: log₂(n!) ≈ n log₂ n − n log₂ e = Ω(n log n). So any comparison sort makes ≥ Ω(n log n) comparisons in the worst case. ∎
    3. *Quicksort worst case:* with last-element pivot, sorted input causes each partition to split off one element (the pivot at the end). n partitions, each scanning ~n elements → n + (n-1) + ... = O(n²). *Prevention:* random pivot (expected O(n log n) with high probability), median-of-three, or shuffle before sort.
    4. *Merge vs. quicksort:* merge — stable, O(n) space, O(n log n) worst, slower constants. Quicksort — unstable, O(1) space, O(n log n) average but O(n²) worst, faster constants. *When to choose:* merge when stability or worst-case guarantee required; quicksort when speed matters and you control the pivot.
    5. *Bubble sort trace on `[4, 1, 3, 2]`:* Pass 1 — (4,1)→swap, (4,3)→swap, (4,2)→swap → `[1, 3, 2, 4]`. Pass 2 — (1,3) keep, (3,2)→swap → `[1, 2, 3, 4]`. Pass 3 — no swap → done. **Total swaps: 4.**
    6. *Insertion sort trace on `[4, 1, 3, 2]`:* i=1, key=1, shift 4 → `[1, 4, 3, 2]`. i=2, key=3, shift 4 → `[1, 3, 4, 2]`. i=3, key=2, shift 4, shift 3 → `[1, 2, 3, 4]`. **Total shifts: 4.**
    7. *Merge sort trace on `[4, 1, 3, 2]`:* divide → `[4,1]` and `[3,2]`. Sort each → `[1,4]` and `[2,3]`. Merge → `[1,2,3,4]`. **Cost: O(n log n) = O(4 log 4) = O(8) comparisons.**
    8. *Why stability matters:* for multi-key sorting, you sort by one key, then by another. Stability preserves the first sort as a tiebreaker. Example: sort employees by department, then by name — a stable sort yields alphabetically-ordered names within each department. An unstable sort scrambles the name order within each department.
    9. *Insertion sort for small subarrays:* its constant factor is tiny (~1 comparison per element when data is already nearly sorted from prior partitioning). The overhead of quicksort's partition and recursion dominates for < ~20 elements. So switch to insertion sort at the base case — faster than continuing to recurse.
    10. *Sort choice:*
        - (a) **10M ints in memory:** dual-pivot quicksort (fastest in practice).
        - (b) **1M objects, stable:** Timsort (Java/Python default).
        - (c) **Nearly sorted:** insertion sort (O(n) best case).
        - (d) **O(1) space:** heapsort (guaranteed O(n log n), O(1) space).

---

## Further reading

- **CLRS**, Chapters 2 (Insertion), 4 (Divide & conquer), 7 (Quicksort), 8 (Sorting lower bounds).
- **Weiss**, Chapter 7 (Sorting).
- **Lafore**, Chapter 3 (Simple sorting), Chapter 6 (Recursion & merge sort), Chapter 7 (Advanced sorting).
- **Sedgewick & Wayne**, Chapters 2.1–2.5 (Elementary, merge, quicksort, priority queues, applications).
- **Knuth**, *The Art of Computer Programming*, Volume 3 — the definitive reference.
- **Timsort:** [en.wikipedia.org/wiki/Timsort](https://en.wikipedia.org/wiki/Timsort) — the algorithm behind Java and Python sorts.
- **Dual-pivot quicksort** paper (Yaroslavskiy, 2009) — the algorithm Java uses for primitives.
- **VisuAlgo** — [visualgo.net/en/sorting](https://visualgo.net/en/sorting) — animated comparisons of all six sorts.

---

## Suggested tutorial problems

1. **Bubble trace.** Sort `[5, 3, 1, 4, 2]` with bubble sort. Count passes, comparisons, swaps.

2. **Insertion trace.** Same array, insertion sort. Count comparisons, shifts.

3. **Selection trace.** Same array, selection sort. Count swaps.

4. **Merge trace.** Same array, merge sort. Show all merge steps and total comparisons.

5. **Quicksort trace.** Same array, quicksort with last-element pivot. Show the partition steps.

6. **Quicksort worst case.** Sort `[1, 2, 3, 4, 5, 6]` with last-element pivot quicksort. Count comparisons. Compare with sorted input.

7. **Heapsort trace.** Same array as problem 1, heapsort. Show build-heap and extract-max steps.

8. **Stability test.** Given `[(3,A),(1,B),(2,C),(3,D),(1,E)]`, sort by key with merge sort and with quicksort. Which preserves A before D and B before E?

9. **Nearly-sorted analysis.** For an array that is 90% sorted (only a few elements out of place), which sort is fastest? Estimate comparisons.

10. **Comparison count.** For `n = 1000`, estimate comparisons for each sort: best, average, worst.

11. **External sort.** Design a sort for 10 GB of data on a machine with 1 GB of RAM. What's the strategy? (Hint: external merge sort.)

12. **Sorting integers.** Given an array of 10 million integers in `[0, 1000]`, which sort is fastest? Why? (Hint: counting sort.)