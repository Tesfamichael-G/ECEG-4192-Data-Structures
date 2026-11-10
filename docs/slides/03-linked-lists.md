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
  js: assets/lecture-03.js
---

# ECEG-4192

## Chapter 3 — Linked Lists

Singly · Doubly · Circular

Semester VII · 2025/26

<aside class="notes">
Linked lists are the first structure that beats arrays on a specific workload: frequent insert/delete with pointer access.
</aside>

---

## Learning outcomes

By the end of this chapter you should be able to:

1. Explain the **motivation** for linked lists over arrays.

2. Describe a **node** and how nodes link together.

3. Implement **insert, delete, search** on a singly linked list.

4. Analyse the **cost** of each operation.

5. Distinguish **singly, doubly, circular** linked lists.

6. Reverse a linked list in **O(n)** time and **O(1)** space.

---

## 3.1 — Why linked lists?

Arrays have two pain points:

<div class="fragment">

- **Fixed size** — you must know $n$ upfront.
- **O(n) insert/delete in the middle** — shifting.

</div>

<div class="fragment">

Linked lists offer:

- **Dynamic size** — grow and shrink as needed.
- **O(1) insert/delete** — with a pointer to the right place.

</div>

<div class="fragment">

!!! warning "Trade-off"
    **No O(1) random access.**

</div>

---

## 3.2 — Node structure

```c
struct Node {
    int data;
    struct Node *next;
};
```

<div class="fragment">

Each node holds:

- **data** — the payload.
- **next** — pointer to the next node (or NULL).

</div>

<div class="fragment">

Nodes are allocated on the **heap** (via `malloc` / `new`).

They are **not contiguous** — connected by pointers.

</div>

---

## Visual layout

```
head → [10|•] → [20|•] → [30|•] → NULL
```

The **head** pointer is the entry to the list.

<div class="fragment">

Traversal is linear: follow `next` until `NULL`.

```c
Node* cur = head;
while (cur) {
    printf("%d\n", cur->data);
    cur = cur->next;
}
```

Time: **O(n)** — must visit every node.

</div>

---

## 3.3 — Singly linked list operations

| Operation | Complexity | Notes |
|---|---|---|
| **Insert at head** | **O(1)** | Rewire two pointers |
| **Insert at tail** | **O(n)** | Walk to end (unless tail pointer) |
| **Insert after node** | **O(1)** | Given a pointer |
| **Delete at head** | **O(1)** | Rewire head |
| **Delete by value** | **O(n)** | Search then unlink |
| **Search** | **O(n)** | Linear scan |
| **Access by index** | **O(n)** | Walk $i$ steps |

---

## Insert at head — O(1)

```c
Node* insert_head(Node* head, int val) {
    Node* n = malloc(sizeof(Node));
    n->data = val;
    n->next = head;
    return n;
}
```

<div class="fragment">

**Two pointer rewires.** No shifting. No traversal.

This is why **linked lists shine for stack push operations**.

</div>

---

## Insert at tail — O(n) without tail pointer

```c
Node* insert_tail(Node* head, int val) {
    Node* n = malloc(sizeof(Node));
    n->data = val; n->next = NULL;
    if (!head) return n;
    Node* cur = head;
    while (cur->next) cur = cur->next;   // walk to end
    cur->next = n;
    return head;
}
```

<div class="fragment">

**$n$ steps** to reach the tail.

!!! tip "Fix"
    Store a `tail` pointer — then insert-at-tail is **O(1)**.

</div>

---

## Delete by value — O(n)

```c
Node* delete(Node* head, int val) {
    Node *cur = head, *prev = NULL;
    while (cur && cur->data != val) {
        prev = cur; cur = cur->next;
    }
    if (!cur) return head;               // not found
    if (prev) prev->next = cur->next;    // unlink from middle
    else head = cur->next;               // deleting head
    free(cur);
    return head;
}
```

<div class="fragment">

**O(n)** to find, **O(1)** to unlink. Total **O(n)**.

</div>

---

## 3.4 — Doubly linked list

```c
struct DNode {
    int data;
    struct DNode *prev, *next;
};
```

Two pointers per node:

- **prev** — backward link.
- **next** — forward link.

<div class="fragment">

```
NULL ← [•|10|•] ⇄ [•|20|•] ⇄ [•|30|•] → NULL
```

</div>

<div class="fragment">

**Advantages:**

- Bidirectional traversal.
- **O(1) delete given a node pointer** — no need for predecessor search.

**Cost:** one extra pointer per node = ~50% memory overhead on small ints.

</div>

---

## Delete given a node — O(1) in doubly list

```c
void delete_node(DNode* n) {
    if (n->prev) n->prev->next = n->next;
    if (n->next) n->next->prev = n->prev;
    free(n);
}
```

<div class="fragment">

**Two pointer updates.** No traversal.

**In a singly linked list**, the same operation is **O(n)** — you must find the predecessor first.

This is the **main reason** doubly linked lists exist.

</div>

---

## 3.5 — Circular linked list

Last node points back to the **head** instead of `NULL`.

```
head → [10|•] → [20|•] → [30|•] ─┐
         ↑                        │
         └────────────────────────┘
```

<div class="fragment">

**Uses:**

- **Round-robin scheduling** — cycle through processes.
- **Circular buffers** — ring buffers in audio, networking.
- **Josephus problem** — cyclic elimination.

</div>

<div class="fragment">

!!! warning "Care needed"
    Traversal must check "back to head" — otherwise infinite loop.

</div>

---

## 3.6 — Worked example: reverse a linked list

Reverse `[10 → 20 → 30 → NULL]` to `[30 → 20 → 10 → NULL]` in **O(n)** time and **O(1)** space.

```c
Node* reverse(Node* head) {
    Node* prev = NULL;
    Node* cur = head;
    while (cur) {
        Node* next = cur->next;   // save next
        cur->next = prev;         // reverse link
        prev = cur;               // advance prev
        cur = next;               // advance cur
    }
    return prev;
}
```

<div class="fragment">

**Three pointers** — `prev`, `cur`, `next`. **One pass. O(1) extra space.**

</div>

---

## Tracing the reversal

Start: `[10 → 20 → 30]`, prev = NULL, cur = 10.

| Step | cur | next | after `cur->next = prev` | list so far |
|:---:|:---:|:---:|---|:---:|
| 1 | 10 | 20 | 10 → NULL | `[10]` |
| 2 | 20 | 30 | 20 → 10 | `[20 → 10]` |
| 3 | 30 | NULL | 30 → 20 | `[30 → 20 → 10]` |

<div class="fragment">

Return prev = 30. **Reversed.**

</div>

---

## Chapter summary

1. **Node** = data + pointer. Nodes are linked, not contiguous.

2. **Singly linked**: O(1) insert-at-head, O(n) search, O(n) delete-by-value.

3. **Doubly linked**: O(1) delete given a node; bidirectional.

4. **Circular**: no NULL terminator; cycles. Used in scheduling and buffers.

5. **Trade-off** vs arrays: fast insert/delete, slow random access.

6. **Reversal** is the classic interview problem — three pointers, one pass.

---

## Review questions

1. Compare arrays and linked lists for insert/delete/access. Give a use case where each is preferable.

2. Why is insert at head O(1) in a singly linked list?

3. What extra operation does a doubly linked list enable? Why can't a singly linked list do it in O(1)?

4. When would you prefer a circular linked list?

5. Write pseudocode to reverse a singly linked list in O(n) time and O(1) space.

6. Given two sorted linked lists, merge them into one sorted list. Analyse the complexity.

> Answers: **Solutions → Chapter 3**.

---

## References

- **Weiss**, Ch. 3 — linked lists and their applications.
- **Lafore**, Ch. 5 — linked lists in Java.
- **Sedgewick & Wayne**, §1.3 — bags, queues, stacks via lists.
- **CLRS**, §10.2 — linked-list primitives.