# Chapter 1 — Introduction, Data Abstraction, and Analysis of Algorithms

!!! abstract "Chapter at a glance"
    **Contact hours:** 2 lecture + 1 tutorial + 2 lab · **Home study:** ~5 h
    
    **Coverage (syllabus):** introduction · overview of data structures · data
    abstraction · introduction to the analysis of algorithms · **Big-O notation**.

---

## Learning outcomes

By the end of this chapter you should be able to:

1. Say what a **data structure** is, and why a course about them exists.
2. Distinguish an **abstract data type (ADT)** from a **data structure implementation**.
3. Write the **Big-O** bounds of simple loops and nested loops.
4. Distinguish **Big-O**, **Big-$\Omega$**, and **Big-$\Theta$** notation.
5. Analyse the complexity of a **recursive** algorithm by writing its recurrence and solving it.
6. Recognise the **common complexity classes** (O(1), O(log n), O(n), O(n log n), O(n²), O(2ⁿ)) and give one example of each.
7. Explain why **Big-O ignores constants** but why those constants still matter in practice.

---

## 1.1 Why a course on data structures?

### 1.1.1 The problem

Programs spend most of their time **finding, storing, and retrieving** data. How that data is laid out in memory decides whether a program finishes in milliseconds or hours.

Two implementations of the same idea can differ by a factor of **millions** in runtime. Not a small percentage — a factor of 10⁶.

**Concrete example.** Suppose you store $n = 10{,}000$ student records and want to find one by name:

- **Unsorted array, scan every record:** up to $10{,}000$ comparisons.
- **Sorted array, binary search:** about $\log_2(10{,}000) \approx 14$ comparisons.
- **Hash table:** 1–2 comparisons, on average.

Same data. Same computer. Same programmer. Different storage strategy — **700× faster**.

### 1.1.2 What a data structure is

!!! note "Definition — Data structure"
    A **data structure** is a *way of organizing data in memory* together with
    *the set of operations* that can be performed on that data efficiently.

Notice both halves of the definition:

- The **layout** — where each item lives in memory, how items relate to each other.
- The **operations** — `insert`, `delete`, `search`, `iterate`, `findMinimum` — each has a cost.

A data structure is not just "how the data is stored." It is **how the data is stored plus what it lets you do cheaply.**

### 1.1.3 Two roles of a data structure

In practice you do two things with data structures:

1. **Use existing ones** — pick from the standard library, justify the choice with complexity analysis.
2. **Create new ones** — when no standard structure fits, build a new one out of the existing building blocks.

The rest of this course develops both skills: the first through implementation of the classics, the second through the reasoning that makes each classic work.

### 1.1.4 The recurring theme — time versus space

Every data structure trades **time** against **space**. Almost always:

- **More space** = **faster access** (arrays: contiguous, O(1) indexing, but wasteful if sparse).
- **Less space** = **slower access** (linked lists: no wasted memory, but O(n) lookup).

There is no free lunch. The rest of the course is a study of specific trade-offs.

---

## 1.2 Data abstraction — ADT versus implementation

### 1.2.1 The idea of abstraction

**Abstraction** means *hiding the details behind a simple interface*. You already know this from OOP:

- You use `List<String>` without knowing whether it's an `ArrayList` or a `LinkedList` inside.
- You call `add(x)` without thinking about resizing or pointer updates.

Data structures push this idea further. They separate **what** a structure does from **how** it does it.

### 1.2.2 ADT — Abstract Data Type

!!! note "Definition — Abstract data type (ADT)"
    An **abstract data type** is a specification of a set of values and a set of
    operations, *independent of any particular implementation*.

An ADT says: "There is a collection of elements, and you can do these things with them." It does not say: "Elements are stored in an array" or "Elements are stored in nodes connected by pointers."

**Examples of ADTs you'll meet in this course:**

| ADT | Operations specified | Possible implementations |
|---|---|---|
| **List** | insert, delete, get, size, iterate | Array, linked list, dynamic array |
| **Stack** | push, pop, peek, isEmpty | Array, linked list |
| **Queue** | enqueue, dequeue, peek | Array, circular array, linked list |
| **Map / Dictionary** | put, get, remove, containsKey | Hash table, balanced tree |
| **Set** | add, remove, contains | Hash table, balanced tree |
| **Priority Queue** | insert, extractMin | Binary heap, sorted array |

Notice the pattern: **one ADT, many implementations**. The **interface** is the same; the **cost of operations** differs.

### 1.2.3 Why abstraction pays off

Three concrete reasons:

**1. Choose the implementation that fits.**

A program that only needs `insert` at the front and `iterate` is well-served by a linked list. A program that needs random access by index wants an array. **The ADT tells you what the caller needs; the implementation is the engineer's decision.**

**2. Change it later without breaking callers.**

If your code calls only `stack.push(x)` and `stack.pop()`, you can swap the underlying array for a linked list without touching the caller. The **interface is the contract**, and the implementation is free to change.

**3. Reason at the interface level.**

When reasoning about correctness or complexity, you can say "this algorithm makes $n$ calls to `insert`" — that's true regardless of implementation. Cost analysis then multiplies by the per-operation cost of the chosen implementation.

### Worked example 1.1 — Same ADT, different costs

Consider a **List** ADT with three operations: `insertAtEnd(x)`, `get(i)`, and `deleteAtEnd()`.

Two implementations:

**Array-based list** (contiguous memory):

| Operation | Cost | Why |
|---|:---:|---|
| `insertAtEnd(x)` | **O(1)** amortised | Append at index `size`; occasionally double the array (amortised) |
| `get(i)` | **O(1)** | Direct indexing: `array[i]` |
| `deleteAtEnd()` | **O(1)** | Decrement `size` |

**Linked-list** (nodes with `next` pointers):

| Operation | Cost | Why |
|---|:---:|---|
| `insertAtEnd(x)` | **O(n)** (or O(1) with tail pointer) | Walk to the end |
| `get(i)` | **O(n)** | Walk `i` steps |
| `deleteAtEnd()` | **O(n)** (or O(1) with doubly-linked + tail) | Walk to the end |

**Interpretation.** Same three operations, same names, same semantics. The array implementation is **dramatically faster** for these three operations. If you had chosen a linked list without thinking, you would have made the program hundreds of times slower for no reason.

**The lesson.** The ADT tells you *what to expect*. Complexity analysis tells you *which implementation delivers*.

### Worked example 1.2 — Choosing the right ADT for a problem

You are building a **text editor** that needs to:

- Insert characters at the cursor position
- Delete the character before the cursor
- Move the cursor left or right
- Undo the last operation

Which ADT fits?

**Candidate 1: Array (dynamic).** Insert at position `p` costs O(n) because everything after `p` shifts. For a 100,000-character document, editing near the start shifts 100,000 characters — perceptibly slow.

**Candidate 2: Linked list (doubly-linked).** Insert at cursor = O(1) (splice between two nodes). Cursor move = O(1). Undo = O(1) with a stack of operations. **This fits.**

**Choice:** a **doubly-linked list** plus a **stack** for undo.

**Justification.** The operations that must be fast (insert, delete, cursor move, undo) are all O(1) in this combination. The operations that would be slow (random access by index) are not needed — the editor never jumps to "position 72,391" directly.

This is the pattern: **write down the operations, write down the cost of each in each candidate implementation, pick the one whose hot path is cheapest.**

---

## 1.3 Analysing algorithms — the need for a formal measure

### 1.3.1 Why not just "run it and time it"?

Suppose you write a function `sort` and want to know how fast it is. You could:

```python
import time
start = time.time()
sort(data)
elapsed = time.time() - start
print(f"{elapsed:.3f} s")
```

This gives you a number, but the number is **useless for comparison** across:

- **Machines.** A laptop and a phone give different times.
- **Languages.** The same algorithm in Python and C differ by 100×.
- **Data sizes.** The result on 100 items tells you nothing about 10⁶ items.
- **Inputs.** The same algorithm can be fast on sorted data and slow on random data.

What we want is a **machine-independent, size-relative** measure of how the algorithm's cost **grows** with input size.

### 1.3.2 Counting operations

Instead of wall-clock time, **count primitive operations**:

- The number of comparisons
- The number of assignments
- The number of array accesses
- The number of nodes visited

Different primitives have different physical times, but in asymptotic analysis **they all count as one "step."** The result is a **step count** $T(n)$ as a function of input size $n$.

**Example.** The array-sum function:

```java
int sum(int[] a) {
    int total = 0;                 // 1 step
    for (int i = 0; i < a.length; i++) {   // n+1 comparisons, n increments
        total += a[i];             // n additions
    }
    return total;                  // 1 step
}
```

Total steps: $1 + (n+1) + n + n + 1 = 3n + 3$.

We say: **$T(n) = 3n + 3$**, i.e., "the cost grows **linearly** in $n$."

### 1.3.3 Growth, not absolute value

The key question is not "how many steps at $n = 100$?" but "how many steps at $n = 1000$, $n = 10^6$?"

| $n$ | $T(n) = 3n + 3$ | $T(n) = 0.001 n^2 + 5$ |
|:---:|:---:|:---:|
| 10 | 33 | 5.1 |
| 100 | 303 | 15 |
| 1,000 | 3,003 | 1,005 |
| 10,000 | 30,003 | 100,005 |
| 1,000,000 | 3,000,003 | 1,000,000,005 |

For small $n$, the quadratic (with a small leading constant) is **faster**. For large $n$, the linear algorithm wins by a factor of ~300,000. **Growth rate is what matters, and Big-O captures it.**

---

## 1.4 Big-O notation

### 1.4.1 The formal definition

!!! abstract "Definition — Big-O"
    We write $f(n) = O(g(n))$ if there exist positive constants $c$ and $n_0$ such
    that

    $$
    f(n) \le c \cdot g(n) \quad \text{for all } n \ge n_0.
    $$

    In words: **$f$ grows no faster than a constant times $g$**, eventually.

**The intuition.** $f(n) = O(g(n))$ means "$g$ is an **upper bound** on the growth of $f$." It ignores constants and small values of $n$ — what's left is the **shape of the growth curve**.

### 1.4.2 Proving a Big-O claim

To prove $f(n) = O(g(n))$, you need to **find** constants $c$ and $n_0$.

### Worked example 1.3 — Prove $3n + 3 = O(n)$

We need $c$ and $n_0$ such that $3n + 3 \le c \cdot n$ for all $n \ge n_0$.

**Try $c = 4$:** $3n + 3 \le 4n$ ⟺ $3 \le n$. This holds for all $n \ge 3$.

**Try $c = 6$:** $3n + 3 \le 6n$ ⟺ $3 \le 3n$ ⟺ $1 \le n$. Holds for $n \ge 1$.

**Conclusion.** $3n + 3 = O(n)$, with witness $c = 6$, $n_0 = 1$. ✓

**Interpretation.** For large $n$, the linear term dominates; the constant term is negligible.

### Worked example 1.4 — Prove $n^2 - 5n + 7 = O(n^2)$

We need $c, n_0$ such that $n^2 - 5n + 7 \le c n^2$ for all $n \ge n_0$.

**Try $c = 2$:** $n^2 - 5n + 7 \le 2n^2$ ⟺ $7 - 5n \le n^2$. Since $n^2 \ge 0$ and $7 - 5n \le 7$, we need $7 \le n^2$ — holds for $n \ge 3$.

**Conclusion.** $n^2 - 5n + 7 = O(n^2)$, with witness $c = 2$, $n_0 = 3$. ✓

**Interpretation.** For large $n$, the $n^2$ term dominates both the linear and the constant terms.

### Worked example 1.5 — Prove a *tighter* bound

Prove $5n^2 + 3n \ne O(n)$.

**Proof by contradiction.** Suppose $5n^2 + 3n = O(n)$. Then there exist $c, n_0$ such that $5n^2 + 3n \le c \cdot n$ for all $n \ge n_0$.

Divide by $n$ (positive): $5n + 3 \le c$ for all $n \ge n_0$.

But $5n + 3$ grows without bound as $n$ grows. For any fixed $c$, there exists $n > (c - 3)/5$ where the inequality **fails**. Contradiction.

**Conclusion.** $5n^2 + 3n \ne O(n)$ — the quadratic term makes the growth strictly faster than linear.

### 1.4.3 Rules of Big-O arithmetic

The following rules let you simplify expressions quickly:

| Rule | Statement | Example |
|---|---|---|
| **Drop constants** | $c \cdot f(n) = O(f(n))$ | $7n = O(n)$ |
| **Drop lower-order terms** | $f(n) + g(n) = O(\max(f, g))$ | $n^2 + n = O(n^2)$ |
| **Transitivity** | $f = O(g)$ and $g = O(h)$ ⇒ $f = O(h)$ | — |
| **Sum rule** | $f + g = O(\max(f, g))$ | $n \log n + n = O(n \log n)$ |
| **Product rule** | $f \cdot g = O(f \cdot g)$ | $n \cdot \log n = O(n \log n)$ |

**The trick that catches students.** In $O(n^2 + n)$, the $n$ term is **not** "small" — it's just **dominated by** $n^2$ for large $n$. The $n^2 + n$ expression is *exact*; the $O(n^2)$ is a simplification that captures the dominant growth.

---

## 1.5 Big-$\Omega$ and Big-$\Theta$

### 1.5.1 Big-$\Omega$ — a lower bound

!!! abstract "Definition — Big-$\Omega$"
    We write $f(n) = \Omega(g(n))$ if there exist positive constants $c$ and $n_0$
    such that

    $$
    f(n) \ge c \cdot g(n) \quad \text{for all } n \ge n_0.
    $$

    In words: **$f$ grows at least as fast as a constant times $g$.**

$f = \Omega(g)$ means "$g$ is a **lower bound**." For example, any sort that must examine every element has $\Omega(n)$ runtime — no faster than linear is possible.

### 1.5.2 Big-$\Theta$ — a tight bound

!!! abstract "Definition — Big-$\Theta$"
    $f(n) = \Theta(g(n))$ if $f(n) = O(g(n))$ **and** $f(n) = \Omega(g(n))$.

    In words: **$f$ and $g$ grow at the same rate**, up to constants.

$\Theta$ is the strongest and most useful statement. "Sorting by comparison is $\Theta(n \log n)$" says: no comparison sort is faster than $n \log n$, and algorithms exist that achieve it.

### 1.5.3 The hierarchy at a glance

| Notation | Meaning | Analogy |
|---|---|---|
| $f = O(g)$ | $f$ grows no faster than $g$ | $f \le g$ (up to constant) |
| $f = \Omega(g)$ | $f$ grows no slower than $g$ | $f \ge g$ (up to constant) |
| $f = \Theta(g)$ | $f$ grows at the same rate as $g$ | $f = g$ (up to constant) |
| $f = o(g)$ | $f$ grows strictly slower than $g$ | $f < g$ (strictly) |
| $f = \omega(g)$ | $f$ grows strictly faster than $g$ | $f > g$ (strictly) |

### Worked example 1.6 — Three statements about one function

Consider $f(n) = 2n^2 + 5n$.

| Claim | True? | Witness |
|---|:---:|---|
| $f(n) = O(n^2)$ | ✓ | $c = 3$, $n_0 = 5$ (verify: $2n^2 + 5n \le 3n^2$ ⟺ $5n \le n^2$ ⟺ $n \ge 5$) |
| $f(n) = \Omega(n^2)$ | ✓ | $c = 1$, $n_0 = 1$ (verify: $2n^2 + 5n \ge n^2$ ✓) |
| $f(n) = \Theta(n^2)$ | ✓ | Follows from both |
| $f(n) = O(n)$ | ✗ | See Worked example 1.5 method |
| $f(n) = \Theta(n)$ | ✗ | Would require the false $O(n)$ bound |

**Conclusion.** $f(n) = \Theta(n^2)$ — the tightest statement.

---

## 1.6 Analysing loops

The single most important skill in complexity analysis is **reading a loop and writing its Big-O**. Here is the procedure.

### 1.6.1 Single loop

**Rule.** If a loop runs a constant number of operations per iteration and executes $k$ iterations, its cost is $O(k)$.

```java
for (int i = 0; i < n; i++) {
    sum += a[i];
}
```

Iterations: $n$. Per iteration: constant. **Total: $O(n)$.**

### Worked example 1.7 — Loop with doubling index

```java
for (int i = 1; i < n; i = i * 2) {
    sum += i;
}
```

How many iterations? Each step **doubles** `i`. Starting at 1, after $k$ iterations we have $i = 2^k$. Stop when $2^k \ge n$, i.e., $k = \lceil \log_2 n \rceil$.

**Total: $O(\log n)$.**

This pattern appears constantly — binary search, balanced-tree walks, exponentiation by squaring. **Doubling or halving the index means $\log n$ iterations.**

### 1.6.2 Nested loops

**Rule.** Multiply the iteration counts.

### Worked example 1.8 — Two independent nested loops

```java
for (int i = 0; i < n; i++) {
    for (int j = 0; j < n; j++) {
        sum += a[i][j];
    }
}
```

Outer: $n$ iterations. Inner: $n$ iterations **for each outer iteration**. Total: $n \times n = n^2$.

**$O(n^2)$.**

### Worked example 1.9 — Nested with an "upper triangle" inner loop

```java
for (int i = 0; i < n; i++) {
    for (int j = i; j < n; j++) {
        sum += a[i][j];
    }
}
```

Inner loop runs $n - i$ times, for outer index $i = 0, 1, \ldots, n-1$.

Total iterations: $\sum_{i=0}^{n-1} (n - i) = n + (n-1) + \ldots + 1 = \frac{n(n+1)}{2}$.

**Total: $O(n^2)$** (the $n^2/2$ leading term dominates; the $\frac12$ constant drops).

**The lesson:** halving the iteration count (from $n^2$ to $n^2/2$) does **not** change the complexity class. **Big-O ignores constants.**

### Worked example 1.10 — Loop where the input drives the iterations

```java
void print(int n) {
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < i; j++) {   // note: j < i, not j < n
            System.out.println(i + j);
        }
    }
}
```

Inner: $i$ iterations per outer step $i$. Total: $\sum_{i=0}^{n-1} i = \frac{(n-1)n}{2} = O(n^2)$.

Same class as Worked example 1.9 — a triangular pattern is $O(n^2)$.

---

## 1.7 Analysing recursion

For recursive algorithms, we don't count iterations. We write a **recurrence relation** — an equation for $T(n)$ in terms of smaller inputs — and solve it.

### 1.7.1 The template

For a recursive function of input size $n$ that:

- Performs some constant work,
- Makes $a$ recursive calls on inputs of size $n/b$,

the recurrence is:

$$
T(n) = a \cdot T(n/b) + O(1).
$$

Its solution depends on whether $a > b$, $a = b$, or $a < b$.

### 1.7.2 The three common cases

| Case | Recurrence | Solution | Examples |
|---|---|---|---|
| **No branching** ($a = 1$) | $T(n) = T(n - 1) + O(1)$ | **$O(n)$** | Factorial, linear search |
| **Halving** ($b = 2, a = 1$) | $T(n) = T(n/2) + O(1)$ | **$O(\log n)$** | Binary search |
| **Two halvings** ($b = 2, a = 2$) | $T(n) = 2\,T(n/2) + O(n)$ | **$O(n \log n)$** | Merge sort |
| **Exponential** ($b=2, a>2$ or $b=1$) | $T(n) = 2^n$ | **$O(2^n)$** | Naive Fibonacci |

### Worked example 1.11 — Linear recursion (factorial)

```java
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}
```

Recurrence: $T(n) = T(n-1) + O(1)$.

Unrolling: $T(n) = T(n-1) + c = T(n-2) + 2c = \ldots = T(1) + (n-1)c = O(n)$.

**Total: $O(n)$.** The recursion **does not multiply** — each call produces exactly one smaller call, so the work is linear.

### Worked example 1.12 — Binary search

```java
int binarySearch(int[] a, int target, int lo, int hi) {
    if (lo > hi) return -1;
    int mid = (lo + hi) / 2;
    if (a[mid] == target) return mid;
    if (a[mid] < target) return binarySearch(a, target, mid + 1, hi);
    return binarySearch(a, target, lo, mid - 1);
}
```

The input is **halved** each call. Recurrence: $T(n) = T(n/2) + O(1)$.

Unrolling: $T(n) = T(n/2) + c = T(n/4) + 2c = \ldots = T(1) + c \cdot \log_2 n = O(\log n)$.

**Total: $O(\log n)$.**

Why? Because $n / 2^k = 1$ when $k = \log_2 n$. That's the depth of the recursion tree.

### Worked example 1.13 — Merge sort

The recursive call splits the array in half and **processes both halves**:

```java
void mergeSort(int[] a, int lo, int hi) {
    if (lo >= hi) return;
    int mid = (lo + hi) / 2;
    mergeSort(a, lo, mid);       // recurse on left half
    mergeSort(a, mid + 1, hi);   // recurse on right half
    merge(a, lo, mid, hi);       // O(n) merge
}
```

Recurrence: $T(n) = 2\,T(n/2) + O(n)$.

**Recursion tree approach:** at depth $d$, there are $2^d$ nodes, each of size $n / 2^d$. Work at depth $d$: $2^d \times O(n / 2^d) = O(n)$. Depth: $\log_2 n$. Total: $O(n \log n)$.

**Total: $O(n \log n)$.** Merge sort is fundamentally better than bubble/insertion/selection sort ($O(n^2)$) for large inputs — this is the payoff of the analysis we'll do in Chapter 8.

### Worked example 1.14 — Naive Fibonacci (why analysis matters)

```java
int fib(int n) {
    if (n <= 1) return n;
    return fib(n-1) + fib(n-2);
}
```

Recurrence: $T(n) = T(n-1) + T(n-2) + O(1)$.

This is exponential — $T(n) = \Theta(\phi^n)$ with $\phi \approx 1.618$. $T(50) \approx 2 \times 10^{10}$ — **hours** on a laptop.

**Contrast with an iterative version:**

```java
int fib(int n) {
    if (n <= 1) return n;
    int a = 0, b = 1;
    for (int i = 2; i <= n; i++) { int c = a + b; a = b; b = c; }
    return b;
}
```

**Total: $O(n)$.** For $n = 50$, that's 50 additions — a microsecond.

**Same math, same answer, 10⁶-times-faster implementation.** The reason: **the naive recursion repeats work** — `fib(n-2)` is computed inside both `fib(n-1)` and `fib(n)`. Complexity analysis reveals this — and tells you to write the iterative version.

---

## 1.8 The common complexity classes

Every algorithm you meet in this course fits in one of these classes.

### 1.8.1 The table

| Class | Name | Grows | Typical example |
|:---:|---|---|---|
| **O(1)** | constant | does not grow | array indexing, hash lookup (average) |
| **O(log n)** | logarithmic | very slowly | binary search, balanced-tree search |
| **O(n)** | linear | linearly | array scan, linked-list traversal |
| **O(n log n)** | linearithmic | slightly super-linear | merge sort, quicksort (average) |
| **O(n²)** | quadratic | quadratically | bubble/insertion/selection sort, nested loops |
| **O(n³)** | cubic | cubically | naive matrix multiplication |
| **O(2ⁿ)** | exponential | explodes | naive Fibonacci, exhaustive subset enumeration |
| **O(n!)** | factorial | explodes faster | brute-force TSP |

### 1.8.2 How big is "big"?

For $n = 1{,}000{,}000$ (1 million items) at $10^9$ operations per second:

| Class | Operations | Time |
|:---:|:---:|:---:|
| $O(1)$ | 1 | 1 ns |
| $O(\log n)$ | 20 | 20 ns |
| $O(n)$ | 10⁶ | 1 ms |
| $O(n \log n)$ | 2 × 10⁷ | 20 ms |
| $O(n²)$ | 10¹² | **11.6 days** |
| $O(2ⁿ)$ | astronomical | **billions of years** |

**The practical message.** Anything worse than $O(n \log n)$ is **unusable** on million-item inputs. This is why the analysis matters — it tells you which algorithms are actually deployable.

---

## 1.9 Constants matter — but not the way you might think

### 1.9.1 Why Big-O ignores constants

Big-O is a **statement about growth**, not about performance. Two algorithms with the same Big-O may differ 100× in absolute speed:

- Algorithm A: $T_A(n) = 100\,n$
- Algorithm B: $T_B(n) = 0.1\,n$

Both are $O(n)$. For $n = 1000$, A takes $100{,}000$ steps, B takes $100$ steps. **B is 1000× faster**, but Big-O cannot see it.

### 1.9.2 Why constants matter anyway

For **small** $n$, constants dominate. The crossover point:

- Algorithm A: $T_A(n) = 100\,n$ (fast per step)
- Algorithm B: $T_B(n) = 0.1\,n^2$ (slow per step, but quadratic)

At $n = 10$: A takes $1000$ steps, B takes $10$ steps. **B is 100× faster.**
At $n = 1000$: A takes $100{,}000$ steps, B takes $100{,}000$ steps. **Tie.**
At $n = 10{,}000$: A takes $10^6$ steps, B takes $10^7$ steps. **A is 10× faster.**

**The crossover at $n = 1000$.** For small datasets, use the "slower" algorithm. For large datasets, use the "faster" one. **This is why quicksort (average $O(n \log n)$, but with a small constant) is faster than merge sort in practice on small to medium inputs**, even though both are $O(n \log n)$ — and why insertion sort ($O(n^2)$ but tiny constant) beats both on arrays of $< 20$ elements.

!!! tip "Practical rule"
    **Big-O tells you what scales.** Constants tell you **what fits in cache, what fits in a register, what dominates at your actual problem size.** Always use both. Never pick an algorithm by Big-O alone when the actual data is small.

---

## 1.10 Space complexity

Time is only half the story. **Space complexity** measures **how much extra memory** an algorithm uses, as a function of input size $n$.

**Auxiliary space** — memory in addition to the input.
**Total space** — input + auxiliary.

**Example.** Array reversal:

```java
// In-place reversal: O(1) auxiliary space
for (int i = 0; i < n/2; i++) {
    int t = a[i]; a[i] = a[n-1-i]; a[n-1-i] = t;
}

// Out-of-place reversal: O(n) auxiliary space
int[] b = new int[n];
for (int i = 0; i < n; i++) b[i] = a[n-1-i];
```

Both are $O(n)$ time. But the first uses $O(1)$ auxiliary space; the second uses $O(n)$. On a memory-constrained embedded device, this matters.

!!! note "Recursion uses space too"
    Every recursive call occupies **a stack frame** — typically tens to hundreds of bytes. So a recursion of depth $d$ uses $O(d)$ auxiliary space. **Fibonacci's naive recursion uses $O(n)$ stack space** — the depth is $n$.

---

## 1.11 Chapter summary

1. A **data structure** is a way of organizing data **and** the operations on it. The layout and the operations are equally important.
2. **Abstraction** — an **ADT** specifies *what*, an **implementation** specifies *how*. One ADT, many implementations.
3. **Big-O** describes the **growth** of an algorithm's cost as input size increases. It **drops constants and lower-order terms**.
4. **Big-$\Omega$** is a lower bound, **Big-$\Theta$** is a tight bound. $\Theta$ is the strongest and most useful.
5. **Loops** — count iterations; multiply for nested loops.
6. **Recursion** — write the recurrence, solve it. No-branching → $O(n)$; halving → $O(\log n)$; two-halving → $O(n \log n)$; exponential → $O(2^n)$.
7. The **common complexity classes** are $O(1)$, $O(\log n)$, $O(n)$, $O(n \log n)$, $O(n^2)$, $O(n^3)$, $O(2^n)$, $O(n!)$. Anything worse than $O(n \log n)$ is un-deployable on large data.
8. **Constants matter.** Big-O gives the growth shape; constants give the practical performance. Both matter, especially for small $n$.
9. **Space complexity** matters as much as time. Recursion costs stack space.

---

## 1.12 Quiz

### A. Multiple choice

1. What does it mean for $f(n) = O(g(n))$?

   (a) $f$ is always smaller than $g$ (b) $f$ grows no faster than $g$, up to a constant
   (c) $f$ and $g$ have equal value at every $n$ (d) $f$ is an algorithm, $g$ a program

2. Which is **not** a valid Big-O simplification?

   (a) $5n + 3 = O(n)$ (b) $n^2 + n = O(n^2)$ (c) $n \log n + n = O(n \log n)$ (d) $n^2 = O(n)$

3. A loop `for (int i = 0; i < n; i++)` has complexity:

   (a) $O(1)$ (b) $O(\log n)$ (c) $O(n)$ (d) $O(n^2)$

4. A loop `for (int i = 1; i < n; i *= 2)` has complexity:

   (a) $O(1)$ (b) $O(\log n)$ (c) $O(n)$ (d) $O(n^2)$

5. Two nested loops, each iterating $n$ times, give:

   (a) $O(n)$ (b) $O(n \log n)$ (c) $O(n^2)$ (d) $O(2^n)$

6. The recurrence $T(n) = 2\,T(n/2) + O(n)$ solves to:

   (a) $O(n)$ (b) $O(n \log n)$ (c) $O(n^2)$ (d) $O(2^n)$

7. The recurrence $T(n) = T(n-1) + O(1)$ solves to:

   (a) $O(1)$ (b) $O(\log n)$ (c) $O(n)$ (d) $O(n \log n)$

8. Big-$\Theta$ notation means:

   (a) an upper bound (b) a lower bound (c) a tight upper **and** lower bound (d) the average case

9. Which of these is the **slowest-growing** function?

   (a) $O(n)$ (b) $O(\log n)$ (c) $O(n \log n)$ (d) $O(n^2)$

10. The naive recursive Fibonacci is which complexity class?

    (a) $O(n)$ (b) $O(n \log n)$ (c) $O(n^2)$ (d) $O(2^n)$

11. An array reversal done in place uses what auxiliary space?

    (a) $O(1)$ (b) $O(\log n)$ (c) $O(n)$ (d) $O(n^2)$

12. If Algorithm A is $O(n)$ with constant 100, and B is $O(n^2)$ with constant 0.1, which is **faster for small $n$**?

    (a) A always (b) B for small $n$, A for large $n$ (c) A always for $n \ge 1$ (d) they are equal for all $n$

### B. Fill in the blank

1. A ______ data type (ADT) specifies a set of values and operations, independent of any implementation.

2. The Big-O definition requires finding two constants: $c$ and ______.

3. The recurrence for **binary search** is $T(n) = T(n/2) + O(1)$, which solves to $T(n) = O(\underline{\ \ \ \ })$.

4. Two nested loops, one counting to $n$ and one counting to $\log n$, give $O(\underline{\ \ \ \ })$.

5. In Big-O notation, $5n^2 + 100n + 7 = O(\underline{\ \ \ \ })$.

6. Big-$\Theta$ means both Big-O **and** Big-______ hold.

7. The naive recursive Fibonacci algorithm is exponential because each call ______ the same subproblems.

8. The auxiliary space used by a recursion of depth $d$ is $O(\underline{\ \ \ \ })$.

9. An algorithm that halves the input on each recursive call runs in $O(\underline{\ \ \ \ })$.

10. An algorithm that doubles a counter each iteration runs in $O(\underline{\ \ \ \ })$.

### C. Define / explain

1. **Define** a data structure. Why must the definition include operations, not just layout?
2. **Distinguish** an ADT from an implementation. Give one ADT and two different implementations, and explain when each is preferred.
3. **State** the formal definition of Big-O. Use it to **prove** that $4n^2 + 3n + 1 = O(n^2)$.
4. **Distinguish** Big-O, Big-$\Omega$, and Big-$\Theta$. Give one example function for which $f = O(g)$, $f = \Omega(g)$, **and** $f = \Theta(g)$ all hold.
5. **Analyse** this code and give its Big-O:

   ```java
   for (int i = 0; i < n; i++) {
       for (int j = i; j < n; j++) {
           for (int k = 0; k < 10; k++) {
               sum += a[i] * a[j];
           }
       }
   }
   ```

6. **Write** the recurrence for the following function and solve it:

   ```java
   int f(int n) {
       if (n <= 1) return 1;
       return f(n/2) + f(n/2) + n;
   }
   ```

7. **Explain** why Big-O ignores constants, and why constants still matter in practice. Give one concrete example where the "slower" algorithm is actually faster.
8. **Compare** the **time** and **auxiliary space** complexity of computing the $n$-th Fibonacci number via recursion versus iteration.

---

## 1.13 Answer key

??? success "A. Multiple choice"
    1. **(b)** — $f$ grows no faster than $g$, up to a constant.
    2. **(d)** — $n^2 = O(n)$ is false; the quadratic term dominates.
    3. **(c)** — $O(n)$.
    4. **(b)** — $O(\log n)$.
    5. **(c)** — $O(n^2)$.
    6. **(b)** — $O(n \log n)$.
    7. **(c)** — $O(n)$.
    8. **(c)** — a tight bound (upper **and** lower).
    9. **(b)** — $\log n$ is the slowest-growing.
    10. **(d)** — $O(2^n)$.
    11. **(a)** — $O(1)$.
    12. **(b)** — B for small $n$, A for large $n$ (crossover around $n = 1000$).

??? success "B. Fill in the blank"
    1. abstract
    2. $n_0$
    3. $\log n$
    4. $n \log n$
    5. $n^2$
    6. $\Omega$
    7. recomputes (repeats)
    8. $d$
    9. $\log n$
    10. $\log n$

??? success "C. Define / explain — model answers"
    1. *Data structure:* a way of organizing data in memory **together with** the operations on that data. *Why include operations:* layout alone does not determine what you can do cheaply. Two layouts with the same data may have wildly different costs for `insert`, `search`, `delete`. A data structure is defined by what it *enables*, not just where the bits go.
    2. *ADT vs. implementation:* ADT = the *interface*, specification of operations independent of representation. Implementation = the *concrete realization*, with specific memory layout and operation costs. *Example:* the **List** ADT (insert, delete, get, size). *Implementation 1:* array-backed — O(1) `get(i)`, O(n) `insert(0, x)`. *Implementation 2:* linked list — O(n) `get(i)`, O(1) `insert(0, x)`. *Preferred:* array when reads dominate and index access is frequent (e.g., storing a matrix); linked list when insertions/deletions at arbitrary positions dominate (e.g., a text editor's cursor).
    3. *Definition:* $f(n) = O(g(n))$ if $\exists c, n_0 > 0$ such that $f(n) \le c \cdot g(n)$ for all $n \ge n_0$. *Proof of $4n^2 + 3n + 1 = O(n^2)$:* Need $c, n_0$ with $4n^2 + 3n + 1 \le c n^2$ for all $n \ge n_0$. Take $c = 8$: $4n^2 + 3n + 1 \le 8n^2$ ⟺ $3n + 1 \le 4n^2$ ⟺ $4n^2 - 3n - 1 \ge 0$ ⟺ $(4n+1)(n-1) \ge 0$, which holds for $n \ge 1$. Witness: $c = 8$, $n_0 = 1$.
    4. *Definitions:* $O$ = upper bound, $\Omega$ = lower bound, $\Theta$ = both. *Example:* $f(n) = 3n$ and $g(n) = n$. *Then:* $3n = O(n)$ (take $c=3$, $n_0=1$); $3n = \Omega(n)$ (take $c=3$, $n_0=1$); $3n = \Theta(n)$ (both hold).
    5. *Analysis:* The outermost loop runs $n$ times. For each outer iteration, the inner runs $n - i$ times, giving the triangular sum $\sum_i (n-i) = \frac{n(n+1)}{2} = O(n^2)$. The innermost loop runs a **constant** 10 times — a constant factor does not change the class. **Total: $O(n^2)$.** (Precisely $10 \cdot n(n+1)/2$ — still $O(n^2)$.)
    6. *Recurrence:* $T(n) = 2\,T(n/2) + O(n)$. This is the merge-sort recurrence. Recursion-tree solution: at depth $d$, there are $2^d$ nodes each with size $n/2^d$, work per level $O(n)$, depth $\log_2 n$. Total: **$O(n \log n)$**.
    7. *Why Big-O ignores constants:* it is a statement about *growth*, not absolute time. For comparing algorithms on *large* inputs, constants are dominated by the growth rate. *Why constants still matter:* (i) for *small* inputs, the constant dominates — insertion sort with tiny constant beats merge sort up to $n \approx 20$; (ii) cache behavior — a cache-friendly O(n²) can beat a cache-thrashing O(n log n); (iii) memory constraints — an O(n) algorithm using $O(n)$ extra space may be unusable on an embedded device, whereas an O(n²) algorithm with $O(1)$ space may be preferred. *Concrete example:* Python's `sorted()` uses Timsort ($O(n \log n)$, small constant) but switches to insertion sort ($O(n^2)$) inside sorted blocks of ≤ 64 elements — because the smaller constant wins for small arrays.
    8. *Time and space comparison — Fibonacci:*
       - *Recursive:* Time $O(2^n)$ (recomputes subproblems exponentially); auxiliary space $O(n)$ (depth of recursion). For $n = 50$: time ≈ hours, stack depth 50.
       - *Iterative:* Time $O(n)$ (single loop); auxiliary space $O(1)$ (two variables). For $n = 50$: time ≈ microseconds, memory ≈ 8 bytes.
       **Iterative wins overwhelmingly on both time and space.**

---

## 1.14 Further reading

- **Weiss**, *Data Structures and Algorithm Analysis*, Chapters 1–2 — the standard reference for Big-O in a data-structures context.
- **Lafore**, *Data Structures & Algorithms in Java*, Chapters 1–2 — gentler introduction with working code.
- **CLRS**, Chapter 3 (growth of functions) — the formal definition of $O$, $\Omega$, $\Theta$, and the Master Theorem.
- **Sedgewick & Wayne**, *Algorithms*, Section 1.4 (analysis of algorithms) — clear explanation of the doubling method.
- **Open Data Structures** (free online) Chapter 1 — complexity primer with code in multiple languages.
- **VisuAlgo** — [visualgo.net](https://visualgo.net/) — animate any algorithm from this chapter to build intuition.
- **MIT 6.006** lecture 1 (Big-O) — [OCW](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/) — first 50 minutes are the classic introduction.
- **Python Tutor** — [pythontutor.com](https://pythontutor.com/) — step through the recursion examples above and watch the stack grow.

---

## 1.15 Suggested tutorial problems

1. **Prove Big-O statements.** Prove each:
   - $7n^2 + 4n + 2 = O(n^2)$
   - $3n + \log n = O(n)$
   - $100\log n + 5 = O(\log n)$

2. **Disprove Big-O statements.** Show that:
   - $n^2 \ne O(n)$
   - $n \log n \ne O(n)$
   - $2^n \ne O(n^{100})$

3. **Analyse these loops** — give the Big-O of each:
   - `for (i = 0; i < n; i++) for (j = 0; j < i; j++) { ... }`
   - `for (i = 1; i < n; i *= 3) { ... }`
   - `for (i = 0; i < n; i++) for (j = 0; j < n; j++) for (k = 0; k < n; k++) { ... }`
   - `for (i = 0; i < 100; i++) for (j = 0; j < n; j++) { ... }`

4. **Write a recurrence** for each function and solve it:

   ```java
   int f(int n) {
       if (n == 0) return 0;
       return f(n - 1) + 1;
   }
   ```

   ```java
   int g(int n) {
       if (n == 0) return 0;
       return g(n - 1) + g(n - 1);
   }
   ```

   ```java
   int h(int n) {
       if (n <= 1) return 1;
       return h(n/2) + h(n/2);
   }
   ```

5. **Prove or disprove:** if $f(n) = \Theta(g(n))$ and $g(n) = \Theta(h(n))$, then $f(n) = \Theta(h(n))$.

6. **Non-obvious case.** Give an example of two functions $f$ and $g$ for which $f = O(g)$ but $f \ne \Theta(g)$.

7. **Same function, three bounds.** For $f(n) = 10n^3 + 100n^2 + 1000n + 10000$, give witnesses $(c, n_0)$ for all three: $f = O(n^3)$, $f = \Omega(n^3)$, $f = \Theta(n^3)$.

8. **Space matters.** An algorithm runs in $O(n)$ time but uses $O(n)$ auxiliary space. Can you always replace it with an $O(1)$-space version? Give an example where you can, and one where you can't.

9. **The crossover.** Algorithm A is $5n + 10$ (per step cost $1$ ns); B is $0.001n^2$ (per step cost $0.001$ ns). Find the $n$ at which B becomes slower than A.

10. **Practical comparison.** On your laptop, time the naive recursive Fibonacci vs. the iterative version for $n = 20, 25, 30, 35$. Plot the ratio of times. What do you see? What does Big-O predict?