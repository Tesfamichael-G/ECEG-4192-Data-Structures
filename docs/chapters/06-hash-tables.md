# Chapter 6 — Hash Tables

!!! abstract "Chapter at a glance"
    **Contact hours:** 2 lecture + 1 tutorial + 2 lab · **Home study:** ~5 h
    
    **Coverage (syllabus):** hashing · hash functions · collisions · open addressing ·
    separate chaining.

---

## Learning outcomes

By the end of this chapter you should be able to:

1. Explain the **hash table idea** — mapping keys to array indices via a hash function.
2. Describe the properties of a **good hash function** (deterministic, uniform, fast).
3. Resolve collisions using **separate chaining** — linked lists at each bucket.
4. Resolve collisions using **open addressing** — linear probing, quadratic probing, double hashing.
5. Compute and reason about the **load factor** and its effect on performance.
6. Analyse the **average-case O(1)** and **worst-case O(n)** costs of hash tables.
7. Handle **deletion** correctly in open addressing (tombstones).
8. Implement hash tables in Java and Python.
9. Recognise common pitfalls (bad hash functions, adversarial inputs, integer overflow in hashing).
10. Choose between hash tables, balanced BSTs, and arrays for a given problem.

---

## 6.1 The hash table idea

### 6.1.1 The gap

Arrays give **O(1) access by index** — but indices are 0, 1, 2, … Keys are usually not small integers. If you have a set of `{name, email}` pairs and want `get("Alice")` in O(1), arrays won't work directly.

The idea: **map the key to an array index via a hash function**, and store the value at that index.

```
key "Alice" → hash(key) → 42 → store value at array[42]
```

Retrieval: same computation → same index → O(1) lookup.

### 6.1.2 The core insight

A **hash function** converts an arbitrary key (string, object, integer) into an **array index** in a bounded range `[0, m - 1]`, where `m` is the table size:

$$
\boxed{\;h : \text{Keys} \to [0, m-1]\;}
$$

Because the input range (all possible keys) is much larger than the output range (`m` slots), **collisions are inevitable** — two different keys hashing to the same index.

### 6.1.3 The cost guarantee

- **Best / average case:** O(1) for insert, lookup, delete — because hash + index is a constant number of operations.
- **Worst case:** O(n) — if all keys collide, the table degenerates to a linked list.

The goal of hash table design is to make the **average case O(1)** under realistic assumptions (random keys, good hash function, load factor bounded).

---

## 6.2 Hash functions

### 6.2.1 Properties of a good hash function

| Property | Meaning |
|---|---|
| **Deterministic** | `h(k)` returns the same value every time |
| **Uniform** | Each index is equally likely to be produced (for random inputs) |
| **Fast** | O(1) to compute, with a small constant |
| **Avalanche** | A small change in input produces a large change in output |
| **Bounded** | Returns index in `[0, m-1]` |

### 6.2.2 The division method

$$
\boxed{\;h(k) = k \bmod m\;}
$$

**Rules:**
- **`m` should be prime** — avoids patterns in input keys causing systematic collisions.
- **`m` should not be a power of 2** — for integer keys, `k mod 2^p` just takes the low `p` bits, discarding the rest; if inputs share low bits, collisions explode.
- **Good choice:** `m` a prime far from any power of 2, e.g., 97, 199, 4093.

### 6.2.3 The multiplication method

$$
\boxed{\;h(k) = \lfloor m \cdot (k \cdot A \bmod 1) \rfloor, \quad 0 < A < 1.\;}
$$

**Recommended `A`** (Knuth): $A \approx (\sqrt 5 - 1)/2 \approx 0.6180339887$.

**Why it works:** multiplication by `A`, keep the fractional part, scale by `m`. Result is in `[0, m)`. **Less sensitive to `m`'s value** than the division method — often used when `m` is a power of 2 for speed.

**When to use:** integer keys where you want `m` to be a power of 2 (for bit-shift efficiency).

### 6.2.4 Universal hashing

**Problem:** any *fixed* hash function can be defeated by an adversary who knows it — e.g., if `h(k) = k mod 97` and the adversary controls the input, they can send all keys as `97 · i` and defeat the table.

**Solution:** choose the hash function **randomly at runtime** from a **universal family**. An adversary who doesn't know which hash function is active can't predict collisions.

**Example family:**

$$
\boxed{\;h_{a,b}(k) = ((a \cdot k + b) \bmod p) \bmod m\;}
$$

where `p` is a prime larger than the largest key, and `a, b` are chosen uniformly at random at startup. Any two distinct keys collide with probability ≤ 1/m.

### Worked example 6.1 — Division method, collisions depend on `m`

**Keys:** `{10, 20, 30, 40, 50}`.

**Table size `m = 10`:** `h(k) = k mod 10` → `{0, 0, 0, 0, 0}` — **all collide** at index 0. Terrible.

**Table size `m = 7`:** `h(k) = k mod 7` → `{3, 6, 2, 5, 1}` — **all distinct**. Good.

**Table size `m = 11`:** `h(k) = k mod 11` → `{10, 9, 8, 7, 6}` — **all distinct**. Good.

**The lesson:** for these keys (all multiples of 10), `m = 10` is the worst possible choice. **Prime `m` avoids this trap** because it doesn't share factors with the keys' common divisor.

### Worked example 6.2 — String hashing via polynomial

**Polynomial rolling hash:**

$$
\boxed{\;h(s) = \left(\sum_{i=0}^{n-1} s[i] \cdot p^i\right) \bmod m\;}
$$

where `p` is a small prime (typically 31 or 131) and `s[i]` is the ASCII code of character `i`.

**Example:** `s = "cat"`, ASCII 99, 97, 116. `p = 31`, `m = 1009`.

$$
h = (99 + 97 \cdot 31 + 116 \cdot 31^2) \bmod 1009.
$$

Compute `31^2 = 961`:
- `99 · 1 = 99`
- `97 · 31 = 3007`
- `116 · 961 = 111,476`

Sum = `114,582`. `114,582 mod 1009 = ?`

`1009 · 113 = 114,017`. `114,582 - 114,017 = 565`. So `h("cat") = 565`.

**Why `p = 31`?** Good distribution for English strings; small enough to avoid overflow in 32-bit ints (with modular reduction per term). Java's `String.hashCode()` uses `p = 31` for this reason.

### Worked example 6.3 — Adversarial input defeats a fixed hash

**Setup:** `h(k) = k mod 1009`. An adversary sends keys `{1009, 2018, 3027, 4036, ...}` — all multiples of 1009.

**Every key hashes to 0.** All n keys collide at bucket 0 → every operation is O(n).

**Defense:** universal hashing (6.2.4) — the adversary can't predict the modulus because it's chosen randomly at runtime.

**Real-world attack:** in 2011, a "HashDoS" attack hit several web frameworks by sending form data with many colliding keys. Server CPU was exhausted on hash lookups. The fix was to switch to randomized hashing.

---

## 6.3 Separate chaining

### 6.3.1 The idea

Each slot in the table is a **linked list**. Colliding elements are stored in the same list.

```
Table:
[0] → null
[1] → ("Alice", 25) → ("Bob", 30) → null
[2] → null
[3] → ("Carol", 40) → null
[4] → ("Dave", 50) → ("Eve", 60) → ("Frank", 35) → null
[5] → null
```

### 6.3.2 Operations

```java
class ChainedHashTable<K, V> {
    class Node {
        K key; V value; Node next;
        Node(K k, V v) { key = k; value = v; }
    }
    private Node[] buckets;
    private int size;

    ChainedHashTable(int capacity) {
        buckets = new Node[capacity];
        size = 0;
    }

    private int hash(K key) {
        return Math.floorMod(key.hashCode(), buckets.length);
    }

    void put(K key, V value) {
        int i = hash(key);
        for (Node n = buckets[i]; n != null; n = n.next) {
            if (n.key.equals(key)) { n.value = value; return; }
        }
        Node newNode = new Node(key, value);
        newNode.next = buckets[i];
        buckets[i] = newNode;
        size++;
        if (size > buckets.length * 0.75) resize(buckets.length * 2);
    }

    V get(K key) {
        int i = hash(key);
        for (Node n = buckets[i]; n != null; n = n.next) {
            if (n.key.equals(key)) return n.value;
        }
        return null;
    }

    V remove(K key) {
        int i = hash(key);
        Node cur = buckets[i], prev = null;
        while (cur != null && !cur.key.equals(key)) {
            prev = cur; cur = cur.next;
        }
        if (cur == null) return null;
        if (prev == null) buckets[i] = cur.next;
        else              prev.next = cur.next;
        size--;
        return cur.value;
    }

    private void resize(int newCap) {
        Node[] old = buckets;
        buckets = new Node[newCap];
        size = 0;
        for (Node n : old) {
            for (Node cur = n; cur != null; cur = cur.next) put(cur.key, cur.value);
        }
    }
}
```

### 6.3.3 Cost analysis

Let `n` = number of elements, `m` = table size, **load factor** $\alpha = n/m$.

**Average case (uniform hashing):**

| Operation | Cost |
|---|:---:|
| `put` | O(1 + α) — hash + walk the list |
| `get` | O(1 + α) — hash + walk the list |
| `remove` | O(1 + α) |
| `resize` | O(n) when triggered — amortised O(1) |

**Worst case (all keys collide):** O(n) per operation — the table is one big list.

**Practical rule:** keep $\alpha \le 1$ (typically 0.75) → average O(1) with high probability.

### 6.3.4 Why $\alpha \le 1$?

Under uniform hashing, the expected length of a chain is exactly $\alpha$. For $\alpha = 0.75$:
- Expected chain length per lookup ≈ 0.75.
- Total expected comparisons per lookup ≈ `1 + α ≈ 1.75` — **constant**.

At $\alpha = 10$:
- Expected chain length = 10.
- Total ≈ 11 comparisons — 10× worse than $\alpha = 0.75$.

**Resize before $\alpha$ exceeds 0.75** to keep O(1) average.

### Worked example 6.4 — Separate chaining trace

**Setup:** `m = 5`, keys inserted in order: `{15, 11, 27, 8, 12}`, `h(k) = k mod 5`.

| Step | Insert key | `h(k)` | Bucket 0 | 1 | 2 | 3 | 4 |
|:---:|:---:|:---:|---|---|---|---|---|
| 1 | 15 | 0 | **15** | — | — | — | — |
| 2 | 11 | 1 | 15 | **11** | — | — | — |
| 3 | 27 | 2 | 15 | 11 | **27** | — | — |
| 4 | 8 | 3 | 15 | 11 | 27 | **8** | — |
| 5 | 12 | 2 | 15 | 11 | **27 → 12** | 8 | — |

**Key 12 collides with 27 at bucket 2.** Inserted at the front of the chain: `27 → 12`.

**Lookup 12:** `h(12) = 2` → walk bucket 2: `27` (≠ 12) → `12` ✓ — 2 comparisons.

**Lookup 8:** `h(8) = 3` → bucket 3: `8` ✓ — 1 comparison.

**Cost.** 5 inserts, average ~1.4 comparisons each. **O(1) average.** ✓

---

## 6.4 Open addressing

### 6.4.1 The idea

Instead of linked lists, **store everything directly in the array**. On collision, **probe** other slots in a deterministic sequence until an empty one is found.

```
Slots:  [ 0 ][ 1 ][ 2 ][ 3 ][ 4 ]
Insert 15 → slot 0 (empty)  ✓
Insert 11 → slot 1 (empty)  ✓
Insert 27 → slot 2 (empty)  ✓
Insert 8  → slot 3 (empty)  ✓
Insert 12 → slot 2 (occupied by 27) → probe slot 3 (occupied by 8) → probe slot 4 (empty)  ✓
```

**No pointers per element** — the table is a flat array. **Excellent cache locality** — potentially much faster than chaining in practice.

**The catch:** lookup must **reproduce the probe sequence** to find elements. And **deletion is tricky** (§6.4.5).

### 6.4.2 Linear probing

$$
\boxed{\;h(k, i) = (h(k) + i) \bmod m, \quad i = 0, 1, 2, \ldots\;}
$$

**Simplest sequence:** try `h(k)`, then `h(k)+1`, then `h(k)+2`, … wrapping around.

**Problem — clustering.** Once multiple consecutive slots are filled, any new key whose hash lands in the cluster must traverse the entire cluster. Long clusters grow longer — **primary clustering** — quickly degrading to O(n) average.

### 6.4.3 Quadratic probing

$$
\boxed{\;h(k, i) = (h(k) + c_1 i + c_2 i^2) \bmod m, \quad i = 0, 1, 2, \ldots\;}
$$

Common choice: $c_1 = 0, c_2 = 1$ → `h(k, i) = (h(k) + i²) mod m`.

**Reduces primary clustering** — probe sequence jumps more widely.

**Problem — secondary clustering.** All keys with the same initial hash `h(k)` follow the same probe sequence, so they still collide repeatedly.

**Catch:** quadratic probing may not visit all slots for arbitrary `m`. For `m` prime and $\alpha < 0.5$, it visits at least half the table — usually sufficient.

### 6.4.4 Double hashing

$$
\boxed{\;h(k, i) = (h_1(k) + i \cdot h_2(k)) \bmod m, \quad i = 0, 1, 2, \ldots\;}
$$

**Second hash function `h_2(k)`** determines the step size for key `k`. Keys with the same `h_1` but different `h_2` follow **different probe sequences** — no clustering at all.

**Good choices:**
- `h_1(k) = k mod m`
- `h_2(k) = 1 + (k mod m')` where `m'` is slightly smaller than `m` (often `m' = m - 1` or a prime).

**The `+1` is critical** — if `h_2(k) = 0`, the probe sequence is `h_1(k), h_1(k), h_1(k), …` — infinite loop. Adding 1 forces `h_2(k) ≥ 1`.

**Requirement:** `h_2(k)` must be **relatively prime to `m`** — then the probe sequence visits all slots.

**Best of the three** — no clustering, uniform probing. **Cost: two hash computations per probe.**

### 6.4.5 Deletion — tombstones

**The problem:** suppose you have entries at slots `[5]`, `[6]`, `[7]` (all from collisions on the same initial hash). Now delete `[6]`. If you simply set `[6]` to empty, a lookup for `[7]` will probe `[5]`, then `[6]` (empty!) — and conclude `[7]` doesn't exist, **even though it does**.

**Solution: tombstones.** Mark deleted slots with a special `DELETED` marker. Lookup continues past tombstones; insertion can reuse them.

```java
enum SlotState { EMPTY, OCCUPIED, DELETED }

class TombstoneHashTable {
    private int[] keys;
    private Object[] values;
    private SlotState[] state;
    private int size;

    TombstoneHashTable(int capacity) {
        keys = new int[capacity];
        values = new Object[capacity];
        state = new SlotState[capacity];
        java.util.Arrays.fill(state, SlotState.EMPTY);
    }

    void put(int key, Object value) {
        int i = Math.floorMod(key, keys.length);
        int firstDeleted = -1;
        for (int probe = 0; probe < keys.length; probe++) {
            int idx = (i + probe) % keys.length;
            if (state[idx] == SlotState.OCCUPIED) {
                if (keys[idx] == key) { values[idx] = value; return; }
            } else if (state[idx] == SlotState.DELETED) {
                if (firstDeleted == -1) firstDeleted = idx;
            } else {
                // EMPTY: insert here (or at firstDeleted if any)
                int ins = (firstDeleted != -1) ? firstDeleted : idx;
                keys[ins] = key;
                values[ins] = value;
                state[ins] = SlotState.OCCUPIED;
                size++;
                return;
            }
        }
    }

    Object get(int key) {
        int i = Math.floorMod(key, keys.length);
        for (int probe = 0; probe < keys.length; probe++) {
            int idx = (i + probe) % keys.length;
            if (state[idx] == SlotState.EMPTY) return null;   // stop only on EMPTY
            if (state[idx] == SlotState.OCCUPIED && keys[idx] == key) return values[idx];
        }
        return null;
    }

    void remove(int key) {
        int i = Math.floorMod(key, keys.length);
        for (int probe = 0; probe < keys.length; probe++) {
            int idx = (i + probe) % keys.length;
            if (state[idx] == SlotState.EMPTY) return;
            if (state[idx] == SlotState.OCCUPIED && keys[idx] == key) {
                state[idx] = SlotState.DELETED;
                size--;
                return;
            }
        }
    }
}
```

**The key rules:**

- **Lookup stops only on EMPTY**, never on DELETED.
- **Insert reuses DELETED slots** (record the first one seen).
- **Long runs of tombstones degrade performance** — a rehash (resize) sweeps them away.

### Worked example 6.5 — Linear probing trace

**Setup:** `m = 7`, insert keys `{10, 20, 15, 7, 22, 1}`, `h(k) = k mod 7`.

| Insert | `h(k)` | Probe sequence | Final slot |
|:---:|:---:|---|:---:|
| 10 | 3 | 3 | **3** |
| 20 | 6 | 6 | **6** |
| 15 | 1 | 1 | **1** |
| 7  | 0 | 0 | **0** |
| 22 | 1 | 1 (occupied by 15), 2 (empty) | **2** |
| 1  | 1 | 1 (15), 2 (22), 3 (10), 4 (empty) | **4** |

**Table after all inserts:**

| Slot | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Key | 7 | 15 | 22 | 10 | 1 | — | 20 |

**Lookup 1:** `h(1) = 1` → probe slots 1 (15), 2 (22), 3 (10), 4 (1) ✓ — **4 probes**.

**The cluster formed at slots 1–4 is the "primary clustering" problem in action.** With just 6 inserts, a lookup touches 4 slots. As the table fills, clusters merge and lookups degrade fast.

### Worked example 6.6 — Quadratic probing trace

**Setup:** `m = 7`, insert keys `{10, 20, 15, 7, 22, 1}`, same hash function, quadratic probing: `h(k, i) = (h(k) + i²) mod 7`.

**Probe sequences for the offending keys:**

- Key 22, `h = 1`: probes 1, 2 (=1+1), 5 (=1+4), 3 (=1+9), 4 (=1+16), 6 (=1+25), 0 (=1+36).
- Key 1, `h = 1`: probes 1, 2, 5, 3, 4, 6, 0 — **same sequence** (secondary clustering).

**Trace:**

| Insert | `h(k)` | Probe sequence | Final slot |
|:---:|:---:|---|:---:|
| 10 | 3 | 3 | 3 |
| 20 | 6 | 6 | 6 |
| 15 | 1 | 1 | 1 |
| 7 | 0 | 0 | 0 |
| 22 | 1 | 1 (15), 2 (empty) | **2** |
| 1 | 1 | 1 (15), 2 (22), 5 (empty) | **5** |

**Notice the difference:** key 1 lands at slot 5 (not 4) because the probe sequence for keys hashing to 1 is `1, 2, 5, 3, 4, 6, 0` — a **different sequence than linear probing**.

**Advantage:** key 1 skipped over slots 3 and 4 (which were occupied but far from `h=1`), reaching empty slot 5 in 3 probes. **No primary clustering** — the cluster at slot 1 didn't get any worse.

**Disadvantage:** secondary clustering remains — any new key with `h(k) = 1` will follow the same sequence.

---

## 6.5 Load factor and rehashing

### 6.5.1 Load factor

$$
\boxed{\;\alpha = \frac{n}{m}\;}
$$

where `n` = number of stored elements, `m` = table size.

**Critical for both methods:**

| Method | Max α before resize |
|---|---|
| Separate chaining | ~0.75–1.0 |
| Linear probing | ~0.5 |
| Quadratic probing | ~0.5 |
| Double hashing | ~0.75 |

**Linear probing degrades faster** — at α = 0.5, expected probe count is ~2.5; at α = 0.9, it's ~50. **For open addressing, resize aggressively.**

### 6.5.2 Expected probe counts (theoretical)

Under uniform hashing:

| Method | Expected probes (successful lookup) | Expected probes (unsuccessful) |
|---|:---:|:---:|
| Separate chaining | 1 + α/2 | α |
| Linear probing | ½ · (1 + 1/(1-α)) | ½ · (1 + 1/(1-α)²) |
| Quadratic probing / double hashing | -ln(1-α) / α | 1/(1-α) |

**The takeaway:** all methods are O(1) as long as **α is bounded away from 1**. For linear probing, α < 0.5 keeps the average lookup under 3 probes.

### 6.5.3 Rehashing

When α exceeds the threshold:

1. **Allocate a new table** of double the size.
2. **Reinsert every element** — recompute hash, insert fresh.

```java
private void rehash(int newCap) {
    int[] oldKeys = keys;
    Object[] oldValues = values;
    keys = new int[newCap];
    values = new Object[newCap];
    java.util.Arrays.fill(state, SlotState.EMPTY);
    // state array must be reallocated to newCap
    state = new SlotState[newCap];
    java.util.Arrays.fill(state, SlotState.EMPTY);
    size = 0;
    for (int i = 0; i < oldKeys.length; i++) {
        if (oldStates[i] == SlotState.OCCUPIED) {
            put(oldKeys[i], oldValues[i]);
        }
    }
}
```

**Cost:** O(n) — one full reinsertion pass.

**Amortised cost:** O(1) per operation — because the table doubles on each rehash, the total rehash cost over all operations is O(n), giving O(1) amortised per insertion (same doubling analysis as Chapter 2).

**Prime table sizes.** For the division-method hash, doubling a prime gives a non-prime — rehash to the **next prime** instead of exactly double. Many libraries ship a table of "next prime" values.

---

## 6.6 Java and Python implementations

### 6.6.1 Java's `HashMap`

Java's `HashMap` uses **separate chaining**, converting chains to red-black trees (Chapter 5) when a bucket exceeds 8 nodes. This hybrid gives O(log n) worst case rather than O(n) — a defence against adversarial inputs.

```java
Map<String, Integer> map = new HashMap<>();
map.put("Alice", 25);
map.get("Alice");           // 25
map.containsKey("Bob");     // false
map.remove("Alice");
map.getOrDefault("Charlie", 0);  // 0
```

### 6.6.2 Python's `dict`

Python's `dict` uses **open addressing with a custom probing scheme** and preserves **insertion order** (since Python 3.7). Average O(1) for `get`, `set`, `del`.

```python
d = {}
d["Alice"] = 25
d.get("Alice")              # 25
"Bob" in d                  # False
del d["Alice"]
```

**Python's `hash()`** — `hash(key)` — the built-in hash function is used by `dict` and `set`. Custom classes can define `__hash__` and `__eq__` for custom hashing.

### 6.6.3 What to use in production

**Always use the language's built-in hash table** unless you need custom behaviour (e.g., deterministic ordering, custom hash).

- **Java:** `HashMap`, `LinkedHashMap` (preserves insertion order), `ConcurrentHashMap` (thread-safe).
- **Python:** `dict`, `collections.OrderedDict` (equivalent to `dict` since 3.7), `collections.Counter`.
- **C++:** `std::unordered_map`, `std::unordered_set`.

**Rolling your own** is educational (this chapter) but error-prone in production.

---

## 6.7 Hash tables vs. balanced BSTs

| Aspect | Hash table | Balanced BST (AVL / RB) |
|---|---|---|
| **Lookup** | O(1) average, O(n) worst | **O(log n) guaranteed** |
| **Insert** | O(1) average | O(log n) |
| **Delete** | O(1) average | O(log n) |
| **Sorted iteration** | **not supported** | **O(n) — in-order** |
| **Range query** | not supported | O(log n + k) |
| **Predecessor / successor** | not supported | O(log n) |
| **Min / max** | not supported | O(log n) |
| **Worst-case bound** | O(n) if all collide | O(log n) |
| **Memory** | moderate (array + chains) | higher (pointers) |

**How to choose:**

- **Need ordered operations (range, sorted iteration, min/max)** → **balanced BST** (or `TreeMap` in Java, sorted containers).
- **Only need lookup/insert/delete** → **hash table** (faster in practice).
- **Adversarial input** → hash table with **randomised hashing** (universal hashing), or balanced BST for guaranteed bounds.
- **Small number of elements (< 50)** → array with linear search may be faster (avoid the constant overhead).

---

## Worked example 6.7 — When does hashing beat binary search?

**Setup:** A dataset of `n = 1000` records, each identified by an integer key. Two approaches:

**Approach 1 — Hash table.** `m = 1500`, α ≈ 0.67. Average successful lookup ≈ `1 + α/2 ≈ 1.33` comparisons.

**Approach 2 — Sorted array + binary search.** `⌈log₂ 1000⌉ = 10` comparisons.

**Hash table wins by ~8× on comparison count.**

**But:**

- **Cache effects.** A hash lookup touches a random slot → cache miss. Binary search touches 10 slots, all in the first few cache lines (since data is contiguous). **On small n, binary search may be faster in wall-clock.**
- **Sorted array supports range queries.** Hash table doesn't.
- **For sorted-insertion workloads**, a hash table requires no sorting cost — but a sorted array does.

**Rule:** for **pure equality lookups** on large `n` (> 10⁴), hash table wins. For **ordered queries** or **small n**, balanced BST or sorted array.

### Worked example 6.8 — Two-sum problem with a hash set

**Problem.** Given an array and a target `T`, find two elements summing to `T`.

**Brute force:** O(n²).

**Sorted array + two-pointer:** O(n log n) sort + O(n) scan.

**Hash set approach:** O(n) expected.

```java
boolean hasTwoSum(int[] a, int target) {
    Set<Integer> seen = new HashSet<>();
    for (int x : a) {
        if (seen.contains(target - x)) return true;
        seen.add(x);
    }
    return false;
}
```

**Why O(n):** each element `x` requires one hash lookup (O(1)) and one hash insert (O(1)). Total: 2n hash operations.

**The hash table lets us check "have we seen `target - x`?" in O(1) instead of O(n).** This is a classic example of trading space for time — the hash set uses O(n) extra space to avoid O(n) scans.

**Compare with sorted-array approach:** O(n log n) total, no extra space beyond sorting in place. **Trade-off:** hash set is faster (O(n) vs. O(n log n)) but uses O(n) memory.

---

## Chapter summary

1. **Hash tables** map keys to array indices via a hash function — **O(1) average-case** insert, lookup, delete.
2. **Good hash functions** are deterministic, uniform, fast, and have avalanche behaviour.
3. The **division method** `h(k) = k mod m` requires `m` prime and not a power of 2.
4. **String hashing** uses polynomial rolling hashes (Java uses `p = 31`).
5. **Separate chaining** stores collisions in linked lists; **O(1 + α)** average per operation.
6. **Open addressing** stores collisions in the same array, probing alternatives — **no pointer overhead** but needs careful deletion (tombstones).
7. **Linear probing** suffers **primary clustering**; **quadratic probing** reduces it; **double hashing** eliminates it.
8. **Tombstones** (`DELETED` markers) enable deletion in open addressing — lookup stops only on EMPTY.
9. Keep the **load factor below 0.5–0.75**; rehash (double size, reinsert) when it exceeds this.
10. **Hash tables excel at lookup by key** but cannot answer ordered queries — **balanced BSTs (AVL) are the choice for ordered operations**.

---

## Quiz

### A. Multiple choice

1. A hash table's average-case lookup time is:

   (a) O(1) (b) O(log n) (c) O(n) (d) O(n log n)

2. Its worst-case lookup is:

   (a) O(1) (b) O(log n) (c) O(n) (d) O(n²)

3. For `h(k) = k mod m`, `m` should be:

   (a) a power of 2 (b) **prime** (c) even (d) equal to `n`

4. Separate chaining resolves collisions using:

   (a) open addresses (b) **linked lists at each bucket** (c) probing (d) rehashing

5. Linear probing is affected by:

   (a) **primary clustering** (b) secondary clustering (c) tombstones (d) adversarial input

6. Double hashing eliminates:

   (a) primary only (b) secondary only (c) **both primary and secondary clustering** (d) nothing

7. Deletion in open addressing uses:

   (a) null values (b) **tombstones** (c) linked lists (d) rescaling

8. Load factor α should be kept below roughly:

   (a) 0.1 (b) **0.5–0.75** (c) 1.5 (d) 3

9. Rehashing during insertion has what amortised cost per operation?

   (a) O(1) (b) O(log n) (c) O(n) (d) O(n log n)

10. Which cannot be done efficiently on a hash table?

    (a) insert (b) lookup (c) **range query** (d) delete

11. Which defense prevents an adversary from crafting all-colliding inputs?

    (a) red-black tree at each bucket (b) **randomised hashing** (c) larger `m`
    (d) fewer elements

12. Java's `HashMap` converts long chains to:

    (a) arrays (b) stacks (c) **red-black trees** (d) graphs

### B. Fill in the blank

1. A hash function maps keys to indices in the range **[0, \_\_\_\_\_** - 1].

2. Two keys hashing to the same index produce a **\_\_\_\_\_**.

3. The division method computes `h(k) = k mod \_\_\_\_\_**.

4. For the division method, `m` should be a **\_\_\_\_\_**.

5. Java's `String.hashCode()` uses polynomial hashing with `p = \_\_\_\_\_**.

6. In **separate chaining**, each bucket holds a **\_\_\_\_\_**.

7. In **open addressing**, collisions are stored in the same **\_\_\_\_\_**.

8. **Linear probing** suffers from **\_\_\_\_\_** clustering.

9. **Double hashing** uses a second hash function to determine the **\_\_\_\_\_** between probes.

10. Deleted slots in open addressing are marked with a **\_\_\_\_\_**.

### C. Define / explain

1. Explain the **hash table idea**. Why can we achieve O(1) lookup in the average case?

2. State the properties of a **good hash function**. Why is prime `m` important for the division method?

3. Describe **separate chaining** and its cost analysis in terms of load factor `α`.

4. Describe **open addressing**. What is the **critical difference** from chaining in terms of memory and cache?

5. Compare **linear probing, quadratic probing, and double hashing**. Which clustering problem does each have or solve?

6. Explain the **tombstone problem** in open addressing. Why can't we simply set deleted slots to EMPTY?

7. What is **universal hashing**? Give the definition of a universal family and explain why it defends against adversarial input.

8. When should you use a **hash table** vs. a **balanced BST**? Give three criteria.

9. Describe the **polynomial rolling hash** for strings. Why `p = 31`?

10. Explain how a hash **set** can solve the **two-sum problem** in O(n). What is the space cost?

---

## Answer key

??? success "A. Multiple choice"
    1. **(a)** O(1) average.
    2. **(c)** O(n) worst case.
    3. **(b)** prime.
    4. **(b)** linked lists at each bucket.
    5. **(a)** primary clustering.
    6. **(c)** both primary and secondary.
    7. **(b)** tombstones.
    8. **(b)** 0.5–0.75.
    9. **(a)** O(1) amortised.
    10. **(c)** range query.
    11. **(b)** randomised hashing.
    12. **(c)** red-black trees.

??? success "B. Fill in the blank"
    1. $m$
    2. collision
    3. $m$
    4. prime
    5. 31
    6. linked list
    7. array
    8. primary
    9. step (or interval)
    10. tombstone

??? success "C. Define / explain — model answers"
    1. *Hash table idea:* a hash function `h(k)` maps each key to an array index; storing and retrieving is then a constant-time array access at that index. *Why O(1) average:* under uniform hashing, each lookup lands on a bucket with expected load α = n/m. If α is bounded (e.g., 0.75), the expected number of comparisons is ~1 + α/2 — a **constant**. So average case is O(1). Worst case is O(n) if all keys collide, but with good hash + bounded α this is astronomically unlikely.
    2. *Properties:* deterministic, uniform over `[0, m-1]`, fast to compute (O(1) with small constant), avalanche (small input change → large output change). *Why prime `m`:* if `m` has nontrivial factors, keys that share those factors all hash to multiples of those factors, creating systematic collisions. Example: `m = 10`, keys `{10, 20, 30}` — all multiples of 10 — all land at index 0. Prime `m` has no nontrivial factors, so no such pattern arises.
    3. *Separate chaining:* each table slot is a linked list. On collision, append (or prepend) to the list. Cost: `O(1 + α)` where α = n/m is the load factor. Under uniform hashing, expected chain length = α; expected comparisons per lookup ≈ 1 + α/2. Keep α ≤ 0.75 → constant cost per operation. Resize (double) when α exceeds threshold.
    4. *Open addressing:* all keys stored **directly in the array**; on collision, probe a deterministic sequence of alternative indices. *Memory/cache:* no per-element pointers → half the memory of chaining. Excellent cache locality because the whole table is contiguous. *Trade-off:* deletion requires tombstones; performance degrades faster as α grows (linear probing at α = 0.9 is 50 probes per lookup).
    5. *Linear probing:* `h(k, i) = (h(k) + i) mod m` — simplest, but suffers **primary clustering** (runs of occupied slots grow). *Quadratic probing:* `h(k, i) = (h(k) + c₁i + c₂i²) mod m` — breaks primary clustering, but suffers **secondary clustering** (same initial hash → same probe sequence). *Double hashing:* `h(k, i) = (h₁(k) + i·h₂(k)) mod m` — no clustering at all, since different keys with the same `h₁` have different probe sequences (determined by `h₂`). Best performance, but requires **two hash computations** per probe.
    6. *Tombstone problem:* suppose keys `{10, 27, 8}` all hash to slot 3. Using linear probing: slot 3 = 10, slot 4 = 27, slot 5 = 8. Now delete slot 4 (27). If we set slot 4 = EMPTY, a lookup for 8 will probe slot 3 (10, no), then slot 4 (EMPTY → stop) → **returns "not found"**, even though 8 is at slot 5. *Solution:* set slot 4 = DELETED (tombstone). Lookup continues past DELETED slots; insertion can reuse them. *Consequence:* long tombstone chains degrade performance — periodically rehash to sweep them.
    7. *Universal hashing:* a family `H` of hash functions such that for any two distinct keys `k₁ ≠ k₂`, the probability (over random choice of `h ∈ H`) that `h(k₁) = h(k₂)` is at most `1/m`. *Definition family:* `h_{a,b}(k) = ((a·k + b) mod p) mod m` for `p` prime, `a, b` chosen uniformly at random (with `a ≠ 0`). *Defends against adversarial input:* the adversary doesn't know which hash function is active, so they cannot craft all-colliding inputs. Even if they send keys with a predicted pattern, the random choice of `a, b` means the expected number of collisions is bounded by α. This is the standard defence against HashDoS attacks.
    8. *Hash vs. balanced BST:* (i) **Ordered operations** (range, min/max, predecessor) → BST (hash tables can't answer these efficiently). (ii) **Worst-case guarantee** — BST gives O(log n) worst case, hash table O(n) worst case → BST for adversarial input (or hash with universal hashing). (iii) **Insertion-order preservation** (e.g., LRU tracking) → BST with a custom key, or a separate linked list. For **pure unordered lookup** on large datasets, hash table wins in average practice.
    9. *Polynomial rolling hash:* `h(s) = (Σ s[i] · p^i) mod m`. Each character is weighted by a power of `p`. *Why `p = 31`:* (i) small enough that arithmetic stays in machine integers; (ii) prime, so no divisor-related pattern in the output; (iii) for English text, gives good distribution — adjacent strings (differing in one character) produce very different hashes. `p = 131` is also common (better for long strings). Java's `String.hashCode()` uses 31.
    10. *Two-sum with a hash set:* iterate through the array once. For each element `x`, check if `(target − x)` is already in the set (O(1)); if yes, return true; otherwise insert `x` into the set. Total: 2n hash operations = **O(n)** time. *Space:* O(n) for the set. *Trade-off:* beats the sorted-array approach's O(n log n) time but uses O(n) extra memory.

---

## Further reading

- **CLRS**, Chapter 11 (Hash Tables) — the definitive treatment; universal hashing in detail.
- **Weiss**, Chapter 5 (Hashing).
- **Lafore**, Chapter 11 (Hash Tables).
- **Sedgewick & Wayne**, Chapter 3.4 (Hash Tables).
- **Open Data Structures**, Chapter 5 (Hash Tables) — with rigorous analysis of open addressing.
- **Knuth**, *The Art of Computer Programming*, Volume 3 (Sorting and Searching) — Section 6.4 (Hashing) — the classic.
- **Joel Spolsky**, "The Law of Leaky Abstractions" — [joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/](https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/) — relevant for understanding why hash tables fail in adversarial settings.
- **"HashDoS"**: [cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2011-4461](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2011-4461) — the class of attacks that motivated universal hashing in production.

---

## Suggested tutorial problems

1. **Division method, `m` choice.** Insert `{3, 6, 9, 12, 15}` into tables of size 4, 5, and 7 with `h(k) = k mod m`. Count the number of collisions in each. Which `m` is best? Why?

2. **Multiplication method.** Compute `h(k) = ⌊m · (k · 0.618 mod 1)⌋` for `m = 16` and keys `1, 2, 3, 4, 5`. Plot the distribution of 100 random keys.

3. **String hash.** Compute polynomial hash of `"dog"` with `p = 31` and `m = 101`. Then `"god"` — do they differ?

4. **Chaining trace.** With `m = 5`, insert keys `{12, 17, 22, 27, 32, 37}` using `h(k) = k mod 5`. Draw the chained table. What is the maximum chain length?

5. **Linear probing trace.** Same keys, same hash, using linear probing with `m = 7`. Show the table after each insert. What is the maximum probe count?

6. **Quadratic probing trace.** Same as problem 5 but with quadratic probing. Show the table.

7. **Double hashing.** For `m = 11`, `h₁(k) = k mod 11`, `h₂(k) = 1 + (k mod 10)`, insert keys `{22, 33, 44, 55, 66}`. Show each key's probe sequence.

8. **Tombstone trace.** Insert keys `{10, 20, 30}` into `m = 5` table with linear probing. Delete 20. Now look up 30 — how many probes? Now delete 10 and look up 30 — how many probes?

9. **Load factor.** At α = 0.5 with linear probing, how many probes on average for successful and unsuccessful lookups?

10. **Amortised rehashing.** Insert 1000 elements into a hash table starting with size 2, doubling whenever α > 0.75. Compute the total number of rehash steps and the amortised cost per insert.

11. **Universal hashing.** Let `p = 1009`, `m = 20`. Choose `a = 17`, `b = 43`. Compute `h(50)`, `h(150)`, `h(250)`. Are these keys likely to collide under this hash? What about under `h(k) = k mod 20`?

12. **Two-sum.** Given `[3, 5, 2, 7, 11, 15]` and target `9`, use a hash set to find a pair. Trace each step.

13. **Longest substring without repeating characters.** Given a string, find the longest substring without repeating characters. Use a sliding window with a hash map (char → last index). O(n) time, O(min(n, alphabet_size)) space.

14. **Group anagrams.** Given a list of strings, group anagrams together. Use a hash map from sorted-string → list of anagrams.

15. **Subarray sum equals K.** Given an array of integers and `K`, count the number of contiguous subarrays summing to `K`. Use a hash map of prefix sums.