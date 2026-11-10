# Chapter 5 — Trees

!!! abstract "Chapter at a glance"
    **Contact hours:** 2 lecture + 1 tutorial + 2 lab · **Home study:** ~5 h
    
    **Coverage (syllabus):** definition and implementation · types of trees
    (binary, binary search, AVL, B+-trees, heaps) · application of trees.

---

## Learning outcomes

By the end of this chapter you should be able to:

1. Define the **tree terminology**: root, parent, child, leaf, subtree, depth, height.
2. Represent trees with **left/right pointers** and with **array-based heap layout**.
3. Implement all four **tree traversals** — in-order, pre-order, post-order, level-order.
4. Implement and analyse a **Binary Search Tree** — insert, search, delete — in best, average, and worst cases.
5. Explain when a BST degrades to a **linked list** and why balance matters.
6. Perform **AVL rotations** (left, right, left-right, right-left) to restore balance after insertion.
7. Implement a **binary heap** with O(log n) insert and O(log n) extract-min.
8. Use a heap to implement **heapsort** (preview of Chapter 8) and a **priority queue**.
9. Explain the role of **B+-trees** in databases (conceptual, brief).
10. Select the right tree structure for a given application.

---

## 5.1 Tree terminology

### 5.1.1 Definition

A **tree** is a connected acyclic graph. Every non-empty tree has:

- **Root** — the topmost node.
- **Leaves** — nodes with no children.
- **Internal nodes** — nodes with at least one child.

For a node $x$:

| Term | Meaning |
|---|---|
| **Parent** | The node directly above `x` |
| **Children** | Nodes directly below `x` |
| **Siblings** | Nodes with the same parent |
| **Ancestors** | `x`, its parent, parent's parent, … up to root |
| **Descendants** | `x`, its children, their children, … |
| **Subtree rooted at x** | `x` and all its descendants |
| **Depth of x** | Number of edges from root to `x` |
| **Height of x** | Number of edges from `x` to deepest descendant leaf |
| **Height of tree** | Height of root |
| **Size** | Number of nodes |

**Convention:** height of a leaf is 0; height of an empty tree is −1.

### 5.1.2 Binary tree

A **binary tree** has at most **two** children per node (left and right).

```java
class TreeNode {
    int      value;
    TreeNode left, right;

    TreeNode(int v) { this.value = v; }
}
```

### 5.1.3 Complete and full binary trees

| Type | Rule |
|---|---|
| **Full** | Every node has 0 or 2 children |
| **Complete** | All levels except possibly the last are full, and last level is filled left-to-right |
| **Perfect** | Every level is completely full (a complete tree with $2^h$ leaves) |

**Complete binary trees** are the key concept for **heaps** — they fit compactly in an array with no wasted space.

### 5.1.4 Height bounds

For a binary tree with $n$ nodes:

- **Minimum height:** $\lfloor \log_2 n \rfloor$ (the tree is as balanced as possible).
- **Maximum height:** $n - 1$ (a "linked list" shape — every node has one child).

**Why this matters:** every tree operation's cost depends on height. Balanced trees → O(log n); skewed trees → O(n).

---

## 5.2 Binary tree traversals

The four traversals differ in the **order** they visit a node relative to its children.

### 5.2.1 In-order (left, root, right)

```java
void inOrder(TreeNode node) {
    if (node == null) return;
    inOrder(node.left);
    System.out.println(node.value);
    inOrder(node.right);
}
```

For a **BST**, in-order gives **sorted order**.

### 5.2.2 Pre-order (root, left, right)

```java
void preOrder(TreeNode node) {
    if (node == null) return;
    System.out.println(node.value);
    preOrder(node.left);
    preOrder(node.right);
}
```

Used for **tree serialisation** — writing a tree to a file in a form that can be read back to produce the same tree.

### 5.2.3 Post-order (left, right, root)

```java
void postOrder(TreeNode node) {
    if (node == null) return;
    postOrder(node.left);
    postOrder(node.right);
    System.out.println(node.value);
}
```

Used for **deleting a tree** — children are deleted before the parent, so nothing is orphaned.

### 5.2.4 Level-order (BFS)

```java
void levelOrder(TreeNode root) {
    if (root == null) return;
    Queue<TreeNode> q = new ArrayDeque<>();
    q.add(root);
    while (!q.isEmpty()) {
        TreeNode cur = q.remove();
        System.out.println(cur.value);
        if (cur.left  != null) q.add(cur.left);
        if (cur.right != null) q.add(cur.right);
    }
}
```

Uses a **queue** (Chapter 4). Visits nodes level by level — root, all children, all grandchildren, …

### 5.2.5 The four traversals on one tree

```
        A
       / \
      B   C
     / \ / \
    D  E F  G
```

| Traversal | Order |
|---|---|
| In-order | D B E A F C G |
| Pre-order | A B D E C F G |
| Post-order | D E B F G C A |
| Level-order | A B C D E F G |

**Cost of all four:** O(n) time, O(h) space (h = height, from recursion or queue).

---

## 5.3 Binary Search Tree (BST)

### 5.3.1 The BST property

A **Binary Search Tree** is a binary tree where, for every node:

- **All values in the left subtree are < the node's value.**
- **All values in the right subtree are > the node's value.**

This property makes **search, insert, and delete** O(h) — where h is the tree height.

### 5.3.2 Search

```java
TreeNode search(TreeNode root, int target) {
    while (root != null) {
        if (target == root.value) return root;
        if (target < root.value)   root = root.left;
        else                       root = root.right;
    }
    return null;
}
```

At each step, discard half the remaining tree (in a balanced tree). **O(h)**.

### 5.3.3 Insert

```java
TreeNode insert(TreeNode root, int value) {
    if (root == null) return new TreeNode(value);
    if (value < root.value) root.left  = insert(root.left, value);
    else                    root.right = insert(root.right, value);
    return root;
}
```

Find the leaf position where the new value belongs, attach it there. **O(h)**.

### 5.3.4 Delete — the tricky one

Three cases:

**Case 1 — Leaf node (no children).** Simply remove it.

**Case 2 — One child.** Splice the child up to replace the node.

**Case 3 — Two children.** Replace the node's value with its **in-order successor** (the smallest value in the right subtree), then delete the successor.

```java
TreeNode delete(TreeNode root, int value) {
    if (root == null) return null;
    if (value < root.value)      root.left  = delete(root.left, value);
    else if (value > root.value) root.right = delete(root.right, value);
    else {
        // found the node
        if (root.left == null)  return root.right;   // case 1 or 2 (right child only)
        if (root.right == null) return root.left;    // case 2 (left child only)
        // case 3: two children
        TreeNode succ = root.right;
        while (succ.left != null) succ = succ.left;   // find in-order successor
        root.value = succ.value;
        root.right = delete(root.right, succ.value);
    }
    return root;
}
```

**Cost:** O(h) — walk down to find the node, plus O(h) to find the successor in the worst case.

### 5.3.5 BST complexity — the big warning

All BST operations are **O(h)**, where h is the tree height.

| Insertion order | Tree shape | Height | Cost per operation |
|:---:|---|:---:|:---:|
| Random | Roughly balanced | O(log n) | **O(log n)** |
| Sorted ascending | Linked list rightward | O(n) | **O(n)** ← **disaster** |

**The degenerate case.** Inserting `1, 2, 3, 4, 5, 6, 7, 8, 9, 10` in sorted order produces a **right-skewed** BST that's functionally a linked list:

```
1
 \
  2
   \
    3
     \
      4
       ...
```

Height = n − 1. Search, insert, delete all O(n). **The BST loses its advantage.**

**This is why balanced trees exist** — AVL (5.6), red-black (in the standard library), B-trees (5.8) — they guarantee h = O(log n) regardless of insertion order.

---

## 5.4 AVL trees

### 5.4.1 The balance condition

An **AVL tree** (Adelson-Velsky & Landis, 1962) is a BST where, for every node, the **balance factor** is −1, 0, or +1:

$$
\text{balance}(x) = \text{height}(x.\text{right}) - \text{height}(x.\text{left})
$$

**Invariant:** |balance(x)| ≤ 1 for all nodes.

**Result:** AVL trees guarantee **h = O(log n)**. All operations are O(log n) worst case.

### 5.4.2 The four rotation cases

When insertion breaks the AVL invariant, one of four rotations restores it.

**Case 1 — Right rotation (LL imbalance).** Balance(x) = −2 (left-heavy), and the new node went into the **left child's left subtree**.

```
        x                         y
       / \                       / \
      y   T3       →            z   x
     / \                       / \ / \
    z   T2                    T1 T2 T3 T4
   / \
  T1  T4
```

**Operation:** y becomes the new root of this subtree; x becomes y's right child; y's old right subtree becomes x's left.

**Case 2 — Left rotation (RR imbalance).** Balance(x) = +2 (right-heavy), and the new node went into the **right child's right subtree**. Mirror of Case 1.

**Case 3 — Left-right rotation (LR imbalance).** Balance(x) = −2, but the new node went into the **left child's right subtree**.

**Two rotations:** first a **left rotation on the left child** (converts to Case 1), then a **right rotation on x**.

**Case 4 — Right-left rotation (RL imbalance).** Balance(x) = +2, new node into the **right child's left subtree**. Two rotations: right on right child, then left on x.

### 5.4.3 Worked example — right rotation

**Before:**
```
        30 (bf = -2)
        /
      20 (bf = -1)
      /
    10 (bf = 0)
```

**After right rotation at 30:**
```
      20 (bf = 0)
     /  \
   10    30
```

Height restored. Root is 20, all balance factors 0.

### 5.4.4 Worked example — left-right (LR) rotation

**Before:**
```
        30 (bf = -2)
        /
      10 (bf = +1)
        \
        20
```

**Step 1 — left rotation at 10:**
```
        30 (bf = -2)
        /
      20
      /
    10
```

**Step 2 — right rotation at 30:**
```
      20 (bf = 0)
     /  \
   10    30
```

Both steps together = one **LR rotation**.

### 5.4.5 Insert trace — build an AVL from 1..7

Inserting `1, 2, 3, 4, 5, 6, 7` in sorted order:

| Insert | Tree (before) | Action | Tree (after) |
|:---:|---|---|---|
| 1 | (empty) | insert | `1` |
| 2 | `1` | insert right | `1 \ 2` |
| 3 | `1 \ 2` | bf(1) = +2 → RR case → **left rotate at 1** | `2 / 1 \ 3` |
| 4 | `2 / 1 \ 3` | insert right of 3 | `2 / 1 \ 3 \ 4` |
| 5 | `...` | bf(3) = +2 → RR case → **left rotate at 3** | `2 / 1 \ 4 / 3 \ 5` |
| 6 | `...` | insert right of 5 | `2 / 1 \ 4 / 3 \ 5 \ 6` |
| 7 | `...` | bf(4) = +2 → RR case → **left rotate at 4** | `4 / 2 \ 6 / 1 \ 3 \ 5 \ 7` — balanced ✓ |

**Balanced after every step.** This is the AVL guarantee: sequential insertion never produces a skewed tree.

---

## 5.5 Heaps

### 5.5.1 Definition

A **binary heap** is a **complete binary tree** satisfying the **heap property**:

- **Min-heap:** value(parent) ≤ value(children) — the minimum is at the root.
- **Max-heap:** value(parent) ≥ value(children) — the maximum is at the root.

**Crucially:** a heap is **not a BST**. Left and right subtrees have no ordering relative to each other — only parent-vs-child matters.

### 5.5.2 Array representation

Because a heap is a **complete** binary tree, it fits in an array with no wasted space:

```
Index:   0   1   2   3   4   5   6
Value:  [2,  5,  8, 10,  6, 15,  9]
```

| Relation | Formula (0-indexed) |
|---|---|
| Parent of `i` | `(i - 1) / 2` |
| Left child of `i` | `2*i + 1` |
| Right child of `i` | `2*i + 2` |

**No pointers needed.** Parent and child indices are computed arithmetically — the heap is a **cache-friendly** data structure by construction.

### 5.5.3 Insert — sift-up

**Algorithm:**

1. Append the new value at the end (index `size`).
2. **Sift up** — compare with parent; swap if violated. Repeat until invariant restored.

```java
class MinHeap {
    private int[] data = new int[16];
    private int size = 0;

    void insert(int x) {
        if (size == data.length) resize(data.length * 2);
        data[size] = x;
        siftUp(size);
        size++;
    }

    private void siftUp(int i) {
        while (i > 0) {
            int parent = (i - 1) / 2;
            if (data[i] >= data[parent]) break;
            swap(i, parent);
            i = parent;
        }
    }

    private void swap(int i, int j) {
        int t = data[i]; data[i] = data[j]; data[j] = t;
    }
    // resize as in Chapter 2
}
```

**Cost: O(log n).** Sift-up does at most one swap per level, and the tree height is $\lfloor \log_2 n \rfloor$.

### 5.5.4 Extract-min — sift-down

**Algorithm:**

1. Save the root value (the min).
2. Move the **last** element to the root; decrement `size`.
3. **Sift down** — compare with children; swap with the smaller child if violated. Repeat.

```java
int extractMin() {
    if (size == 0) throw new RuntimeException("Heap empty");
    int minVal = data[0];
    data[0] = data[size - 1];
    size--;
    siftDown(0);
    return minVal;
}

private void siftDown(int i) {
    while (true) {
        int left = 2*i + 1, right = 2*i + 2, smallest = i;
        if (left  < size && data[left]  < data[smallest]) smallest = left;
        if (right < size && data[right] < data[smallest]) smallest = right;
        if (smallest == i) break;
        swap(i, smallest);
        i = smallest;
    }
}
```

**Cost: O(log n).**

### 5.5.5 Peek-min — O(1)

```java
int peekMin() {
    if (size == 0) throw new RuntimeException("Heap empty");
    return data[0];
}
```

**The root is always the minimum.** That's the whole point.

### 5.5.6 Building a heap from n elements — O(n)

Naive build: n insertions, each O(log n) → **O(n log n)**.

**Better:** build in **O(n)** using bottom-up `siftDown` from the last internal node.

```java
void heapify(int[] arr) {
    data = arr;
    size = arr.length;
    for (int i = size/2 - 1; i >= 0; i--) siftDown(i);
}
```

**Why O(n)?** Most nodes are near the bottom and need **0 or 1 swap**. Only a few near the root need many swaps. The total work is bounded by:

$$
\sum_{h=0}^{\log n} \frac{n}{2^{h+1}} \cdot O(h) = O(n).
$$

This is the geometric-decay sum from Chapter 2 — the number of nodes decreases exponentially with height, which cancels the increasing per-node cost.

### 5.5.7 Heapsort — sort using a heap

**Algorithm:**

1. Build a **max-heap** from the array — O(n).
2. Repeatedly: swap the root (max) with the last element, decrement size, sift-down the new root — O(log n) each.
3. After n iterations, the array is sorted.

```java
void heapsort(int[] a) {
    int n = a.length;
    // build max-heap
    for (int i = n/2 - 1; i >= 0; i--) siftDown(a, i, n);
    // extract one element at a time
    for (int i = n - 1; i > 0; i--) {
        swap(a, 0, i);
        siftDown(a, 0, i);
    }
}
```

**Cost: O(n log n)** — O(n) build + n × O(log n) extract.

**In-place: O(1) auxiliary space.** This is heapsort's key advantage over merge sort.

**Not stable.** Equal values may be reordered.

---

## Worked example 5.1 — Build a BST from sorted input (worst case)

**Insertion order:** `10, 20, 30, 40, 50`.

**Tree after each insertion:**

```
Insert 10:  10
Insert 20:  10 \ 20
Insert 30:  10 \ 20 \ 30
Insert 40:  10 \ 20 \ 30 \ 40
Insert 50:  10 \ 20 \ 30 \ 40 \ 50
```

**Height: 4** (n − 1 = 5 − 1 = 4). Every operation is **O(n)**, not O(log n).

**Why?** Each new key is larger than all existing keys, so it always goes to the rightmost leaf. The tree is a right-skewed list.

**Fix:** use a **self-balancing** BST — AVL (5.4) or red-black.

---

## Worked example 5.2 — Build a BST from random order (best case)

**Insertion order:** `50, 30, 70, 20, 40, 60, 80`.

**Tree after each insertion:**

```
50
→
   50
   / \
  30  70
→
      50
     /  \
    30   70
   / \   /
  20 40 60
→
      50
     /  \
    30   70
   / \   / \
  20 40 60 80
```

**Height: 2.** All operations **O(log n)** (log₂ 7 ≈ 2.8).

**Compare with Worked example 5.1.** Same seven keys, different insertion order → dramatically different shape.

**The lesson:** BST performance depends on insertion order, which the programmer usually cannot control. **Balanced BSTs are the safe choice.**

---

## Worked example 5.3 — In-order traversal of a BST yields sorted order

**BST:**
```
        50
       /  \
      30   70
     / \   / \
    20 40 60 80
```

**In-order trace (left, root, right):**

Starting at root 50, we first go to left subtree (root 30), which itself goes to left (root 20):

```
- Descend to 20, no left → visit 20
- Back to 30 → visit 30
- Right of 30 → descend to 40, no left → visit 40 → no right
- Back to 50 → visit 50
- Right of 50 → descend to 70, left = 60 → descend to 60 → visit 60
- Back to 70 → visit 70
- Right of 70 → descend to 80 → visit 80
```

**Output:** `20, 30, 40, 50, 60, 70, 80` — **sorted**. ✓

**Why this is a theorem.** The in-order traversal visits, at every node, the entire left subtree (all values < node), then the node itself, then the entire right subtree (all values > node). By induction, the whole traversal visits values in ascending order.

**This is the primary use of in-order traversal** — extract sorted data from a BST in O(n).

**Cost.** O(n) time, O(h) space.

---

## Worked example 5.4 — Delete from a BST, all three cases

**Starting tree:**
```
        50
       /  \
      30   70
     / \   / \
    20 40 60 80
```

**Case 1 — Delete leaf 20.**

Simply remove node 20 and set parent's `left = null`:

```
        50
       /  \
      30   70
       \   / \
       40 60 80
```

**O(h)** to find, O(1) to remove.

**Case 2 — Delete node 30 (one child — 40).**

Replace 30 with its child 40:

```
        50
       /  \
      40   70
          / \
         60 80
```

**O(h)**.

**Case 3 — Delete node 70 (two children — 60 and 80).**

**Find the in-order successor** of 70 — the smallest value in its right subtree. The right subtree is `{80, 60}`. The smallest is **60**.

**Replace 70's value with 60, then delete 60 from its original position** (which is a leaf):

```
        50
       /  \
      40   60
            \
            80
```

**O(h)** total — find node (O(h)), find successor (O(h) worst case), delete successor (O(h)).

**Why in-order successor and not predecessor?** Either works. Both preserve the BST invariant. **Successor** is the smallest value greater than the deleted node; **predecessor** is the largest smaller. Choosing one consistently is fine — some textbooks alternate for balance, but that's an over-optimisation.

---

## Worked example 5.5 — AVL right rotation, step by step

**Before:** Right-heavy tree at node 30.

```
        30
       /  \
      10   40
            \
            50
```

**Balance factors:**
- Node 50: bf = 0
- Node 40: bf = +1
- Node 30: bf = +2 → **violated** (RR case)

**Action:** left rotation at 30.

**Result:**
```
        40
       /  \
      30   50
     /
    10
```

**Balance factors after:**
- Node 10: bf = 0
- Node 30: bf = 0
- Node 50: bf = 0
- Node 40: bf = 0

**All invariant.** ✓

**When does this happen?** Inserting 50 into a tree `{30, 10, 40}` causes RR imbalance. Since RR is one of the 4 AVL cases, single left rotation fixes it.

---

## Worked example 5.6 — AVL left-right rotation

**Before:** Left-heavy tree with right-heavy left child.

```
        30
       /  
      10
        \
        20
```

**Balance factors:**
- Node 20: bf = 0
- Node 10: bf = +1 (right child 20, no left)
- Node 30: bf = −2 → violated, and left child is right-heavy → **LR case**

**Step 1 — left rotation at 10:**

```
        30
       /
      20
      /
     10
```

Now it's LL case (left-heavy at 30, left child 20 is left-heavy). 

**Step 2 — right rotation at 30:**

```
      20
     /  \
   10    30
```

**All balanced.** ✓

**Rule of thumb:** whenever the tree is "zigzagged" (heavy on one side, but the heavy subtree is heavy on the other side), you need **two rotations**.

---

## Worked example 5.7 — AVL insertion trace (concrete)

**Build an AVL by inserting `10, 20, 30, 40, 50, 25` in that order.**

**Insert 10:** `10` — balanced.

**Insert 20:** `10 \ 20` — bf(10) = +1, OK.

**Insert 30:** `10 \ 20 \ 30` — bf(10) = +2, RR → **left rotate at 10**:

```
    20
   /  \
  10   30
```

**Insert 40:** `20 / 10 \ 30 \ 40` — bf(30) = +1, OK.

**Insert 50:** `20 / 10 \ 30 \ 40 \ 50` — bf(30) = +2, RR → **left rotate at 30**:

```
      20
     /  \
    10   40
        /  \
       30   50
```

**Insert 25:** goes to right of 20 → left of 30. New tree:

```
      20
     /  \
    10   40
        /  \
       30   50
      /
     25
```

**Balance factors:**
- 25: bf = 0
- 30: bf = −1 (left child 25, no right)
- 40: bf = −1 (left child 30, right child 50)
- 20: bf = +1 (right child 40, left 10)
- 10: bf = 0

**All balance factors within [−1, +1].** No rotation needed. ✓

**AVL invariant maintained throughout 6 insertions.** Each insertion was O(log n).

---

## Worked example 5.8 — Min-heap insert trace

**Start:** empty heap.

**Insert 5, 3, 8, 1, 9, 2 in order.**

**Insert 5:** array = `[5]`.

**Insert 3:** array = `[5, 3]`. Sift-up: parent(1)=0, `data[1]=3 < data[0]=5` → swap. Array = `[3, 5]`.

**Insert 8:** array = `[3, 5, 8]`. Parent(2)=0, `data[2]=8 ≥ data[0]=3` → no swap. Array = `[3, 5, 8]`.

**Insert 1:** array = `[3, 5, 8, 1]`. Sift-up: parent(3)=1, `data[3]=1 < data[1]=5` → swap. Array = `[3, 1, 8, 5]`. Now parent of 1 is 0, `data[1]=1 < data[0]=3` → swap. Array = `[1, 3, 8, 5]`.

**Insert 9:** array = `[1, 3, 8, 5, 9]`. Parent(4)=1, `data[4]=9 ≥ data[1]=3` → no swap. Array = `[1, 3, 8, 5, 9]`.

**Insert 2:** array = `[1, 3, 8, 5, 9, 2]`. Parent(5)=2, `data[5]=2 < data[2]=8` → swap. Array = `[1, 3, 2, 5, 9, 8]`. Parent of 2 (index 2) is 0, `data[2]=2 ≥ data[0]=1` → no swap.

**Final array:** `[1, 3, 2, 5, 9, 8]`.

**Verify min-heap property:**
- Parent 0 (1): children 3, 2. Both ≥ 1. ✓
- Parent 1 (3): children 5, 9. Both ≥ 3. ✓
- Parent 2 (2): child 8. 8 ≥ 2. ✓

**Valid min-heap.** Root is 1 — the minimum. ✓

**Cost.** Each sift-up is at most log n swaps. Six insertions → O(6 log 6) = O(n log n) total.

---

## Worked example 5.9 — Min-heap extract-min trace

**Start:** heap `[1, 3, 2, 5, 9, 8]` (from Example 5.8).

**Extract-min:**

1. Save `min = 1` (root).
2. Move last element (8) to root. Array = `[8, 3, 2, 5, 9]`.
3. Sift-down from index 0:
   - Children of 0: indices 1 (3) and 2 (2). Smaller child is 2 (index 2). `data[0]=8 > 2` → swap. Array = `[2, 3, 8, 5, 9]`.
   - At index 2: no children (2*2+1=5 ≥ size 5). Stop.

**Return:** 1.

**Resulting array:** `[2, 3, 8, 5, 9]`.

**Verify min-heap:**
- Root 2: children 3, 8. Both ≥ 2. ✓
- Node 3 (index 1): children 5, 9. Both ≥ 3. ✓

**Valid min-heap.** ✓

**Cost.** O(log n) — at most log n sift-down swaps.

---

## Worked example 5.10 — Heapify (bottom-up heap construction)

**Input array:** `[4, 10, 3, 5, 1, 2]` (unsorted).

**Build min-heap from bottom up:**

**Start:** `[4, 10, 3, 5, 1, 2]`, size = 6. Last internal node index = `6/2 - 1 = 2`.

**Sift-down at index 2 (value 3):**
- Children: indices 5 (value 2). Only left child exists (2*2+2=6 ≥ size).
- 3 > 2 → swap. Array = `[4, 10, 2, 5, 1, 3]`.

**Sift-down at index 1 (value 10):**
- Children: indices 3 (value 5) and 4 (value 1). Smaller is 1.
- 10 > 1 → swap. Array = `[4, 1, 2, 5, 10, 3]`.
- At index 4: no children. Stop.

**Sift-down at index 0 (value 4):**
- Children: 1 (value 1), 2 (value 2). Smaller is 1.
- 4 > 1 → swap. Array = `[1, 4, 2, 5, 10, 3]`.
- At index 1 (value 4): children 3 (value 5), 4 (value 10). Both ≥ 4 → no swap.

**Final array:** `[1, 4, 2, 5, 10, 3]`.

**Verify:**
- Root 1: children 4, 2. Both ≥ 1. ✓
- Node 4: children 5, 10. Both ≥ 4. ✓
- Node 2: child 3. 3 ≥ 2. ✓

**Valid min-heap.** ✓

**Cost.** O(n) — this is the O(n) heapify from §5.5.6. Compare to n × insert = O(n log n). The bottom-up approach is strictly faster.

---

## Worked example 5.11 — Heapsort on a small array

**Input:** `[5, 2, 8, 1, 9, 3]`, sorted ascending.

**Step 1 — Build a max-heap.** (Similar to Example 5.10 but with max-heap comparisons.) Result: `[9, 5, 8, 1, 2, 3]`.

**Step 2 — Extract max repeatedly** (swap root with last, sift-down on remaining).

| Iteration | Array | Swap | Sift-down |
|:---:|---|---|---|
| 1 | `[9, 5, 8, 1, 2, 3]` | swap 9 and 3 | sift-down on `[3, 5, 8, 1, 2]` → `[8, 5, 3, 1, 2, 9]` |
| 2 | `[8, 5, 3, 1, 2, 9]` | swap 8 and 2 | sift-down on `[2, 5, 3, 1]` → `[5, 2, 3, 1, 8, 9]` |
| 3 | `[5, 2, 3, 1, 8, 9]` | swap 5 and 1 | sift-down on `[1, 2, 3]` → `[3, 2, 1, 5, 8, 9]` |
| 4 | `[3, 2, 1, 5, 8, 9]` | swap 3 and 1 | sift-down on `[1, 2]` → `[2, 1, 3, 5, 8, 9]` |
| 5 | `[2, 1, 3, 5, 8, 9]` | swap 2 and 1 | `[1, 2, 3, 5, 8, 9]` — **sorted** ✓ |

**Result:** `[1, 2, 3, 5, 8, 9]` — sorted ascending.

**Cost.** O(n log n) — n extract-max operations, each O(log n).

**In-place.** Only O(1) auxiliary space — the same array is used for the heap and the output.

---

## 5.6 B+-trees (brief)

**B+-trees** are the workhorse of **databases and file systems** — a variation of balanced search trees designed for **disk access**, where reading a node is expensive.

**Key differences from a binary tree:**

- **High branching factor** — each node holds hundreds of keys (not just 2). A tree of height 3 holds millions of keys.
- **All values in leaves** — internal nodes hold only routing keys. This makes leaves dense and fast to scan.
- **Leaves linked** — sequential access (range queries) walks the leaf chain.

**Why the branching factor matters.** Each level of the tree corresponds to **one disk read**. With binary tree depth ~20 for a million records, you'd do ~20 disk reads. With a B+-tree branching factor 200, depth is ~3 → only **3 disk reads**. **This is why databases use B+-trees, not binary trees.**

**Preview only.** Full treatment is beyond this course — see CLRS Chapter 18 or a database textbook for depth. This course's focus is the binary structures.

---

## 5.7 Choosing a tree structure

| Application | Structure |
|---|---|
| **Sorted iteration** + **ordered operations** (range, predecessor) | **BST** (if balanced) or **AVL / red-black** |
| **Frequent insertions in unpredictable order** | **Balanced BST** (AVL / red-black) — never plain BST |
| **Priority queue / min or max access** | **Binary heap** |
| **Sorting (in-place)** | **Heapsort** |
| **DB index / filesystem** | **B+-tree** |
| **Range queries on disk** | **B+-tree** |
| **Prefix search / strings** | **Trie** (not covered here) |

---

## Chapter summary

1. **Tree terminology** — root, leaf, height, depth, ancestors, subtree. Height bounds: O(log n) minimum for n nodes, O(n) maximum.
2. **Binary tree** — at most 2 children per node. **Complete** binary trees fit in arrays compactly (heaps).
3. **Four traversals** — in-order (sorted for BST), pre-order (serialization), post-order (deletion), level-order (BFS with queue). All O(n).
4. **BST** — every operation is O(h). In-order traversal yields sorted order.
5. **BST can degrade** to a linked list with sorted insertion → O(n) per operation. This motivates **AVL**.
6. **AVL** — BST with |balance factor| ≤ 1. Guarantees h = O(log n). **Four rotation cases** — LL, RR, LR, RL.
7. **Heap** — complete binary tree with parent-child ordering. Array-representable with index arithmetic: parent = `(i-1)/2`, children = `2i+1`, `2i+2`.
8. **Heap insert** — append then sift-up. **O(log n)**. **Extract-min** — move last to root then sift-down. **O(log n)**.
9. **Heapify** — build a heap from n elements in **O(n)** (not O(n log n)) — geometric series argument.
10. **Heapsort** — O(n log n) time, O(1) space, not stable.
11. **B+-trees** — high branching factor (hundreds of children per node), disk-oriented — used by all major databases.

---

## Quiz

### A. Multiple choice

1. In a **BST**, in-order traversal yields:

   (a) pre-order (b) post-order (c) **sorted order** (d) reverse-sorted order

2. Search in a **balanced** BST of n nodes is:

   (a) O(1) (b) **O(log n)** (c) O(n) (d) O(n log n)

3. Search in a **skewed** BST (inserted sorted) of n nodes is:

   (a) O(1) (b) O(log n) (c) **O(n)** (d) O(n log n)

4. Which rotation restores balance in the LR case?

   (a) single right (b) single left (c) **left then right** (d) right then left

5. A **max-heap** guarantees:

   (a) all nodes sorted (b) **each node ≥ its children** (c) left subtree < right subtree (d) root is median

6. In a 0-indexed heap array, the left child of index `i` is:

   (a) `i/2` (b) `i+1` (c) **`2i+1`** (d) `2i+2`

7. Insert into a heap of size n costs:

   (a) O(1) (b) **O(log n)** (c) O(n) (d) O(n log n)

8. Building a heap from n unsorted elements (bottom-up) costs:

   (a) O(1) (b) O(log n) (c) **O(n)** (d) O(n log n)

9. **Heapsort** time and space complexity:

   (a) **O(n log n), O(1)** (b) O(n log n), O(n) (c) O(n²), O(1) (d) O(n), O(n)

10. Which tree does **not** guarantee O(log n) operations?

    (a) AVL (b) **naive BST** (c) red-black (d) B+-tree

11. The in-order successor of a node in a BST is:

    (a) its parent (b) **the smallest value in its right subtree** (c) the largest value in its left subtree (d) always a leaf

12. A **B+-tree** is preferred for databases because:

    (a) it's simpler (b) **its high branching factor reduces disk I/O** (c) it uses less memory (d) it's faster on CPU

### B. Fill in the blank

1. The **height** of a leaf in a binary tree is **\_\_\_\_\_**.

2. A binary tree with $n$ nodes has minimum height **\_\_\_\_\_** and maximum height **\_\_\_\_\_**.

3. In-order traversal of a BST gives values in **\_\_\_\_\_** order.

4. BST search is O(\_\_\_\_\_) where h is the tree height.

5. An **AVL tree** maintains balance factor in **\_\_\_\_\_** for every node.

6. The **LR rotation** consists of a **\_\_\_\_\_** rotation on the left child followed by a **\_\_\_\_\_** rotation on the original root.

7. In a 0-indexed array heap, the parent of index `i` is at index **\_\_\_\_\_**.

8. Insert into a heap: append, then **\_\_\_\_\_**.

9. Extract-min: move last element to root, then **\_\_\_\_\_**.

10. **Heapify** builds a heap from n elements in **O(\_\_\_\_\_)** time.

### C. Define / explain

1. Define a **tree**. What distinguishes a tree from a general graph?

2. State the **BST property**. Show that in-order traversal of a BST gives sorted order.

3. **Insertion sort** on an array has O(n²) worst case. Does a **BST-based sort** have the same problem? Explain.

4. Describe the **four AVL rotation cases** (LL, RR, LR, RL). Which rotation does each one apply?

5. Prove that a **binary heap** with n elements has height ⌊log₂ n⌋.

6. Explain why **building a heap bottom-up** is O(n), not O(n log n).

7. Describe the **three cases of BST deletion**. Which case requires the in-order successor?

8. Compare a **BST** and a **heap** on: ordering, operations supported, time complexity of each.

9. What is a **B+-tree**? Why is it preferred over a BST in databases?

10. For each scenario, choose the best tree structure and justify: (a) storing sorted data with fast lookup; (b) priority queue; (c) DB index; (d) sorting in-place.

---

## Answer key

??? success "A. Multiple choice"
    1. **(c)** sorted order.
    2. **(b)** O(log n).
    3. **(c)** O(n).
    4. **(c)** left then right.
    5. **(b)** each node ≥ children.
    6. **(c)** 2i+1.
    7. **(b)** O(log n).
    8. **(c)** O(n).
    9. **(a)** O(n log n), O(1).
    10. **(b)** naive BST.
    11. **(b)** smallest in right subtree.
    12. **(b)** high branching factor reduces disk I/O.

??? success "B. Fill in the blank"
    1. 0
    2. ⌊log₂ n⌋; n−1
    3. ascending (sorted)
    4. $h$
    5. {−1, 0, +1}
    6. left; right
    7. `(i - 1) / 2`
    8. sift-up
    9. sift-down
    10. $n$

??? success "C. Define / explain — model answers"
    1. *Tree:* a **connected acyclic graph**. *Distinguished from a general graph:* trees have no cycles (unique path between any two nodes), and if we root the tree at one node, every node has a unique parent. A graph may have cycles, may be disconnected, and has no inherent root/parent structure. Any n-node tree has exactly n−1 edges.
    2. *BST property:* for every node, all values in the left subtree are strictly less than the node's value, and all values in the right subtree are strictly greater. *In-order sorted proof:* by induction on tree structure. For a leaf, the (empty) left, the leaf value, and the (empty) right give sorted output trivially. For an internal node, the left subtree's in-order traversal is sorted and contains all values < node; the right subtree's is sorted and contains all values > node. Concatenating (left, node, right) gives a fully sorted sequence.
    3. *BST-based sort:* insert all elements into a BST, then in-order traverse. **Cost:** n insertions, each O(h). For a balanced BST, O(n log n). For a **skewed** BST (sorted insertion), O(n²). **So yes, BST sort has the same worst case as insertion sort.** Fix: use an AVL or red-black tree → O(n log n) worst case (this is "tree sort"). In practice, heapsort or merge sort are preferred because they don't need pointer overhead.
    4. *Four AVL rotation cases:*
       - **LL** — the new node went into the left subtree's left subtree. Fix: **single right rotation**.
       - **RR** — the new node went into the right subtree's right subtree. Fix: **single left rotation**.
       - **LR** — the new node went into the left subtree's right subtree. Fix: **left rotation on the left child, then right rotation on the root**.
       - **RL** — the new node went into the right subtree's left subtree. Fix: **right rotation on the right child, then left rotation on the root**.
    5. *Heap height:* a heap is a **complete** binary tree — all levels full except possibly the last. Level 0 holds 1 node, level 1 holds up to 2, level $k$ holds up to $2^k$. Total nodes up to level $h$: $\sum_{k=0}^{h} 2^k = 2^{h+1} - 1$. If a heap has $n$ nodes, then $n < 2^{h+2}$ and $n \ge 2^{h} - 1$. Simplifying: $h = \lfloor \log_2 n \rfloor$. **The height is the floor of log₂ n.**
    6. *O(n) heapify:* sift-down is called once for each internal node, from the last internal node up to the root. A node at height $h$ performs at most $h$ swaps. The number of nodes at height $h$ is $\lceil n/2^{h+1} \rceil$. Total work: $\sum_{h=0}^{\log n} \lceil n/2^{h+1} \rceil \cdot h \le n \sum_{h=0}^{\log n} h/2^{h} = O(n)$ — the geometric series $\sum h/2^h = 2$ converges. **The number of nodes decreases exponentially with height, cancelling the linear per-node work.**
    7. *BST deletion, three cases:*
       - **Leaf** (no children) — remove directly.
       - **One child** — splice the child up.
       - **Two children** — replace the node's value with its **in-order successor** (smallest value in its right subtree), then delete the successor (which has at most one child — reducing to case 1 or 2).
    8. *BST vs. heap:*
       - **Ordering:** BST has ordering across all values; heap has ordering only parent-to-child.
       - **Operations:** BST supports search/insert/delete/predecessor/successor/range in O(h); heap supports insert/extract-min/peek-min and that's roughly it.
       - **Time:** BST O(h) per op (worst O(n)); balanced BST O(log n); heap O(log n) insert/extract, O(1) peek.
       - **Space:** heap is array-based (compact, cache-friendly); BST needs pointers per node.
    9. *B+-tree:* a balanced multiway search tree with high branching factor (typically hundreds of children per node). All values stored in leaves; leaves linked for sequential access. *Why for DBs:* the branching factor makes height very small — typically **height 3** holds millions of records. Each tree level corresponds to one disk read, so lookups cost **~3 disk reads** instead of ~20 for a BST. Range queries are efficient because of the linked-leaf structure.
    10. *Choice of structure:*
        - (a) **Sorted data with fast lookup:** balanced BST (AVL or red-black). **Not** a plain BST — sorted insertion would degrade to O(n).
        - (b) **Priority queue:** binary heap. **Not** a BST — heap's O(1) peek and O(log n) extract-min are optimal for priority ordering; BST's log n for min is worse and it has unnecessary ordering.
        - (c) **DB index:** B+-tree. **Not** a BST — the high branching factor reduces disk I/O dramatically.
        - (d) **Sorting in-place:** heapsort (which uses a heap). **Not** a BST — a BST sorts in O(n log n) but uses O(n) extra memory for pointers; heapsort is O(1) extra.

---

## Further reading

- **Weiss**, Chapters 4–6 (Trees, BST, AVL, B-trees, hash tables).
- **Lafore**, Chapters 8–11 (Binary trees, red-black, heaps, B-trees).
- **CLRS**, Chapters 6 (Heaps), 12 (BST), 13 (Red-black), 18 (B-trees).
- **Sedgewick & Wayne**, Chapter 3 (Search trees).
- **Open Data Structures**, Chapters 6 (Binary trees), 7 (Random BST), 8 (Treaps), 10 (Heaps) — rigorous with full implementations.
- **VisuAlgo** — [visualgo.net](https://visualgo.net/) — animated BST, AVL rotations, heap operations.
- **"The Anatomy of a Large-Scale Hypertextual Web Search Engine"** (Brin & Page, 1998) — how Google used BSTs for URL lookups.
- **PostgreSQL documentation**, "Indexes" — how B+-trees work in a real database.

---

## Suggested tutorial problems

1. **In-order traversal.** Given tree with root 8, children 3 (with 1, 6) and 10 (with 14), what is the in-order traversal? Pre-order? Post-order?

2. **BST insert trace.** Insert `50, 30, 70, 20, 40, 60, 80` into an empty BST. Draw the tree. Verify in-order traversal.

3. **BST degenerate case.** Insert `1, 2, 3, 4, 5` in that order. Draw the tree. What is its height? Compare with the height of an AVL built from the same keys.

4. **BST delete.** From the tree in problem 2, delete 30. Which case does this fall into? Trace the resulting tree.

5. **BST delete, two children.** Delete 50 from the tree in problem 2. Which case? Find the in-order successor. Show the tree before and after.

6. **AVL RR rotation.** Build an AVL by inserting `10, 20, 30`. Which rotation? Trace the rotation on the board.

7. **AVL LR rotation.** Insert `30, 10, 20` in that order. Which case? Perform the rotation step by step.

8. **AVL insertion trace.** Build an AVL by inserting `1..10` in sorted order. After each insertion, show the tree. How many rotations in total?

9. **Heap insert.** Insert `10, 20, 5, 3, 15, 8` into an empty min-heap. Show the array and sift-up at each step.

10. **Heap extract.** Extract the minimum from the heap in problem 9. Show the sift-down.

11. **Heapify.** Convert `[5, 8, 3, 9, 1, 7, 2]` into a min-heap using bottom-up heapify. Show the array after each `siftDown` call.

12. **Heapsort.** Sort `[4, 1, 3, 2, 16, 9, 10, 14, 8, 7]` using heapsort. Show the array after each extract-max step.

13. **Max vs. min heap.** Explain how to find the **maximum** in a min-heap (in linear time, not O(1)). How would you find it in a max-heap?

14. **BST validity.** Design an algorithm to check whether a given binary tree is a valid BST. Give the O(n) recursive approach. What's a common bug in this problem?

15. **In-order to tree.** Given the in-order and pre-order traversals of a binary tree, reconstruct the tree. Show on `in-order = [4, 2, 5, 1, 3]`, `pre-order = [1, 2, 4, 5, 3]`.

16. **Level-order to tree.** Given a level-order traversal, can you always reconstruct the tree? Why or why not?

17. **B+-tree motivation.** A database holds 1 million records. Compare the number of disk reads needed to find one record using: (a) a sorted array with binary search, (b) a binary search tree, (c) a B+-tree with branching factor 200. Assume each tree node = 1 disk read, and the sorted array stores each record contiguously (so binary search = log₂(10⁶) ≈ 20 disk reads).