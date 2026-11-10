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
  js: assets/lecture-07.js
---

# ECEG-4192

## Chapter 7 — Graphs

Representations · Traversals · Shortest Paths · MST

Semester VII · 2025/26

<aside class="notes">
Graphs are the most general structure in this course. Trees, lists, and networks are all special graphs.
</aside>

---

## Learning outcomes

By the end of this chapter you should be able to:

1. Define graphs and use **graph terminology** (degree, path, cycle, connected).

2. Choose **adjacency matrix** or **adjacency list** based on density.

3. Perform **DFS** and **BFS** in $O(V+E)$.

4. Explain **when** DFS or BFS applies.

5. Run **Dijkstra** on non-negative weighted graphs.

6. Compare **Prim** and **Kruskal** for MST.

---

## 7.1 — Graph definition

???+ note "Definition — Graph"
    $G = (V, E)$: a set of **vertices** $V$ and a set of **edges** $E$ connecting pairs of vertices.

<div class="fragment">

**Variants:**

- **Directed** (digraph) — edges have direction.
- **Undirected** — edges are two-way.
- **Weighted** — edges carry a cost.
- **Unweighted** — edges are uniform.

</div>

---

## 7.2 — Graph terminology

<div class="fragment">

- **Degree** — number of edges incident to a vertex.
- **Path** — sequence of vertices connected by edges.
- **Cycle** — path that starts and ends at the same vertex.

</div>

<div class="fragment">

- **Connected** — path between any two vertices (undirected).
- **Strongly connected** — directed, path in both directions.
- **DAG** — directed acyclic graph.
- **Sparse** — $|E| \approx |V|$.
- **Dense** — $|E| \approx |V|^2$.

</div>

---

## 7.3 — Adjacency matrix

$n \times n$ matrix: $A[i][j] = 1$ (or weight) if edge $i \to j$.

```
     A B C D
   A 0 1 1 0
   B 0 0 0 1
   C 0 1 0 1
   D 0 0 0 0
```

| Aspect | Value |
|---|---|
| Edge lookup | **O(1)** |
| Memory | **O(V²)** |
| Enumerate neighbors | O(V) |

<div class="fragment">

**Use:** dense graphs, frequent edge-existence checks.

**Don't use:** sparse graphs — massive waste.

</div>

---

## 7.4 — Adjacency list

Array of lists: `adj[u]` = neighbors of $u$.

```
A → [B, C]
B → [D]
C → [B, D]
D → []
```

| Aspect | Value |
|---|---|
| Edge lookup | **O(degree(u))** |
| Memory | **O(V + E)** |
| Enumerate neighbors | **O(degree(u))** |

<div class="fragment">

**Use:** sparse graphs, neighbor iteration (DFS, BFS, Dijkstra).

**Modern practice:** adjacency list is the default.

</div>

---

## Choosing a representation

| Scenario | Matrix | List |
|---|:---:|:---:|
| Dense graph ($E \sim V^2$) | ✅ | ❌ |
| Sparse graph ($E \sim V$) | ❌ | ✅ |
| Frequent edge lookup | ✅ | ❌ |
| Iterate all neighbors | ❌ | ✅ |
| Memory-constrained | ❌ | ✅ |

<div class="fragment">

**Rule of thumb:** use adjacency list unless you have a specific reason for matrix.

</div>

---

## 7.5 — Depth-First Search (DFS)

```python
def dfs(u, visited, adj):
    visited[u] = True
    for v in adj[u]:
        if not visited[v]:
            dfs(v, visited, adj)
```

<div class="fragment">

**Uses a stack** — explicit or via recursion.

**O(V + E)** time.

</div>

<div class="fragment">

**Applications:**

- **Connected components** — count them.
- **Cycle detection**.
- **Topological sort** (on DAG).
- **Maze solving**, **backtracking**.

</div>

---

## DFS traversal trace

```
     A
    / \
   B   C
  / \   \
 D   E   F
```

Start at A:

<div class="fragment">

1. A → B → D (backtrack) → E (backtrack) → A.
2. A → C → F (backtrack) → A.
3. Done.

**DFS order:** A, B, D, E, C, F.

**Stack in action** — depth pursued before breadth.

</div>

---

## 7.6 — Breadth-First Search (BFS)

```python
from collections import deque
def bfs(s, adj):
    visited = {s}
    q = deque([s])
    while q:
        u = q.popleft()
        for v in adj[u]:
            if v not in visited:
                visited.add(v)
                q.append(v)
```

<div class="fragment">

**Uses a queue.**

**O(V + E)** time.

</div>

<div class="fragment">

**Applications:**

- **Shortest path** (unweighted) — the number of edges.
- **Level-order traversal**.
- **Bipartite check**.
- **Social network** — degrees of separation.

</div>

---

## DFS vs BFS

| Aspect | DFS | BFS |
|---|---|---|
| Data structure | Stack | Queue |
| Explores | Deep first | Wide first |
| Shortest path (unweighted) | ❌ | ✅ |
| Memory | $O(h)$ — height | $O(w)$ — width |
| Typical use | Cycles, topo sort | Shortest path |

<div class="fragment">

**Rule:** shortest path on unweighted graph → **BFS**. Structural exploration, cycles → **DFS**.

</div>

---

## 7.7 — Shortest path — Dijkstra

???+ note "Goal"
    Single-source shortest path on a graph with **non-negative edge weights**.

<div class="fragment">

**Algorithm sketch:**

1. Initialize $dist[s] = 0$, all others $\infty$.
2. Repeatedly extract the vertex $u$ with minimum $dist[u]$ not yet visited.
3. For each neighbor $v$: relax $dist[v] = \min(dist[v], dist[u] + w(u,v))$.

</div>

<div class="fragment">

**Requires a priority queue** — typically a binary heap.

**Fails** on **negative weights** — use Bellman-Ford instead.

</div>

---

## Dijkstra — code

```python
import heapq
def dijkstra(s, adj, n):
    dist = [float('inf')] * n
    dist[s] = 0
    pq = [(0, s)]                        # (distance, vertex)
    while pq:
        d, u = heapq.heappop(pq)
        if d > dist[u]:
            continue                     # stale entry
        for v, w in adj[u]:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                heapq.heappush(pq, (dist[v], v))
    return dist
```

<div class="fragment">

**Complexity:** $O((V + E) \log V)$ with binary heap.

**Why log V?** Heap operations per edge relaxation.

</div>

---

## Dijkstra trace (small example)

Graph: `A-B=4, A-C=2, B-C=1, B-D=5, C-D=8, C-E=10, D-E=2`.

Start at A:

| Step | Extract | dist A B C D E |
|:---:|---|---|
| 0 | — | 0 ∞ ∞ ∞ ∞ |
| 1 | A (0) | 0 4 2 ∞ ∞ |
| 2 | C (2) | 0 3 2 10 12 |
| 3 | B (3) | 0 3 2 8 12 |
| 4 | D (8) | 0 3 2 8 10 |
| 5 | E (10) | 0 3 2 8 10 |

<div class="fragment">

**Shortest distances from A:** A=0, B=3, C=2, D=8, E=10.

</div>

---

## 7.8 — Minimum spanning tree (MST)

???+ note "Definition — MST"
    A subset of edges that **connects all vertices**, is **acyclic** (a tree), and has **minimum total weight**.

<div class="fragment">

**For a connected graph**, an MST always exists.

**Two classic algorithms:**

- **Prim's** — grow the tree one vertex at a time.
- **Kruskal's** — grow the tree one edge at a time.

</div>

---

## Prim's algorithm

**Grow the tree from a starting vertex.**

<div class="fragment">

1. Start with vertex $s$; tree = $\{s\}$.
2. Repeatedly add the **cheapest edge** connecting the tree to a vertex **not yet in the tree**.

</div>

<div class="fragment">

**Like Dijkstra** — but the priority is **edge weight to the tree**, not distance from source.

**Complexity:** $O((V + E) \log V)$ with binary heap.

**Good for:** dense graphs, online/growing graphs.

</div>

---

## Kruskal's algorithm

**Grow by adding cheapest edges that don't form a cycle.**

<div class="fragment">

1. Sort all edges by weight.
2. Add each edge, **skipping** if it would create a cycle (use **Union-Find**).

```python
def kruskal(edges, n):
    edges.sort(key=lambda e: e[2])
    uf = UnionFind(n)
    mst = []
    for u, v, w in edges:
        if uf.find(u) != uf.find(v):
            uf.union(u, v)
            mst.append((u, v, w))
    return mst
```

</div>

<div class="fragment">

**Complexity:** $O(E \log E)$ — dominated by sorting.

**Good for:** sparse graphs.

</div>

---

## Prim vs Kruskal

| Aspect | Prim | Kruskal |
|---|---|---|
| Approach | Grow from vertex | Grow from edge |
| Data structure | Heap + graph | Sorted edges + Union-Find |
| Complexity | $O((V+E)\log V)$ | $O(E \log E)$ |
| Best for | Dense graph | Sparse graph |
| Needs connectivity | From start vertex | Handles disconnected (forest) |

<div class="fragment">

**Both produce *an* MST** — weights are equal, tree may differ.

</div>

---

## Chapter summary

1. **Graph** = vertices + edges. Directed/undirected, weighted/unweighted.

2. **Adjacency matrix** — O(1) edge lookup, O(V²) memory.

3. **Adjacency list** — O(V+E) memory, default choice.

4. **DFS** — stack-based, deep-first, cycles, topo sort.

5. **BFS** — queue-based, wide-first, shortest unweighted paths.

6. **Dijkstra** — weighted shortest paths (non-negative), O((V+E)log V).

7. **Prim / Kruskal** — MST, both correct; pick by graph density.

---

## Review questions

1. Compare adjacency matrix and adjacency list. When is each preferable?

2. Trace DFS and BFS on a given graph.

3. Why does BFS find shortest unweighted paths but Dijkstra is needed for weighted?

4. Show Dijkstra's steps on the small graph in the trace.

5. Compare Prim's and Kruskal's algorithms.

6. What happens if Dijkstra is run on a graph with negative edges? Why does it fail?

> Answers: **Solutions → Chapter 7**.

---

## References

- **Weiss**, Ch. 9 — graph algorithms.
- **Lafore**, Ch. 13 (graphs), Ch. 14 (weighted graphs).
- **CLRS**, Ch. 22 (BFS/DFS), Ch. 24 (Dijkstra), Ch. 23 (MST).
- **Sedgewick & Wayne**, Ch. 4.