# Solutions to Chapter 4 — Stacks & Queues

[:octicons-arrow-left-24: Back to Chapter 4](../chapters/04-stacks-queues.md)

---

## 1. Stack trace — push 1,2,3; pop ×2; push 4,5; pop ×1

| Operation | Stack (top on right) |
|---|---|
| push 1 | [1] |
| push 2 | [1, 2] |
| push 3 | [1, 2, 3] |
| pop → 3 | [1, 2] |
| pop → 2 | [1] |
| push 4 | [1, 4] |
| push 5 | [1, 4, 5] |
| pop → 5 | [1, 4] |

**Final top = 4.**

---

## 2. Postfix evaluation — `"4 5 + 6 2 - * 3 +"`

| Token | Action | Stack |
|:---:|---|---|
| 4 | push | [4] |
| 5 | push | [4, 5] |
| + | pop 5, 4; push 9 | [9] |
| 6 | push | [9, 6] |
| 2 | push | [9, 6, 2] |
| − | pop 2, 6; push 4 | [9, 4] |
| * | pop 4, 9; push 36 | [36] |
| 3 | push | [36, 3] |
| + | pop 3, 36; push 39 | [39] |

**Result: 39** ✓

Check: `(4+5) × (6−2) + 3 = 9 × 4 + 3 = 39`. ✓

---

## 3. Balanced parentheses

| Input | Balanced? | Why |
|---|:---:|---|
| `"(()"` | ✗ | Stack not empty at end (2 opens, 1 close) |
| `"()[]{}"` | ✓ | All matched |
| `"([{}])"` | ✓ | Nested correctly |
| `"([)]"` | ✗ | Mismatch: `)` pops `[` |

---

## 4. Infix → postfix for `"(A + B) * (C - D)"`

| Token | Output | Operator stack |
|:---:|---|---|
| `(` | — | `(` |
| `A` | A | `(` |
| `+` | A | `( +` |
| `B` | A B | `( +` |
| `)` | A B + | — (pop `+`, then `(`) |
| `*` | A B + | `*` |
| `(` | A B + | `* (` |
| `C` | A B + C | `* (` |
| `-` | A B + C | `* ( -` |
| `D` | A B + C D | `* ( -` |
| `)` | A B + C D - | `*` (pop `-`, `(`) |
| end | A B + C D - * | — |

**Postfix: `A B + C D - *`** ✓

---

## 5. Circular queue trace — capacity 4

| Operation | data [0..3] | front | size |
|:---:|---|:---:|:---:|
| enqueue 1 | [1,_,_,_] | 0 | 1 |
| enqueue 2 | [1,2,_,_] | 0 | 2 |
| enqueue 3 | [1,2,3,_] | 0 | 3 |
| dequeue → 1 | [_,2,3,_] | 1 | 2 |
| enqueue 4 | [_,2,3,4] | 1 | 3 |
| enqueue 5 | [5,2,3,4] (wrap!) | 1 | 4 |
| dequeue → 2 | [5,_,3,4] | 2 | 3 |
| enqueue 6 | [5,6,3,4] (wrap!) | 2 | 4 |

**Wrap points:** enqueue 5 wraps to index 0; enqueue 6 wraps to index 1.

---

## 6. Two-stack infix — `"10 + 20 * (30 - 15)"`

| Token | Num stack | Op stack |
|:---:|---|---|
| 10 | [10] | — |
| + | [10] | `+` |
| 20 | [10, 20] | `+` |
| * | [10, 20] | `+ *` |
| ( | [10, 20] | `+ * (` |
| 30 | [10, 20, 30] | `+ * (` |
| − | [10, 20, 30] | `+ * ( −` |
| 15 | [10, 20, 30, 15] | `+ * ( −` |
| ) | [10, 20, 15] | `+ *` (apply −, pop `(`) |
| end: apply * | [10, 300] | `+` |
| end: apply + | [310] | — |

**Result: 310** ✓

Check: `10 + 20 × (30 − 15) = 10 + 20 × 15 = 310`. ✓

---

## 7. Undo/redo

Start: text = "", undoStack = [], redoStack = [].

| Action | text | undoStack | redoStack |
|---|---|:---:|:---:|
| type "abc" | "abc" | [""] | [] |
| undo | "" | [] | ["abc"] |
| undo | "" (no-op) | [] | ["abc"] |
| redo | "abc" | [""] | [] |
| type "d" | "abcd" | ["", "abc"] | [] (cleared) |

---

## 8. Palindrome with deque — `"racecar"`

- Remove first `'r'`, last `'r'` → equal. Remaining: `"aceca"`.
- Remove `'a'`, `'a'` → equal. `"cec"`.
- Remove `'c'`, `'c'` → equal. `"e"`.
- Single element → done.

**Palindrome** ✓. **O(n) time, O(n) space** (deque holds up to n).

---

## 9. Sliding window maximum — `[1,3,-1,-3,5,3,6,7]`, k=3

Monotonic deque (decreasing order):

| Window | Deque (indices) | Max |
|:---:|---|:---:|
| [1,3,-1] | [3, -1] (indices 1, 2) | 3 |
| [3,-1,-3] | [3, -1, -3] (1,2,3) | 3 |
| [-1,-3,5] | [5] (index 4) | 5 |
| [-3,5,3] | [5, 3] (4,5) | 5 |
| [5,3,6] | [6] (index 6) | 6 |
| [3,6,7] | [7] (index 7) | 7 |

**Output: [3, 3, 5, 5, 6, 7]** ✓

---

## 10. Recursion depth — count-down from n

At `n = 100,000`: **crashes** with `StackOverflowError` — call stack has a fixed size (~1 MB); each frame is ~100 bytes; 10⁵ frames × 100 bytes = 10 MB > 1 MB.

**Iterative rewrite:**

```java
void countDown(int n) {
    while (n >= 0) { System.out.println(n); n--; }
}
```

**Memory:** O(1) — a single loop. **Speed:** microseconds vs. crash.

---

## 11. Round-robin scheduler

Processes P1 (5ms), P2 (3ms), P3 (8ms). Quantum 2ms.

| Turn | Process | Remaining |
|:---:|---|---|
| 1 | P1 (2ms) | P1=3 |
| 2 | P2 (2ms) | P2=1 |
| 3 | P3 (2ms) | P3=6 |
| 4 | P1 (2ms) | P1=1 |
| 5 | P2 (1ms → done) | P2=0 |
| 6 | P3 (2ms) | P3=4 |
| 7 | P1 (1ms → done) | P1=0 |
| 8 | P3 (2ms) | P3=2 |
| 9 | P3 (2ms → done) | P3=0 |

**Completion order:** P2, P1, P3.

---

## 12. Array stack vs. linked-list stack for 10⁶ push + 10⁶ pop

**Array stack** preferred — amortised O(1), excellent cache locality (all `push`es touch the same cache line region), ~2-3 ns per operation.

**Linked-list stack** — O(1) worst case but each push is a heap allocation (~20-30 ns), each pop deallocates. Cache misses per operation.

**Winner: array stack** for typical workloads.

---

## 13. BFS on a tree

Root=1, children 2,3; grandchildren 4,5 (from 2), 6,7 (from 3).

**BFS order:** 1, 2, 3, 4, 5, 6, 7 — **level by level**. ✓

---

## 14. Queue from two stacks

Two stacks: `in_stack` (enqueue target) and `out_stack` (dequeue target).

- **enqueue(x):** push onto `in_stack`. O(1).
- **dequeue():** if `out_stack` is empty, pop all from `in_stack` and push onto `out_stack`; then pop from `out_stack`.

**Amortised cost:** each element is pushed to `in_stack` once, transferred to `out_stack` once, popped from `out_stack` once. Total 3 pushes/pops per element = **O(1) amortised**.

---

## 15. Wildcard valid parentheses

`(` `)` `*` (wildcard = `(`, `)`, or empty).

**Greedy (two-pass) approach:**
1. Left-to-right: treat `*` as `(` and `)`. Track `low` (min unmatched opens) and `high` (max unmatched opens). If `high < 0`, return false.
2. After the pass, if `low == 0`, return true.

**O(n) time, O(1) space.**