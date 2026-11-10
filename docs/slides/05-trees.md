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
  js: assets/lecture-05.js
---

# ECEG-4192

## Chapter 5 — Trees

Binary · BST · AVL · Heaps · B+-Trees

Semester VII · 2025/26

<aside class="notes">
Trees are the first structure that gives O(log n) search on dynamic data. Everything that follows — hash tables, graphs — builds on this idea.
</aside>

---

## Learning outcomes

By the end of this chapter you should be able to:

1. Use **tree terminology** (root, leaf, height, depth).

2. Distinguish **full, complete, perfect** binary trees.

3. Perform all four **traversals**: preorder, inorder, postorder, level-order.

4. Analyse **BST** operations.

5. Explain how **AVL rotations** maintain balance.

6. Explain how **heaps** work and are stored in arrays.

7. Describe the role of **B+-trees** in databases.

---

## 5.1 — Tree terminology

```
        A          ← root (depth 0)
       / \
      B   C        ← depth 1
     / \   \
    D   E   F      ← leaves (depth 2)
```

<div class="fragment">

- **Root** — top node.
- **Parent / child** — connected by edges.
- **Leaf** — no children.
- **Height** — longest path root → leaf (edges).
- **Depth** — path from root to node.

</div>

---

## 5.2 — Binary tree

Each node has **at most two children** (left, right).

<div class="fragment">

**Variants:**

- **Full**: every node has 0 or 2 children.
- **Complete**: all levels filled except possibly the last, filled left-to-right.
- **Perfect**: all internal nodes have 2 children, all leaves at the same depth.

</div>

```
Full:       Complete:       Perfect:
   A           A              A
  / \         / \            / \
 B   C       B   C          B   C
    / \     /  /           / \ / \
   D   E   D  E           D  E F  G
```

---

## 5.3 — Tree traversals

| Traversal | Order | Use |
|---|---|---|
| **Preorder** | Root → Left → Right | Copy tree, prefix expr |
| **Inorder** | Left → Root → Right | **Sorted output (BST)** |
| **Postorder** | Left → Right → Root | Delete tree, postfix |
| **Level-order** | BFS by level | Width, level print |

```python
def inorder(node):
    if node:
        inorder(node.left)
        print(node.val)
        inorder(node.right)
```

<div class="fragment">

**All traversals: O(n)** — visit every node once.

</div>

---

## Traversal trace

```
        1
       / \
      2   3
     / \   \
    4   5   6
```

| Traversal | Output |
|---|---|
| Preorder | 1 2 4 5 3 6 |
| Inorder | 4 2 5 1 3 6 |
| Postorder | 4 5 2 6 3 1 |
| Level-order | 1 2 3 4 5 6 |

---

## 5.4 — Binary Search Tree (BST)

???+ note "Definition — BST"
    A binary tree where, for every node:
    
    **left subtree values < node value < right subtree values.**

```
        50
       /  \
      30   70
     / \   / \
    20 40 60 80
```

<div class="fragment">

**Inorder traversal yields sorted output** — this is the defining feature.

</div>

---

## BST operations

```python
def search(node, key):
    if not node or node.val == key:
        return node
    return search(node.left, key) if key < node.val \
           else search(node.right, key)
```

| Operation | Average | Worst (skewed) |
|---|---|---|
| Search | **O(log n)** | **O(n)** |
| Insert | **O(log n)** | **O(n)** |
| Delete | **O(log n)** | **O(n)** |

<div class="fragment">

Worst case: inserting sorted data → **linked list**, height $n$.

</div>

---

## BST — deletion cases

Deleting node $X$:

<div class="fragment">

**Case 1 — $X$ is a leaf.** Remove it.

</div>

<div class="fragment">

**Case 2 — $X$ has one child.** Replace $X$ by its child.

</div>

<div class="fragment">

**Case 3 — $X$ has two children.**
Replace $X$ by its **inorder successor** (smallest in right subtree), then delete the successor.

</div>

---

## Why BSTs can degrade

Insert $1, 2, 3, 4, 5$ in order:

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

<div class="fragment">

**Height = 5 = n.**

All operations degrade to **O(n)**.

**Fix:** balance the tree — **AVL**, **red-black**, **B-tree**.

</div>

---

## 5.5 — AVL trees

???+ note "Definition — AVL"
    A **self-balancing BST**: for every node,
    
    $$ |\text{height}(\text{left}) - \text{height}(\text{right})| \le 1. $$

<div class="fragment">

Invented by **Adelson-Velsky & Landis** (1962).

Guarantees height $O(\log n)$ → all operations **O(log n)**.

**Maintained via rotations** after insert/delete.

</div>

---

## AVL rotations

**LL imbalance — right rotate:**

```
      30            20
     /  \          /  \
    20  40  →     10  30
   /                  \
  10                  40
```

<div class="fragment">

**RR imbalance — left rotate:** mirror image.

**LR / RL — double rotations:** left-then-right, or right-then-left.

**One rotation per insert** (or a double rotation) restores balance.

</div>

---

## AVL insertion — algorithm

1. Insert like a normal BST.

2. Walk back up, updating heights.

3. At each node, compute balance factor $h_L - h_R$.

4. If $|h_L - h_R| > 1$, apply the appropriate rotation:
   - **LL** → right rotate.
   - **RR** → left rotate.
   - **LR** → left rotate on left child, then right rotate on node.
   - **RL** → right rotate on right child, then left rotate on node.

<div class="fragment">

**O(log n)** total — same as BST insert, plus $O(1)$ rotation work per level.

</div>

---

## 5.6 — Heaps

???+ note "Definition — Heap"
    A **complete binary tree** with the **heap property**:
    
    - **Max-heap:** parent $\ge$ children.
    - **Min-heap:** parent $\le$ children.

<div class="fragment">

**Complete** → can be stored in an **array**.

Index $i$ has:

- Parent at $\lfloor (i-1)/2 \rfloor$.
- Children at $2i+1$ and $2i+2$.

</div>

---

## Heap stored in an array

Min-heap:

```
        1
       / \
      3   2
     / \ / \
    7  4 5  6
```

Array: `[1, 3, 2, 7, 4, 5, 6]`

<div class="fragment">

- Index 0 (value 1) → children at index 1 (3), index 2 (2). ✓
- Index 1 (value 3) → children at index 3 (7), index 4 (4). ✓

**No pointers. No wasted space.** Cache-friendly.

</div>

---

## Heap operations

| Operation | Complexity |
|---|---|
| `peek` (min or max) | **O(1)** |
| `insert` | **O(log n)** |
| `extract` (min or max) | **O(log n)** |
| `buildHeap` from array | **O(n)** |

<div class="fragment">

**Insert:** append at end, then **sift up** (bubble up while violating heap property). Height $\log n$.

**Extract:** move last to root, then **sift down**. Height $\log n$.

**Build heap:** sift-down from bottom half — total $O(n)$, not $O(n\log n)$ (a classic analysis trick).

</div>

---

## Heap applications

<div class="fragment">

- **Priority queues** — scheduler, Huffman coding, Dijkstra.
- **Heap sort** — $O(n\log n)$, in place.
- **Median maintenance** — two heaps.
- **Top-$k$ selection** — min-heap of size $k$.

</div>

<div class="fragment">

!!! tip "Heap = priority-ordered, not sorted."
    Partial order is cheaper than total order.

</div>

---

## 5.7 — B+-Trees

**Multiway search trees** — every internal node has many children (typically 100–1000).

<div class="fragment">

**Properties:**

- All data in **leaves**.
- Internal nodes hold **keys for routing**.
- Leaves linked → efficient **range queries**.
- High fanout → **low height** → **few disk I/Os**.

</div>

<div class="fragment">

**Height $O(\log_B n)$** where $B$ is fanout.

For $n = 10^9$ and $B = 100$: height $\approx 5$. **Five disk reads to find any record.**

**Used in:** databases (MySQL, PostgreSQL), file systems (NTFS, ext4), Google Bigtable.

</div>

---

## Chapter summary

1. **Tree terminology**: root, leaf, height, depth.

2. **Traversals:** preorder, inorder (sorted for BST), postorder, level-order — all O(n).

3. **BST**: O(log n) average, **O(n) worst** if skewed.

4. **AVL** balances via rotations → guaranteed O(log n).

5. **Heaps** = complete binary trees in arrays — O(log n) insert/extract.

6. **B+-trees** = disk-friendly multiway trees — very low height.

---

## Review questions

1. Define height and depth; illustrate on a tree.

2. Trace preorder, inorder, postorder on a given tree.

3. Why does an unbalanced BST degrade to O(n)?

4. Show the rotation needed to fix an LL imbalance in an AVL tree.

5. Explain how a min-heap is stored in an array. Give the index formula.

6. Why is `buildHeap` O(n), not O(n log n)?

> Answers: **Solutions → Chapter 5**.

---

## References

- **Weiss**, Ch. 4 (trees, AVL), Ch. 6 (heaps).
- **Lafore**, Ch. 8 (binary trees), Ch. 10 (2-3-4 and B-trees).
- **CLRS**, Ch. 12 (BST), Ch. 13 (red-black), Ch. 6 (heaps).
- **Sedgewick & Wayne**, Ch. 3.