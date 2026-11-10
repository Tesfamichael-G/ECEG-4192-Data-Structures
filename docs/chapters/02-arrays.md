# Chapter 2 — Arrays

!!! abstract "Chapter at a glance"
    **Contact hours:** 2 lecture + 1 tutorial + 2 lab · **Home study:** ~5 h
    
    **Coverage (syllabus):** memory allocation · operations in arrays.

---

## Learning outcomes

By the end of this chapter you should be able to:

1. Explain the **contiguous memory model** that underlies arrays.
2. Derive the **indexing arithmetic** that makes `array[i]` O(1).
3. Write and analyse the cost of **insert**, **delete**, **search**, and **traverse** operations on static arrays.
4. Explain **dynamic arrays** and the **amortised analysis** of doubling expansion.
5. Compare **linear search** to **binary search** with Big-O proof.
6. Handle **2D arrays** and understand **row-major** versus **column-major** traversal costs.
7. Recognise problems where arrays are the right data structure and where they are not.

---

## 2.1 The memory model — why arrays are fast

### 2.1.1 Contiguous memory

An **array** stores its elements in **consecutive memory cells**:

```
Memory address:   1000    1004    1008    1012    1016    1020
Element:         [ a0 ]  [ a1 ]  [ a2 ]  [ a3 ]  [ a4 ]  [ a5 ]
Index:             0       1       2       3       4       5
```

If the array starts at address **B** and each element takes **w** bytes (w = 4 for `int` on most systems, w = 8 for `long` / `double`), then the element at index `i` lives at:

$$
\boxed{\;\text{address}(a_i) = B + i \cdot w\;}
$$

This is the **entire reason** arrays are fast. Given `i`, the CPU computes one multiplication and one addition — **a couple of CPU cycles** — and gets the address. No scanning, no pointer chasing.

### 2.1.2 Cache locality — the hidden superpower

Modern CPUs read memory in **cache lines** (typically 64 bytes). When you access `a[i]`, the CPU fetches the entire line containing it — which contains **16 consecutive ints** (64 / 4).

So when you access `a[i]`, `a[i+1]`, `a[i+2]`, … in a loop, **15 of the next 15 accesses hit cache** and cost almost nothing.

```
Access a[0]: misses cache, fetches 16 ints → cache hit for a[1]..a[15]
Access a[16]: misses cache, fetches next 16 ints → cache hit for a[17]..a[31]
```

**Consequence:** a linear scan of an array is often **10–100× faster** in wall-clock terms than iterating a linked list of the same length — even though both are O(n) — because the linked list's nodes are scattered in memory.

We'll come back to this when we compare arrays to linked lists (Chapter 3).

### 2.1.3 Bounds checking (and why it costs almost nothing)

When you write `a[i]` in Java or Python, the runtime checks `0 ≤ i < a.length`. If not, it throws `ArrayIndexOutOfBoundsException` (Java) or `IndexError` (Python).

This check takes **one extra branch** — almost free on modern CPUs (branch prediction handles it). But it's worth knowing: in C or C++, `a[i]` performs **no check**. If `i` is out of bounds, the program reads (or writes) whatever memory happens to be at `B + i*w` — **undefined behaviour**.

---

## 2.2 Static arrays — operations and costs

A **static array** has a fixed size, declared at creation.

### 2.2.1 Traverse

```java
for (int i = 0; i < n; i++) {
    sum += a[i];
}
```

$n$ iterations, each O(1). **Total: O(n).**

### 2.2.2 Search — linear scan

```java
int linearSearch(int[] a, int target) {
    for (int i = 0; i < a.length; i++) {
        if (a[i] == target) return i;
    }
    return -1;
}
```

**Worst case: O(n)** (target is at the end or absent). **Best case: O(1)** (target is first). **Average: O(n)** (uniform distribution).

### 2.2.3 Search — binary search (on sorted arrays)

```java
int binarySearch(int[] a, int target) {
    int lo = 0, hi = a.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;   // avoids overflow of (lo+hi)/2
        if (a[mid] == target) return mid;
        if (a[mid] < target) lo = mid + 1;
        else                 hi = mid - 1;
    }
    return -1;
}
```

The search space **halves** each iteration:

- After 1 iteration: size $n/2$
- After 2: $n/4$
- After $k$: $n / 2^k$

Stop when $n / 2^k < 1$, i.e., $k = \lceil \log_2 n \rceil$. **Total: O(log n).**

**The catch:** binary search requires the array to be **sorted**. Sorting is O(n log n) — you'd pay that cost once if you need many searches.

### 2.2.4 The full cost table

| Operation | Best | Average | Worst |
|---|:---:|:---:|:---:|
| Access `a[i]` | O(1) | O(1) | O(1) |
| Traverse | O(n) | O(n) | O(n) |
| Linear search | O(1) | O(n) | O(n) |
| Binary search (sorted) | O(1) | O(log n) | O(log n) |
| Insert at end (static) | — | — | **not supported** (array is full) |
| Insert at position `p` | O(1) (if `p = n`) | O(n) | O(n) |
| Delete at position `p` | O(1) (if `p = n-1`) | O(n) | O(n) |

The "insert at end" that **isn't supported** for static arrays is a real limitation. That's what dynamic arrays fix (§2.3).

---

## 2.3 Dynamic arrays — resizing and amortised analysis

### 2.3.1 The idea

A **dynamic array** (Java `ArrayList`, Python `list`, C++ `std::vector`) hides a static array inside, with two extra fields:

```java
class DynamicArray {
    private int[] data;
    private int size;     // how many elements actually in use
    
    public DynamicArray() {
        data = new int[1];   // start small
        size = 0;
    }
    
    public void add(int x) {
        if (size == data.length) {
            resize(data.length * 2);   // double the capacity
        }
        data[size++] = x;
    }
    
    private void resize(int newCap) {
        int[] newData = new int[newCap];
        System.arraycopy(data, 0, newData, 0, size);   // O(n) copy
        data = newData;
    }
}
```

The `add` operation is **O(1) most of the time** — but occasionally O(n) when the internal array is full and needs doubling.

### 2.3.2 Amortised analysis — why doubling wins

The key question: **is `add` really O(1)?**

Naively: no. Some calls take O(n) — those that resize. The **worst case per call is O(n)**.

But **amortised over a sequence of many calls**, the average cost per `add` is **O(1)**. Here's the proof.

!!! abstract "Amortised cost of `add` (doubling strategy)"
    Suppose we do $n$ additions starting from an empty dynamic array of capacity 1. Resizes happen at sizes $1, 2, 4, 8, \ldots$ — i.e., at sizes $1, 2, 4, \ldots, 2^k$ where $2^k \le n$.

    Total cost of all resizes:
    $$
    1 + 2 + 4 + \cdots + 2^k = 2^{k+1} - 1 < 2n.
    $$
    
    Total cost of all $n$ insertions (each O(1)): $n$.
    
    **Total cost:** less than $n + 2n = 3n = O(n)$.

    **Amortised cost per `add`:** $3n / n = O(1)$. ✓

The geometric series $\sum 2^i = 2^{k+1} - 1$ is the key insight — the sum of all doubles is less than **twice** the last one. Doubling is what makes this work; if you grew by **+1 each time** (incrementing capacity), the cost would be $1 + 2 + 3 + \ldots + n = O(n^2)$ → O(n) amortised — **no better than a linked list without tail pointer.**

### 2.3.3 Why not grow by 3× or 10×?

You can — but **doubling is the sweet spot**:

| Growth factor | Amortised `add` | Space wasted | Worst-case `add` |
|:---:|:---:|:---:|:---:|
| +1 (increment) | O(n) | minimal | O(n) |
| ×1.5 | O(1) | ≤ 33% | O(n) |
| **×2** | **O(1)** | **≤ 50%** | O(n) |
| ×3 | O(1) | ≤ 67% | O(n) |
| ×10 | O(1) | ≤ 90% | O(n) |

Growth factor 2 has a clean trade-off: amortised O(1), at most 50% wasted capacity. Anything ≥ 2 gives O(1); smaller gives more space efficiency at the cost of more frequent resizes.

### 2.3.4 Shrinking

If you `add` a million elements, then `remove` most of them, the internal array stays at a million — **wasting 4 MB of memory for an empty array**.

The **shrinking strategy**: when `size` drops below `capacity / 4`, halve the capacity.

**Why `capacity / 4` and not `capacity / 2`?** If you shrink at exactly half, then adding one more element triggers an immediate resize **back up**. Alternating add/remove would cause **thrashing** — every operation resizes. Using `capacity / 4` gives a buffer: shrink only when the array is comfortably empty.

---

## 2.4 Insert and delete — the shifting cost

For a **static array** (or the internal array of a dynamic array), insert/delete at a position `p` requires shifting.

### 2.4.1 Insert at position `p`

```java
// Assumes size < data.length
void insertAt(int[] data, int size, int p, int x) {
    for (int i = size; i > p; i--) {
        data[i] = data[i - 1];   // shift right
    }
    data[p] = x;
}
```

**Shifts:** elements at positions `p, p+1, …, size-1` move right — that's `size - p` shifts.

| `p` | Shifts | Cost |
|:---:|:---:|:---:|
| 0 (front) | `size` | **O(n)** |
| `size/2` (middle) | `size/2` | O(n) |
| `size` (end) | 0 | **O(1)** |

**Worst case: O(n)** (insert at front). **Best case: O(1)** (insert at end — append).

### 2.4.2 Delete at position `p`

```java
void deleteAt(int[] data, int size, int p) {
    for (int i = p; i < size - 1; i++) {
        data[i] = data[i + 1];   // shift left
    }
}
```

**Shifts:** elements at `p+1, …, size-1` move left — that's `size - 1 - p` shifts.

Same cost structure as insert: O(n) worst case, O(1) when deleting the last element.

!!! warning "Common mistake"
    Deleting the **first** element is the **most expensive** (all `n-1` shift). Deleting the **last** is free. If your algorithm repeatedly deletes from the front, **use a queue or linked list instead** — not an array.

---

## Worked example 2.1 — Proving `a[i]` is O(1)

**Claim.** Accessing `a[i]` on an array takes O(1) time.

**Proof.** The address of `a[i]` is $B + i \cdot w$, where $B$ is the base address and $w$ is the element width. Computing this address requires:

1. One multiplication: $i \cdot w$ — a single CPU instruction.
2. One addition: $B + i \cdot w$ — a single CPU instruction.
3. One memory load from that address — a single CPU instruction.

Total: **3 primitive operations**, independent of $i$ and of the array length. Therefore, O(1). ∎

**Contrast with linked list.** Accessing the $i$-th element of a linked list requires following $i$ `next` pointers — O(i) time, O(n) worst case. This is the entire reason arrays are preferred when access by index is frequent.

---

## Worked example 2.2 — Insert at position `p` in a dynamic array

**Setup.** An array of size 8 with capacity 8 (full). Insert the value `x` at position `p = 3`.

**Step-by-step trace.**

```
Before:
Index:   0   1   2   3   4   5   6   7
Value: [10, 20, 30, 40, 50, 60, 70, 80]
Size = 8, Capacity = 8   → full, must resize first.
```

**Step 1 — Resize** (double to capacity 16, copy 8 elements — O(n)). Now capacity = 16.

**Step 2 — Shift right** from `size-1 = 7` down to `p = 3`:

```
After shift:
Index:   0   1   2   3   4   5   6   7   8
Value: [10, 20, 30, 40, 40, 50, 60, 70, 80]
                     ↑
                     positions 3..7 shifted right
```

Wait — that's not right. Let me re-trace.

Correct shift (from i = 8 down to i = 4):
- `data[8] = data[7]` → 80 at index 8
- `data[7] = data[6]` → 70 at index 7
- `data[6] = data[5]` → 60 at index 6
- `data[5] = data[4]` → 50 at index 5
- `data[4] = data[3]` → 40 at index 4

```
After shift:
Index:   0   1   2   3   4   5   6   7   8
Value: [10, 20, 30, 40, 40, 50, 60, 70, 80]
                     ↑
                     index 3 is now overwritable
```

**Step 3 — Insert** `x = 35` at index 3:

```
After insert:
Index:   0   1   2   3   4   5   6   7   8
Value: [10, 20, 30, 35, 40, 50, 60, 70, 80]
Size = 9, Capacity = 16
```

**Cost.** Resize = O(n) copy (8 elements); shift = 5 = `size - p` operations; insert = 1 write. **Total: O(n).**

---

## Worked example 2.3 — Amortised cost of `add` with doubling

**Setup.** Start with capacity 1, empty. Insert 17 elements one at a time. Track the total number of element copies.

| Insert # | Size before | Capacity | Action | Copies this step | Total copies |
|:---:|:---:|:---:|---|---:|---:|
| 1 | 0 | 1 | append (no resize) | 0 | 0 |
| 2 | 1 | 1 | **resize to 2** | 1 | 1 |
| 3 | 2 | 2 | **resize to 4** | 2 | 3 |
| 4 | 3 | 4 | append | 0 | 3 |
| 5 | 4 | 4 | **resize to 8** | 4 | 7 |
| 6–9 | 5–8 | 8 | append | 0 | 7 |
| 10 | 8 | 8 | **resize to 16** | 8 | 15 |
| 11–17 | 9–16 | 16 | append | 0 | 15 |

**Total copies for 17 insertions: 15.**

**Amortised copies per insertion:** $15 / 17 < 1$ — **constant**, less than 1 per insertion.

The general pattern: inserting $n$ elements into a doubling dynamic array copies at most $2n - 1$ elements total. Per insertion: $< 2$. **O(1) amortised.** ✓

---

## Worked example 2.4 — Why growing by +1 is O(n²)

**Setup.** Start with capacity 1, empty. Insert $n$ elements, but **each resize grows capacity by 1** (worst-case strategy).

| Insert # | Size before | Capacity | Copies |
|:---:|:---:|:---:|---:|
| 1 | 0 | 1 | 0 |
| 2 | 1 | 1 | 1 |
| 3 | 2 | 2 | 2 |
| 4 | 3 | 3 | 3 |
| ... | ... | ... | ... |
| $n$ | $n-1$ | $n-1$ | $n-1$ |

Total copies: $0 + 1 + 2 + \cdots + (n-1) = \frac{n(n-1)}{2} = \Theta(n^2)$.

**Amortised cost per insert:** $\Theta(n^2) / n = \Theta(n)$ — **per insert, quadratic in the worst case**.

**Contrast with doubling:** doubling gave total copies $< 2n$ (linear); +1 gives $n^2/2$ (quadratic). **The growth factor is the difference between linear and quadratic overall cost.**

---

## Worked example 2.5 — Binary search trace

Search for `42` in `[2, 5, 8, 12, 16, 23, 38, 42, 56, 72, 91]` (sorted, n = 11).

| Iteration | `lo` | `hi` | `mid` | `a[mid]` | Compare |
|:---:|:---:|:---:|:---:|:---:|---|
| 1 | 0 | 10 | 5 | 23 | 42 > 23 → `lo = 6` |
| 2 | 6 | 10 | 8 | 56 | 42 < 56 → `hi = 7` |
| 3 | 6 | 7 | 6 | 38 | 42 > 38 → `lo = 7` |
| 4 | 7 | 7 | 7 | 42 | **found at index 7** ✓ |

**Iterations: 4.** Theory: $\lceil \log_2 11 \rceil = 4$. ✓

**Comparison with linear search.** Linear search for `42` would take **8 iterations** (indices 0 through 7). Binary search took **4**. At $n = 10^6$, binary search takes ~20 iterations; linear search takes up to a million. **The gap is astronomical.**

---

## Worked example 2.6 — Reverse an array in place

**Goal.** Reverse `[1, 2, 3, 4, 5]` → `[5, 4, 3, 2, 1]` using O(1) auxiliary space.

**Algorithm — two-pointer swap.**

```java
void reverse(int[] a) {
    int left = 0, right = a.length - 1;
    while (left < right) {
        int t = a[left];
        a[left] = a[right];
        a[right] = t;
        left++;
        right--;
    }
}
```

**Trace on `[1, 2, 3, 4, 5]`:**

| Step | `left` | `right` | Array |
|:---:|:---:|:---:|---|
| 0 | 0 | 4 | `[1, 2, 3, 4, 5]` |
| 1 | 1 | 3 | `[5, 2, 3, 4, 1]` |
| 2 | 2 | 2 | `[5, 4, 3, 2, 1]` — `left == right`, stop |

**Cost.** $\lfloor n/2 \rfloor$ swaps, each O(1). **Time: O(n).** **Auxiliary space: O(1).**

**Contrast — out-of-place reversal** using a new array would also be O(n) time but O(n) space. **In-place is strictly better** if you can modify the input.

Python version (same idea, more concise):

```python
def reverse(a):
    left, right = 0, len(a) - 1
    while left < right:
        a[left], a[right] = a[right], a[left]
        left += 1
        right -= 1
```

---

## Worked example 2.7 — Rotate an array by `k` positions

**Goal.** Rotate `[1, 2, 3, 4, 5, 6, 7]` right by `k = 3` → `[5, 6, 7, 1, 2, 3, 4]`.

**Approach 1 (naive).** Shift each element by `k` positions, one element at a time. $\Theta(n \cdot k)$ — quadratic if `k` is large.

**Approach 2 (using auxiliary array).** Copy elements to their new positions in a new array, then copy back. **O(n) time, O(n) space.**

**Approach 3 (in-place, O(1) space).** The **"reverse three times"** trick:

1. Reverse the whole array: `[7, 6, 5, 4, 3, 2, 1]`
2. Reverse the first `k` elements: `[5, 6, 7, 4, 3, 2, 1]`
3. Reverse the remaining `n-k` elements: `[5, 6, 7, 1, 2, 3, 4]`

Wait — step 2 reverses the **first k** (indices 0..k-1 = 0..2), giving `[5, 6, 7, ...]`. ✓

Step 3 reverses **the rest** (indices k..n-1 = 3..6), giving `[..., 1, 2, 3, 4]`. ✓

Combined: `[5, 6, 7, 1, 2, 3, 4]` — correct! ✓

```java
void rotate(int[] a, int k) {
    int n = a.length;
    k = k % n;              // handle k > n
    if (k == 0) return;
    reverse(a, 0, n - 1);
    reverse(a, 0, k - 1);
    reverse(a, k, n - 1);
}

void reverse(int[] a, int from, int to) {
    while (from < to) {
        int t = a[from]; a[from] = a[to]; a[to] = t;
        from++; to--;
    }
}
```

**Cost.** Three reversals, each O(n). **Total: O(n) time, O(1) space.** This is the **optimal** solution — beats the auxiliary-array approach on space.

---

## Worked example 2.8 — Two-pointer sum in a sorted array

**Problem.** Given a **sorted** array and a target `T`, find two elements that sum to `T`.

**Brute force.** Two nested loops: O(n²).

**Two-pointer (sorted array).**

```java
boolean hasTwoSum(int[] a, int T) {
    int left = 0, right = a.length - 1;
    while (left < right) {
        int sum = a[left] + a[right];
        if (sum == T) return true;
        if (sum < T) left++;
        else         right--;
    }
    return false;
}
```

**Why it works.** Because the array is **sorted**:

- If `sum < T`: the current pair is too small. Increase `left` (larger element).
- If `sum > T`: the current pair is too large. Decrease `right` (smaller element).

**Correctness argument.** Suppose the true pair is `(a[i], a[j])` with `i < j`. At any iteration, either:

- `left < i` or `right > j`: the algorithm hasn't reached the pair yet, but the sum tells it which direction to go — and the pair will eventually be reached.
- If the algorithm ever sets `left > i` (i.e., passed `i`), then the previous step must have had `a[left] + a[right] < T`, meaning even with the largest possible `right`, `left` was too small — so `left = i` would have been a solution candidate. Contradiction.

So the algorithm never skips the pair.

**Cost.** `left` and `right` move toward each other at most $n$ total steps. **O(n) time, O(1) space.**

**Contrast.** Brute force is O(n²). Two-pointer is O(n). **The sorted precondition enables the linear algorithm.**

---

## Worked example 2.9 — Traversing a 2D array: row-major vs. column-major

**Setup.** An array `int[][] matrix` of size $n \times n$. Two traversals:

**Row-major** (outer loop over rows):
```java
for (int i = 0; i < n; i++)
    for (int j = 0; j < n; j++)
        sum += matrix[i][j];
```

**Column-major** (outer loop over columns):
```java
for (int j = 0; j < n; j++)
    for (int i = 0; i < n; i++)
        sum += matrix[i][j];
```

**Big-O: both are O(n²).** Same iteration count.

**Wall-clock: row-major is often 2–5× faster.**

**Why?** Java (like C, C++, Python's NumPy) stores 2D arrays in **row-major order** — element `matrix[i][j]` is at address $B + (i \cdot n + j) \cdot w$.

Row-major traversal visits consecutive addresses → excellent cache behaviour.
Column-major traversal jumps by `n * w` between accesses → poor cache behaviour (each access misses).

**The Big-O is the same, but the constant is different** — exactly the "constants matter" lesson from Chapter 1.

**Python (NumPy) is the same:** NumPy is row-major by default, so iterating over rows is faster than iterating over columns.

---

## 2.5 When arrays are the right choice

Arrays excel at:

- **Random access by index** — O(1) per access; no other structure does better.
- **Dense iteration** — visiting every element is cache-friendly and fast.
- **Fixed-size collections** — if you know the size, arrays are the simplest and most efficient.
- **Storing data that needs sorting** — arrays are the natural home for sort algorithms.

Arrays are the wrong choice when:

- **Frequent insertions or deletions** in the middle or front — O(n) each shift.
- **Size unknown and highly variable** — dynamic arrays amortise well but still resize occasionally.
- **Very sparse data** — a dense array would waste memory.

The next chapters show the alternatives — linked lists for flexible size and cheap insertion at the front, hash tables for fast lookup, trees for ordered data.

---

## Chapter summary

1. Arrays store elements **contiguously**; the address of `a[i]` is `B + i*w`, computable in **O(1)**.
2. **Cache locality** makes arrays ~10–100× faster than pointers in wall-clock terms for linear scans.
3. **Static arrays** support O(1) access and O(n) insert/delete at interior positions (shifting cost).
4. **Binary search** on a sorted array is **O(log n)** — but requires the array to be sorted.
5. **Dynamic arrays** grow by doubling; **amortised `add` cost is O(1)** because the sum of resize copies is < 2n.
6. Growing by **+1** instead of doubling gives **O(n) amortised** — a factor-of-n penalty.
7. **Shrinking** should happen at `capacity/4`, not `capacity/2`, to avoid thrashing.
8. **In-place** operations (reverse, rotate) can achieve O(1) auxiliary space.
9. **Two-pointer** techniques exploit sortedness for O(n) algorithms where brute force is O(n²).
10. **Row-major traversal** of a 2D array is faster than column-major in row-major languages — same Big-O, better constants.

---

## Quiz

### A. Multiple choice

1. `a[i]` is O(1) because:

   (a) arrays are immutable (b) the address is computable with a fixed number of operations
   (c) arrays are always sorted (d) Python caches results

2. Inserting at the **front** of an n-element array takes:

   (a) O(1) (b) O(log n) (c) O(n) (d) O(n²)

3. Binary search requires:

   (a) the array to be unsorted (b) the array to be sorted (c) the array to contain only integers
   (d) the array to have an even length

4. Adding `n` elements to a dynamic array with **doubling** has total cost:

   (a) O(n) (b) O(n log n) (c) O(n²) (d) O(2ⁿ)

5. Adding `n` elements to a dynamic array growing by **+1** has total cost:

   (a) O(n) (b) O(n log n) (c) O(n²) (d) O(2ⁿ)

6. Amortised cost of one `add` with doubling is:

   (a) O(1) (b) O(log n) (c) O(n) (d) O(n²)

7. Rotating an array by `k` positions in place (reverse-three-times trick) has cost:

   (a) O(n) time, O(1) space (b) O(n log n) time, O(n) space
   (c) O(n²) time, O(1) space (d) O(n) time, O(n) space

8. Row-major traversal of a 2D array in Java is faster than column-major because:

   (a) row-major uses less memory (b) rows have fewer elements (c) cache locality is better
   (d) row-major uses a different algorithm

9. Auxiliary space of an in-place array reversal is:

   (a) O(1) (b) O(log n) (c) O(n) (d) O(n²)

10. Shrinking a dynamic array should trigger at:

    (a) `size == capacity / 2` (b) `size == capacity / 4` (c) `size == 0` (d) never

11. Two-pointer sum in a **sorted** array is:

    (a) O(1) (b) O(log n) (c) O(n) (d) O(n²)

12. A static array of size 100 is full. Trying to `add` a 101st element:

    (a) silently drops it (b) throws an error or corrupts memory
    (c) grows automatically (d) queues it internally

### B. Fill in the blank

1. An element at index `i` in an array with base `B` and width `w` is at address **\_\_\_\_\_**.

2. Binary search on a sorted array of size `n` runs in **O(\_\_\_\_\_)**.

3. Inserting at the **end** of a static array (if not full) costs **O(\_\_\_\_\_)**.

4. Dynamic arrays grown by **doubling** have total copies $< $\_\_\_\_ for `n` insertions.

5. The amortised cost of `add` with doubling is **O(\_\_\_\_\_)**.

6. An in-place reversal of an array uses **O(\_\_\_\_\_)** auxiliary space.

7. Rotating an array in place uses the **\_\_\_\_\_** three-times trick.

8. Row-major traversal of `matrix[i][j]` in Java accesses consecutive **\_\_\_\_\_**.

9. The two-pointer sum trick requires the array to be **\_\_\_\_\_**.

10. Shrinking a dynamic array at `capacity/4` avoids **\_\_\_\_\_** (repeated resize).

### C. Define / explain

1. Explain why `a[i]` is O(1) using the address arithmetic formula. Contrast with a linked list's `get(i)`.

2. Describe the **cache locality advantage** of arrays over linked lists. Why is it not visible in Big-O analysis?

3. Derive the **amortised cost** of `add` with doubling, showing that `n` insertions copy < 2n elements total.

4. Explain why growing by +1 gives **O(n) amortised** instead of O(1). Show the total-copies calculation.

5. Compare the cost of `insert at position p` for `p = 0`, `p = n/2`, and `p = n` in a static array. Which is cheapest and why?

6. State the **binary search algorithm**. Prove that it runs in O(log n).

7. Describe the **reverse-three-times** algorithm for array rotation. Prove correctness using the rotation equation.

8. Explain the **two-pointer sum** algorithm. Why does it work only on sorted arrays?

9. Explain the difference between **time complexity** and **wall-clock performance** for 2D array traversal. Give an example where Big-O is the same but wall-clock is 5× different.

10. Give three situations where arrays are the **wrong choice** and suggest what data structure would be better.

---

## Answer key

??? success "A. Multiple choice"
    1. **(b)** — fixed number of operations to compute the address.
    2. **(c)** — O(n) shifting.
    3. **(b)** — sorted array required.
    4. **(a)** — O(n) total with doubling.
    5. **(c)** — O(n²) with +1 growth.
    6. **(a)** — O(1) amortised.
    7. **(a)** — O(n) time, O(1) space.
    8. **(c)** — cache locality.
    9. **(a)** — O(1).
    10. **(b)** — `capacity/4` avoids thrashing.
    11. **(c)** — O(n).
    12. **(b)** — error or corruption (undefined behaviour in C, exception in Java).

??? success "B. Fill in the blank"
    1. $B + i \cdot w$
    2. $\log n$
    3. $O(1)$
    4. $2n$
    5. $O(1)$
    6. $O(1)$
    7. reverse
    8. addresses (or memory locations)
    9. sorted
    10. thrashing

??? success "C. Define / explain — model answers"
    1. *Address arithmetic:* `a[i]` lives at `B + i*w`. Computing this takes one multiplication, one addition, one memory load — a **constant number of CPU instructions**, independent of `i` and `n`. Therefore O(1). *Contrast with linked list:* the $i$-th element requires following $i$ `next` pointers, so `get(i)` is O(i) and worst case O(n). This is why arrays win on random-access workloads.
    2. *Cache locality:* the CPU fetches 64-byte cache lines containing **16 consecutive ints**. Row-major traversal of an array touches each line once, reusing it 15 times. *Linked list:* nodes are scattered in memory — each access likely misses cache. *Why Big-O misses this:* Big-O counts **operations**, not **memory accesses**. Both traversals count $n$ operations, but the array traversal only has $n/16$ cache misses; the linked list traversal has $n$. **Wall-clock reflects the cache**; Big-O does not.
    3. *Amortised analysis with doubling:* resizes happen at sizes $1, 2, 4, 8, \ldots, 2^k$ where $2^k \le n$. Total copies: $1 + 2 + 4 + \ldots + 2^k = 2^{k+1} - 1 < 2n$. Per-insertion amortised: $< 2 = O(1)$. The key is the **geometric series** — the sum of all doubles is less than 2× the largest.
    4. *+1 growth:* resizes happen at **every** size $1, 2, \ldots, n-1$, copying $1, 2, \ldots, n-1$ elements respectively. Total: $1 + 2 + \ldots + n-1 = n(n-1)/2 = O(n^2)$. Per-insertion amortised: $O(n)$. The sum is **arithmetic**, not geometric — hence quadratic, not linear.
    5. *Insert at position `p`:* shifts elements at `p, p+1, ..., n-1` right, which is `n - p` shifts. *At `p = 0`:* n shifts → O(n) worst case. *At `p = n/2`:* n/2 shifts → O(n). *At `p = n`:* 0 shifts → **O(1)** best case. **The end is cheapest** because no elements need to shift right — it's just an append. This is why the `add` operation (append) is the fastest insertion.
    6. *Binary search:* maintain a search interval `[lo, hi]`. Compute `mid = (lo + hi) / 2`. If `a[mid] == target`, return. If `a[mid] < target`, the target must be in `[mid+1, hi]`. If `a[mid] > target`, in `[lo, mid-1]`. *Proof of O(log n):* each iteration **halves** the interval. Starting from size $n$, after $k$ iterations the interval has size $n / 2^k$. Stop when size < 1, i.e., $k = \lceil \log_2 n \rceil$. Therefore $T(n) = O(\log n)$.
    7. *Reverse-three-times:* Let $A = UV$ where $U$ is the first $n-k$ elements and $V$ is the last $k$. Rotation is $(UV)^R$ = $VU$. The trick: reverse all → $V^R U^R$; reverse first $k$ → $V U^R$; reverse rest → $VU$. ✓ Correct.
    8. *Two-pointer sum:* maintain `left = 0`, `right = n-1`. Compute `sum = a[left] + a[right]`. If `sum == target`, found. If `sum < target`, `left++` (need larger element). If `sum > target`, `right--` (need smaller). *Why sorted required:* the direction choice depends on whether `a[left]` and `a[right]` are the smallest and largest remaining candidates — true only in a sorted array. If unsorted, `a[left]` could be anything, so the pointer direction is meaningless. **O(n) time, O(1) space.**
    9. *Time vs. wall-clock:* Big-O counts **operations**, independent of memory hierarchy. *Example:* row-major vs. column-major traversal of a 1000×1000 matrix both take 10⁶ iterations. But row-major accesses consecutive addresses → good cache → ~5 ms. Column-major accesses strided addresses → cache misses every iteration → ~25 ms. **Same operations, different memory access patterns, 5× wall-clock difference.**
    10. *Wrong choice of array:* (i) **Frequent insertions/deletions in the middle** — every insert/delete is O(n); use a linked list or balanced tree. (ii) **Size highly variable** — resizing is amortised O(1) but still costly; use a linked list for truly unbounded/frequent-variable. (iii) **Sparse data with large gaps** — a dense array wastes memory; use a hash table or sparse representation. (iv) **Frequent lookup by key (not index)** — arrays require O(n) or O(log n) search; use a hash table for O(1) lookup.

---

## Further reading

- **Weiss**, *Data Structures and Algorithm Analysis*, Chapter 3 (Lists, Stacks, Queues) — includes an excellent section on the array vs. linked list trade-off.
- **Lafore**, Chapter 2 (Arrays) — basic array operations.
- **CLRS**, Section 17.4 (Dynamic tables) — amortised analysis of doubling in rigorous form.
- **Sedgewick & Wayne**, *Algorithms*, Section 1.3 (Bags, Queues, Stacks) — the dynamic-array analysis.
- **Joel Spolsky**, "Back to Basics" — [joelonsoftware.com/2001/12/11/back-to-basics/](https://www.joelonsoftware.com/2001/12/11/back-to-basics/) — classic essay on the cache advantage of arrays.
- **Ulrich Drepper**, "What Every Programmer Should Know About Memory" — [people.freebsd.org/~lstewart/articles/cpumemory.pdf](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf) — the definitive reference on why cache locality matters.

---

## Suggested tutorial problems

1. **Prove `a[i]` is O(1).** Write down the address computation and count the primitive operations.

2. **Insertion cost table.** For an array of size 100, compute the number of shifts for insertions at positions `0, 25, 50, 75, 100`. Which position is cheapest?

3. **Amortised cost, traced.** Start with capacity 1, empty dynamic array with doubling. Insert 32 elements, tracking the total copies. Verify the total is < 2n.

4. **+1 growth.** Repeat problem 3 but with +1 growth. Compute the total copies for `n = 32`. What is the ratio to doubling?

5. **Binary search tree of decisions.** For an array of size 1024, how many binary-search iterations are needed to find the first element? The last? A missing element?

6. **Reverse a subarray.** Write `reverse(a, i, j)` that reverses the segment `a[i..j]` in place. What is its cost?

7. **Rotate by `k`.** Implement the three-reversal rotation. Verify by hand on `[1, 2, 3, 4, 5]` with `k = 2`.

8. **Two-pointer sum, missing pair.** In the sorted array `[1, 3, 5, 7, 9, 11]`, find two elements summing to `15`. Trace each step of the two-pointer algorithm.

9. **2D array traversal.** Allocate `int[1000][1000]`. Time a row-major scan vs. column-major scan. Report the ratio. Explain the difference in cache terms.

10. **Dynamic array shrink policy.** Suppose the shrink trigger is `size < capacity / 2`. Show an adversarial sequence of insert/delete operations that alternates between growing and shrinking, causing O(n) work per operation. Then show that `capacity / 4` avoids this.

11. **Array vs. linked list — quantitative.** For `n = 100,000` random lookups on a sorted array (binary search) vs. a sorted linked list (linear search), estimate the wall-clock ratio. Which wins by how much?

12. **Two-pointer sum on unsorted arrays.** Can you adapt the two-pointer algorithm to work on an unsorted array by **pre-sorting** it first? What is the total cost, and how does it compare to brute force?

13. **Sliding-window average.** Given an array and an integer `k`, find the maximum average of any `k` consecutive elements. Design an O(n) algorithm using a sliding window. Why is the naive algorithm O(n·k)?

14. **Duplicate detection.** Given an unsorted array of `n` integers, find whether any value appears twice. Give O(n²), O(n log n), and O(n) solutions. What are their respective space costs?

15. **Missing number.** An array contains `n-1` distinct integers from `{1, 2, …, n}` (one is missing). Find the missing number in O(n) time and O(1) space. (Hint: use arithmetic or XOR.)