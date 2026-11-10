# Solutions to Chapter 7 — Graphs

[:octicons-arrow-left-24: Back to Chapter 7](../chapters/07-graphs.md)

---

## 1. Adjacency matrix vs. list — graph from Example 7.1 (n=6, m=7)

**Adjacency matrix:** 6×6 = 36 ints × 4 bytes = **144 bytes**.

**Adjacency list:** 6 header pointers × 8 bytes = 48 bytes, plus 14 entries (2 × 7 for undirected) × (int + pointer) = 14 × 12 = 168 bytes, total ≈ **216 bytes**.

**For sparse graphs (m ≈ n), memory is comparable**, but list scales with m, matrix with n². For n=10,000, m=20,000: matrix = 400 MB, list ≈ 240 KB — **1600× difference**.

---

## 2. BFS from 4 on Example 7.1 graph

Adjacency: 0: [1, 2], 1: [0, 3, 4], 2: [0, 4], 3: [1, 5], 4: [1, 2, 5], 5: [3, 4].

BFS from 4:

| Step | Queue | Visited |
|:---:|---|---|
| 0 | [4] | {4} |
| 1 | [1, 2, 5] | {4, 1, 2, 5} |
| 2 | [2, 5, 0, 3] | + {0, 3} |
| 3 | [5, 0, 3] | (none new from 2) |
| 4 | [0, 3] | (none new from 5) |
| 5 | [3] | (none new from 0) |
| 6 | [] | (none new from 3) |

**Visit order:** 4, 1, 2, 5, 0, 3. **Distances from 4:** [2, 1, 1, 2, 0, 1].

---

## 3. DFS from 2 (recursive, sorted neighbors)

DFS(2) → visit 2 → neighbors [0, 4].
- DFS(0) → visit 0 → neighbors [1, 2].
  - DFS(1) → visit 1 → neighbors [0, 3, 4].
    - DFS(3) → visit 3 → neighbors [1, 5].
      - DFS(5) → visit 5 → neighbors [3, 4].
        - DFS(4) → visit 4 → neighbors [1, 2, 5] — all visited.

**Visit order:** 2, 0, 1, 3, 5, 4.

**DFS tree edges:** 2→0, 0→1, 1→3, 3→5, 5→4.

---

## 4. Connected components

```java
int countComponents(List<List<Integer>> adj) {
    boolean[] visited = new boolean[adj.size()];
    int count = 0;
    for (int i = 0; i < adj.size(); i++) {
        if (!visited[i]) {
            bfsMark(adj, i, visited);
            count++;
        }
    }
    return count;
}
```

**O(V + E) time.**

---

## 5. Bipartite check

Color vertices with 2 colors such that every edge connects different colors. During BFS, alternate colors on each level. If an edge connects same-color vertices, graph is **not bipartite**.

**O(V + E) time.**

---

## 6. Cycle detection (undirected)

DFS: track parent. If a visited neighbor is not the parent, there's a cycle.

```java
boolean hasCycle(int u, int parent, boolean[] visited, List<List<Integer>> adj) {
    visited[u] = true;
    for (int v : adj.get(u)) {
        if (!visited[v]) {
            if (hasCycle(v, u, visited, adj)) return true;
        } else if (v != parent) return true;
    }
    return false;
}
```

**O(V + E) time.**

---

## 7. Cycle detection (directed)

3-color DFS: `WHITE` (unvisited), `GRAY` (in stack), `BLACK` (fully explored). Cycle if any back edge to a `GRAY` node.

The undirected approach (parent check) fails on directed graphs because `u → v` and `u → w → v` is not a cycle but a diamond — parent check flags `v` as seen which is wrong.

**O(V + E) time.**

---

## 8. Topological sort

**DFS approach:** reverse postorder.

```java
void topoDFS(int u, boolean[] visited, Deque<Integer> stack, List<List<Integer>> adj) {
    visited[u] = true;
    for (int v : adj.get(u)) if (!visited[v]) topoDFS(v, visited, stack, adj);
    stack.push(u);
}
// after all DFS starts, stack (bottom to top) is topological order
```

**Kahn's algorithm (BFS):** compute in-degrees; enqueue nodes with 0 in-degree; pop, decrement neighbors' in-degrees, enqueue new 0s.

Both **O(V + E)**. Result may differ if multiple valid orderings exist.

---

## 9. Dijkstra from vertex 2 (Example 7.3 graph)

Edges: 0→1 (4), 0→2 (1), 1→3 (1), 2→1 (3), 2→3 (5), 3→4 (2), 2→4 (6).

Start: dist = [∞, ∞, 0, ∞, ∞].

| Step | Pop | dist [0..4] |
|:---:|---|---|
| 1 | (2, 0) | [∞, 3, 0, 5, 6] |
| 2 | (1, 3) | [∞, 3, 0, 4, 6] (1→3: 3+1=4 < 5) |
| 3 | (3, 4) | [∞, 3, 0, 4, 6] (3→4: 4+2=6, no change) |
| 4 | (4, 6) | [∞, 3, 0, 4, 6] |

**Final:** dist = [∞, 3, 0, 4, 6]. Node 0 unreachable. ✓

---

## 10. Dijkstra fails with negative edges

Graph: `s→u (1), s→v (4), u→v (-3)`.

**Dijkstra from s:**
- Pop s (dist 0), relax: u←1, v←4.
- Pop u (dist 1), relax u→v: 1 + (-3) = -2 < 4 → **but wait**, if v was already popped at dist 4, we'd have missed this. In this graph, u is popped before v (dist 1 < 4), so v gets updated to -2. **Dijkstra actually works here.**

Let me construct a failure case: `s→u (1), s→v (2), u→v (-5)`.
- Pop s (dist 0), relax: u←1, v←2.
- Pop u (dist 1), relax u→v: 1 + (-5) = -4 < 2 → update v←-4. **Still works** because u is popped before v.

Construct: `s→u (5), s→v (3), u→v (-10)`, `u→w (1)`, `v→w (1)`.

- Pop s (0), relax: u←5, v←3.
- Pop v (3), relax v→w: 3+1=4 → w←4. Also v→u? No edge.
- Pop w (4), no outgoing. **w finalized at 4.**
- Pop u (5), relax u→v: 5+(-10)=-5 < 3 → **update v←-5**. relax u→w: 5+1=6 > 4, no.
- Pop v (-5). Done.

But w's dist was finalized at 4 — should have been 5 + (-10) + 1 = **-4** via u→v→w. Dijkstra missed it because it finalized w before relaxing u.

**Correct answer:** w = -4. Dijkstra returns 4. **Bellman–Ford gives -4.** ✓

---

## 11. Bellman–Ford with negative cycle

Graph: edges `(0,1,4), (0,2,5), (1,2,-3), (2,3,-1)`.

| Pass | dist [0..3] |
|:---:|---|
| init | [0, ∞, ∞, ∞] |
| 1 | [0, 4, 1, 0] |
| 2 | [0, 4, 1, 0] (stable) |
| 3 | [0, 4, 1, 0] (stable) |

**Converged in 2 passes.** No negative cycle.

Add `(3,1,-10)` → cycle 1→2→3→1: -3 + (-1) + (-10) = -14, negative cycle.

**Pass 3** would detect: relax (3,1) with dist[3] = 0: 0 + (-10) = -10 < 4 → update dist[1] = -10. **Change on the V-th pass → negative cycle detected.**

---

## 12. Kruskal union-find trace (Example 7.5)

Sorted edges: (1,2,1), (3,4,2), (3,5,4), (0,2,5), (1,3,5), (2,3,5), (0,1,6), (2,4,6), (4,5,6).

| Edge | Weight | Union? | Components |
|:---:|:---:|:---:|---|
| (1,2) | 1 | ✓ (roots different) | {1,2}, {3}, {4}, {5}, {0} |
| (3,4) | 2 | ✓ | {1,2}, {3,4}, {5}, {0} |
| (3,5) | 4 | ✓ | {1,2}, {3,4,5}, {0} |
| (0,2) | 5 | ✓ | {0,1,2}, {3,4,5} |
| (1,3) | 5 | ✓ | {0,1,2,3,4,5} — **done** |
| (2,3) | 5 | ✗ (same component) | — |
| (0,1) | 6 | ✗ | — |
| ... | | | |

**MST weight: 1 + 2 + 4 + 5 + 5 = 17.** ✓

---

## 13. Prim from vertex 3

| Step | Tree | Min edge to outside | Add |
|:---:|---|---|---|
| 1 | {3} | (3,4,2) | 4 (w=2) |
| 2 | {3,4} | (3,5,4) | 5 (w=4) |
| 3 | {3,4,5} | (2,3,5) or (1,3,5) — tie | 2 (w=5) |
| 4 | {3,4,5,2} | (1,2,1) or (1,3,5) | 1 (w=1) |
| 5 | {3,4,5,2,1} | (0,2,5) | 0 (w=5) |
| done | all | — | — |

**MST weight: 2 + 4 + 5 + 1 + 5 = 17.** ✓ Same as Kruskal.

---

## 14. MST uniqueness

MST is **unique** iff all edge weights are distinct. If two edges have equal weight, there may be multiple MSTs (as in Example 7.5, where edges (1,3), (2,3), (1,3) all have weight 5 — choosing either (1,3) or a different weight-5 edge gives a valid MST).

Example: triangle with all weights 1. Any two edges form an MST — 3 possible MSTs, all weight 2.

---

## 15. Road network

**Representation:** adjacency list — road networks are **sparse** (average city has ~3-4 roads per intersection).

**Shortest route:** Dijkstra — road weights are non-negative (distance).

**Cheapest construction:** Kruskal or Prim for MST — connects all towns with minimum total road length.

---

## 16. Social network — 100M users, path from Alice to Bob

Adjacency matrix: 100M × 100M ints × 4 bytes = **40 quadrillion bytes** = **40 petabytes**. Impossible.

Adjacency list: 100M users × ~200 friends × 12 bytes ≈ **240 GB**. Feasible on a large server.

**Use adjacency list** — sparse graph.