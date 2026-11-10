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
  js: assets/lecture-06.js
---

# ECEG-4192

## Chapter 6 — Hash Tables

Hashing · Collisions · Open Addressing · Chaining

Semester VII · 2025/26

<aside class="notes">
Hash tables give average O(1) lookups — the fastest dictionary we will see. The price is worst-case O(n).
</aside>

---

## Learning outcomes

By the end of this chapter you should be able to:

1. Explain the **hash table** concept.

2. Identify properties of a **good hash function**.

3. Explain **why collisions happen** and are unavoidable.

4. Distinguish **open addressing** and **separate chaining**.

5. Compare **linear probing, quadratic probing, double hashing**.

6. Analyse complexity of hash operations **average vs worst**.

---

## 6.1 — Hash table concept

???+ note "Definition — Hash table"
    A data structure that maps **keys → values** using a **hash function** $h(k)$ that computes an index in $[0, m-1]$.

```
key "alice" → h("alice") = 3 → table[3] = (alice, 42)
```

<div class="fragment">

**Average access: O(1).**

**Used in:** dictionaries, sets, caches, symbol tables, databases.

</div>

---

## Why hash tables work

| Operation | Array | Linked list | Hash table (avg) |
|---|:---:|:---:|:---:|
| Search | O(n) | O(n) | **O(1)** |
| Insert | O(n) | O(n) | **O(1)** |
| Delete | O(n) | O(1)* | **O(1)** |

<div class="fragment">

**"Average O(1)" is the whole point** — near-array speed with near-list flexibility.

!!! warning "The catch"
    The O(1) is **average**, not worst. Bad hash function or full table → **O(n)**.

</div>

---

## 6.2 — Hash functions

**Good hash function properties:**

<div class="fragment">

- **Deterministic** — same key always → same hash.
- **Uniform** — keys spread evenly across buckets.
- **Fast** — O(1) to compute.

</div>

<div class="fragment">

**Common methods:**

- **Division:** $h(k) = k \bmod m$ (choose $m$ prime).
- **Multiplication:** $h(k) = \lfloor m(kA \bmod 1) \rfloor$ with $0 < A < 1$.
- **Universal hashing:** randomly chosen from a family — beats adversarial inputs.

</div>

---

## Division method — the classic

$$ h(k) = k \bmod m $$

**Choose $m$ prime**, not a power of 2.

<div class="fragment">

Why? If $m = 2^p$, then $h(k)$ = the low $p$ bits of $k$. If keys all end in the same bits (e.g., they're all even), the low bits carry no information — mass collisions.

</div>

<div class="fragment">

**Prime $m$** makes every bit of $k$ affect the result.

- **Bad:** $m = 1000$ (many composite factors).
- **Good:** $m = 997$ (prime near 1000).

</div>

---

## 6.3 — Collisions

???+ note "Definition — Collision"
    Two distinct keys $k_1 \ne k_2$ with $h(k_1) = h(k_2)$.

<div class="fragment">

**Collisions are unavoidable** when #keys > $m$ (pigeonhole principle).

Even with #keys < $m$, they happen — birthday paradox says collisions occur with $\sqrt{m}$ keys.

</div>

<div class="fragment">

**Two resolution strategies:**

1. **Open addressing** — probe other slots.
2. **Separate chaining** — linked list per bucket.

</div>

---

## 6.4 — Open addressing

**On collision, probe alternate slots in a sequence.**

<div class="fragment">

**Linear probing:** $h(k), h(k)+1, h(k)+2, \ldots$ (mod $m$).

**Quadratic probing:** $h(k), h(k)+1^2, h(k)+2^2, \ldots$

**Double hashing:** $h_1(k), h_1(k)+h_2(k), h_1(k)+2h_2(k), \ldots$

</div>

<div class="fragment">

**Search** follows the same probe sequence until the key is found or an empty slot is hit.

</div>

---

## Linear probing — trace

Table size $m = 7$, hash $h(k) = k \bmod 7$. Insert 10, 20, 15, 7, 22.

| Insert | $h(k)$ | Probe sequence | Placed at |
|:---:|:---:|---|:---:|
| 10 | 3 | 3 | 3 |
| 20 | 6 | 6 | 6 |
| 15 | 1 | 1 | 1 |
| 7 | 0 | 0 | 0 |
| 22 | 1 | 1, 2 | 2 |

<div class="fragment">

Search for 22: start at 1 → 2 → found. **2 probes.**

**Watch:** 15 at 1, 22 at 2 — clustering around index 1.

</div>

---

## Clustering — linear probing's weakness

**Primary clustering:** long runs of occupied slots grow longer, making future insertions slower.

<div class="fragment">

With load factor $\alpha$, expected probes ≈ $\frac{1}{2}(1 + 1/(1-\alpha)^2)$. Blows up as $\alpha \to 1$.

</div>

<div class="fragment">

**Fix 1 — Quadratic probing:** less primary clustering, but still **secondary clustering** (same hash → same probe sequence).

**Fix 2 — Double hashing:** eliminates secondary clustering. **Best in practice.**

</div>

---

## Load factor

???+ note "Definition — Load factor"
    $$ \alpha = \frac{n}{m} $$
    
    where $n$ = number of stored keys, $m$ = table size.

<div class="fragment">

**Rules of thumb:**

- Open addressing: keep $\alpha < 0.7$.
- Chaining: $\alpha$ can exceed 1.

**When $\alpha$ grows too large:** **resize** the table (typically double), **rehash all keys**.

**Resize costs O(n)** — amortised O(1) per insert (same argument as dynamic arrays).

</div>

---

## Deletion in open addressing

**Problem:** you can't just set the slot to empty — a later search may have probed past it.

<div class="fragment">

**Solution: tombstones.**

```
[10] [TOMBSTONE] [15] [22] [ ] ...
```

</div>

<div class="fragment">

**Search:** treat tombstone as "keep probing."

**Insert:** treat tombstone as empty (reuse it).

**Delete:** replace with tombstone.

**Cost:** tombstones accumulate. **Periodic rehash** cleans them.

</div>

---

## 6.5 — Separate chaining

Each bucket holds a **linked list** of entries.

```
table[0] → NULL
table[1] → [k1,v1] → [k3,v3] → NULL
table[2] → [k2,v2] → NULL
table[3] → NULL
```

<div class="fragment">

**Search:** hash to bucket, scan the list.

**Insert:** hash to bucket, prepend to list — O(1).

**Delete:** hash to bucket, unlink node — O(1) (with doubly linked or predecessor).

</div>

---

## Chaining — complexity

Average chain length = $\alpha = n/m$.

| Operation | Average | Worst |
|---|:---:|:---:|
| Search | **O(1 + α)** | O(n) |
| Insert | **O(1)** | O(n) |
| Delete | **O(1)** | O(n) |

<div class="fragment">

If $m \propto n$, then $\alpha = O(1)$ → **all operations O(1) average**.

Worst case: all keys collide → **linked list**, **O(n)**.

</div>

---

## Open addressing vs chaining

| Aspect | Open addressing | Chaining |
|---|---|---|
| Memory | **One array** — no pointers | Array + per-entry pointer overhead |
| Cache | **Cache-friendly** | Pointer chasing |
| Load factor | Must stay < 0.7 | Can exceed 1 |
| Delete | **Tricky (tombstones)** | **Simple** |
| Worst-case | O(n) | O(n) |
| Clustering | Susceptible | Not susceptible |

<div class="fragment">

**Modern practice:** open addressing with **Robin Hood hashing** or **cuckoo hashing** often wins in cache-heavy workloads (e.g., Rust's `HashMap`).

</div>

---

## 6.6 — Complexity summary

| Operation | Average | Worst |
|---|:---:|:---:|
| Search | **O(1)** | O(n) |
| Insert | **O(1)** | O(n) |
| Delete | **O(1)** | O(n) |

<div class="fragment">

**Average assumes:** good hash function **and** low load factor.

**Worst:** all keys collide — degenerate to linear scan.

**Adversarial inputs:** if keys are chosen by an attacker who knows the hash function, worst case is achievable — **defend with universal hashing** or **random seeds**.

</div>

---

## Chapter summary

1. **Hash table** = array + hash function + collision resolution.

2. **Average O(1)** for search/insert/delete — **worst O(n)**.

3. **Good hash function** = deterministic, uniform, fast.

4. **Collisions are unavoidable** — resolved by **open addressing** or **chaining**.

5. **Open addressing** (probe): cache-friendly, deletion tricky (tombstones).

6. **Chaining** (lists): simple deletion, pointer overhead.

7. **Load factor** determines performance — keep it low, resize when high.

---

## Review questions

1. What is a hash function? List three properties of a good one.

2. Explain linear vs quadratic probing. What is clustering?

3. Compare open addressing and separate chaining.

4. Why does the load factor matter? What happens as α → 1?

5. Trace insertion of `[5, 15, 25, 35]` into a table of size 7, with $h(k) = k \bmod 7$. Show probes.

6. Why are **tombstones** needed in open-addressing deletion? What happens without them?

> Answers: **Solutions → Chapter 6**.

---

## References

- **Weiss**, Ch. 5 — hashing in depth.
- **Lafore**, Ch. 11 — hash tables in Java.
- **CLRS**, Ch. 11 — hashing, universal hashing, open addressing.
- **Sedgewick & Wayne**, §3.4 — hash tables.