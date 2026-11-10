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
  js: assets/lecture-02.js
---

# ECEG-4192

## Chapter 2 — Arrays

Memory allocation · Operations · Complexity

<aside class="notes">
Arrays are the foundation. Everything else — stacks, queues, heaps, hash tables — is built on the array.
</aside>

---

## Learning outcomes

By the end of this chapter you should be able to:

1. Explain what an array **is** and how it is laid out in memory.

2. Compute the **address** of any element in **O(1)**.

3. Analyse the cost of **insert, delete, search** on arrays.

4. Explain **row-major** vs **column-major** storage.

5. Reason about the **amortised** cost of dynamic arrays.

6. Choose between arrays and linked lists for a given workload.

---

## 2.1 — What is an array?

???+ note "Definition — Array"
    A collection of elements of the **same type**, stored in a **contiguous** block of memory, accessed by an integer **index** (0-based in C, C++, Java, Python).

```c
int a[5] = {10, 20, 30, 40, 50};
// a[0]=10, a[1]=20, a[2]=30, a[3]=40, a[4]=50
```

<div class="fragment">

Key properties:

- **Homogeneous** — all elements same type.
- **Contiguous** — one block of memory.
- **Fixed size** (in statically typed languages).
- **Indexed** — 0-based.

</div>

---

## 2.2 — Memory allocation

Elements stored **contiguously**. Address of element $i$:

$$ \text{addr}(a[i]) = \text{base} + i \times \text{sizeof(type)} $$

<div class="fragment">

!!! tip "This is why array access is O(1)."
    One multiplication, one addition, one read.
    
    No traversal. No pointer chasing. Just arithmetic.

</div>

---

## Layout example (int array, 4 bytes each)

| Index | Address | Value |
|:---:|:---:|:---:|
| 0 | 1000 | 10 |
| 1 | 1004 | 20 |
| 2 | 1008 | 30 |
| 3 | 1012 | 40 |
| 4 | 1016 | 50 |

<div class="fragment">

`a[3]` → address $1000 + 3 \times 4 = 1012$ → value 40.

**One arithmetic step. Constant time.**

</div>

---

## 2.3 — Basic operations

| Operation | Complexity | Why |
|---|---|---|
| **Access by index** | **O(1)** | Direct arithmetic |
| **Search (unsorted)** | **O(n)** | Linear scan |
| **Search (sorted)** | **O(log n)** | Binary search |
| **Insert at end** | **O(1)*** | If capacity allows |
| **Insert at position** | **O(n)** | Shift elements |
| **Delete at position** | **O(n)** | Shift elements |

*Amortised for dynamic arrays — occasionally resize.

---

## Insertion — shifting

```c
// Insert value at index k (array has n elements, capacity >= n+1)
for (int i = n; i > k; i--)
    a[i] = a[i-1];
a[k] = value;
n++;
```

<div class="fragment">

If $k = 0$: **$n$ shifts** = $O(n)$.

If $k = n$: **0 shifts** = $O(1)$.

**Average cost:** $n/2$ shifts = $O(n)$.

</div>

---

## Deletion — shifting

```c
// Delete element at index k
for (int i = k; i < n-1; i++)
    a[i] = a[i+1];
n--;
```

<div class="fragment">

If $k = 0$: **$n-1$ shifts** = $O(n)$.

If $k = n-1$: **0 shifts** = $O(1)$.

**Average cost:** $O(n)$.

</div>

<div class="fragment">

!!! warning "Lesson"
    Arrays are **fast at the end, slow in the middle**.

</div>

---

## 2.4 — Multidimensional arrays

**2D array** `int m[3][4];` → 12 contiguous ints.

Two storage orders:

<div class="fragment">

**Row-major (C, C++, Python/NumPy default):**
`m[0][0], m[0][1], ..., m[0][3], m[1][0], ...`

**Column-major (Fortran, MATLAB):**
`m[0][0], m[1][0], m[2][0], m[0][1], ...`

</div>

<div class="fragment">

Access `m[i][j]` in row-major:

$$ \text{addr} = \text{base} + (i \cdot \text{cols} + j) \times \text{sizeof(type)} $$

</div>

---

## Why storage order matters

**Cache behavior.** Consecutive memory addresses get fetched together.

???+ note "Row-major + row-wise iteration"
    ```c
    for (int i = 0; i < rows; i++)
        for (int j = 0; j < cols; j++)
            sum += m[i][j];     // sequential — cache-friendly
    ```

???+ note "Row-major + column-wise iteration"
    ```c
    for (int j = 0; j < cols; j++)
        for (int i = 0; i < rows; i++)
            sum += m[i][j];     // strided — cache-thrashing
    ```

<div class="fragment">

**Same O(rows × cols) time. Up to 10× difference in practice.**

</div>

---

## 2.5 — Dynamic arrays — amortised analysis

Python `list`, Java `ArrayList`, C++ `std::vector` — these are **dynamic arrays**.

???+ note "Growth strategy"
    When full, allocate **2×** the current size and copy old elements.

<div class="fragment">

**Cost of $n$ appends:**

- Most appends: $O(1)$ — just write at the next slot.
- Occasional append: $O(n)$ — copy everything.

Total over $n$ appends: $1 + 1 + \ldots + 1$ (most) + $n + 2n + \ldots$ (copies).

</div>

<div class="fragment">

**Amortised cost per append: O(1).**

The occasional expensive operation is **paid for** by the many cheap ones.

</div>

---

## Worked example — amortised cost

Appending $n = 16$ items to an empty dynamic array (capacity doubling):

| Append # | Cost | Capacity after |
|:---:|:---:|:---:|
| 1 | 1 (alloc 1) | 1 |
| 2 | 2 (copy 1) | 2 |
| 3 | 1 | 4 |
| 4 | 2 | 4 |
| 5–8 | 1 each, then copy 4 | 8 |
| 9–16 | 1 each, then copy 8 | 16 |

<div class="fragment">

**Total work for 16 appends: ~32.**

Average: **2 ops per append** = **amortised O(1)**.

</div>

---

## 2.6 — Arrays vs linked lists

| Use case | Array | Linked list |
|---|:---:|:---:|
| Random access by index | ✅ O(1) | ❌ O(n) |
| Insert/delete at end | ✅ O(1)* | ✅ O(1) with tail |
| Insert/delete in middle | ❌ O(n) | ✅ O(1) with pointer |
| Memory locality | ✅ Cache-friendly | ❌ Pointer chasing |
| Memory overhead | ✅ Minimal | ❌ Pointers per node |
| Fixed capacity | ❌ | ✅ Dynamic |

<div class="fragment">

**Pick the structure whose *hot path* is cheapest.**

</div>

---

## Chapter summary

1. **Array** = contiguous, homogeneous, indexed block.

2. **Access is O(1)** — arithmetic address calculation.

3. **Insert/delete in middle = O(n)** — requires shifts.

4. **Insert/delete at end = O(1)** — with amortised analysis for dynamic arrays.

5. **Row-major vs column-major** — determines cache behavior.

6. **Amortised O(1)** append via capacity doubling.

7. Arrays are the base for **stacks, queues, heaps, hash tables**.

---

## Review questions

1. Why is array access O(1)? Give the address formula.

2. Explain why inserting at the beginning of an array is O(n).

3. Compare row-major and column-major storage. Which does C use?

4. Show the amortised argument for dynamic-array append.

5. When would you prefer an array over a linked list? Give a concrete example.

6. Give the memory overhead comparison for $n$ ints in an array vs a linked list.

> Answers: **Solutions → Chapter 2**.

---

## References

- **Weiss**, Ch. 2 — array implementation details.
- **Lafore**, Ch. 2 — simple arrays in Java.
- **CLRS**, §10.1 — stacks and queues as array applications.
- **Sedgewick & Wayne**, §1.4 — dynamic array resizing.