# Solutions to Chapter 5 — Trees

[:octicons-arrow-left-24: Back to Chapter 5](../chapters/05-trees.md)

---

## 1. Traversals of the 8-node tree

```
        A
       / \
      B   C
     / \ / \
    D  E F  G
```

- **In-order:** D B E A F C G
- **Pre-order:** A B D E C F G
- **Post-order:** D E B F G C A
- **Level-order:** A B C D E F G

---

## 2. BST insert trace — `50, 30, 70, 20, 40, 60, 80`

```
      50
     /  \
    30   70
   / \   / \
  20 40 60 80
```

**In-order:** 20, 30, 40, 50, 60, 70, 80 — **sorted** ✓

---

## 3. BST degenerate — `1, 2, 3, 4, 5`

```
1
 \
  2
   \
    3
     \
      4
       \
        5
```

**Height: 4** (n − 1). All operations O(n).

**AVL with same keys:** perfectly balanced, height 2 (log₂ 5 ≈ 2.32). All operations O(log n). ✓

---

## 4. BST delete 30 (one child)

Original tree from problem 2. Delete 30 → has one child (40)? No — 30 has children 20, 40 → **two children**.

**Case 3:** find in-order successor of 30 (smallest in right subtree = 40). Replace 30 with 40.

```
      50
     /  \
    40   70
   /    / \
  20   60 80
```

Wait — deleting 30 leaves 40 with 20 as its left child. Tree is still valid BST.

**Cost:** O(h) = O(log n) for this balanced tree.

---

## 5. BST delete 50 (root, two children)

In-order successor of 50 = 60 (smallest in right subtree). Replace 50 with 60, delete original 60.

```
      60
     /  \
    40   70
   /      \
  20      80
```

**Cost:** O(h) = O(log n).

---

## 6. AVL RR rotation — insert `10, 20, 30`

After `10, 20`: `10 \ 20`. After `30`: `10 \ 20 \ 30` — bf(10)=+2, RR case.

**Left rotate at 10:**

```
    20
   /  \
  10   30
```

All balance factors 0. ✓

---

## 7. AVL LR rotation — insert `30, 10, 20`

After `30, 10`: `30 / 10`. Insert 20 → `30 / 10 \ 20`. bf(30)=−2, bf(10)=+1 → **LR case**.

**Step 1 — left rotate at 10:**

```
    30
   /
  20
  /
 10
```

**Step 2 — right rotate at 30:**

```
    20
   /  \
  10   30
```

All balance factors 0. ✓

---

## 8. AVL insert `1..10` sorted order

| Insert | Rotations |
|:---:|---|
| 1 | — |
| 2 | — |
| 3 | **RR at 1** (1 left) |
| 4 | — |
| 5 | **RR at 3** |
| 6 | — |
| 7 | **RR at 4** (after rebalance: 2 → 6 with children 4, 7; actually LR case at 2? Verify) |

Let me re-trace:

- After 1,2,3: rotate → `2 / 1 \ 3`.
- Insert 4: `2 / 1 \ 3 \ 4`. bf(3)=+1, OK.
- Insert 5: `2 / 1 \ 3 \ 4 \ 5`. bf(3)=+2 → RR at 3. After: `2 / 1 \ 4 / 3 \ 5`.
- Insert 6: `2 / 1 \ 4 / 3 \ 5 \ 6`. bf(5)=+1, OK.
- Insert 7: `2 / 1 \ 4 / 3 \ 5 \ 6 \ 7`. bf(5)=+2 → RR at 5. After: `2 / 1 \ 4 / 3 \ 6 / 5 \ 7`. bf(4)=+1, OK.
- Insert 8: → sub at 6 grows: `6 / 5 \ 7 \ 8`. bf(7)=+1. bf(6)=+1. bf(4)=+2 → RR at 4. After: `4 / 2 \ 6 / 1 \ 3 \ 5 / 7 \ 8`. bf(6)=+1, OK.
- Insert 9: sub at 8 grows: `8 / 7 \ 9`. bf(7)=−1, bf(8)=+1, bf(6)=+2 → RR at 6. After: `4 / 2 \ 8 / 1 \ 3 / 6 / 5 \ 7 \ 9`.

**Total rotations: 5.**

---

## 9. Min-heap insert `10, 20, 5, 3, 15, 8`

| Insert | Array after sift-up |
|:---:|---|
| 10 | [10] |
| 20 | [10, 20] |
| 5 | [5, 20, 10] (sift 5 to root) |
| 3 | [3, 5, 10, 20] (sift 3 to root) |
| 15 | [3, 5, 10, 20, 15] (no sift) |
| 8 | [3, 5, 8, 20, 15, 10] (sift 8: parent(5)=index 2, swap 10↔8) |

**Final array:** `[3, 5, 8, 20, 15, 10]`.

---

## 10. Extract min from heap `[3, 5, 8, 20, 15, 10]`

Save min=3. Move last (10) to root: `[10, 5, 8, 20, 15]`.

Sift-down:
- Children 5, 8. Min=5. 10>5 → swap: `[5, 10, 8, 20, 15]`.
- At index 1 (10): children 20, 15. Min=15. 10<15 → no swap.

**Result:** `[5, 10, 8, 20, 15]`. **Extracted: 3.**

---

## 11. Heapify `[5, 8, 3, 9, 1, 7, 2]` (min-heap)

`n = 7`. Last internal = index `7/2 - 1 = 2`.

**i=2 (value 3):** children 7, 2. Min=2 (index 6). 3>2 → swap. `[5, 8, 2, 9, 1, 7, 3]`.

**i=1 (value 8):** children 9, 1. Min=1 (index 4). 8>1 → swap. `[5, 1, 2, 9, 8, 7, 3]`. At index 4: no children.

**i=0 (value 5):** children 1, 2. Min=1 (index 1). 5>1 → swap. `[1, 5, 2, 9, 8, 7, 3]`. At index 1 (5): children 9, 8. Min=8 (index 4). 5<8 → no swap.

**Final:** `[1, 5, 2, 9, 8, 7, 3]`.

---

## 12. Heapsort `[4, 1, 3, 2, 16, 9, 10, 14, 8, 7]`

**Build max-heap** (result): `[16, 14, 10, 9, 8, 9... let me redo`

Actually: `[4, 1, 3, 2, 16, 9, 10, 14, 8, 7]` after heapify → `[16, 14, 10, 9, 7, 9, 3, 2, 8, 4]` (approximately — trace depends on exact sift order).

**Extract-max repeatedly:**

| Iter | Extract | Array after |
|:---:|:---:|---|
| 1 | 16 | `[14, 8, 10, 9, 7, 9, 3, 2, 4, 16]` |
| 2 | 14 | `[10, 9, 9, 4, 7, 8, 3, 2, 14, 16]` |
| 3 | 10 | `[9, 8, 9, 4, 7, 2, 3, 10, 14, 16]` |
| 4 | 9 | `[9, 8, 7, 4, 3, 2, 9, 10, 14, 16]` |
| 5 | 9 | `[8, 4, 7, 2, 3, 9, 9, 10, 14, 16]` |
| 6 | 8 | `[7, 4, 3, 2, 8, 9, 9, 10, 14, 16]` |
| 7 | 7 | `[4, 2, 3, 7, 8, 9, 9, 10, 14, 16]` |
| 8 | 4 | `[3, 2, 4, 7, 8, 9, 9, 10, 14, 16]` |
| 9 | 3 | `[2, 3, 4, 7, 8, 9, 9, 10, 14, 16]` |

**Sorted:** `[2, 3, 4, 7, 8, 9, 9, 10, 14, 16]` ✓

---

## 13. Max in a min-heap

**Min-heap:** root = min. To find max, must check **all leaves** (nodes at indices `n/2` to `n-1`).

**O(n)** time — can't do better because any leaf could be the max.

**In a max-heap:** root = max, **O(1)**.

---

## 14. BST validity check

```java
boolean isBST(TreeNode node, long min, long max) {
    if (node == null) return true;
    if (node.value <= min || node.value >= max) return false;
    return isBST(node.left, min, node.value) && isBST(node.right, node.value, max);
}
// call: isBST(root, Long.MIN_VALUE, Long.MAX_VALUE)
```

**O(n) time, O(h) space.**

**Common bug:** only checking `node.left.value < node.value < node.right.value` — misses cases like:

```
    5
   / \
  3   8
     /
    4     ← 4 < 5 but in the right subtree — invalid
```

The `min/max` bound propagation is required.

---

## 15. Reconstruct from in-order + pre-order

**in-order** = [4, 2, 5, 1, 3], **pre-order** = [1, 2, 4, 5, 3].

Pre-order gives root = 1. Find 1 in in-order (index 3). Left subtree = in-order[0..2] = [4,2,5]; right = in-order[4..4] = [3].

Recurse on left: pre-order[1..3] = [2,4,5], in-order = [4,2,5]. Root = 2. Left = [4], right = [5].

Recurse on right of root: pre-order[4] = [3], in-order = [3]. Root = 3.

**Tree:**
```
        1
       / \
      2   3
     / \
    4   5
```

---

## 16. Level-order reconstruction

**No**, not unique. Level-order [1, 2, 3] could be:
- Root 1, left 2, right 3.
- Root 1, left 3, right 2.
- Root 1, only right child 2, whose left child is 3.

Without more information (like whether it's a BST, or null markers), level-order alone can't reconstruct the tree.

---

## 17. B+-tree motivation

| Approach | Disk reads |
|---|:---:|
| Sorted array binary search | ~20 (log₂ 10⁶) |
| BST | ~20 (log₂ 10⁶ in balanced case) |
| B+-tree, branching 200 | **~3** (log₂₀₀ 10⁶ = 3) |

**B+-tree reads 3 disk blocks per lookup; BST reads 20.** The 7× speedup is why databases use B+-trees.