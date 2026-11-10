# Solutions to Chapter 3 — Linked Lists

[:octicons-arrow-left-24: Back to Chapter 3](../chapters/03-linked-lists.md)

---

## 1. Insert at head — build `5→4→3→2→1` from `1,2,3,4,5`

```
Insert 1: 1
Insert 2: 2 → 1
Insert 3: 3 → 2 → 1
Insert 4: 4 → 3 → 2 → 1
Insert 5: 5 → 4 → 3 → 2 → 1
```

Order reversed relative to insertion. **O(1) per insert.**

---

## 2. Insert at tail — build `1→2→3→4→5`

With tail pointer: each insert O(1). Final: `1 → 2 → 3 → 4 → 5`.

Without tail pointer: each insert walks to end — **O(n²) total**.

---

## 3. Delete all occurrences of `v`

```java
Node deleteAll(Node head, int v) {
    while (head != null && head.value == v) head = head.next;
    Node cur = head;
    while (cur != null && cur.next != null) {
        if (cur.next.value == v) cur.next = cur.next.next;
        else cur = cur.next;
    }
    return head;
}
```

**Edge cases:** v at head (loop at top), v is the only value (loop runs to null → return null). **O(n)**.

---

## 4. Reverse — iterative vs. recursive

**Iterative:** three pointers (prev, cur, next). **O(n) time, O(1) space.**

**Recursive:** reverse(head.next), then head.next.next = head; head.next = null. **O(n) time, O(n) stack space.**

**Iterative is preferred** for large lists.

---

## 5. Merge `[1,3,5]` + `[2,4,6]`

Using dummy node: `dummy → 1 → 2 → 3 → 4 → 5 → 6 → null`. **O(m+n) time, O(1) space.**

---

## 6. Remove duplicates

**Sorted list:** single pass, skip equal neighbors. **O(n)**.

**Unsorted list:** hash set to track seen values. **O(n) time, O(n) space**. Or sort first (O(n log n)) then use sorted approach.

---

## 7. Detect and remove cycle

**Floyd's algorithm** detects the meeting point. Then find entry point: reset slow to head, advance both one step until they meet → entry point. Then set prev.next = null (or the predecessor of the entry point) to break the cycle.

---


## 8. Middle with two pointers — trace on `1→2→3→4→5`

| Step | slow | fast |
|:---:|---|---|
| 0 | 1 | 1 |
| 1 | 2 | 3 |
| 2 | 3 | 5 |
| exit | 3 | 5 (fast.next = null) |

**Middle = 3** ✓

On `1→2→3→4`:

| Step | slow | fast |
|:---:|---|---|
| 0 | 1 | 1 |
| 1 | 2 | 3 |
| 2 | 3 | null |
| exit | 3 | null |

**Middle = 3** (upper middle of 2,3).

---

## 9. Merge sort a linked list

Split by finding the middle (`slow`, `fast`), sort each half recursively, merge.

**Cost.** Same recurrence as array merge sort: `T(n) = 2 T(n/2) + O(n)` → **O(n log n)**.

**Better than array merge sort** for linked lists: split doesn't need O(n) auxiliary space (you just detach the next pointer at mid). Merge is O(1) space. So **O(n log n) time, O(log n) space** (only recursion stack).

---

## 10. Josephus

For `J(7, 3)`:
- Eliminations: 3, 6, 2, 7, 5, 1
- **Survivor: 4** ✓

Circular list simulation: O(n·k) time (n eliminations, each k steps).
Closed-form recursion: `J(n,k) = (J(n-1,k) + k) % n`, base `J(1,k)=0` → **O(n)** time.

---

## 11. LRU cache

Hash map (key → node) + doubly-linked list (recency order). Head = most recent, tail = least recent.

- `get(k)`: O(1) lookup + O(1) move to head.
- `put(k,v)`: O(1) insert/update + O(1) evict tail if full.

**Every operation O(1).**

---

## 12. Palindrome check on a linked list

1. Find middle (two-pointer). O(n).
2. Reverse second half. O(n).
3. Compare first half with reversed second. O(n).
4. (Optional) Reverse second half back. O(n).

**Total O(n) time, O(1) space.**

---

## 13. Add two numbers

`2→4→3` + `5→6→4` (reversed digits):
- 342 + 465 = 807
- Result list: `7 → 0 → 8` ✓

**Algorithm.** Walk both lists simultaneously, sum digit + carry, create new node per digit.

---

## 14. Copy list with random pointers

**Three-pass O(n) approach:**
1. For each original node, insert a clone immediately after it: `A → A' → B → B' → …`
2. Set `clone.random = original.random.next` (uses cloned layout).
3. Split the interleaved list into original and clone.

**O(n) time, O(1) extra space.**

---

## 15. Intersection of two lists

1. Compute lengths `lenA`, `lenB`.
2. Advance the head of the longer list by `|lenA − lenB|`.
3. Walk both in lockstep — first common node is the intersection.

**O(m + n) time, O(1) space.**