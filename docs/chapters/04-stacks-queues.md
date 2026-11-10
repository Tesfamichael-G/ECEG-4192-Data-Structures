# Chapter 4 — Stacks and Queues

!!! abstract "Chapter at a glance"
    **Contact hours:** 2 lecture + 1 tutorial + 2 lab · **Home study:** ~5 h
    
    **Coverage (syllabus):** stacks (definition, implementation, application) ·
    queues (definition, implementation, application).

---

## Learning outcomes

By the end of this chapter you should be able to:

1. State the **LIFO** (stack) and **FIFO** (queue) principles and their ADT operations.
2. Implement a stack using both an **array** and a **linked list**, with all O(1) operations.
3. Evaluate a **postfix expression** with a stack — the classic stack algorithm.
4. Detect **balanced parentheses** and **matching brackets** with a stack.
5. Implement a **circular queue** on an array — avoiding the "wrap-around" bug.
6. Implement a **queue** using a linked list with a tail pointer — O(1) enqueue and dequeue.
7. Recognise the applications of **deque** and **priority queue** (deferred to Chapter 5).
8. Choose between array and linked-list implementations based on the operation mix.

---

## 4.1 The stack — LIFO

### 4.1.1 The principle

A **stack** is a collection with a single access point — the **top**. Items can only be added or removed at the top. This gives **LIFO** (Last In, First Out) behaviour.

Mental model: a stack of plates. You add at the top, you take from the top. The bottom plate is the last one you'll ever reach.

### 4.1.2 The ADT

The Stack ADT specifies these operations:

| Operation | Effect | Returns |
|---|---|---|
| `push(x)` | Insert `x` at the top | — |
| `pop()` | Remove and return the top element | Top value |
| `peek()` / `top()` | Return the top without removing | Top value |
| `isEmpty()` | Check whether the stack has elements | `true` / `false` |
| `size()` | Number of elements | `int` |

All operations must be **O(1)** — anything else defeats the purpose.

### 4.1.3 The underlying idea — one end only

A stack is a **special case of a list** where you only operate at one end. That means:

- **No searching** — you can't reach into the middle.
- **No random access** — only the top is visible.
- **Both array and linked-list** can implement it, and both give O(1) push/pop.

---

## 4.2 Stack implementations

### 4.2.1 Array-based stack

```java
class ArrayStack {
    private int[] data;
    private int top;   // index of the top element; -1 when empty

    ArrayStack(int capacity) {
        data = new int[capacity];
        top = -1;
    }

    void push(int x) {
        if (top == data.length - 1) throw new RuntimeException("Stack full");
        data[++top] = x;
    }

    int pop() {
        if (isEmpty()) throw new RuntimeException("Stack empty");
        return data[top--];
    }

    int peek() {
        if (isEmpty()) throw new RuntimeException("Stack empty");
        return data[top];
    }

    boolean isEmpty() { return top == -1; }
    int size() { return top + 1; }
}
```

**Cost:** all operations **O(1)**. Push might need to **resize** if the array is full — see the dynamic version below.

**Drawback:** fixed capacity. Push onto a full stack throws — unless you resize.

### 4.2.2 Dynamic array-based stack

Grow by doubling when full, shrink at capacity/4 (same policy as Chapter 2 dynamic arrays):

```java
class DynamicArrayStack {
    private int[] data = new int[1];
    private int top = -1;

    void push(int x) {
        if (top == data.length - 1) resize(data.length * 2);
        data[++top] = x;
    }

    int pop() {
        int x = data[top--];
        if (top > 0 && top < data.length / 4) resize(data.length / 2);
        return x;
    }

    private void resize(int cap) {
        int[] nd = new int[cap];
        System.arraycopy(data, 0, nd, 0, top + 1);
        data = nd;
    }
    // peek, isEmpty, size as above
}
```

**Amortised push/pop:** O(1) — same analysis as Chapter 2 (geometric series bound).

**Worst-case single push/pop:** O(n) when a resize occurs.

### 4.2.3 Linked-list stack

Push at head, pop from head — both O(1):

```java
class LinkedStack {
    private Node head;

    void push(int x) {
        Node n = new Node(x);
        n.next = head;
        head = n;
    }

    int pop() {
        if (head == null) throw new RuntimeException("Stack empty");
        int x = head.value;
        head = head.next;
        return x;
    }

    int peek() {
        if (head == null) throw new RuntimeException("Stack empty");
        return head.value;
    }

    boolean isEmpty() { return head == null; }
}
```

**Cost:** all operations **O(1) worst case** — no resize, no amortisation needed.

**Trade-off vs. array:**

| Aspect | Array stack | Linked-list stack |
|---|---|---|
| Push (amortised) | O(1) | O(1) |
| Push (worst case) | O(n) on resize | O(1) |
| Memory per element | 4 bytes (int) | 4 bytes + 8 for `next` = 16 bytes |
| Cache locality | Excellent | Poor |
| Total memory | Allocated capacity (may waste up to 50%) | Exact |

**Practical choice:** dynamic array in almost every case — the cache advantage is worth the occasional resize. Linked-list stack only when you need **strict O(1) worst case** (real-time systems) or the number of elements varies wildly.

### 4.2.4 Python — same idea

Python's built-in `list` works as a stack with `append` and `pop`:

```python
stack = []
stack.append(10)    # push
stack.append(20)
top = stack[-1]     # peek
value = stack.pop() # pop
```

`append` and `pop()` at the end are **O(1) amortised** (Python's list uses the same doubling policy). Perfect stack.

---

## 4.3 Stack applications

### 4.3.1 Balanced parentheses

**Problem.** Given a string of `()`, `[]`, `{}`, determine if every opening bracket is closed by the correct closing bracket, in the right order.

```java
boolean isBalanced(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        if (c == '(' || c == '[' || c == '{') {
            stack.push(c);
        } else if (c == ')' || c == ']' || c == '}') {
            if (stack.isEmpty()) return false;
            char open = stack.pop();
            if (c == ')' && open != '(') return false;
            if (c == ']' && open != '[') return false;
            if (c == '}' && open != '{') return false;
        }
    }
    return stack.isEmpty();   // stack must be empty at end
}
```

**Cost.** O(n) time, O(n) space worst case (all opening brackets).

**Why a stack?** The most recently opened bracket must be the next one closed. That's LIFO. No other data structure handles this as cleanly.

### 4.3.2 Postfix (Reverse Polish) evaluation

**Notation.** Normal math is **infix** (`3 + 4`). **Postfix** writes operators after their operands: `3 4 +`. This eliminates the need for parentheses and follows a simple left-to-right rule.

Examples:
- `3 + 4` → `3 4 +` = 7
- `(3 + 4) × 5` → `3 4 + 5 ×` = 35
- `3 + 4 × 5` → `3 4 5 × +` = 23

```java
int evalPostfix(String expr) {
    Deque<Integer> stack = new ArrayDeque<>();
    for (String tok : expr.trim().split("\\s+")) {
        if (isNumber(tok)) {
            stack.push(Integer.parseInt(tok));
        } else {
            int b = stack.pop();
            int a = stack.pop();
            int r = switch (tok) {
                case "+" -> a + b;
                case "-" -> a - b;
                case "*" -> a * b;
                case "/" -> a / b;
                default  -> throw new IllegalArgumentException(tok);
            };
            stack.push(r);
        }
    }
    return stack.pop();
}
```

**Cost.** O(n) time, O(n) space. **Every token processed exactly once.** No parsing complexity, no operator precedence, no parentheses.

**Why postfix matters.** All compilers lower expressions to postfix (or a similar stack-based representation) before generating code. The stack is what makes expression evaluation trivial.

### 4.3.3 Expression conversion (infix → postfix)

Given infix `3 + 4 × 5`, use **Dijkstra's shunting-yard** to produce postfix.

**Rules:**
- Number: emit directly.
- Operator: pop and emit operators of **higher or equal precedence** from the stack, then push the new operator.
- `(`: push.
- `)`: pop and emit until `(`; discard the `(`.

```java
String infixToPostfix(String expr) {
    StringBuilder out = new StringBuilder();
    Deque<Character> ops = new ArrayDeque<>();
    for (char c : expr.toCharArray()) {
        if (Character.isDigit(c)) out.append(c).append(' ');
        else if (c == '(') ops.push(c);
        else if (c == ')') {
            while (!ops.isEmpty() && ops.peek() != '(') out.append(ops.pop()).append(' ');
            ops.pop();   // discard '('
        } else if ("+-*/".indexOf(c) >= 0) {
            while (!ops.isEmpty() && prec(ops.peek()) >= prec(c))
                out.append(ops.pop()).append(' ');
            ops.push(c);
        }
    }
    while (!ops.isEmpty()) out.append(ops.pop()).append(' ');
    return out.toString().trim();
}

int prec(char op) {
    return (op == '*' || op == '/') ? 2 : (op == '+' || op == '-') ? 1 : 0;
}
```

**Trace** on `3 + 4 * 5`:

| Token | Output | Operator stack |
|:---:|---|---|
| `3` | `3` | — |
| `+` | `3` | `+` |
| `4` | `3 4` | `+` |
| `*` | `3 4` | `+ *` |
| `5` | `3 4 5` | `+ *` |
| end | `3 4 5 * +` | — |

**Correct:** `3 4 5 * +` evaluates to `3 + (4 * 5) = 23`. ✓

**Cost.** O(n) time, O(n) space.

### 4.3.4 Undo/redo

A simple text editor's **undo** feature is a stack of editor states. Each edit pushes the previous state; undo pops and restores.

**Redo** requires a **second stack** — every undo pushes onto the redo stack; a new edit clears the redo stack.

This is not just a toy — it's the same pattern used in browsers (back/forward buttons), IDEs (Ctrl+Z/Ctrl+Y), and version-control tools.

### 4.3.5 Function calls — the call stack

Every function call in every programming language uses a **stack** — the **call stack**:

- Calling a function: push a **stack frame** (arguments, local variables, return address).
- Returning: pop the frame.
- Recursion: each recursive call is a new frame — depth of recursion = stack depth.

**This is why** deep recursion overflows the stack — the OS allocates a fixed-size call stack (typically 1–8 MB) and the program crashes when exceeded. Anything that grows the stack unboundedly (naive Fibonacci on $n = 10^6$) will crash — not run slowly, **crash**.

### 4.3.6 Other stack applications

- **Depth-first search (DFS)** — iterative DFS uses an explicit stack (Chapter 7).
- **Backtracking algorithms** — maze solving, N-queens, sudoku — push/pop choices.
- **Tower of Hanoi** — recursion is literally a stack.
- **Syntax parsing** — compilers use a stack to parse nested constructs.

---

## 4.4 The queue — FIFO

### 4.4.1 The principle

A **queue** is a collection where items are added at one end (**rear**) and removed at the other (**front**). This gives **FIFO** (First In, First Out).

Mental model: a checkout line. The first customer in is the first out.

### 4.4.2 The ADT

| Operation | Effect |
|---|---|
| `enqueue(x)` | Insert `x` at the rear |
| `dequeue()` | Remove and return the front element |
| `peek()` / `front()` | Return the front without removing |
| `isEmpty()` | Check whether empty |
| `size()` | Number of elements |

All operations must be **O(1)**.

### 4.4.3 The naive array implementation — and why it fails

Tempting first attempt:

```java
class NaiveQueue {
    int[] data = new int[100];
    int front = 0, rear = 0;

    void enqueue(int x) { data[rear++] = x; }
    int dequeue() { return data[front++]; }
}
```

The bug: **`front` and `rear` both move forward, never resetting**. After 100 enqueues and 100 dequeues, the array is "empty" but `front = 100`, `rear = 100` — the next enqueue overflows even though there's space.

**Fix 1: shift every dequeue.** Move all elements left by one — **O(n) per dequeue** → O(n²) for n operations. Unacceptable.

**Fix 2: wrap around — the circular queue.**

---

## 4.5 The circular queue

### 4.5.1 The idea

Treat the array as a **circle**: index `(i + 1) % capacity` wraps around. Front and rear both advance by 1 modulo capacity; when front catches up to rear, the queue is empty.

```java
class CircularQueue {
    private int[] data;
    private int front, size;
    private final int capacity;

    CircularQueue(int capacity) {
        this.capacity = capacity;
        data = new int[capacity];
        front = 0;
        size = 0;
    }

    void enqueue(int x) {
        if (size == capacity) throw new RuntimeException("Queue full");
        data[(front + size) % capacity] = x;
        size++;
    }

    int dequeue() {
        if (isEmpty()) throw new RuntimeException("Queue empty");
        int x = data[front];
        front = (front + 1) % capacity;
        size--;
        return x;
    }

    int peek() {
        if (isEmpty()) throw new RuntimeException("Queue empty");
        return data[front];
    }

    boolean isEmpty() { return size == 0; }
    int size() { return size; }
}
```

**Cost.** All operations **O(1)** — constant time, no shifting, no waste.

**Key design choice: use `size` not `(rear - front)`.** If you track `rear` and `front` both, you can't distinguish "empty" from "full" without an extra flag or a deliberate one-slot gap. Tracking `size` makes both checks trivial.

### 4.5.2 Why `size` matters

Consider the alternative:

```java
int front = 0, rear = 0;
boolean isEmpty() { return front == rear; }
boolean isFull()  { return front == rear; }   // oops — same condition
```

Empty and full look identical without extra state. Options:

- **Sacrifice one slot:** only `capacity - 1` elements stored; full when `(rear + 1) % capacity == front`.
- **Track `size`:** straightforward, no wasted slot. This is the cleaner approach.

### 4.5.3 Trace

**Setup:** `capacity = 5`, empty.

| Operation | `data` (index 0..4) | `front` | `size` |
|:---:|---|:---:|:---:|
| enqueue(10) | `[10, _, _, _, _]` | 0 | 1 |
| enqueue(20) | `[10, 20, _, _, _]` | 0 | 2 |
| enqueue(30) | `[10, 20, 30, _, _]` | 0 | 3 |
| dequeue() → 10 | `[_, 20, 30, _, _]` | 1 | 2 |
| dequeue() → 20 | `[_, _, 30, _, _]` | 2 | 1 |
| enqueue(40) | `[_, _, 30, 40, _]` | 2 | 2 |
| enqueue(50) | `[_, _, 30, 40, 50]` | 2 | 3 |
| enqueue(60) | `[60, _, 30, 40, 50]` (wrapped!) | 2 | 4 |
| dequeue() → 30 | `[60, _, _, 40, 50]` | 3 | 3 |
| dequeue() → 40 | `[60, _, _, _, 50]` | 4 | 2 |
| dequeue() → 50 | `[60, _, _, _, _]` | 0 | 1 |
| dequeue() → 60 | `[_, _, _, _, _]` | 1 | 0 |

**The wrap-around** keeps the queue O(1) per operation — no shifting, no waste. This is the standard queue implementation in every real system.

---

## 4.6 Linked-list queue

Alternative — no capacity limit:

```java
class LinkedQueue {
    private Node head, tail;

    void enqueue(int x) {
        Node n = new Node(x);
        if (tail == null) { head = tail = n; }
        else { tail.next = n; tail = n; }
    }

    int dequeue() {
        if (head == null) throw new RuntimeException("Queue empty");
        int x = head.value;
        head = head.next;
        if (head == null) tail = null;   // queue became empty
        return x;
    }

    int peek() {
        if (head == null) throw new RuntimeException("Queue empty");
        return head.value;
    }

    boolean isEmpty() { return head == null; }
}
```

**Cost.** All operations **O(1)**. Both enqueue and dequeue modify the ends.

**Why both `head` and `tail` pointers matter.** Enqueue at `tail`, dequeue from `head` — both O(1). With only `head`, enqueue would be O(n).

**The empty-queue edge case.** After the last dequeue, `head = null` but `tail` still points to the old node — a bug unless you explicitly reset `tail = null`. This is one of the two "classic" queue bugs (the other is the same for enqueue when starting from empty).

**Trade-off vs. circular array:**

| Aspect | Circular array | Linked list |
|---|---|---|
| Enqueue / dequeue | O(1) | O(1) |
| Capacity | Fixed (must resize) | Unbounded |
| Memory per element | 4 bytes | 4 + 8 = 16 bytes |
| Cache locality | Excellent | Poor |
| Bug risk | Wrap-around index arithmetic | Null-pointer bugs |

**Practical choice:** circular array for fixed-size buffers (network packets, streaming); linked list when unbounded growth is needed.

---

## 4.7 Variants — deque and priority queue

### 4.7.1 Deque (double-ended queue)

A **deque** allows insert and remove at **both ends** — O(1) at each end.

Operations: `addFirst`, `addLast`, `removeFirst`, `removeLast`, `peekFirst`, `peekLast`.

**Implementation:** doubly-linked list, or circular array (with two valid pointers).

**Use cases:**

- **Sliding window maximum** — maintain a monotonic deque.
- **Palindrome check** — remove from both ends, compare.
- **Undo/redo** — push and pop at both ends.
- **Work-stealing schedulers** — threads steal from the *other* end.

Java's `ArrayDeque` is the standard `Deque` implementation — fast, cache-friendly, O(1) amortised at both ends. **Prefer `ArrayDeque` over `Stack` or `LinkedList`** in Java unless you need linked semantics.

### 4.7.2 Priority queue

A **priority queue** — not FIFO or LIFO — returns the element with the **highest (or lowest) priority** first.

Standard implementation: **binary heap** (Chapter 5). All operations O(log n) except peek (O(1)).

**Use cases:** Dijkstra's algorithm (Chapter 7), Huffman coding, task scheduling by priority, event simulation.

We defer the full treatment to Chapter 5 — the heap is a tree.

---

## Worked example 4.1 — Balanced parentheses trace

**Input:** `"([{}])"`.

```java
Deque<Character> stack = new ArrayDeque<>();
```

**Trace:**

| Char | Action | Stack (top on right) |
|:---:|---|---|
| `(` | push | `(` |
| `[` | push | `( [` |
| `{` | push | `( [ {` |
| `}` | pop `{`, matches | `( [` |
| `]` | pop `[`, matches | `(` |
| `)` | pop `(`, matches | (empty) |

**Result:** stack empty at end → **balanced** ✓

**Input:** `"([)]"`:

| Char | Action | Stack |
|:---:|---|---|
| `(` | push | `(` |
| `[` | push | `( [` |
| `)` | pop `[` — **mismatch** | — |

**Result:** mismatch → **not balanced** ✓

**Cost.** O(n) time, O(n) space.

---

## Worked example 4.2 — Postfix evaluation trace

**Input:** `"3 4 + 5 *"` — that's `(3 + 4) × 5 = 35`.

**Trace:**

| Token | Action | Stack |
|:---:|---|---|
| `3` | push | `[3]` |
| `4` | push | `[3, 4]` |
| `+` | pop 4, pop 3, push 7 | `[7]` |
| `5` | push | `[7, 5]` |
| `*` | pop 5, pop 7, push 35 | `[35]` |

**Result:** `35` ✓

**Note the order of pops.** For `a b -`, pop `b` first, then `a`, compute `a - b`. **Order matters** for non-commutative operators (`-`, `/`).

---

## Worked example 4.3 — Infix to postfix

**Input:** `"a + b * c - d"` (no parentheses — standard precedence).

**Precedence:** `*` and `/` = 2, `+` and `-` = 1.

| Token | Output | Operator stack |
|:---:|---|---|
| `a` | `a` | — |
| `+` | `a` | `+` |
| `b` | `a b` | `+` |
| `*` | `a b` | `+ *` |
| `c` | `a b c` | `+ *` |
| `-` | `a b c * +` (pop `*`, then `+`) | `-` |
| `d` | `a b c * + d` | `-` |
| end | `a b c * + d -` | — |

**Result:** `a b c * + d -`

**Sanity check.** Infix `a + b*c - d` = `a + (b*c) - d`. Postfix `a b c * + d -` = `(a + b*c) - d`. ✓

**The `-` case.** When the new operator `-` arrives, pop and emit all higher-or-equal precedence ops (`*`, `+`) first — then push `-`.

---

## Worked example 4.4 — Two-stack expression evaluator (infix with parentheses)

**Input:** `"3 + 4 * (2 - 1)"`.

Two stacks: one for numbers, one for operators.

```java
int evalInfix(String s) {
    Deque<Integer> nums = new ArrayDeque<>();
    Deque<Character> ops = new ArrayDeque<>();
    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if (c == ' ') continue;
        if (Character.isDigit(c)) {
            int n = 0;
            while (i < s.length() && Character.isDigit(s.charAt(i))) {
                n = n * 10 + (s.charAt(i++) - '0');
            }
            i--;
            nums.push(n);
        } else if (c == '(') ops.push(c);
        else if (c == ')') {
            while (ops.peek() != '(') apply(nums, ops);
            ops.pop();
        } else {   // operator
            while (!ops.isEmpty() && prec(ops.peek()) >= prec(c)) apply(nums, ops);
            ops.push(c);
        }
    }
    while (!ops.isEmpty()) apply(nums, ops);
    return nums.pop();
}

void apply(Deque<Integer> nums, Deque<Character> ops) {
    char op = ops.pop();
    int b = nums.pop(), a = nums.pop();
    nums.push(op == '+' ? a + b : op == '-' ? a - b : op == '*' ? a * b : a / b);
}
```

**Trace:**

| Char | Action | Num stack | Op stack |
|:---:|---|---|---|
| `3` | push | `[3]` | — |
| `+` | push | `[3]` | `+` |
| `4` | push | `[3, 4]` | `+` |
| `*` | push (higher prec) | `[3, 4]` | `+ *` |
| `(` | push | `[3, 4]` | `+ * (` |
| `2` | push | `[3, 4, 2]` | `+ * (` |
| `-` | push | `[3, 4, 2]` | `+ * ( -` |
| `1` | push | `[3, 4, 2, 1]` | `+ * ( -` |
| `)` | apply `-` | `[3, 4, 1]` | `+ *` |
| end | apply `*` | `[3, 4]` | `+` |
| end | apply `+` | `[7]` | — |

**Result:** `7` ✓ (`3 + 4*(2-1) = 3 + 4 = 7`)

**Cost.** O(n) time, O(n) space.

---

## Worked example 4.5 — Circular queue trace

**Setup:** `capacity = 3`.

| Operation | `data` (index 0..2) | `front` | `size` |
|:---:|---|:---:|:---:|
| enqueue(1) | `[1, _, _]` | 0 | 1 |
| enqueue(2) | `[1, 2, _]` | 0 | 2 |
| enqueue(3) | `[1, 2, 3]` | 0 | 3 (full) |
| dequeue() → 1 | `[_, 2, 3]` | 1 | 2 |
| enqueue(4) | `[4, 2, 3]` (wrapped: index `(1+2) % 3 = 0`) | 1 | 3 |
| enqueue(5) | **throw**: size == capacity | — | — |
| dequeue() → 2 | `[4, _, 3]` | 2 | 2 |
| enqueue(5) | `[4, 5, 3]` (index `(2+2) % 3 = 1`) | 2 | 3 |

**The wrap-around** is what lets us reuse positions 0 and 1 without shifting. **Without wrap**, we'd hit the "array full" condition at position 3 even though only 2 elements exist.

---

## Worked example 4.6 — Queue used for BFS (preview)

**Queue-based BFS** on a tree/graph is the canonical **queue application**. Each node is enqueued when discovered; dequeued when processed.

```java
void bfs(Node root) {
    if (root == null) return;
    Queue<Node> q = new ArrayDeque<>();
    q.add(root);
    while (!q.isEmpty()) {
        Node cur = q.remove();
        System.out.println(cur.value);
        if (cur.left  != null) q.add(cur.left);
        if (cur.right != null) q.add(cur.right);
    }
}
```

**Why a queue?** BFS visits nodes in **level order** — root, then all its children, then all grandchildren, etc. A queue is FIFO, so nodes are processed in the order discovered. A stack would give **DFS** instead (LIFO — dive deep first).

**Cost.** O(n) time per traversal, O(w) space where w = max width of tree. Chapter 7 covers graph BFS fully.

---

## 4.8 Comparison table

| Aspect | Stack | Queue | Deque | Priority queue |
|---|---|---|---|---|
| Order | LIFO | FIFO | Both ends | Priority |
| Push/enqueue | O(1) | O(1) | O(1) at each end | O(log n) |
| Pop/dequeue | O(1) | O(1) | O(1) at each end | O(log n) |
| Peek | O(1) | O(1) | O(1) | O(1) |
| Implementation | Array / list | Circular array / list | List or array | Binary heap |
| Example applications | Undo, DFS, parsing | BFS, scheduling, buffers | Sliding window | Dijkstra |

---

## Chapter summary

1. **Stack** is LIFO — push, pop, peek at one end. **O(1)** operations.
2. **Array stack** (amortised O(1)) is faster than linked-list stack due to cache locality; linked-list stack is O(1) **worst case** (no resize).
3. **Balanced parentheses** and **postfix evaluation** are the two canonical stack applications — O(n) time each.
4. **Infix to postfix** via shunting-yard algorithm — O(n) time.
5. The **function call stack** is a physical implementation of the stack ADT — recursion depth = stack depth.
6. **Queue** is FIFO — enqueue at rear, dequeue at front. **O(1)** operations.
7. **Circular array** queue wraps `front` and `size` modulo capacity — no shifting, no waste.
8. **Linked-list queue** with `head` and `tail` — O(1) at both ends.
9. **Deque** — O(1) at both ends; Java's `ArrayDeque` is the standard implementation.
10. **Priority queue** returns the highest-priority element — implemented with a **binary heap** (Chapter 5).

---

## Quiz

### A. Multiple choice

1. A stack operates on which principle?

   (a) FIFO (b) LIFO (c) Priority (d) Random

2. Push and pop on an **array-based** stack are:

   (a) O(1) each (b) O(log n) each (c) O(n) each (d) O(n²) each

3. Postfix expression `"5 3 2 * +"` evaluates to:

   (a) 16 (b) 11 (c) 10 (d) 25

4. Balanced parentheses detection uses a stack because:

   (a) brackets are LIFO-nested (b) it's the only option (c) it uses less memory
   (d) it's faster than a queue

5. A **circular queue** avoids the "array full" bug by:

   (a) shifting elements (b) wrapping indices modulo capacity (c) using a linked list
   (d) always keeping the array half empty

6. Enqueue and dequeue on a **circular array** queue are:

   (a) O(1) each (b) O(log n) each (c) O(n) each (d) O(n²) each

7. In a linked-list queue, **enqueue at tail** requires:

   (a) walking to the end (b) a tail pointer for O(1) (c) a head pointer only (d) a doubly-linked list

8. `ArrayDeque` in Java is:

   (a) array-based (b) linked-list based (c) hash-based (d) tree-based

9. `"a + b * c"` in postfix is:

   (a) `a b c * +` (b) `a b + c *` (c) `+ a * b c` (d) `a b * c +`

10. A queue is used for **BFS** because:

    (a) it dives deep (b) it processes nodes in discovery order (c) it's faster than a stack
    (d) it uses less memory

11. In the two-stack infix evaluator, the two stacks hold:

    (a) operands and results (b) numbers and operators (c) values and pointers (d) inputs and outputs

12. **Priority queue** is typically implemented with:

    (a) array (b) binary heap (c) hash table (d) stack

### B. Fill in the blank

1. A stack operates on **\_\_\_\_\_** (Last In, First Out).

2. A queue operates on **\_\_\_\_\_** (First In, First Out).

3. Push on a dynamic array stack is **O(\_\_\_\_\_)** amortised.

4. The **postfix** expression `"3 4 +"` evaluates to **\_\_\_\_\_**.

5. Balanced parentheses uses the stack's **\_\_\_\_\_** property.

6. A circular queue wraps indices with the modulo of **\_\_\_\_\_** (the array's size).

7. Dequeue from a linked-list queue with a tail pointer costs **O(\_\_\_\_\_)**.

8. `"a b + c *"` in infix is **\_\_\_\_\_**.

9. The **shunting-yard** algorithm converts **\_\_\_\_\_** to postfix.

10. `ArrayDeque` in Java is preferred over `Stack` because it's **\_\_\_\_\_** -friendly.

### C. Define / explain

1. State the **stack ADT** — its operations and their complexity.

2. Explain why **postfix evaluation** is simpler than infix evaluation — what algorithm does infix require that postfix doesn't?

3. Implement a **balanced parentheses** checker in pseudocode. Why is a stack the right choice?

4. Describe the **circular queue** wrap-around logic. Draw a diagram of enqueue/dequeue wrapping.

5. Explain why **`size`** is a better state variable than tracking `front` and `rear` independently for a circular queue.

6. Compare **array-based** and **linked-list** stacks on: worst-case push, memory per element, cache locality.

7. Describe the **two-stack infix evaluator**. What does each stack contain?

8. Explain why **BFS** uses a queue and **DFS** uses a stack (or recursion). What property of the graph traversal does each data structure give?

9. What is a **deque**? Give two applications.

10. Explain the difference between a **queue** and a **priority queue**. When would you prefer a priority queue?

---

## Answer key

??? success "A. Multiple choice"
    1. **(b)** LIFO.
    2. **(a)** O(1).
    3. **(b)** 11 (`5 + 3*2 = 11`).
    4. **(a)** LIFO-nested.
    5. **(b)** wrap indices.
    6. **(a)** O(1).
    7. **(b)** tail pointer.
    8. **(a)** array-based.
    9. **(a)** `a b c * +`.
    10. **(b)** discovery order.
    11. **(b)** numbers and operators.
    12. **(b)** binary heap.

??? success "B. Fill in the blank"
    1. LIFO
    2. FIFO
    3. $1$
    4. 7
    5. LIFO
    6. capacity
    7. $1$
    8. `(a + b) * c`
    9. infix
    10. cache

??? success "C. Define / explain — model answers"
    1. *Stack ADT:* operations `push(x)`, `pop()`, `peek()`, `isEmpty()`, `size()`. All must be O(1). LIFO ordering: last element pushed is first element popped. Implementation is either array-backed (amortised O(1)) or linked-list-backed (strict O(1)).
    2. *Postfix vs. infix evaluation:* postfix processes tokens left-to-right — push operands, pop for operators. No precedence rules, no parentheses. *Infix* requires either two passes (convert to postfix first) or two stacks (operators and operands, with precedence rules and parenthesised sub-expressions). The postfix representation *embeds* precedence and grouping structurally; infix forces the evaluator to reconstruct them.
    3. *Balanced parentheses pseudocode:* for each character: if opening bracket, push. If closing, pop and check it matches. At end, stack must be empty. Return true iff no mismatch and empty at end. *Why a stack:* the most recently opened bracket must be closed first — that's LIFO. Any other structure (queue, list, hash) would give wrong answers because they don't respect nesting order.
    4. *Circular queue:* maintain `front` (index of oldest element) and `size` (element count). `enqueue(x)` writes at `(front + size) % capacity`, then `size++`. `dequeue()` reads `data[front]`, then `front = (front + 1) % capacity` and `size--`. *Diagram:* as `front` and `size` advance past the end, they wrap to index 0 — the array is treated as a ring. No element moves.
    5. *Why `size` beats `front`+`rear`:* if you track `front` and `rear` as two independent indices, the condition `front == rear` is ambiguous — it's both "empty" (initial state) and "full" (wrapped around). Resolving this requires either a waste-one-slot convention (store at most capacity-1 elements) or an extra `size` variable anyway. Tracking `size` directly makes both `isEmpty` and `isFull` trivial and unambiguous — no discarded slot, no extra flags.
    6. *Array vs. linked-list stack:* (i) worst-case push — array O(n) on resize, linked-list O(1); (ii) memory per element — array 4 bytes (int), linked-list 16 bytes (value + next pointer, 64-bit); (iii) cache locality — array has 10–100× better cache hit rate for sequential push/pop. *Practical choice:* array stack unless strict O(1) worst case is required (real-time systems).
    7. *Two-stack infix evaluator:* one stack holds **operands** (numbers), the other holds **operators** (and parentheses). When an operator arrives, pop and apply operators of higher-or-equal precedence; when `)` is seen, pop and apply until `(`; at the end, apply all remaining operators. Result: one number on the operand stack.
    8. *BFS uses a queue; DFS uses a stack (or recursion):* BFS must process nodes in **breadth-first** order — nearest first, then next-nearest. A FIFO queue naturally produces this: enqueue all children of a dequeued node; they get processed after all currently queued nodes. DFS must go deep first — LIFO stack or recursion naturally pushes the "next child" onto the front of the processing order. *Property:* BFS gives shortest-path distance in unweighted graphs; DFS gives structural properties (cycles, topological order).
    9. *Deque (double-ended queue):* O(1) insert and remove at both ends. Operations: `addFirst`, `addLast`, `removeFirst`, `removeLast`, `peekFirst`, `peekLast`. *Applications:* (i) **sliding-window maximum** — monotonic deque tracks candidates; (ii) **palindrome check** — compare from both ends inward.
    10. *Queue vs. priority queue:* a queue processes elements in **FIFO** order; a priority queue processes elements in **priority** order — highest (or lowest) priority first, regardless of insertion order. *Prefer priority queue when:* the ordering that matters is priority, not arrival time — e.g., Dijkstra's algorithm (process nearest unexplored node first), task scheduling by priority, event simulation (earliest event first), Huffman coding (merge two smallest first).

---

## Further reading

- **Weiss**, Chapter 3 (Lists, Stacks, Queues).
- **Lafore**, Chapters 4–5 (Stacks and Queues).
- **CLRS**, Section 10.1 (Stacks and queues).
- **Open Data Structures**, Chapters 4–5 (Stacks and Queues) — rigorous treatment with all variations.
- **Sedgewick & Wayne**, Section 1.3 — with visualisations.
- **VisuAlgo** — [visualgo.net](https://visualgo.net/) — animated stack/queue operations and application demos.
- **Joel Spolsky**, "The Law of Leaky Abstractions" — [joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/](https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/) — relevant for the array vs. linked-list trade-off.

---

## Suggested tutorial problems

1. **Stack trace.** Push `1, 2, 3`, pop twice, push `4, 5`, pop once. Trace the stack after each operation. What is the final top?

2. **Postfix evaluation.** Evaluate `"4 5 + 6 2 - * 3 +"` step by step. What is the result?

3. **Balanced parentheses.** Check `"(()"`, `"()[]{}"`, `"([{}])"`, `"([)]"`. Which are balanced?

4. **Infix to postfix.** Convert `"(A + B) * (C - D)"` to postfix using the shunting-yard algorithm. Trace the operator stack at each step.

5. **Circular queue, hand-traced.** Start with capacity 4, empty. Trace: enqueue 1, 2, 3; dequeue; enqueue 4, 5; dequeue; enqueue 6. Show `data`, `front`, `size` after each operation.

6. **Two-stack infix.** Evaluate `"10 + 20 * (30 - 15)"` using the two-stack method. Show the stacks after each step.

7. **Undo/redo.** Design an undo/redo system for a simple text editor using two stacks. Draw the state after: type "abc", undo, undo, redo, type "d".

8. **Palindrome check with deque.** Given a string, use a deque to check palindrome in O(n) time, O(n) space. Trace on `"racecar"`.

9. **Sliding window maximum.** Given `[1, 3, -1, -3, 5, 3, 6, 7]` and `k = 3`, use a monotonic deque to find the max in each window. Trace the deque state at each step.

10. **Recursion depth.** Write a recursive function that counts down from n to 0. At n = 100,000, does it crash? Why? Rewrite it iteratively with an explicit stack. Compare memory usage.

11. **Queue used for scheduling.** Simulate a simple round-robin scheduler: three processes with execution times 5, 3, 8 ms; time quantum 2 ms. Show the order of completion.

12. **Comparison.** For an application that does 10⁶ push operations and 10⁶ pop operations, would you choose an array stack or a linked-list stack? Justify with analysis.

13. **BFS with a queue.** On a binary tree with root at 1, children 2, 3; grandchildren 4, 5, 6, 7 — trace BFS using a queue. What order are nodes visited in?

14. **Queue with two stacks.** Implement a FIFO queue using two LIFO stacks (in-stack and out-stack). What is the amortised cost of enqueue and dequeue?

15. **Valid parentheses with wildcard.** Given a string containing `(`, `)`, and `*` (wildcard that can be `(`, `)`, or empty), determine if there's any valid assignment. (Hint: two-stack or greedy approach.)