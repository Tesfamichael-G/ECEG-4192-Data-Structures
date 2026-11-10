# Chapter 3 — Linked Lists

!!! abstract "Chapter at a glance"
    **Contact hours:** 2 lecture + 1 tutorial + 2 lab · **Home study:** ~5 h
    
    **Coverage (syllabus):** memory allocation · operations in linked lists · variations
    of linked lists: circular, doubly-linked.

---

## Learning outcomes

By the end of this chapter you should be able to:

1. Explain the **node-and-pointer** memory model that underlies linked lists.
2. Implement a **singly-linked list** with insert, delete, search, and traverse.
3. Implement a **doubly-linked list** and explain when the extra pointer pays off.
4. Implement a **circular linked list** and identify its applications.
5. **Reverse**, **merge**, and **sort** a linked list.
6. Detect cycles and find the middle of a linked list using **Floyd's algorithm**.
7. Compare linked lists to arrays with a full operation-by-operation cost table.
8. Justify choosing linked list over array (or vice versa) for a given problem.

---

## 3.1 Node structure and pointers

### 3.1.1 The node

A **node** holds one value and (at least) one pointer to the next node:

```java
class Node {
    int  value;
    Node next;   // reference to the next node, or null

    Node(int v) { this.value = v; this.next = null; }
}
```

In memory, a linked list is **not contiguous**. Each node is a separate heap allocation:

```
Heap:
  [Node{value:10, next:→}]  at address 0x1000
  [Node{value:20, next:→}]  at address 0x2050
  [Node{value:30, next:→}]  at address 0x3f00
  ...
```

The `next` pointer stores the address of the following node. The list is formed by **following** these pointers from the **head**:

```
head → 10 → 20 → 30 → null
```

### 3.1.2 The null terminator

The last node's `next` is **null** — the standard signal for "end of list." Every traversal loop must check for `null` before dereferencing:

```java
Node cur = head;
while (cur != null) {
    System.out.println(cur.value);
    cur = cur.next;
}
```

**Null pointer bugs are the dominant bug class in linked-list code.** Forget the null check, and you'll get `NullPointerException` at runtime (Java) or a segfault (C).

### 3.1.3 Why not just use an array?

Different layout → different costs:

| Operation | Array | Linked list |
|---|:---:|:---:|
| Access by index `a[i]` | **O(1)** | O(n) |
| Insert at front | O(n) | **O(1)** |
| Insert at end | **O(1)** | O(1) with tail pointer |
| Delete at front | O(n) | **O(1)** |
| Cache locality | **Excellent** | Poor |

The linked list trades **random access** for **cheap insertion and deletion at known positions**. Chapter 3 is about knowing when this trade is worth it.

---

## 3.2 Singly-linked list — operations

### 3.2.1 Insert at head (front)

```java
Node head = null;

void insertAtHead(int x) {
    Node n = new Node(x);
    n.next = head;      // new node points to old head
    head = n;            // update head pointer
}
```

**Cost: O(1).** Regardless of list length.

**Trace.** Insert 30, 20, 10 in that order:

| Step | List |
|:---:|---|
| 0 | `null` |
| Insert 30 | `30 → null` |
| Insert 20 | `20 → 30 → null` |
| Insert 10 | `10 → 20 → 30 → null` |

The **order of insertion is reversed** in the final list — an important consequence of insert-at-head.

### 3.2.2 Insert at tail (end) — without tail pointer

```java
void insertAtTail(int x) {
    Node n = new Node(x);
    if (head == null) {
        head = n;
        return;
    }
    Node cur = head;
    while (cur.next != null) cur = cur.next;   // walk to last
    cur.next = n;
}
```

**Cost: O(n)** — must traverse to the end every time.

### 3.2.3 Insert at tail — with tail pointer

```java
Node head = null, tail = null;

void insertAtTail(int x) {
    Node n = new Node(x);
    if (head == null) { head = n; tail = n; return; }
    tail.next = n;
    tail = n;
}
```

**Cost: O(1).** The extra `tail` pointer costs one reference (8 bytes), saves O(n) per append.

### 3.2.4 Delete by value

```java
void delete(int x) {
    if (head == null) return;
    if (head.value == x) { head = head.next; return; }
    Node cur = head;
    while (cur.next != null && cur.next.value != x) cur = cur.next;
    if (cur.next != null) cur.next = cur.next.next;   // skip over the match
}
```

**Cost: O(n)** — search for the value, then O(1) to unlink.

**Careful trace.** List `10 → 20 → 30 → null`, delete 20:

- `cur = 10`; `cur.next.value = 20 == x` → stop.
- `cur.next = cur.next.next` → `cur.next = 30`.
- Result: `10 → 30 → null` ✓

### 3.2.5 Traverse — O(n)

```java
void print() {
    for (Node cur = head; cur != null; cur = cur.next) {
        System.out.println(cur.value);
    }
}
```

**O(n) time, O(1) space.**

### 3.2.6 Search — O(n)

```java
boolean contains(int x) {
    for (Node cur = head; cur != null; cur = cur.next) {
        if (cur.value == x) return true;
    }
    return false;
}
```

**O(n) worst case.** Even with tail pointer, search requires walking.

### 3.2.7 Full cost table (singly-linked)

| Operation | Cost |
|---|:---:|
| Insert at head | **O(1)** |
| Insert at tail (no tail) | O(n) |
| Insert at tail (with tail) | **O(1)** |
| Insert after given node | **O(1)** |
| Delete at head | **O(1)** |
| Delete by value | O(n) |
| Access by index | O(n) |
| Search | O(n) |
| Traverse | O(n) |

**The pattern:** anything involving the **head** is O(1); anything requiring **reaching a specific position** is O(n).

---

## 3.3 Doubly-linked list

### 3.3.1 Node structure

A doubly-linked node adds a `prev` pointer:

```java
class Node {
    int  value;
    Node prev;
    Node next;
    Node(int v) { this.value = v; }
}
```

The list can be traversed **both directions**:

```
null ← 10 ⇄ 20 ⇄ 30 → null
```

### 3.3.2 Why the extra pointer pays off

With `prev`, you can:

- **Traverse backward** — useful for reverse iteration.
- **Delete a node given a pointer to it** in O(1) — no need to walk to find the predecessor.

The last is the big one. In a singly-linked list, deleting a node `x` requires finding its predecessor — O(n). In a doubly-linked list, `x.prev` gives it directly.

```java
void deleteNode(Node x) {
    if (x.prev != null) x.prev.next = x.next;
    else                head = x.next;      // x was head
    if (x.next != null) x.next.prev = x.prev;
    // x is now unlinked; Java GC collects it
}
```

**O(1).** This is why a doubly-linked list is used for **LRU caches** — where nodes are constantly moved and removed by pointer.

### 3.3.3 Insert after a node

```java
void insertAfter(Node x, int val) {
    Node n = new Node(val);
    n.next = x.next;
    n.prev = x;
    if (x.next != null) x.next.prev = n;
    x.next = n;
}
```

**O(1).** Four pointer updates.

### 3.3.4 Cost table (doubly-linked)

| Operation | Singly | Doubly |
|---|:---:|:---:|
| Insert at head | O(1) | O(1) |
| Insert at tail | O(1) with tail | O(1) |
| Insert after node | O(1) | O(1) |
| **Delete given node** | **O(n)** | **O(1)** |
| Traverse forward | O(n) | O(n) |
| Traverse backward | — | O(n) |

**Extra cost:** two pointers per node (16 bytes vs. 8 on 64-bit systems) — **double the memory**.

---

## 3.4 Circular linked list

### 3.4.1 Structure

The last node's `next` points back to the **head** instead of `null`:

```
10 → 20 → 30 → 10 → 20 → 30 → ...
↑                              │
└──────────────────────────────┘
```

### 3.4.2 When to use a circular list

- **Round-robin scheduling** — a scheduler cycles through processes; after the last, wraps to the first.
- **Circular buffers** — streaming audio/video, where data wraps around.
- **Josephus problem** — a group elimination game.
- **Anywhere "next after last" means "first"** — the pointer structure naturally models this.

### 3.4.3 Careful — no null terminator

Circular lists have **no null** to signal the end. Traversal must **stop when it returns to the head** — otherwise it loops forever:

```java
void printCircular(Node head) {
    if (head == null) return;
    Node cur = head;
    do {
        System.out.println(cur.value);
        cur = cur.next;
    } while (cur != head);
}
```

The `do...while` guarantees at least one iteration, and stops when `cur` returns to `head`.

---

## Worked example 3.1 — Insert at head, step by step

**Goal.** Build `10 → 20 → 30` by inserting at head.

**Code:**

```java
void insertAtHead(int x) {
    Node n = new Node(x);
    n.next = head;
    head = n;
}
```

**Trace.**

**Insert 10:** `n = Node(10)`, `n.next = null`, `head = n`. List: `10 → null`.

**Insert 20:** `n = Node(20)`, `n.next = head (10)`, `head = n (20)`. List: `20 → 10 → null`.

**Insert 30:** `n = Node(30)`, `n.next = head (20)`, `head = n (30)`. List: `30 → 20 → 10 → null`.

**Final list:** `30 → 20 → 10 → null` — **reversed** relative to insertion order.

**Cost.** Three insertions, each **O(1)**. **Total: O(1) regardless of list length.**

**Contrast with array.** To build `[30, 20, 10]` by inserting at the front of an array: each insertion shifts all existing elements → O(n) per insert → O(n²) for n inserts. **Linked list is n× faster here.**

---

## Worked example 3.2 — Insert at tail with tail pointer

**Goal.** Build `10 → 20 → 30` in original order.

**Code:**

```java
Node head = null, tail = null;

void insertAtTail(int x) {
    Node n = new Node(x);
    if (head == null) { head = n; tail = n; return; }
    tail.next = n;
    tail = n;
}
```

**Trace.**

**Insert 10:** `head = tail = Node(10)`. List: `10 → null`.

**Insert 20:** `tail.next = Node(20)` (so `10.next = 20`); `tail = Node(20)`. List: `10 → 20 → null`.

**Insert 30:** `tail.next = Node(30)` (so `20.next = 30`); `tail = Node(30)`. List: `10 → 20 → 30 → null`.

**Final list:** `10 → 20 → 30 → null` — **original order** preserved.

**Cost.** Three insertions, each **O(1)**. **Total: O(1)** — but requires the extra `tail` pointer.

**Trade-off.** Without the tail pointer, insert-at-tail is O(n) per call; with it, O(1). The tail pointer is worth 8 bytes of memory in almost every scenario where you append.

---

## Worked example 3.3 — Delete by value — full trace

**Goal.** Delete 20 from `10 → 20 → 30 → 40 → null`.

```java
void delete(int x) {
    if (head == null) return;
    if (head.value == x) { head = head.next; return; }
    Node cur = head;
    while (cur.next != null && cur.next.value != x) cur = cur.next;
    if (cur.next != null) cur.next = cur.next.next;
}
```

**Trace.**

- `head = 10`. `10.value == 20`? No.
- `cur = 10`. `cur.next.value = 20 == 20`? No, we're checking `cur.next.value`. Wait — `cur.next = Node(20)`, so `cur.next.value = 20`. Match! Exit loop.

Wait — that's the check `cur.next.value != x` — we exit when `cur.next.value == x`, i.e., when `cur.next` is the node to delete.

- `cur = 10`. `cur.next = Node(20)`, `cur.next.value = 20`. Match → exit loop.
- `cur.next = cur.next.next` → `cur.next = Node(30)`.
- List: `10 → 30 → 40 → null`.

**Correct.**

**Cost.** 1 comparison of `head`, 1 comparison of `cur.next`, 1 pointer update. **Total: O(n) worst case** — if the target is at the end, `cur` must walk the whole list.

**Common bug.** Forgetting to handle `head` as a special case. If `x = 10` (head), the loop condition `cur.next.value != x` will never fire — `cur` starts at head, and `head.value == x` is not checked in the loop. The special case at the top is mandatory.

---

## Worked example 3.4 — Reverse a singly-linked list

**Goal.** Reverse `1 → 2 → 3 → 4 → 5 → null` in place → `5 → 4 → 3 → 2 → 1 → null`.

**Approach 1 — iterative three-pointer trick.**

```java
Node reverse(Node head) {
    Node prev = null, cur = head;
    while (cur != null) {
        Node next = cur.next;   // save next
        cur.next = prev;        // reverse current's pointer
        prev = cur;             // advance prev
        cur = next;             // advance cur
    }
    return prev;                 // new head
}
```

**Trace on `1 → 2 → 3 → null`:**

| Iteration | `prev` | `cur` | `cur.next` (saved) | After `cur.next = prev` | After advance |
|:---:|---|---|---|---|---|
| 0 | null | Node(1) | Node(2) | 1 → null | prev=1, cur=2 |
| 1 | Node(1) | Node(2) | Node(3) | 2 → 1 | prev=2, cur=3 |
| 2 | Node(2) | Node(3) | null | 3 → 2 | prev=3, cur=null |
| exit | Node(3) | null | — | — | return prev (3) |

**Final list:** `3 → 2 → 1 → null` — reversed. **Return `prev` as new head.** ✓

**Cost.** One pass, each node visited once. **O(n) time, O(1) space.**

**Approach 2 — recursive.**

```java
Node reverse(Node head) {
    if (head == null || head.next == null) return head;
    Node newHead = reverse(head.next);
    head.next.next = head;
    head.next = null;
    return newHead;
}
```

**Cost.** O(n) time, **O(n) stack space** (recursion depth = n). The iterative version is preferred for large lists.

**Why reversal is easy on linked lists but O(n) on arrays.** Reversing a linked list just flips pointers — no data movement. Reversing an array requires swapping `n/2` pairs. Both are O(n) time, but the **constant is much smaller** for linked lists (3 pointer writes per node vs. 2 array writes + 2 reads per pair).

---

## Worked example 3.5 — Merge two sorted lists

**Goal.** Given two sorted lists `1 → 3 → 5` and `2 → 4 → 6`, produce `1 → 2 → 3 → 4 → 5 → 6`.

```java
Node merge(Node a, Node b) {
    Node dummy = new Node(0);
    Node tail = dummy;
    while (a != null && b != null) {
        if (a.value <= b.value) { tail.next = a; a = a.next; }
        else                    { tail.next = b; b = b.next; }
        tail = tail.next;
    }
    tail.next = (a != null) ? a : b;   // append remaining
    return dummy.next;
}
```

**Trace on `1 → 3 → 5` and `2 → 4 → 6`:**

| Step | `a` | `b` | `tail.next` | Output so far |
|:---:|---|---|---|---|
| init | 1 | 2 | null | `dummy` |
| 1 | 3 | 2 | 1 | `dummy → 1` |
| 2 | 3 | 4 | 2 | `→ 1 → 2` |
| 3 | 5 | 4 | 3 | `→ 1 → 2 → 3` |
| 4 | 5 | 6 | 4 | `→ 1 → 2 → 3 → 4` |
| 5 | null | 6 | 5 | `→ 1 → 2 → 3 → 4 → 5` |
| exit | null | 6 | — | append remainder: `→ 6` |

**Final:** `1 → 2 → 3 → 4 → 5 → 6 → null` ✓

**Cost.** Each node visited once. **O(m + n) time, O(1) space** (reuses existing nodes).

**The `dummy` node trick.** Using `Node dummy = new Node(0)` avoids the "first iteration sets head" special case. The dummy is discarded (`return dummy.next`) — the actual data starts from `dummy.next`.

This is **the most important linked-list idiom** in competitive programming and production. Learn it, use it, teach it.

**Why merge is easier on linked lists than arrays.** Merging two arrays requires shifting to make room for the merged element — O(n²) if done naively. Merging two **sorted** arrays is O(m + n) with a new output array, but O(m + n) extra space. Merging two **linked lists** is O(m + n) time and O(1) space — just splice pointers.

---

## Worked example 3.6 — Detect a cycle (Floyd's tortoise and hare)

**Problem.** Given a linked list, determine whether it contains a **cycle** (a node whose `next` eventually points back to an earlier node).

**Naive approach.** Use a `HashSet` of visited nodes; detect duplicates. **O(n) time, O(n) space.**

**Floyd's algorithm — O(1) space.**

```java
boolean hasCycle(Node head) {
    Node slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

**Why it works.** If there's a cycle, the `slow` pointer (1 step/iter) and `fast` (2 steps/iter) eventually meet **inside the cycle**. If there's no cycle, `fast` reaches `null` and the loop terminates.

**Trace with a cycle** on `1 → 2 → 3 → 4 → 2 → (3 → 4 → 2 → ...)`:

| Step | `slow` | `fast` |
|:---:|---|---|
| init | 1 | 1 |
| 1 | 2 | 3 |
| 2 | 3 | 2 (wrapped) |
| 3 | 4 | 4 |
| 4 | 2 | 3 |
| 5 | 3 | 2 |
| 6 | 4 | 4 — **match!** return true |

**Cost.** O(n) time, O(1) space.

**The tortoise and hare insight.** In a cycle of length $c$, once both pointers are in the cycle, the gap between them decreases by 1 per iteration (fast gains 1 on slow). After at most $c$ iterations, they meet. **Total time:** O(n + c) = O(n).

**Extension.** Once they meet, to find the **start of the cycle**: reset `slow` to `head`; advance both `slow` and `fast` one step at a time. They meet at the cycle start. This follows from the fact that the distance from head to cycle-start equals the distance from meeting point to cycle-start.

---

## Worked example 3.7 — Find the middle of a linked list

**Problem.** Given a linked list, find the middle node in one pass.

```java
Node middle(Node head) {
    Node slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}
```

**Trace on `1 → 2 → 3 → 4 → 5` (odd length):**

| Step | `slow` | `fast` |
|:---:|---|---|
| init | 1 | 1 |
| 1 | 2 | 3 |
| 2 | 3 | 5 |
| exit (fast.next = null) | **3** | 5 |

**Middle is node 3.** ✓

**Trace on `1 → 2 → 3 → 4` (even length):**

| Step | `slow` | `fast` |
|:---:|---|---|
| init | 1 | 1 |
| 1 | 2 | 3 |
| 2 | 3 | null |
| exit | **3** | null |

**Middle is node 3** — the "upper middle" of 2 and 3. If you want the lower middle, initialize `fast = head.next` instead.

**Cost.** O(n) time, O(1) space.

**This two-pointer pattern** — `slow` + `fast` — is one of the most reusable tools for linked-list problems. Floyd's cycle detection (Example 3.6), middle-finding (this example), and palindrome-checking all use it.

---

## Worked example 3.8 — Doubly-linked list: LRU cache

**LRU (Least Recently Used) cache** is the classic application of a doubly-linked list. The cache keeps the `k` most recently used items. When full, it evicts the **least** recently used.

**Design.**

- **HashMap:** key → node, for O(1) lookup.
- **Doubly-linked list:** maintains recency order — head = most recent, tail = least recent.

**Operations:**

- `get(key)`: if key exists, move node to head (mark as recently used). O(1).
- `put(key, value)`: if key exists, update and move to head. If new and cache full, evict tail. Insert new node at head. O(1).

```java
class LRUCache {
    class Node { int key, value; Node prev, next; }
    
    Map<Integer, Node> map = new HashMap<>();
    Node head = new Node(), tail = new Node();   // sentinel nodes
    int capacity;
    
    LRUCache(int cap) {
        capacity = cap;
        head.next = tail;
        tail.prev = head;
    }
    
    int get(int key) {
        Node n = map.get(key);
        if (n == null) return -1;
        moveToHead(n);
        return n.value;
    }
    
    void put(int key, int value) {
        Node n = map.get(key);
        if (n != null) {
            n.value = value;
            moveToHead(n);
        } else {
            if (map.size() == capacity) {
                Node lru = tail.prev;
                removeNode(lru);
                map.remove(lru.key);
            }
            Node nn = new Node();
            nn.key = key; nn.value = value;
            insertAtHead(nn);
            map.put(key, nn);
        }
    }
    
    void moveToHead(Node n) { removeNode(n); insertAtHead(n); }
    void removeNode(Node n)  { n.prev.next = n.next; n.next.prev = n.prev; }
    void insertAtHead(Node n) {
        n.next = head.next; n.prev = head;
        head.next.prev = n; head.next = n;
    }
}
```

**Cost.** Every operation is **O(1)**. The doubly-linked list gives O(1) removal of any node; the hash map gives O(1) lookup.

**Why not just a hash map alone?** A hash map is unordered — it can't tell you which entry is "least recently used." The doubly-linked list maintains the ordering. Together, they provide O(1) LRU semantics.

**Why not just a linked list alone?** Finding a node by key would be O(n). The hash map provides O(1) access to the node.

**This pattern — hash map + doubly-linked list — is used in every LRU cache in production**, from Redis to Linux page caches to web browsers.

---

## Worked example 3.9 — Josephus problem with a circular list

**Setup.** $n$ people numbered 1 to $n$ sit in a circle. Starting from 1, we skip $k-1$ people and eliminate the $k$-th. Then from the next person, skip $k-1$ and eliminate the $k$-th. Repeat until one survives.

**Example:** $n = 7$, $k = 3$ → survivors eliminated in order: 3, 6, 2, 7, 5, 1. Survivor: 4.

**With a circular linked list:**

```java
int josephus(int n, int k) {
    Node head = null, tail = null;
    for (int i = 1; i <= n; i++) {
        Node nn = new Node(i);
        if (head == null) { head = nn; tail = nn; }
        else { tail.next = nn; tail = nn; }
    }
    tail.next = head;   // make it circular
    
    Node cur = head, prev = tail;
    while (cur.next != cur) {
        for (int i = 0; i < k - 1; i++) {
            prev = cur;
            cur = cur.next;
        }
        // eliminate cur
        prev.next = cur.next;
        cur = cur.next;
    }
    return cur.value;
}
```

**Trace (n=3, k=2):**

Initial circle: `1 → 2 → 3 → (back to 1)`.

- Skip 1, eliminate 2: `1 → 3 → (back to 1)`.
- Skip 3, eliminate 1: `3 → (back to 3)`.
- Survivor: **3**.

**Closed form (for the record):** the survivor is $J(n, k) = (J(n-1, k) + k) \bmod n$ with $J(1, k) = 0$, giving positions 0-indexed. The linked-list simulation is what most candidates write in interviews, then the recursion.

**Cost.** Each elimination is O(k) (skipping), total O(nk). The recursion is O(n) — but the linked-list version is easier to understand and to verify.

---

## 3.5 Comparison — arrays vs. linked lists

| Operation | Array | Linked list |
|---|:---:|:---:|
| **Random access `a[i]`** | **O(1)** | O(n) |
| Search (unsorted) | O(n) | O(n) |
| Search (sorted) | **O(log n)** | O(n) |
| Insert at front | O(n) | **O(1)** |
| Insert at end | **O(1)** amortised | O(1) with tail |
| Insert in middle (position known) | O(n) | **O(1)** |
| Insert in middle (must find) | O(n) | O(n) |
| Delete at front | O(n) | **O(1)** |
| Delete given node pointer | — | **O(1)** (doubly) |
| Cache locality | **Excellent** | Poor |
| Memory overhead | **None** | 8–16 bytes per node |

**How to choose:**

- **Random access dominates?** → Array.
- **Insert/delete at front dominates?** → Linked list.
- **Insert/delete in middle dominates and you can find the node cheaply?** → Doubly-linked list.
- **Sorted lookup dominates?** → Array (binary search) or balanced tree (Chapter 5).
- **Cache friendliness matters?** → Array — 10–100× faster in wall-clock for linear scans.

---

## Chapter summary

1. A **node** holds a value and one or two **pointers** (`next`, `prev`). The list is formed by following pointers from the **head**.
2. **Singly-linked lists** support O(1) insert at head, O(1) insert at tail with a tail pointer, O(n) search, and O(n) random access.
3. **Doubly-linked lists** add a `prev` pointer, enabling O(1) delete of a given node — the basis of **LRU caches**.
4. **Circular lists** have no null terminator; traversal stops when it returns to the head.
5. **Reversal, merge, cycle detection, and middle-finding** are the classic patterns — all use the two-pointer trick.
6. The **`dummy` node** trick eliminates special-casing the head — learn it and use it everywhere.
7. **Arrays win on random access and cache locality**; linked lists win on insertion/deletion at known positions.
8. Choose the right structure based on the **operation mix**, not on a generic preference.

---

## Quiz

### A. Multiple choice

1. In a singly-linked list, `insert at head` costs:

   (a) O(1) (b) O(log n) (c) O(n) (d) O(n²)

2. In a singly-linked list **without** a tail pointer, `insert at tail` costs:

   (a) O(1) (b) O(log n) (c) O(n) (d) O(n²)

3. Adding a `prev` pointer to each node lets you delete a given node in:

   (a) O(1) (b) O(log n) (c) O(n) (d) O(n²)

4. Floyd's cycle-detection algorithm uses:

   (a) a hash set (b) a stack (c) two pointers at different speeds (d) recursion

5. Reversing a singly-linked list in place takes:

   (a) O(1) time, O(1) space (b) O(n) time, O(1) space
   (c) O(n) time, O(n) space (d) O(n log n) time, O(1) space

6. The `dummy` node trick is used to:

   (a) save memory (b) eliminate the special case of the first insertion
   (c) avoid null checks (d) make the list circular

7. Merging two sorted lists of sizes m and n takes:

   (a) O(m + n) time, O(m + n) space (b) O(m + n) time, O(1) space
   (c) O(m · n) time (d) O(log m + log n) time

8. In a **circular** linked list, traversal must stop when:

   (a) `next == null` (b) the pointer returns to head (c) a counter reaches n
   (d) the value becomes negative

9. In Floyd's algorithm, the two pointers advance at:

   (a) the same speed (b) 1 and 2 steps per iteration (c) 1 and log n steps
   (d) 1 and n steps

10. The LRU cache uses which combination?

    (a) array + hash map (b) doubly-linked list + hash map
    (c) stack + queue (d) two arrays

11. Deleting the **first** element of a singly-linked list costs:

    (a) O(1) (b) O(log n) (c) O(n) (d) O(n²)

12. Accessing the **i-th** element of a linked list costs:

    (a) O(1) (b) O(log n) (c) O(i) → O(n) worst case (d) O(n²)

### B. Fill in the blank

1. A node in a linked list contains a value and a **\_\_\_\_** pointer.

2. Inserting at head of a singly-linked list costs **O(\_\_\_\_\_)**.

3. The last node's `next` is **\_\_\_\_\_** in a non-circular singly-linked list.

4. A doubly-linked list node has both `next` and **\_\_\_\_\_** pointers.

5. Floyd's algorithm runs in O(n) time and **O(\_\_\_\_\_** space.

6. The **\_\_\_\_\_** node trick eliminates special-casing when building a new list.

7. LRU cache combines a hash map with a **\_\_\_\_\_** linked list.

8. In a circular linked list, the last node's `next` points to the **\_\_\_\_\_**.

9. Reversing a singly-linked list uses **\_\_\_\_\_** pointers (`prev`, `cur`, `next`).

10. Finding the middle with two pointers uses one that moves 1 step and one that moves **\_\_\_\_\_** steps.

### C. Define / explain

1. Explain the **node-and-pointer** model. Why is a linked list's memory scattered, not contiguous?

2. Compare the **cost of `insert at head`** in an array vs. a singly-linked list. Why does the linked list win by a factor of n?

3. Explain why **`insert at tail`** is O(n) without a tail pointer but O(1) with one. What is the trade-off?

4. Describe the **`dummy` node** trick. Give one scenario where it eliminates nontrivial bugs.

5. Explain the **tortoise-and-hare** algorithm (Floyd's cycle detection). Prove that if a cycle exists, the two pointers must meet.

6. Compare **singly-linked** and **doubly-linked** lists on: insert at head, delete a given node, memory per node, and traversal in reverse.

7. Describe the **reverse-a-linked-list** algorithm. Why does it need three pointers?

8. Explain the **LRU cache** design. Why not just a hash map? Why not just a linked list?

9. Compare the **cost of accessing the i-th element** in an array vs. a linked list. Give one application where this difference decides the choice.

10. Explain why **cache locality** makes arrays faster than linked lists for linear scans, even though both are O(n). What is the wall-clock factor typically?

---

## Answer key

??? success "A. Multiple choice"
    1. **(a)** O(1).
    2. **(c)** O(n).
    3. **(a)** O(1).
    4. **(c)** two pointers at different speeds.
    5. **(b)** O(n) time, O(1) space.
    6. **(b)** eliminate first-insertion special case.
    7. **(b)** O(m + n) time, O(1) space (for linked lists).
    8. **(b)** returns to head.
    9. **(b)** 1 and 2.
    10. **(b)** doubly-linked + hash map.
    11. **(a)** O(1).
    12. **(c)** O(i), worst O(n).

??? success "B. Fill in the blank"
    1. `next`
    2. $O(1)$
    3. `null`
    4. `prev`
    5. $1$
    6. dummy
    7. doubly-linked
    8. head
    9. three
    10. 2

??? success "C. Define / explain — model answers"
    1. *Node-and-pointer model:* each node is a heap allocation containing a value and a pointer to the next node. The list is formed by following pointers. *Why scattered:* each node is a **separate** allocation — the allocator places them wherever there's free memory. Unlike arrays, there is no guarantee of adjacency — a node's "next" may be anywhere in the heap. This is why random access requires walking the list.
    2. *Insert at head — array vs. linked list:* array insertion at index 0 requires shifting all `n` existing elements right → O(n). Linked-list insertion creates one node, sets `n.next = head`, and updates `head` → O(1). **The n× difference** comes from the array's contiguous layout: to make room at the front, every element must move. The linked list just redirects a pointer.
    3. *Insert at tail — without tail pointer:* must walk from head to the last node, O(n). *With tail pointer:* `tail.next = newNode; tail = newNode` → O(1). *Trade-off:* the tail pointer is 8 bytes (one reference) per list; you save O(n) per append. Any application that appends more than a few times should use the tail pointer.
    4. *Dummy node:* create a throwaway `Node(0)` at the start, use it as the "previous" for building, and return `dummy.next` at the end. *Scenario:* merging two sorted lists (Example 3.5) — without the dummy, every iteration needs a conditional to check "is this the first element?" — with the dummy, the loop is uniform, no special cases, no bugs. Used everywhere in competitive programming.
    5. *Tortoise and hare:* both pointers start at head. Each iteration, `slow = slow.next` (1 step) and `fast = fast.next.next` (2 steps). *Proof of cycle detection:* if there's a cycle of length $c$, once both pointers are inside the cycle, the gap between them decreases by 1 per iteration. After at most $c$ iterations, they meet. If there's no cycle, `fast` (or `fast.next`) becomes `null` and the loop terminates. **O(n) time, O(1) space.**
    6. *Singly vs. doubly:*
       - Insert at head: O(1) both.
       - Delete given node: O(n) singly (need predecessor), **O(1) doubly** (prev pointer).
       - Memory: 8 bytes singly, 16 bytes doubly (on 64-bit).
       - Traversal in reverse: impossible singly, O(n) doubly.
    7. *Reverse:* three pointers — `prev` (last reversed node), `cur` (node being reversed), `next` (temporary save of `cur.next`). Loop: save `next = cur.next`, set `cur.next = prev`, advance `prev = cur`, `cur = next`. Return `prev`. Each node visited once, O(n) time, O(1) space.
    8. *LRU cache design:* a hash map gives O(1) key lookup, but has **no order** — it can't tell you "least recently used." A doubly-linked list maintains the recency order — head is most recent, tail is least recent. Together: O(1) get (via hash map), O(1) put (via hash map + O(1) unlink/relink on doubly-linked list). Neither structure alone suffices.
    9. *i-th element access:* array O(1) (address arithmetic); linked list O(i) (walk i steps). *Application:* a program that stores a 10-million-element temperature time-series and repeatedly accesses `a[t]` for random `t` **must use an array** — a linked list would be 10 million times slower per access. Conversely, a program that repeatedly inserts at the front of a growing list **must use a linked list** — an array would shift all elements each time.
    10. *Cache locality:* modern CPUs fetch 64-byte cache lines; an array stores 16 consecutive `int`s in one line → 1 miss, 15 hits per 16 accesses. A linked list stores each node in a separate heap allocation → **every access misses cache**. *Wall-clock factor:* typically **5–20×** slower for linked-list traversal than array traversal on the same data, despite both being O(n). This is why **Big-O is not the whole story** — constants, especially cache misses, dominate for small-to-medium data.

---

## Further reading

- **Weiss**, Chapter 3 — linked lists, stacks, queues.
- **Lafore**, Chapters 5–6 — linked list and doubly-linked list.
- **CLRS**, Section 10.2 (Linked lists).
- **Sedgewick & Wayne**, Section 1.3 — the "linked list vs. array" analysis.
- **Open Data Structures**, Chapter 3 — rigorous linked list treatment with all operations.
- **Coding interview practice:** LeetCode "Linked List" tag — 100+ problems, sorted by difficulty. Start with "Reverse Linked List," "Merge Two Sorted Lists," "Linked List Cycle."
- **"What Every Programmer Should Know About Memory"** (Ulrich Drepper) — Section 3.3 on array vs. linked list performance.

---

## Suggested tutorial problems

1. **Insert at head trace.** Build `5 → 4 → 3 → 2 → 1` by inserting `1, 2, 3, 4, 5` in order using `insertAtHead`. Trace each step.

2. **Insert at tail trace.** Same build, but using `insertAtTail` with a tail pointer.

3. **Delete all occurrences.** Given a linked list and a value `v`, remove all occurrences of `v`. Edge cases: `v` is at the head; `v` is the only value.

4. **Reverse a linked list.** Implement the iterative reverse. Then implement the recursive reverse. Compare memory usage.

5. **Merge two sorted lists.** Implement Example 3.5. Verify with `[1, 3, 5]` and `[2, 4, 6]`.

6. **Remove duplicates.** Given a **sorted** linked list, remove all duplicate nodes. Given an **unsorted** linked list, do the same in O(n log n) using a sort first.

7. **Detect and remove a cycle.** Use Floyd's algorithm to detect a cycle; then find and remove the entry point.

8. **Find the middle.** Implement `middle(head)` using the two-pointer approach. Test on odd and even lengths.

9. **Merge sort a linked list.** Implement merge sort — split by finding the middle, sort each half recursively, merge. What is the total cost, and how does it compare to sorting an array?

10. **Josephus.** Implement the circular-list Josephus. Verify $J(7, 3)$ returns 4.

11. **LRU cache.** Implement `LRUCache` with `get` and `put` in O(1). Test with a sequence of operations.

12. **Palindrome check.** Given a linked list, determine whether its values form a palindrome. Solve in O(n) time and O(1) space (hint: find middle, reverse second half, compare, then reverse back).

13. **Add two numbers.** Given two linked lists representing digits of a number (reversed order), return a linked list representing their sum. E.g., `2 → 4 → 3` + `5 → 6 → 4` = `7 → 0 → 8` (342 + 465 = 807).

14. **Copy a list with random pointers.** Each node has `next` and `random` pointers. Copy the list in O(n) time. (Hint: interleave clones with originals, then split.)

15. **Intersection of two lists.** Given two linked lists, find the node where they intersect. Solve in O(m + n) time, O(1) space. (Hint: align tails.)