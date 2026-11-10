# Exam Bank — ECEG-4192 Data Structures

40 questions spanning all chapters. **3 hours. Closed book; one A4 handwritten
formula sheet. 100 marks.** Answer key and grading rubric at the end.

---

## Section A — Introduction & Big-O (10 marks)

**A1. Multiple choice [2].** Which of these is a valid Big-O simplification?

(a) $n^2 + n = O(n)$ (b) $5n + 3 = O(n)$ (c) $n \log n = O(n)$ (d) $n^2 = O(n)$

**A2. Multiple choice [2].** The recurrence $T(n) = 2T(n/2) + O(n)$ solves to:

(a) $O(n)$ (b) $O(n \log n)$ (c) $O(n^2)$ (d) $O(2^n)$

**A3. Short answer [3].** State the formal definition of Big-O. Use it to prove that $4n^2 + 3n = O(n^2)$ with explicit witnesses $(c, n_0)$.

**A4. Derivation [3].** A loop has structure: `for (i = 1; i < n; i *= 3) { for (j = 0; j < i; j++) { ... } }`. Derive the total operation count as a function of $n$ and state its Big-O.

---

## Section B — Arrays (10 marks)

**B1. Multiple choice [2].** Inserting an element at position 0 of an n-element array costs:

(a) O(1) (b) O(log n) (c) O(n) (d) O(n²)

**B2. Multiple choice [2].** Adding $n$ elements to a dynamic array with **doubling** has total cost:

(a) O(n) (b) O(n log n) (c) O(n²) (d) O(2ⁿ)

**B3. Short answer [3].** Prove that `a[i]` on an array takes O(1) time. Write the address arithmetic and count the primitive operations.

**B4. Derivation [3].** An array of size $n$ is rotated by $k$ positions in place using the reverse-three-times trick. Show the three reversals restore the array, and give the time and space complexity.

---

## Section C — Linked Lists (10 marks)

**C1. Multiple choice [2].** Deleting a given node in a **doubly-linked list** (given a pointer to it) costs:

(a) O(1) (b) O(log n) (c) O(n) (d) O(n²)

**C2. Multiple choice [2].** Floyd's cycle-detection algorithm uses:

(a) hash set (b) stack (c) two pointers at different speeds (d) recursion

**C3. Short answer [3].** Draw the state after each step of inserting `10, 20, 30` at the **head** of a singly-linked list. Final list order?

**C4. Derivation [3].** Write the recurrence for the reverse of a singly-linked list (iterative version) and justify that it runs in O(n) time and O(1) space.

---

## Section D — Stacks & Queues (12 marks)

**D1. Multiple choice [2].** The **circular queue** wrap-around exists to avoid:

(a) overflow (b) O(n) shifts on dequeue (c) null pointers (d) recursion

**D2. Multiple choice [2].** Postfix expression `"5 3 2 * +"` evaluates to:

(a) 16 (b) 11 (c) 10 (d) 25

**D3. Short answer [4].** Evaluate `"4 5 + 6 2 - * 3 +"` using a stack. Show the stack after each token.

**D4. Derivation [4].** Implement a FIFO queue using two LIFO stacks. State the amortised cost of enqueue and dequeue, with proof.

---

## Section E — Trees (20 marks)

**E1. Multiple choice [2].** In-order traversal of a BST gives:

(a) pre-order (b) post-order (c) **sorted order** (d) reverse-sorted order

**E2. Multiple choice [2].** AVL trees guarantee what height bound?

(a) O(1) (b) **O(log n)** (c) O(n) (d) O(n²)

**E3. Short answer [3].** Insert `10, 20, 30` into an empty BST. Draw the result. What is its height? Compare with the height of an AVL tree built from the same keys.

**E4. Short answer [3].** Delete node 50 (root) from the tree below. Which case does this fall into? Show the tree after deletion.

```
      50
     /  \
    30   70
   / \   / \
  20 40 60 80
```

**E5. Derivation [5].** Build a min-heap from `[7, 3, 9, 1, 8, 2, 6]` using **bottom-up heapify**. Show the array after each `siftDown` call. Then extract the minimum twice, showing the array after each.

**E6. Derivation [5].** Trace the AVL insertion of `1, 2, 3, 4, 5` in order. After each insertion, state whether a rotation was needed, and if so which type (LL / RR / LR / RL) and show the tree before and after.

---

## Section F — Hash Tables (10 marks)

**F1. Multiple choice [2].** Separate chaining achieves average-case lookup:

(a) O(1) (b) O(log n) (c) O(n) (d) O(n²)

**F2. Multiple choice [2].** Which clustering problem does **linear probing** suffer from?

(a) primary clustering (b) secondary clustering (c) tombstone accumulation (d) integer overflow

**F3. Short answer [3].** Insert `{10, 20, 30, 40}` into a hash table of size `m = 7` using **linear probing**. Use `h(k) = k mod 7`. Show the table after each insertion. What is the maximum probe count?

**F4. Derivation [3].** Explain the **tombstone problem** in open addressing. Give a concrete example (small table) where setting deleted slots to `EMPTY` breaks lookups.

---

## Section G — Graphs (18 marks)

**G1. Multiple choice [2].** Adjacency list memory is:

(a) O(n) (b) **O(n + m)** (c) O(n²) (d) O(2ⁿ)

**G2. Multiple choice [2].** **Dijkstra's algorithm** requires:

(a) non-negative weights (b) at least one negative edge (c) symmetric graph (d) directed graph

**G3. Short answer [3].** Run **BFS** on the following undirected graph from vertex 0. Give the visit order and the distance to each vertex.

```
0 — 1 — 3
|   |   |
2 — 4 — 5
```

**G4. Short answer [3].** For the weighted graph below, run **Dijkstra's algorithm** from vertex 0. Show the distance array after each extraction. Edges: `0→1 (4), 0→2 (1), 1→3 (1), 2→1 (3), 2→3 (5), 3→4 (2), 2→4 (6)`.

**G5. Derivation [4].** Construct a graph with **negative weights** where Dijkstra's algorithm returns the wrong answer but **Bellman–Ford** returns the correct one. Show both algorithms' traces on the graph.

**G6. Derivation [4].** Run **Kruskal's algorithm** on the following 6-vertex graph. Sort edges, process in order, and show the MST.

| Edge | Weight |
|---|:---:|
| (0,1) | 6 |
| (0,2) | 5 |
| (1,2) | 1 |
| (1,3) | 5 |
| (2,3) | 5 |
| (2,4) | 6 |
| (3,4) | 2 |
| (3,5) | 4 |
| (4,5) | 6 |

---

## Section H — Sorting (10 marks)

**H1. Multiple choice [2].** Which sort is **stable** and in-place?

(a) Merge sort (b) Quicksort (c) **Insertion sort** (d) Heapsort

**H2. Multiple choice [2].** Quicksort's worst-case complexity is:

(a) O(n log n) (b) O(n) (c) **O(n²)** (d) O(2ⁿ)

**H3. Short answer [3].** Trace **insertion sort** on `[4, 1, 3, 2]`. Show the array after each outer iteration. Count the total shifts.

**H4. Derivation [3].** Explain why **merge sort** uses O(n) auxiliary space. Give one scenario where **heapsort** is preferred over merge sort despite both being O(n log n).

---

## Answer key

??? success "Section A"

    **A1.** (b) — $5n + 3 = O(n)$. All others fail.

    **A2.** (b) — $O(n \log n)$ (merge-sort recurrence). Recursion tree: depth $\log n$, work per level $O(n)$.

    **A3.** *Definition:* $f(n) = O(g(n))$ if $\exists c, n_0 > 0$ with $f(n) \le c \cdot g(n)$ for all $n \ge n_0$. *Proof:* need $4n^2 + 3n \le c n^2$. Try $c = 7$: $4n^2 + 3n \le 7n^2$ ⟺ $3n \le 3n^2$ ⟺ $n \ge 1$. Witness: $c = 7$, $n_0 = 1$.

    **A4.** Outer loop iterations: $i = 1, 3, 9, 27, \ldots$ — stop when $i \ge n$, i.e., $\log_3 n$ iterations. Inner loop iteration count per outer iteration equals $i$. Total operations: $\sum_{i \in \{1, 3, 9, \ldots\}} i = 1 + 3 + 9 + \ldots + 3^{\log_3 n} = \frac{3^{\log_3 n + 1} - 1}{2} = \Theta(n)$. **Big-O: $O(n)$.**

??? success "Section B"

    **B1.** (c) — O(n) shifting all elements right.

    **B2.** (a) — O(n) total (geometric series sum: $1 + 2 + 4 + \ldots + 2^{k} < 2n$).

    **B3.** Address of `a[i]` = base + `i` × width. Computation: one multiplication, one addition, one memory load = **3 primitive instructions**, independent of `i` and `n`. **O(1).**

    **B4.** *Correctness:* Let array = $A[0..n-1]$. Rotation by $k$ = permute to $A[n-k..n-1]$ followed by $A[0..n-k-1]$. Let $U = A[0..n-k-1]$, $V = A[n-k..n-1]$; result = $VU$. Reverse 1: $A \to (UV)^R = V^R U^R$. Reverse 2: reverse first $k$ elements (which is $V^R$) $\to V$. Now $V U^R$. Reverse 3: reverse $U^R$ portion $\to U$. Result $VU$. ✓ *Cost:* three O(n) reversals = **O(n) time, O(1) space.**

??? success "Section C"

    **C1.** (a) — O(1) using the `prev` pointer.

    **C2.** (c) — two pointers at different speeds (slow 1 step, fast 2 steps per iteration).

    **C3.** Insert 10: `10 → null`. Insert 20: `20 → 10 → null`. Insert 30: `30 → 20 → 10 → null`. **Final: `30 → 20 → 10`** — reversed relative to insertion.

    **C4.** *Recurrence:* one pass, each node visited once → **T(n) = T(n-1) + O(1)**, solving to **O(n)**. *Space:* only `prev`, `cur`, `next` variables — constant per iteration → **O(1) auxiliary**.

??? success "Section D"

    **D1.** (b) — O(n) shifts on dequeue (or O(n²) total).

    **D2.** (b) — 11. Trace: 5, 3, 2 → push. `*` → pop 2, 3, push 6. `+` → pop 6, 5, push 11.

    **D3.** Trace:

    | Token | Action | Stack |
    |:---:|---|---|
    | 4 | push | [4] |
    | 5 | push | [4, 5] |
    | + | pop 5,4; push 9 | [9] |
    | 6 | push | [9, 6] |
    | 2 | push | [9, 6, 2] |
    | − | pop 2,6; push 4 | [9, 4] |
    | * | pop 4,9; push 36 | [36] |
    | 3 | push | [36, 3] |
    | + | pop 3,36; push 39 | [39] |

    **Result: 39.**

    **D4.** Two stacks: `in_stack` (enqueue), `out_stack` (dequeue). **Enqueue:** push to `in_stack`. **Dequeue:** if `out_stack` empty, pop all from `in_stack` onto `out_stack`; pop top of `out_stack`. **Amortised:** each element pushed once to `in_stack`, moved once to `out_stack`, popped once — **3 operations total → O(1) amortised.** Worst case single dequeue is O(n) if `out_stack` empty.

??? success "Section E"

    **E1.** (c) — sorted order.

    **E2.** (b) — O(log n) (balance factor invariant guarantees height ≤ 1.44 log₂ n).

    **E3.** *BST:* `10 \ 20 \ 30` (right-skewed). Height = **2** (edge count) or 3 (node count). *AVL:* after 30, RR rotation at 10 → `20 / 10 \ 30`. Height = **1** (edge count) or 2. Both trees have the same 3 keys — different shapes.

    **E4.** **Case 3 (two children).** In-order successor of 50 is 60 (smallest in right subtree = leftmost of 70's subtree = 60). Replace 50 with 60, delete 60 from its leaf.

    ```
          60
         /  \
        30   70
       / \    \
      20 40   80
    ```

    **E5.** Starting array: `[7, 3, 9, 1, 8, 2, 6]`. `n = 7`, last internal = `7/2 − 1 = 2`.

    - `i=2` (value 9): children 8 (idx 5), 6 (idx 6). Min = 6. 9>6 → swap. `[7, 3, 6, 1, 8, 2, 9]`.
    - `i=1` (value 3): children 1, 8. Min = 1. 3>1 → swap. `[7, 1, 6, 3, 8, 2, 9]`. At idx 3 (value 3): no children.
    - `i=0` (value 7): children 1, 6. Min = 1. 7>1 → swap. `[1, 7, 6, 3, 8, 2, 9]`. At idx 1 (value 7): children 3, 8. Min = 3. 7>3 → swap. `[1, 3, 6, 7, 8, 2, 9]`. At idx 3 (value 7): no children.

    **Heap:** `[1, 3, 6, 7, 8, 2, 9]`.

    *Extract 1 (min):* save 1, move last (9) to root: `[9, 3, 6, 7, 8, 2]`. Sift-down: children 3, 6. Min 3 (idx 1). 9>3 swap → `[3, 9, 6, 7, 8, 2]`. At idx 1 (9): children 7, 8. Min 7 (idx 3). 9>7 swap → `[3, 7, 6, 9, 8, 2]`. At idx 3 (9): no children. **Extracted: 1. Heap: [3, 7, 6, 9, 8, 2].**

    *Extract 3 (min):* save 3, move last (2) to root: `[2, 7, 6, 9, 8]`. Sift-down: children 7, 6. Min 6 (idx 2). 2<6 no swap. **Extracted: 3. Heap: [2, 7, 6, 9, 8].**

    **E6.** AVL insertion trace for `1, 2, 3, 4, 5`:

    | Insert | Tree | Rotation? |
    |:---:|---|---|
    | 1 | `1` | No |
    | 2 | `1 \ 2` | No |
    | 3 | `1 \ 2 \ 3` | **RR at 1** → `2 / 1 \ 3` |
    | 4 | `2 / 1 \ 3 \ 4` | No |
    | 5 | `2 / 1 \ 3 \ 4 \ 5` | **RR at 3** → `2 / 1 \ 4 / 3 \ 5` |

    All trees balanced after rotations. ✓

??? success "Section F"

    **F1.** (a) — O(1) average (O(1 + α) where α is load factor).

    **F2.** (a) — primary clustering (long consecutive runs of occupied slots grow faster).

    **F3.** Hash values: 10 mod 7 = 3, 20 mod 7 = 6, 30 mod 7 = 2, 40 mod 7 = 5. **All distinct** — no collisions. Table: `[_, _, 30, 10, _, 40, 20]` (indices 0..6). Max probes: **1**.

    (If asked "insert 10, 20, 30, 40, 50" with 50 mod 7 = 1 — still distinct. With 40 mod 7 = 5, and 45 mod 7 = 3 → collides with 10 at 3 → probe 4. Max probes: **2**.)

    **F4.** *Tombstone problem:* suppose keys 10, 20, 30 all hash to slot 0 (linear probing). Table: `[10, 20, 30, _, _]`. Delete 20: setting slot 1 to EMPTY. Now lookup 30: hash to 0 → slot 0 (10, no match) → slot 1 (EMPTY → **stop, not found**). But 30 is at slot 2 — the search terminated too early. **Fix:** mark deleted slots as DELETED (tombstone), not EMPTY. Lookup continues past DELETED; only stops on EMPTY.

??? success "Section G"

    **G1.** (b) — O(n + m).

    **G2.** (a) — non-negative weights required.

    **G3.** *BFS from 0:*

    | Step | Queue | Visited |
    |:---:|---|---|
    | 0 | [0] | {0} |
    | 1 | [1, 2] | {0,1,2} |
    | 2 | [2, 3, 4] | + {3,4} |
    | 3 | [3, 4] | (2 has no new) |
    | 4 | [4, 5] | + {5} |
    | 5 | [5] | — |
    | 6 | [] | done |

    **Visit order:** 0, 1, 2, 3, 4, 5. **Distances:** [0, 1, 1, 2, 2, 3].

    **G4.** Dijkstra from 0:

    Init: `dist = [0, ∞, ∞, ∞, ∞]`.

    | Pop | Relaxations | dist |
    |:---:|---|---|
    | (0, 0) | 0→1: 4, 0→2: 1 | [0, 4, 1, ∞, ∞] |
    | (2, 1) | 2→1: 1+3=4 (no change), 2→3: 1+5=6, 2→4: 1+6=7 | [0, 4, 1, 6, 7] |
    | (1, 4) | 1→3: 4+1=5 < 6 → update | [0, 4, 1, 5, 7] |
    | (3, 5) | 3→4: 5+2=7 (no change) | [0, 4, 1, 5, 7] |
    | (4, 7) | none | done |

    **Final: [0, 4, 1, 5, 7].**

    **G5.** Graph: edges `s→u (1), s→v (2), u→v (-5), u→w (10)`.

    **Dijkstra from s:**
    - Pop s(0), relax: u←1, v←2.
    - Pop u(1), relax u→v: 1+(-5) = -4 < 2 → v←-4. u→w: 11.
    - Pop v(-4), no outgoing better.
    - Pop w(10). 

    Actually, to make Dijkstra *fail*, we need it to settle a node before a shorter path is discovered. Classic example: `s→u (1), s→v (2), u→v (-5)`. Dijkstra pops u at 1, then v at -4 (via u), still correct here.

    Failure case: `s→u (5), s→v (3), u→v (-10), u→w (1), v→w (1)`.
    - Pop s(0), relax: u←5, v←3.
    - Pop v(3), relax v→w: 3+1 = 4. w←4.
    - **Pop w(4)** — settled at 4.
    - Pop u(5), relax u→v: 5 + (-10) = -5 < 3 → v←-5. u→w: 5+1 = 6 > 4, no change.
    - Pop v(-5).
    - Dijkstra returns w = **4**.
    - **Bellman–Ford:** correct answer is w = 5 + (-10) + 1 = **-4** (path s→u→v→w). Bellman–Ford gets it right because it relaxes all edges V-1 times.

    **G6.** Kruskal trace:

    Sorted edges: (1,2,1), (3,4,2), (3,5,4), (0,2,5), (1,3,5), (2,3,5), (0,1,6), (2,4,6), (4,5,6).

    | Edge | Add? | MST |
    |:---:|:---:|---|
    | (1,2,1) | ✓ | {(1,2)} |
    | (3,4,2) | ✓ | {(1,2), (3,4)} |
    | (3,5,4) | ✓ | {(1,2), (3,4), (3,5)} |
    | (0,2,5) | ✓ | {(1,2), (3,4), (3,5), (0,2)} |
    | (1,3,5) | ✓ (merges two components) | {(1,2), (3,4), (3,5), (0,2), (1,3)} |
    | (2,3,5) | ✗ (cycle) | — |
    | (0,1,6) | ✗ (cycle) | — |

    **MST edges: {(1,2), (3,4), (3,5), (0,2), (1,3)}, total weight = 17.** ✓

??? success "Section H"

    **H1.** (c) — insertion sort is stable and in-place.

    **H2.** (c) — O(n²).

    **H3.** Trace `[4, 1, 3, 2]`:

    | i | key | shifts | Array |
    |:---:|:---:|:---:|---|
    | 1 | 1 | 1 (4) | [1, 4, 3, 2] |
    | 2 | 3 | 1 (4) | [1, 3, 4, 2] |
    | 3 | 2 | 2 (4, 3) | [1, 2, 3, 4] |

    **Total shifts: 4.**

    **H4.** *Merge sort space:* the merge step needs a temporary array of size $n$ to hold the merged result before copying back. Hence **O(n) auxiliary**. *Heapsort preferred when:* O(1) auxiliary space is required (embedded systems, memory-constrained environments) — heapsort is in-place, merge sort is not. Also: heapsort has O(n log n) **worst case** while quicksort can degrade.

---

## Grading rubric

| Band | Marks | Criteria |
|---|---|---|
| **A** | 90–100 | All correct or with trivial arithmetic slips; traces and derivations rigorous |
| **B** | 75–89 | Most correct; occasional conceptual gaps |
| **C** | 60–74 | Basic understanding; some errors |
| **F** | < 60 | Major gaps |

**Common error checks (deductions):**

- **Wrong Big-O** (dropping leading term or keeping constant): −1 to −2.
- **Confusing BST / heap** ordering: −2.
- **Missing balance factor check** in AVL rotation: −1.
- **Wrong heap array index** (`(i-1)/2` vs. `i/2`): −1.
- **Omitting tombstone in open-addressing delete**: −2.
- **Dijkstra with negative edge**: −2 to −3.
- **Forgetting MST has V−1 edges**: −1.
- **Claiming quicksort stable**: −1.
- **Not showing trace steps**: −2 to −5 (partial credit depends on correct final answer alone).