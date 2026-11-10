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
  js: assets/lecture-04.js
---

# ECEG-4192

## Chapter 4 — Stacks and Queues

LIFO · FIFO · Applications

Semester VII · 2025/26

<aside class="notes">
Two ADTs, both O(1) per operation, both foundational for parsing, graph traversal, and scheduling.
</aside>

---

## Learning outcomes

By the end of this chapter you should be able to:

1. Define the **Stack** ADT (LIFO) and its operations.

2. Implement stacks using **arrays** and **linked lists**.

3. Explain **when** to use each implementation.

4. Define the **Queue** ADT (FIFO) and its operations.

5. Implement queues using **circular arrays** and linked lists.

6. Recognise **applications** of both.

---

## 4.1 — The Stack ADT

???+ note "Definition — Stack"
    A **LIFO** (Last-In, First-Out) collection.
    
    - `push(x)` — add $x$ to the top.
    - `pop()` — remove and return the top.
    - `peek()` / `top()` — return the top without removing.
    - `isEmpty()` — true if empty.

<div class="fragment">

Analogy: a stack of plates — you add and remove from the top.

</div>

---

## Stack operations — visual

```
push(10) → [10]
push(20) → [10, 20]
push(30) → [10, 20, 30]
pop()    → 30, stack = [10, 20]
peek()   → 20
pop()    → 20, stack = [10]
```

<div class="fragment">

**All operations touch only the top.**

**All operations are O(1).**

</div>

---

## 4.2 — Stack: array implementation

```c
#define MAX 100
typedef struct { int data[MAX]; int top; } Stack;

void init(Stack* s) { s->top = 0; }
void push(Stack* s, int v) { s->data[s->top++] = v; }
int  pop (Stack* s)        { return s->data[--s->top]; }
int  peek(Stack* s)        { return s->data[s->top-1]; }
int  isEmpty(Stack* s)     { return s->top == 0; }
```

<div class="fragment">

**O(1) operations.** Simple memory layout.

**Downside:** fixed capacity — potential **overflow**. Must `check full` on push.

</div>

---

## Stack — linked-list implementation

```c
typedef struct Node { int data; struct Node* next; } Node;

typedef struct { Node* top; } Stack;

void push(Stack* s, int v) {
    Node* n = malloc(sizeof(Node));
    n->data = v; n->next = s->top; s->top = n;
}
int pop(Stack* s) {
    Node* n = s->top;
    int v = n->data;
    s->top = n->next;
    free(n);
    return v;
}
```

<div class="fragment">

**O(1)** push/pop. **Dynamic size** — no overflow until memory exhausted.

**Downside:** pointer overhead, worse cache locality.

</div>

---

## 4.3 — Stack applications

<div class="fragment">

- **Function call stack** — recursion, local variables, return addresses.
- **Expression evaluation** — infix → postfix, RPN evaluation.
- **Balanced parentheses** — checking `(`, `{`, `[` matching.

</div>

<div class="fragment">

- **Undo / redo** — in editors.
- **DFS (iterative)** — depth-first search.
- **Backtracking** — maze, N-queens, parsing.
- **Browser back button** — history stack.

</div>

---

## Worked example — balanced parentheses

Check if `{[()]}` is balanced.

```python
def is_balanced(s):
    stack = []
    pairs = {')': '(', '}': '{', ']': '['}
    for c in s:
        if c in '({[':
            stack.append(c)
        elif c in ')}]':
            if not stack or stack.pop() != pairs[c]:
                return False
    return not stack
```

<div class="fragment">

Trace on `{[()]}`: push `{`, push `[`, push `(`, pop `(`=`)`, pop `[`=]`, pop `{`=}. **Empty → balanced.**

**O(n)** time, **O(n)** space.

</div>

---

## 4.4 — The Queue ADT

???+ note "Definition — Queue"
    A **FIFO** (First-In, First-Out) collection.
    
    - `enqueue(x)` — add $x$ to the back.
    - `dequeue()` — remove and return the front.
    - `front()` — return the front without removing.
    - `isEmpty()` — true if empty.

<div class="fragment">

Analogy: a line at a ticket counter — first-come, first-served.

</div>

---

## Queue operations — visual

```
enqueue(10) → [10]
enqueue(20) → [10, 20]
enqueue(30) → [10, 20, 30]
dequeue()   → 10, queue = [20, 30]
front()     → 20
dequeue()   → 20, queue = [30]
```

<div class="fragment">

**Enqueue touches the back; dequeue touches the front.**

**All operations O(1)** — with the right implementation.

</div>

---

## 4.5 — Circular array implementation

A naïve array queue wastes space: after `dequeue`, slots at the front are unusable.

???+ tip "Fix"
    Wrap the index with `% MAX`.

```c
typedef struct {
    int data[MAX]; int front, rear, size;
} Queue;

void enqueue(Queue* q, int v) {
    q->data[q->rear] = v;
    q->rear = (q->rear + 1) % MAX;
    q->size++;
}
int dequeue(Queue* q) {
    int v = q->data[q->front];
    q->front = (q->front + 1) % MAX;
    q->size--;
    return v;
}
```

<div class="fragment">

**O(1)** enqueue/dequeue. **Full capacity used.**

</div>

---

## Circular queue trace

`enqueue` three times, `dequeue` once, `enqueue` one more (size = 4, MAX = 5):

| Action | front | rear | Occupied slots |
|:---:|:---:|:---:|:---:|
| init | 0 | 0 | empty |
| enq A | 0 | 1 | [A] |
| enq B | 0 | 2 | [A,B] |
| enq C | 0 | 3 | [A,B,C] |
| deq (A) | 1 | 3 | [B,C] |
| enq D | 1 | 4 | [B,C,D] |
| enq E | 1 | 0 | [B,C,D,E] ← wrapped |

<div class="fragment">

`rear` wraps around. Space reused.

</div>

---

## Queue — linked-list implementation

```c
typedef struct { Node *front, *rear; } Queue;

void enqueue(Queue* q, int v) {
    Node* n = malloc(sizeof(Node));
    n->data = v; n->next = NULL;
    if (q->rear) q->rear->next = n;
    else q->front = n;
    q->rear = n;
}
int dequeue(Queue* q) {
    Node* n = q->front;
    int v = n->data;
    q->front = n->next;
    if (!q->front) q->rear = NULL;
    free(n);
    return v;
}
```

<div class="fragment">

**O(1)** enqueue/dequeue. **Dynamic size.** No wraparound confusion.

</div>

---

## 4.6 — Queue variants

???+ note "Priority Queue"
    Dequeue returns the **highest-priority** element, not the oldest. Implementation: binary heap (see Ch. 5). Cost: **O(log n)** per operation.

???+ note "Deque (double-ended queue)"
    Insert and remove at **both ends**. O(1) per operation with a doubly-linked list or circular array.

???+ note "Blocking queue"
    Concurrency; `enqueue` waits if full, `dequeue` waits if empty. Used in producer-consumer.

---

## Queue applications

<div class="fragment">

- **BFS** — breadth-first search.
- **Task scheduling** — CPU scheduler, printer queue.
- **Message queues** — Kafka, RabbitMQ.

</div>

<div class="fragment">

- **I/O buffers** — audio, networking.
- **Asynchronous processing** — job queues.
- **Bipartite checking** — via BFS.

</div>

<div class="fragment">

!!! tip "Pattern"
    Queue = **"process in arrival order"**.

</div>

---

## Chapter summary

1. **Stack** — LIFO. All ops O(1). Array or linked-list based.

2. **Queue** — FIFO. All ops O(1). **Circular array** avoids wasted space.

3. **Applications:** stacks for **recursion, parsing, DFS**; queues for **BFS, scheduling, buffers**.

4. **Variants:** priority queue, deque, blocking queue.

5. **Implementation trade-offs:** arrays cache-friendly but fixed; lists dynamic but pointer-heavy.

---

## Review questions

1. Distinguish LIFO and FIFO. Give two real-world examples of each.

2. Show how a stack checks balanced parentheses. Trace `{[()]}`.

3. Convert `A + B * C` to postfix using a stack.

4. Why is a circular queue preferred over a linear array queue? Draw a trace.

5. Give two applications each of stacks and queues.

6. Implement a queue using **two stacks**. Analyse the amortised cost per operation.

> Answers: **Solutions → Chapter 4**.

---

## References

- **Weiss**, Ch. 3 (lists, stacks, queues).
- **Lafore**, Ch. 4 (stacks), Ch. 4 (queues).
- **CLRS**, §10.1 — stacks and queues.
- **Sedgewick & Wayne**, §1.3 — bags, queues, stacks.