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
  js: assets/lecture-01.js
---

# ECEG-4192

## Chapter 1 — Introduction, Data Abstraction, and Analysis of Algorithms

Semester VII · 2025/26

<aside class="notes">
This chapter sets the foundation. Everything that follows — arrays, lists, trees, graphs — is judged by the analysis framework we build today.
</aside>

---

## Learning outcomes

By the end of this chapter you should be able to:

<div class="fragment">

1. Say what a **data structure** is, and why a course about them exists.

</div>

<div class="fragment">

2. Distinguish an **abstract data type (ADT)** from a **data structure implementation**.

</div>

<div class="fragment">

3. Write the **Big-O** bounds of simple loops and nested loops.

</div>

<div class="fragment">

4. Distinguish **Big-O**, **Big-Ω**, and **Big-Θ**.

</div>

<div class="fragment">

5. Analyse the complexity of a **recursive** algorithm by writing its recurrence and solving it.

</div>

<div class="fragment">

6. Recognise the **common complexity classes** — O(1), O(log n), O(n), O(n log n), O(n²), O(2ⁿ) — and give one example of each.

</div>

<div class="fragment">

7. Explain why **Big-O ignores constants** but why constants still matter in practice.

</div>

---

## 1.1 — Why a course on data structures?

### The problem

Programs spend most of their time **finding, storing, and retrieving** data.

How that data is laid out in memory decides whether a program finishes in **milliseconds** or **hours**.

<div class="fragment">

Two implementations of the same idea can differ by a factor of **millions** in runtime. Not a small percentage — a factor of 10⁶.

</div>

<aside class="notes">
Ask the class: how many records does a typical university store? Tens of thousands. A bank? Millions. At those scales, the difference between an unsorted array and a hash table is the difference between a working app and a timeout.
</aside>

---

## Concrete example

Store $n = 10{,}000$ student records, find one by name:

<div class="fragment">

???+ note "Three storage strategies"
    | Approach | Comparisons |
    |---|---|
    | Unsorted array, scan | up to $10{,}000$ |
    | Sorted array, binary search | $\log_2(10{,}000) \approx 14$ |
    | Hash table | 1–2 (average) |

</div>

<div class="fragment">

Same data. Same computer. Same programmer.

**Different storage strategy → ~700× faster.**

</div>

<aside class="notes">
This is the motivating example for the entire course. Notice: the programmer did not write better code. They chose a better data structure. That's the skill we're building.
</aside>

---

## Definition — Data structure

???+ note "Definition"
    A **data structure** is a *way of organizing data in memory* together with *the set of operations* that can be performed on that data efficiently.

<div class="fragment">

Notice **both** halves of the definition:

- The **layout** — where each item lives in memory, how items relate to each other.
- The **operations** — `insert`, `delete`, `search`, `iterate`, `findMinimum` — each has a cost.

</div>

<div class="fragment">

A data structure is **not just "how the data is stored."** It is **how the data is stored *plus* what it lets you do cheaply.**

</div>

---

## Two roles of a data structure

In practice you do two things with data structures:

<div class="fragment">

???+ note "Role 1 — Use existing ones"
    Pick from the standard library. Justify the choice with **complexity analysis**.

</div>

<div class="fragment">

???+ note "Role 2 — Create new ones"
    When no standard structure fits, build a new one out of **existing building blocks**.

</div>

<div class="fragment">

The rest of this course develops both skills: the first through **implementation** of the classics, the second through the **reasoning** that makes each classic work.

</div>

---

## The recurring theme — time vs space

Every data structure trades **time** against **space**. Almost always:

<div class="fragment">

- **More space** = **faster access** (arrays: contiguous, O(1) indexing, but wasteful if sparse).
- **Less space** = **slower access** (linked lists: no wasted memory, but O(n) lookup).

</div>

<div class="fragment">

!!! warning "There is no free lunch."
    The rest of the course is a study of **specific trade-offs**.

</div>

---

## 1.2 — Data abstraction

### The idea of abstraction

**Abstraction** means *hiding the details behind a simple interface*.

<div class="fragment">

You already know this from OOP:

- You use `List<String>` without knowing whether it's an `ArrayList` or a `LinkedList` inside.
- You call `add(x)` without thinking about resizing or pointer updates.

</div>

<div class="fragment">

Data structures push this idea further. They separate **what** a structure does from **how** it does it.

</div>

---

## ADT — Abstract Data Type

???+ note "Definition — Abstract data type (ADT)"
    An **abstract data type** is a specification of a set of values and a set of operations, *independent of any particular implementation*.

<div class="fragment">

An ADT says: *"There is a collection of elements, and you can do these things with them."*

It does **not** say: *"Elements are stored in an array"* or *"Elements are stored in nodes connected by pointers."*

</div>

---

## ADTs you will meet in this course

| ADT | Operations specified | Possible implementations |
|---|---|---|
| **List** | insert, delete, get, size, iterate | Array, linked list, dynamic array |
| **Stack** | push, pop, peek, isEmpty | Array, linked list |
| **Queue** | enqueue, dequeue, peek | Array, circular array, linked list |
| **Map / Dictionary** | put, get, remove, containsKey | Hash table, balanced tree |
| **Set** | add, remove, contains | Hash table, balanced tree |
| **Priority Queue** | insert, extractMin | Binary heap, sorted array |

<div class="fragment">

**The pattern:** one ADT, **many implementations**. The **interface** is the same; the **cost of operations** differs.

</div>

---

## Why abstraction pays off

???+ info "Three concrete reasons"

<div class="fragment">

**1. Choose the implementation that fits.**
A program that only needs `insert` at the front and `iterate` is well-served by a linked list. A program that needs random access by index wants an array.

</div>

<div class="fragment">

**2. Change it later without breaking callers.**
Swap the underlying array for a linked list without touching the caller. The **interface is the contract**.

</div>

<div class="fragment">

**3. Reason at the interface level.**
"This algorithm makes $n$ calls to `insert`" — true regardless of implementation.

</div>

</div>

---

## Worked example 1.1 — Same ADT, different costs

Consider a **List** ADT with `insertAtEnd(x)`, `get(i)`, `deleteAtEnd()`.

<div class="fragment">

???+ note "Array-based list"
    | Operation | Cost | Why |
    |---|:---:|---|
    | `insertAtEnd(x)` | **O(1)** amortised | Append at index `size`; occasionally double |
    | `get(i)` | **O(1)** | Direct indexing |
    | `deleteAtEnd()` | **O(1)** | Decrement `size` |

</div>

<div class="fragment">

???+ note "Linked-list"
    | Operation | Cost | Why |
    |---|:---:|---|
    | `insertAtEnd(x)` | **O(n)** | Walk to the end |
    | `get(i)` | **O(n)** | Walk `i` steps |
    | `deleteAtEnd()` | **O(n)** | Walk to the end |

</div>

<div class="fragment">

**Same operations, same semantics.** The array implementation is **dramatically faster** for these three operations.

</div>

---

## Worked example 1.2 — Choosing an ADT for a text editor

You are building a text editor that needs to: insert characters at cursor, delete before cursor, move cursor, undo.

<div class="fragment">

???+ note "Candidate 1 — Dynamic array"
    Insert at position `p` costs **O(n)** because everything after `p` shifts. For a 100,000-character document, editing near the start shifts 100,000 characters — **perceptibly slow**.

</div>

<div class="fragment">

???+ note "Candidate 2 — Doubly-linked list"
    Insert at cursor = **O(1)** (splice between two nodes). Cursor move = **O(1)**. Undo = **O(1)** with a stack of operations. **This fits.**

</div>

<div class="fragment">

**Choice:** a **doubly-linked list** plus a **stack** for undo. Operations that must be fast are all O(1). Operations that would be slow (random access by index) are not needed.

</div>

---

## 1.3 — The need for a formal measure

### Why not just "run it and time it"?

```python
import time
start = time.time()
sort(data)
elapsed = time.time() - start
```

<div class="fragment">

The number is **useless for comparison** across:

- **Machines** — a laptop and a phone give different times.
- **Languages** — Python vs C differ by 100×.
- **Data sizes** — the result on 100 items tells you nothing about 10⁶.
- **Inputs** — the same algorithm can be fast on sorted data and slow on random data.

</div>

<div class="fragment">

What we want is a **machine-independent, size-relative** measure of how the algorithm's cost **grows** with input size.

</div>

---

## Counting operations

Instead of wall-clock time, **count primitive operations**:

<div class="fragment">

- The number of comparisons
- The number of assignments
- The number of array accesses
- The number of nodes visited

</div>

<div class="fragment">

Different primitives have different physical times, but in asymptotic analysis **they all count as one "step."** The result is a **step count** $T(n)$ as a function of input size $n$.

</div>

---

## Example — counting steps

```java
int sum(int[] a) {
    int total = 0;                        // 1 step
    for (int i = 0; i < a.length; i++) {  // n+1 comparisons, n increments
        total += a[i];                    // n additions
    }
    return total;                         // 1 step
}
```

<div class="fragment">

Total steps: $1 + (n+1) + n + n + 1 = 3n + 3$.

We say: **$T(n) = 3n + 3$** — "the cost grows **linearly** in $n$."

</div>

---

## Growth, not absolute value

The key question is not "how many steps at $n = 100$?" but "how many steps at $n = 10^6$?"

| $n$ | $T(n) = 3n + 3$ | $T(n) = 0.001 n^2 + 5$ |
|:---:|:---:|:---:|
| 10 | 33 | 5.1 |
| 100 | 303 | 15 |
| 1,000 | 3,003 | 1,005 |
| 10,000 | 30,003 | 100,005 |
| 1,000,000 | 3,000,003 | 1,000,000,005 |

<div class="fragment">

For small $n$, the quadratic (with a small leading constant) is **faster**. For large $n$, the linear algorithm wins by a factor of ~300,000.

**Growth rate is what matters, and Big-O captures it.**

</div>

---

## 1.4 — Big-O notation

???+ note "Definition — Big-O"
    We write $f(n) = O(g(n))$ if there exist positive constants $c$ and $n_0$ such that
    
    $$ f(n) \le c \cdot g(n) \quad \text{for all } n \ge n_0. $$
    
    In words: **$f$ grows no faster than a constant times $g$**, eventually.

<div class="fragment">

**Intuition.** $f(n) = O(g(n))$ means "$g$ is an **upper bound** on the growth of $f$."

It ignores constants and small values of $n$ — what's left is the **shape of the growth curve**.

</div>

---

## Worked example 1.3 — Prove $3n + 3 = O(n)$

We need $c$ and $n_0$ such that $3n + 3 \le c \cdot n$ for all $n \ge n_0$.

<div class="fragment">

**Try $c = 4$:** $3n + 3 \le 4n$ ⟺ $3 \le n$. Holds for all $n \ge 3$.

</div>

<div class="fragment">

**Try $c = 6$:** $3n + 3 \le 6n$ ⟺ $3 \le 3n$ ⟺ $1 \le n$. Holds for $n \ge 1$.

</div>

<div class="fragment">

**Conclusion.** $3n + 3 = O(n)$, with witness $c = 6$, $n_0 = 1$. ✓

**Interpretation.** For large $n$, the linear term dominates; the constant is negligible.

</div>

---

## Worked example 1.4 — Prove $n^2 - 5n + 7 = O(n^2)$

We need $c, n_0$ such that $n^2 - 5n + 7 \le c n^2$ for all $n \ge n_0$.

<div class="fragment">

**Try $c = 2$:** $n^2 - 5n + 7 \le 2n^2$ ⟺ $7 - 5n \le n^2$.

Since $n^2 \ge 0$ and $7 - 5n \le 7$, we need $7 \le n^2$ — holds for $n \ge 3$.

</div>

<div class="fragment">

**Conclusion.** $n^2 - 5n + 7 = O(n^2)$, with witness $c = 2$, $n_0 = 3$. ✓

</div>

---

## Worked example 1.5 — Prove a *tighter* bound does not hold

Prove $5n^2 + 3n \ne O(n)$.

<div class="fragment">

**Proof by contradiction.** Suppose $5n^2 + 3n = O(n)$.

Then there exist $c, n_0$ such that $5n^2 + 3n \le c \cdot n$ for all $n \ge n_0$.

</div>

<div class="fragment">

Divide by $n$ (positive): $5n + 3 \le c$ for all $n \ge n_0$.

But $5n + 3$ grows without bound as $n$ grows. For any fixed $c$, there exists $n > (c - 3)/5$ where the inequality **fails**. Contradiction.

</div>

<div class="fragment">

**Conclusion.** $5n^2 + 3n \ne O(n)$ — the quadratic term makes the growth strictly faster than linear.

</div>

---

## Rules of Big-O arithmetic

| Rule | Statement | Example |
|---|---|---|
| **Drop constants** | $c \cdot f(n) = O(f(n))$ | $7n = O(n)$ |
| **Drop lower-order terms** | $f(n) + g(n) = O(\max(f, g))$ | $n^2 + n = O(n^2)$ |
| **Transitivity** | $f = O(g)$ and $g = O(h)$ ⇒ $f = O(h)$ | — |
| **Sum rule** | $f + g = O(\max(f, g))$ | $n \log n + n = O(n \log n)$ |
| **Product rule** | $f \cdot g = O(f \cdot g)$ | $n \cdot \log n = O(n \log n)$ |

<div class="fragment">

!!! warning "The trick that catches students."
    In $O(n^2 + n)$, the $n$ term is **not** "small" — it's just **dominated by** $n^2$ for large $n$. The $n^2 + n$ expression is *exact*; the $O(n^2)$ is a simplification that captures the dominant growth.

</div>

---

## 1.5 — Big-Ω and Big-Θ

???+ note "Definition — Big-Ω"
    We write $f(n) = \Omega(g(n))$ if there exist positive constants $c$ and $n_0$ such that
    
    $$ f(n) \ge c \cdot g(n) \quad \text{for all } n \ge n_0. $$
    
    In words: **$f$ grows at least as fast as a constant times $g$.**

<div class="fragment">

$f = \Omega(g)$ means "$g$ is a **lower bound**."

For example, any sort that must examine every element has $\Omega(n)$ runtime — no faster than linear is possible.

</div>

---

## Definition — Big-Θ

???+ note "Definition — Big-Θ"
    $f(n) = \Theta(g(n))$ if $f(n) = O(g(n))$ **and** $f(n) = \Omega(g(n))$.
    
    In words: **$f$ and $g$ grow at the same rate**, up to constants.

<div class="fragment">

$\Theta$ is the **strongest and most useful** statement.

"Sorting by comparison is $\Theta(n \log n)$" says: no comparison sort is faster than $n \log n$, and algorithms exist that achieve it.

</div>

---

## The hierarchy at a glance

| Notation | Meaning | Analogy |
|---|---|---|
| $f = O(g)$ | $f$ grows no faster than $g$ | $f \le g$ (up to constant) |
| $f = \Omega(g)$ | $f$ grows no slower than $g$ | $f \ge g$ (up to constant) |
| $f = \Theta(g)$ | $f$ grows at the same rate as $g$ | $f = g$ (up to constant) |
| $f = o(g)$ | $f$ grows strictly slower than $g$ | $f < g$ (strictly) |
| $f = \omega(g)$ | $f$ grows strictly faster than $g$ | $f > g$ (strictly) |

---

## Worked example 1.6 — Three statements about one function

Consider $f(n) = 2n^2 + 5n$.

| Claim | True? | Witness |
|---|:---:|---|
| $f(n) = O(n^2)$ | ✓ | $c = 3$, $n_0 = 5$ |
| $f(n) = \Omega(n^2)$ | ✓ | $c = 1$, $n_0 = 1$ |
| $f(n) = \Theta(n^2)$ | ✓ | Follows from both |
| $f(n) = O(n)$ | ✗ | See Worked example 1.5 method |
| $f(n) = \Theta(n)$ | ✗ | Would require the false $O(n)$ bound |

<div class="fragment">

**Conclusion.** $f(n) = \Theta(n^2)$ — the tightest statement.

</div>

---

## 1.6 — Analysing loops

The single most important skill in complexity analysis is **reading a loop and writing its Big-O**.

???+ info "Single loop rule"
    If a loop runs a constant number of operations per iteration and executes $k$ iterations, its cost is $O(k)$.

```java
for (int i = 0; i < n; i++) {
    sum += a[i];
}
```

<div class="fragment">

Iterations: $n$. Per iteration: constant. **Total: $O(n)$.**

</div>

---

## Worked example 1.7 — Loop with doubling index

```java
for (int i = 1; i < n; i = i * 2) {
    sum += i;
}
```

<div class="fragment">

How many iterations? Each step **doubles** `i`. Starting at 1, after $k$ iterations we have $i = 2^k$.

Stop when $2^k \ge n$, i.e., $k = \lceil \log_2 n \rceil$.

</div>

<div class="fragment">

**Total: $O(\log n)$.**

This pattern appears constantly — binary search, balanced-tree walks, exponentiation by squaring.

!!! tip "Rule of thumb"
    **Doubling or halving the index means $\log n$ iterations.**

</div>

---

## Worked example 1.8 — Two independent nested loops

```java
for (int i = 0; i < n; i++) {
    for (int j = 0; j < n; j++) {
        sum += a[i][j];
    }
}
```

<div class="fragment">

Outer: $n$ iterations. Inner: $n$ iterations **for each outer iteration**.

$n \times n = n^2$. **$O(n^2)$.**

</div>

---

## Worked example 1.9 — Nested with an "upper triangle" inner loop

```java
for (int i = 0; i < n; i++) {
    for (int j = i; j < n; j++) {
        sum += a[i][j];
    }
}
```

<div class="fragment">

Inner loop runs $n - i$ times, for outer index $i = 0, 1, \ldots, n-1$.

Total: $\sum_{i=0}^{n-1} (n - i) = n + (n-1) + \ldots + 1 = \frac{n(n+1)}{2}$.

</div>

<div class="fragment">

**Total: $O(n^2)$** — the $n^2/2$ leading term dominates; the $\frac12$ constant drops.

!!! tip "The lesson"
    Halving the iteration count (from $n^2$ to $n^2/2$) does **not** change the complexity class. **Big-O ignores constants.**

</div>

---

## Worked example 1.10 — Loop where the input drives the iterations

```java
void print(int n) {
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < i; j++) {   // note: j < i, not j < n
            System.out.println(i + j);
        }
    }
}
```

<div class="fragment">

Inner: $i$ iterations per outer step $i$. Total: $\sum_{i=0}^{n-1} i = \frac{(n-1)n}{2} = O(n^2)$.

Same class as Worked example 1.9 — a triangular pattern is $O(n^2)$.

</div>

---

## 1.7 — Analysing recursion

For recursive algorithms, we don't count iterations. We write a **recurrence relation** — an equation for $T(n)$ in terms of smaller inputs — and solve it.

???+ info "The template"
    For a recursive function of input size $n$ that:
    
    - Performs some constant work,
    - Makes $a$ recursive calls on inputs of size $n/b$,
    
    the recurrence is: $T(n) = a \cdot T(n/b) + O(1)$.

---

## The three common cases

| Case | Recurrence | Solution | Examples |
|---|---|---|---|
| **No branching** ($a = 1$) | $T(n) = T(n - 1) + O(1)$ | **$O(n)$** | Factorial, linear search |
| **Halving** ($b = 2, a = 1$) | $T(n) = T(n/2) + O(1)$ | **$O(\log n)$** | Binary search |
| **Two halvings** ($b = 2, a = 2$) | $T(n) = 2\,T(n/2) + O(n)$ | **$O(n \log n)$** | Merge sort |
| **Exponential** ($b=2, a>2$) | $T(n) = 2^n$ | **$O(2^n)$** | Naive Fibonacci |

---

## Worked example 1.11 — Linear recursion (factorial)

```java
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}
```

<div class="fragment">

Recurrence: $T(n) = T(n-1) + O(1)$.

Unrolling: $T(n) = T(n-1) + c = T(n-2) + 2c = \ldots = T(1) + (n-1)c = O(n)$.

</div>

<div class="fragment">

**Total: $O(n)$.** The recursion **does not multiply** — each call produces exactly one smaller call, so the work is linear.

</div>

---

## Worked example 1.12 — Binary search

```java
int binarySearch(int[] a, int target, int lo, int hi) {
    if (lo > hi) return -1;
    int mid = (lo + hi) / 2;
    if (a[mid] == target) return mid;
    if (a[mid] < target) return binarySearch(a, target, mid + 1, hi);
    return binarySearch(a, target, lo, mid - 1);
}
```

<div class="fragment">

The input is **halved** each call. Recurrence: $T(n) = T(n/2) + O(1)$.

Unrolling: $T(n) = T(1) + c \cdot \log_2 n = O(\log n)$.

</div>

<div class="fragment">

**Total: $O(\log n)$.**

Why? Because $n / 2^k = 1$ when $k = \log_2 n$. That's the depth of the recursion tree.

</div>

---

## Worked example 1.13 — Merge sort

```java
void mergeSort(int[] a, int lo, int hi) {
    if (lo >= hi) return;
    int mid = (lo + hi) / 2;
    mergeSort(a, lo, mid);       // recurse on left half
    mergeSort(a, mid + 1, hi);   // recurse on right half
    merge(a, lo, mid, hi);       // O(n) merge
}
```

<div class="fragment">

Recurrence: $T(n) = 2\,T(n/2) + O(n)$.

**Recursion tree:** at depth $d$, there are $2^d$ nodes, each of size $n / 2^d$. Work at depth $d$: $2^d \times O(n / 2^d) = O(n)$. Depth: $\log_2 n$. Total: $O(n \log n)$.

</div>

<div class="fragment">

**Total: $O(n \log n)$.** Merge sort is fundamentally better than bubble/insertion/selection sort ($O(n^2)$) for large inputs.

</div>

---

## Worked example 1.14 — Naive Fibonacci

```java
int fib(int n) {
    if (n <= 1) return n;
    return fib(n-1) + fib(n-2);
}
```

<div class="fragment">

Recurrence: $T(n) = T(n-1) + T(n-2) + O(1)$. This is **exponential** — $T(n) = \Theta(\phi^n)$ with $\phi \approx 1.618$.

$T(50) \approx 2 \times 10^{10}$ — **hours** on a laptop.

</div>

<div class="fragment">

???+ tip "Contrast with an iterative version"
    ```java
    int fib(int n) {
        if (n <= 1) return n;
        int a = 0, b = 1;
        for (int i = 2; i <= n; i++) { int c = a + b; a = b; b = c; }
        return b;
    }
    ```
    **Total: $O(n)$.** For $n = 50$, that's 50 additions — a microsecond.

</div>

<div class="fragment">

**Same math, same answer, 10⁶-times-faster implementation.** The reason: the naive recursion **repeats work** — `fib(n-2)` is computed inside both `fib(n-1)` and `fib(n)`.

</div>

---

## 1.8 — The common complexity classes

| Class | Name | Grows | Typical example |
|:---:|---|---|---|
| **O(1)** | constant | does not grow | array indexing, hash lookup |
| **O(log n)** | logarithmic | very slowly | binary search, balanced-tree search |
| **O(n)** | linear | linearly | array scan, linked-list traversal |
| **O(n log n)** | linearithmic | slightly super-linear | merge sort, quicksort (avg) |
| **O(n²)** | quadratic | quadratically | bubble/insertion/selection sort |
| **O(n³)** | cubic | cubically | naive matrix multiplication |
| **O(2ⁿ)** | exponential | explodes | naive Fibonacci |
| **O(n!)** | factorial | explodes faster | brute-force TSP |

---

## How big is "big"?

For $n = 1{,}000{,}000$ (1 million items) at $10^9$ operations per second:

| Class | Operations | Time |
|:---:|:---:|:---:|
| $O(1)$ | 1 | 1 ns |
| $O(\log n)$ | 20 | 20 ns |
| $O(n)$ | 10⁶ | 1 ms |
| $O(n \log n)$ | 2 × 10⁷ | 20 ms |
| $O(n²)$ | 10¹² | **11.6 days** |
| $O(2ⁿ)$ | astronomical | **billions of years** |

<div class="fragment">

!!! warning "The practical message"
    Anything worse than $O(n \log n)$ is **unusable** on million-item inputs. This is why the analysis matters — it tells you which algorithms are actually deployable.

</div>

---

## 1.9 — Constants matter

### Why Big-O ignores constants

Big-O is a **statement about growth**, not about performance. Two algorithms with the same Big-O may differ 100× in absolute speed:

- Algorithm A: $T_A(n) = 100\,n$
- Algorithm B: $T_B(n) = 0.1\,n$

Both are $O(n)$. For $n = 1000$, A takes $100{,}000$ steps, B takes $100$ steps. **B is 1000× faster**, but Big-O cannot see it.

---

## Why constants matter anyway

For **small** $n$, constants dominate. The crossover point:

| $n$ | A: $100n$ | B: $0.1n^2$ |
|:---:|:---:|:---:|
| 10 | 1000 | 10 |
| 1000 | 100,000 | 100,000 |
| 10,000 | 10⁶ | 10⁷ |

<div class="fragment">

**B faster for small $n$; A faster for large $n$.** Crossover at $n = 1000$.

This is why **quicksort** (average $O(n \log n)$ with a small constant) is faster than merge sort in practice on small to medium inputs — and why **insertion sort** ($O(n^2)$ but tiny constant) beats both on arrays of $< 20$ elements.

</div>

---

## Practical rule

!!! tip "Big-O tells you what scales."
    Constants tell you **what fits in cache, what fits in a register, what dominates at your actual problem size.**
    
    Always use both. Never pick an algorithm by Big-O alone when the actual data is small.

---

## 1.10 — Space complexity

Time is only half the story. **Space complexity** measures **how much extra memory** an algorithm uses, as a function of input size $n$.

<div class="fragment">

- **Auxiliary space** — memory in addition to the input.
- **Total space** — input + auxiliary.

</div>

```java
// In-place reversal: O(1) auxiliary space
for (int i = 0; i < n/2; i++) {
    int t = a[i]; a[i] = a[n-1-i]; a[n-1-i] = t;
}

// Out-of-place reversal: O(n) auxiliary space
int[] b = new int[n];
for (int i = 0; i < n; i++) b[i] = a[n-1-i];
```

<div class="fragment">

Both are $O(n)$ time. But the first uses $O(1)$ auxiliary space; the second uses $O(n)$.

</div>

<aside class="notes">
On a memory-constrained embedded device, this difference is decisive. Always report auxiliary space, not just time.
</aside>

---

## Recursion uses space too

!!! note "Every recursive call occupies a stack frame."
    Typically tens to hundreds of bytes. So a recursion of depth $d$ uses $O(d)$ auxiliary space.
    
    **Fibonacci's naive recursion uses $O(n)$ stack space** — the depth is $n$.

---

## Chapter summary

1. A **data structure** is a way of organizing data **and** the operations on it. The layout and the operations are equally important.

2. **Abstraction** — an **ADT** specifies *what*, an **implementation** specifies *how*. One ADT, many implementations.

3. **Big-O** describes the **growth** of an algorithm's cost as input size increases. It **drops constants and lower-order terms**.

4. **Big-Ω** is a lower bound, **Big-Θ** is a tight bound. $\Theta$ is the strongest and most useful.

5. **Loops** — count iterations; multiply for nested loops.

6. **Recursion** — write the recurrence, solve it. No-branching → $O(n)$; halving → $O(\log n)$; two-halving → $O(n \log n)$; exponential → $O(2^n)$.

7. The **common complexity classes** are $O(1)$, $O(\log n)$, $O(n)$, $O(n \log n)$, $O(n^2)$, $O(n^3)$, $O(2^n)$, $O(n!)$.

8. **Constants matter.** Big-O gives the growth shape; constants give the practical performance.

9. **Space complexity** matters as much as time. Recursion costs stack space.

---

## Review questions

<div class="quiz" data-question="ch1-q1">
  <p class="quiz-question">What does it mean for $f(n) = O(g(n))$?</p>
  <button class="quiz-option" data-answer="wrong">$f$ is always smaller than $g$</button>
  <button class="quiz-option" data-answer="correct">$f$ grows no faster than $g$, up to a constant</button>
  <button class="quiz-option" data-answer="wrong">$f$ and $g$ have equal value at every $n$</button>
  <button class="quiz-option" data-answer="wrong">$f$ is an algorithm, $g$ a program</button>
  <p class="quiz-feedback"></p>
</div>

---

## Review questions (cont.)

1. Define a data structure. Why must the definition include operations, not just layout?

2. Distinguish an ADT from an implementation. Give one ADT and two different implementations, and explain when each is preferred.

3. State the formal definition of Big-O. Use it to **prove** that $4n^2 + 3n + 1 = O(n^2)$.

4. Distinguish Big-O, Big-Ω, and Big-Θ. Give one example function for which $f = O(g)$, $f = \Omega(g)$, and $f = \Theta(g)$ all hold.

5. Analyse: triple-nested loop, outer $n$, middle $n-i$, inner constant 10. Big-O?

6. Write the recurrence for `f(n) = f(n/2) + f(n/2) + n` and solve.

7. Why does Big-O ignore constants, and why do constants still matter in practice?

8. Compare time and auxiliary space complexity of recursive vs iterative Fibonacci.

> Answers: **Solutions → Chapter 1**.

---

## References

- **Weiss**, *Data Structures and Algorithm Analysis*, Chapters 1–2.
- **Lafore**, *Data Structures & Algorithms in Java*, Chapters 1–2.
- **CLRS**, Chapter 3 (growth of functions).
- **Sedgewick & Wayne**, *Algorithms*, Section 1.4.
- **Open Data Structures** (free online) Chapter 1.
- **VisuAlgo** — [visualgo.net](https://visualgo.net/)
- **MIT 6.006** Lecture 1 — [OCW](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/)
- **Python Tutor** — [pythontutor.com](https://pythontutor.com/)

---

## Next — Chapter 2

**Arrays — memory, operations, and complexity.**

- Contiguous memory and O(1) indexing
- Insert and delete with shifting
- Multidimensional arrays: row-major vs column-major
- Dynamic arrays and amortised analysis

Every subsequent structure — stacks, queues, heaps, hash tables — starts from the array.