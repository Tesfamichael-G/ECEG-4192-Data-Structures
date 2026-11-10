# Solutions to Chapter 6 — Hash Tables

[:octicons-arrow-left-24: Back to Chapter 6](../chapters/06-hash-tables.md)

---

## 1. Division method — `{3, 6, 9, 12, 15}`

**m = 4:** h(k) = {3, 2, 1, 0, 3} — **collision** between 3 and 15 → 1 collision.

**m = 5:** h(k) = {3, 1, 4, 2, 0} — **all distinct** → 0 collisions.

**m = 7:** h(k) = {3, 6, 2, 5, 1} — **all distinct** → 0 collisions.

**Best: m = 5 or 7** (prime, doesn't share factors with the common difference 3). **Worst: m = 4** (has factor 2, doesn't divide 3 — so 3 and 15 collide).

Wait, m=4: 15 mod 4 = 3, 3 mod 4 = 3 — collide. Why? 15 - 3 = 12, and 12 is divisible by 4. Keys differing by multiples of m collide.

---

## 2. Multiplication method `m = 16`

`h(k) = ⌊16 · ((k · 0.618) mod 1)⌋`.

For k=1..5:
- 1·0.618 = 0.618 → 16·0.618 = 9.88 → **9**
- 2·0.618 = 1.236 → fract = 0.236 → 16·0.236 = 3.78 → **3**
- 3·0.618 = 1.854 → fract = 0.854 → 16·0.854 = 13.66 → **13**
- 4·0.618 = 2.472 → fract = 0.472 → 7.55 → **7**
- 5·0.618 = 3.09 → fract = 0.09 → 1.44 → **1**

**Result:** [9, 3, 13, 7, 1] — **all distinct**, spread across range.

---

## 3. Polynomial hash `"dog"` vs `"god"`, p=31, m=101

`h("dog")`: 100 + 111·31 + 103·31² = 100 + 3441 + 98,983 = 102,524 → 102,524 mod 101 = ?

101 · 1015 = 102,515. 102,524 − 102,515 = **9**.

`h("god")`: 103 + 111·31 + 100·961 = 103 + 3441 + 96,100 = 99,644.

101 · 986 = 99,586. 99,644 − 99,586 = **58**.

**Different (9 vs 58)** — order matters, hash values differ. ✓

---

## 4. Chaining trace, m=5, keys `{12, 17, 22, 27, 32, 37}`

h(k) = k mod 5: {2, 2, 2, 2, 2, 2}. **All collide at bucket 2!**

| Bucket 0 | 1 | 2 | 3 | 4 |
|:---:|:---:|---|---|---|
| — | — | 37 → 32 → 27 → 22 → 17 → 12 | — | — |

**Max chain length: 6.** This is a terrible hash function for these keys (all ≡ 2 mod 5). **Fix:** use a prime `m` that doesn't divide `(17-12) = 5, (22-17) = 5, ...` — wait, all differences are 5, so any `m` that divides 5 collides. **Use m = 7** (prime, doesn't divide 5) → all distinct.

---

## 5. Linear probing, m=7, same keys

h(k) = k mod 7: {5, 3, 1, 6, 4, 2} — **all distinct**, no probing needed.

Hmm, that's different from problem 4 because m changed from 5 to 7. All distinct → 0 probes.

Let me redo with m=5 (same as problem 4):
- Insert 12: h=2, empty → slot 2.
- Insert 17: h=2, occupied by 12 → probe 3, empty → slot 3.
- Insert 22: h=2, occupied by 12 → probe 3 (17) → probe 4 → slot 4.
- Insert 27: h=2 → 3 → 4 → probe 0 → slot 0.
- Insert 32: h=2 → 3 → 4 → 0 → 1 → slot 1.
- Insert 37: h=2 → 3 → 4 → 0 → 1 → 2 → **full** — need resize.

**Table:** [27, 32, 12, 17, 22] (all 5 slots filled, then resize).

**Max probes for insert 37: 6** (checked all 5 slots). Bad.

---

## 6. Quadratic probing, m=7, same keys `{12,17,22,27,32,37}`

h(k) = k mod 7 = {5, 3, 1, 6, 4, 2} — all distinct.

**No probing needed.** All slots filled directly.

---

## 7. Double hashing, m=11, keys `{22,33,44,55,66}`

h₁(k) = k mod 11, h₂(k) = 1 + (k mod 10).

| Key | h₁ | h₂ | Probe sequence |
|:---:|:---:|:---:|---|
| 22 | 0 | 1+2=3 | 0, 3, 6, 9, 1, 4, 7, 10, 2, 5, 8 |
| 33 | 0 | 1+3=4 | 0, 4, 8, 1, 5, 9, 2, 6, 10, 3, 7 |
| 44 | 0 | 1+4=5 | 0, 5, 10, 4, 9, 3, 8, 2, 7, 1, 6 |
| 55 | 0 | 1+5=6 | 0, 6, 1, 7, 2, 8, 3, 9, 4, 10, 5 |
| 66 | 0 | 1+6=7 | 0, 7, 3, 10, 6, 2, 9, 5, 1, 8, 4 |

**All start at h₁=0 (collision)** but follow **different** probe sequences thanks to h₂.

Insertion:
- 22 → slot 0.
- 33 → slot 0 occupied → probe 4 → slot 4.
- 44 → 0 occupied, 4 occupied → probe 10 → slot 10.
- 55 → 0, 4, 10 occupied → probe 1 → slot 1.
- 66 → 0, 4, 10, 1 occupied → probe 7 → slot 7.

**Final:** slot 0=22, 1=55, 4=33, 7=66, 10=44.

**No primary or secondary clustering** — the probe sequences diverge. ✓

---

## 8. Tombstone trace

Insert `{10, 20, 30}` into m=5 with linear probing, h(k) = k mod 5.

- 10 → slot 0.
- 20 → slot 0 occupied → slot 1.
- 30 → slot 0 occupied → slot 1 occupied → slot 2.

Table: [10, 20, 30, _, _].

**Delete 20** → tombstone at slot 1: [10, X, 30, _, _].

**Lookup 30:** h(30)=0 → slot 0 (10, no match) → slot 1 (X, keep going) → slot 2 (30, match) ✓ — **3 probes**.

**Now delete 10** → tombstone at slot 0: [X, X, 30, _, _].

**Lookup 30:** slot 0 (X) → slot 1 (X) → slot 2 (30) ✓ — **3 probes**.

**Without tombstones** (set deleted slots to EMPTY): after deleting 20, slot 1 is EMPTY. Lookup 30 starts at slot 0, sees 10 (no match), then slot 1 (EMPTY → **stop**), returns **"not found"** — WRONG.

**This is why tombstones are required.** ✓

---

## 9. Load factor α = 0.5, linear probing

**Successful lookup:** expected probes = ½(1 + 1/(1−0.5)) = ½(1 + 2) = **1.5**.

**Unsuccessful lookup:** expected probes = ½(1 + 1/(1−0.5)²) = ½(1 + 4) = **2.5**.

---

## 10. Amortised rehashing — 1000 inserts, doubling

Starting size 2, doubling at α > 0.75 → resize at sizes 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024.

Rehash at sizes 2, 4, ..., 512: total work = 2 + 4 + 8 + ... + 512 = **1022 operations**.

Plus 1000 inserts = 2022 operations total. **Amortised per insert: 2.022** = **O(1)**. ✓

---

## 11. Universal hashing — p=1009, m=20, a=17, b=43

h(k) = ((17k + 43) mod 1009) mod 20.

- h(50) = ((850 + 43) mod 1009) mod 20 = (893 mod 1009) mod 20 = 893 mod 20 = **13**.
- h(150) = ((2550 + 43) mod 1009) mod 20 = (2593 mod 1009) mod 20 = (2593 − 2018) mod 20 = 575 mod 20 = **15**.
- h(250) = ((4250 + 43) mod 1009) mod 20 = (4293 mod 1009) mod 20 = (4293 − 4·1009 = 4293 − 4036 = 257) mod 20 = **17**.

**All distinct** under this hash.

Under `h(k) = k mod 20`: 50 mod 20 = 10, 150 mod 20 = 10, 250 mod 20 = 10 — **all collide**. ✓

Universal hashing avoids the pattern.

---

## 12. Two-sum `[3, 5, 2, 7, 11, 15]`, target 9

| Step | x | target − x | in set? | set after |
|:---:|:---:|:---:|:---:|---|
| 1 | 3 | 6 | no | {3} |
| 2 | 5 | 4 | no | {3, 5} |
| 3 | 2 | 7 | no | {3, 5, 2} |
| 4 | 7 | 2 | **yes!** | — return true |

**Pair: (2, 7).** ✓

---

## 13. Longest substring without repeating chars

Sliding window with hash map `char → last index`. Extend right pointer; if char repeats, jump left pointer to `last index + 1`. Track max window size.

**O(n) time, O(min(n, alphabet)) space.**

For `"abcabcbb"`:
- Window "abc" (max 3) → next 'a' repeats → new window "bca" → etc.
- **Answer: 3.**

---

## 14. Group anagrams

For each string, sort characters → use as hash map key. Append original string to the value list.

**O(n · k log k)** time (k = max string length), **O(n · k)** space.

For `["eat", "tea", "tan", "ate", "nat", "bat"]`:
- "eat" sorted = "aet" → ["eat"]
- "tea" sorted = "aet" → ["eat", "tea"]
- "tan" sorted = "ant" → ["tan"]
- "ate" sorted = "aet" → ["eat", "tea", "ate"]
- "nat" sorted = "ant" → ["tan", "nat"]
- "bat" sorted = "abt" → ["bat"]

**Groups:** [["eat","tea","ate"], ["tan","nat"], ["bat"]].

---

## 15. Subarray sum = K

Track prefix sums in a hash map: `prefix → count`. For each index, `target_prefix = current_prefix − K`; if `target_prefix` is in the map, add its count to answer.

**O(n) time, O(n) space.**

For `[1, 1, 1]`, K=2:
- prefix = 1, map {0:1, 1:1}
- prefix = 2, target=0 in map (count=1) → ans += 1. Map {0:1, 1:1, 2:1}.
- prefix = 3, target=1 in map (count=1) → ans += 1. Map {0:1, 1:1, 2:1, 3:1}.

**Answer: 2** (subarrays [1,1] at indices 0-1 and 1-2). ✓