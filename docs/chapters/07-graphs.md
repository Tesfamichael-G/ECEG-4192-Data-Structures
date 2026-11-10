# Chapter 7 — Graphs

!!! abstract "Chapter at a glance"
    **Contact hours:** 2 lecture + 1 tutorial + 2 lab · **Home study:** ~5 h
    
    **Coverage (syllabus):** definition · implementation (adjacency matrix, adjacency
    list) · DFS and BFS · selected graph algorithms (shortest path, minimum spanning
    tree) · applications.

---

## Learning outcomes

By the end of this chapter you should be able to:

1. Define the **graph terminology**: vertex, edge, directed vs. undirected, weighted, cycle, path.
2. Represent a graph using **adjacency matrix**, **adjacency list**, and (for weighted graphs) **edge list**.
3. Explain the **time/memory trade-offs** of each representation.
4. Implement **DFS** and **BFS** — both recursively and iteratively.
5. Use DFS and BFS to solve: **connected components**, **cycle detection**, **topological sort**, **bipartite check**.
6. Implement **Dijkstra's algorithm** for single-source shortest paths (non-negative weights).
7. Implement **Bellman–Ford** for graphs with negative edges; **detect negative cycles**.
8. Implement **Prim's** and **Kruskal's** algorithms for minimum spanning trees.
9. Recognise when to use **Dijkstra vs. Bellman–Ford vs. BFS** for shortest paths.
10. Apply graph algorithms to real problems (networks, routing, dependency resolution, recommendation).

---

## 7.1 Graph terminology

### 7.1.1 Definition

A **graph** $G = (V, E)$ consists of:

- **$V$** — a set of **vertices** (nodes).
- **$E$** — a set of **edges** — pairs of vertices.

| Term | Meaning |
|---|---|
| **Directed** | Edges have direction: $(u, v) \ne (v, u)$ |
| **Undirected** | Edges are symmetric: $\{u, v\} = \{v, u\}$ |
| **Weighted** | Each edge has a numeric **weight** (cost, distance, capacity) |
| **Adjacent** | Two vertices connected by an edge |
| **Degree** | Number of edges touching a vertex |
| **Path** | Sequence of vertices with edges between consecutive ones |
| **Cycle** | Path that starts and ends at the same vertex, with no repeated edges |
| **Connected** | Every vertex reachable from every other (undirected) |
| **Strongly connected** | Every vertex reachable from every other, following edges (directed) |
| **DAG** | Directed Acyclic Graph — directed, no cycles |

### 7.1.2 Sparse vs. dense

A graph with $n$ vertices can have up to $\binom{n}{2} = \frac{n(n-1)}{2}$ edges (undirected).

| Density | Edges `m` | Memory implication |
|---|---|---|
| **Sparse** | $m \approx O(n)$ | Adjacency list is best |
| **Dense** | $m \approx O(n²)$ | Adjacency matrix is fine |

**Most real-world graphs are sparse.** A road network with a million cities has maybe 2–3 million road segments — not $10^{12}$.

---

## 7.2 Graph representations

### 7.2.1 Adjacency matrix

A 2D array `matrix[i][j] = 1` if there's an edge from `i` to `j`; `0` otherwise. For weighted graphs, store the weight instead.

```java
class AdjacencyMatrix {
    int[][] matrix;
    int n;
    
    AdjacencyMatrix(int n) {
        this.n = n;
        this.matrix = new int[n][n];   // 0 = no edge, weight = edge present
    }
    
    void addEdge(int u, int v, int w) {
        matrix[u][v] = w;
        matrix[v][u] = w;   // for undirected
    }
    
    boolean hasEdge(int u, int v) {
        return matrix[u][v] != 0;
    }
    
    List<Integer> neighbors(int u) {
        List<Integer> result = new ArrayList<>();
        for (int v = 0; v < n; v++) {
            if (matrix[u][v] != 0) result.add(v);
        }
        return result;
    }
}
```

| Operation | Cost |
|---|:---:|
| Add edge | O(1) |
| Check edge | **O(1)** |
| List neighbors | O(n) |
| Memory | **O(n²)** |

**Best for:** dense graphs, or when you need O(1) edge lookup. **Worst for:** sparse graphs (wasted memory).

### 7.2.2 Adjacency list

An array of lists — `adj[i]` contains all neighbors of `i`.

```java
class AdjacencyList {
    List<List<Integer>> adj;
    int n;
    
    AdjacencyList(int n) {
        this.n = n;
        this.adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    }
    
    void addEdge(int u, int v) {
        adj.get(u).add(v);
        adj.get(v).add(u);   // for undirected
    }
    
    boolean hasEdge(int u, int v) {
        return adj.get(u).contains(v);   // O(degree(u))
    }
    
    List<Integer> neighbors(int u) {
        return adj.get(u);
    }
}
```

| Operation | Cost |
|---|:---:|
| Add edge | O(1) |
| Check edge | O(degree(u)) — worst O(n) |
| List neighbors | **O(degree(u))** — optimal |
| Memory | **O(n + m)** — optimal for sparse |

**Best for:** sparse graphs (the common case). **Worst for:** edge lookup.

### 7.2.3 Weighted adjacency list

```java
class Edge {
    int to, weight;
    Edge(int to, int w) { this.to = to; this.weight = w; }
}

class WeightedGraph {
    List<List<Edge>> adj;
    int n;
    
    WeightedGraph(int n) {
        this.n = n;
        adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    }
    
    void addEdge(int u, int v, int w) {
        adj.get(u).add(new Edge(v, w));
        adj.get(v).add(new Edge(u, w));
    }
}
```

This is what Dijkstra, Prim, and Kruskal expect.

### 7.2.4 Comparison

| Aspect | Adjacency matrix | Adjacency list |
|---|---|---|
| Memory | O(n²) | **O(n + m)** |
| Add edge | O(1) | O(1) |
| Check edge | **O(1)** | O(degree) |
| List neighbors | O(n) | **O(degree)** |
| Dense graph | fine | O(n²) too — no savings |
| Sparse graph | **wastes O(n²)** | best |

**Rule:** adjacency list unless the graph is dense (> ~30% of possible edges) or the algorithm needs many edge lookups.

---

## 7.3 Breadth-First Search (BFS)

### 7.3.1 The algorithm

```java
void bfs(List<List<Integer>> adj, int start) {
    boolean[] visited = new boolean[adj.size()];
    Queue<Integer> queue = new ArrayDeque<>();
    queue.add(start);
    visited[start] = true;
    while (!queue.isEmpty()) {
        int u = queue.remove();
        System.out.println(u);
        for (int v : adj.get(u)) {
            if (!visited[v]) {
                visited[v] = true;
                queue.add(v);
            }
        }
    }
}
```

**Idea:** expand outward from the start, visiting all distance-1 vertices first, then all distance-2, then all distance-3, etc.

**Cost:** O(V + E) — every vertex and edge visited at most once.

**Space:** O(V) for the queue and visited set.

### 7.3.2 Properties

- **Shortest paths in unweighted graphs.** BFS explores by edge count; the first time it reaches a vertex, that's the shortest path.
- **Level-order in trees** (Chapter 5).
- **Undirected connectivity:** BFS from a start vertex reaches all vertices in its connected component.

### 7.3.3 BFS computes shortest distances

```java
int[] bfsDistances(List<List<Integer>> adj, int start) {
    int[] dist = new int[adj.size()];
    Arrays.fill(dist, -1);
    dist[start] = 0;
    Queue<Integer> queue = new ArrayDeque<>();
    queue.add(start);
    while (!queue.isEmpty()) {
        int u = queue.remove();
        for (int v : adj.get(u)) {
            if (dist[v] == -1) {
                dist[v] = dist[u] + 1;
                queue.add(v);
            }
        }
    }
    return dist;
}
```

**Proof of correctness.** BFS processes vertices in non-decreasing order of distance from `start`. When it reaches `v` from `u`, `dist[v]` is set to `dist[u] + 1`. If a shorter path existed, BFS would have found it earlier (contradiction with the FIFO order). ✓

**Cost:** O(V + E).

---

## 7.4 Depth-First Search (DFS)

### 7.4.1 The algorithm — recursive

```java
void dfs(List<List<Integer>> adj, int u, boolean[] visited) {
    visited[u] = true;
    System.out.println(u);
    for (int v : adj.get(u)) {
        if (!visited[v]) dfs(adj, v, visited);
    }
}
```

**Idea:** dive as deep as possible before backtracking. Follow one neighbor all the way to a dead end, then backtrack.

**Cost:** O(V + E).

**Space:** O(V) for visited + O(h) call stack, where h is the maximum depth.

### 7.4.2 The algorithm — iterative

```java
void dfsIterative(List<List<Integer>> adj, int start) {
    boolean[] visited = new boolean[adj.size()];
    Deque<Integer> stack = new ArrayDeque<>();
    stack.push(start);
    while (!stack.isEmpty()) {
        int u = stack.pop();
        if (visited[u]) continue;
        visited[u] = true;
        System.out.println(u);
        for (int v : adj.get(u)) {
            if (!visited[v]) stack.push(v);
        }
    }
}
```

**Same result as recursive DFS** — but no recursion depth risk.

### 7.4.3 DFS properties

- **Cycle detection** — if we find a back edge to an ancestor, there's a cycle.
- **Topological sort** — reverse postorder of a DFS on a DAG.
- **Connected components** — count separate DFS starts.
- **Bipartite check** — 2-coloring during DFS.
- **DFS tree depth = O(V) worst case** — recursive DFS can stack-overflow on large graphs.

---

## Worked example 7.1 — BFS trace on a small graph

**Graph (undirected):**
```
0 — 1 — 3
|   |   |
2 — 4 — 5
```

**Edges:** `{(0,1), (0,2), (1,3), (1,4), (2,4), (3,5), (4,5)}`.

**BFS from 0:**

| Step | Queue | Visited | `dist` |
|:---:|---|---|---|
| 0 | [0] | {0} | 0:0 |
| 1 | [1, 2] | {0, 1, 2} | 1:1, 2:1 |
| 2 | [2, 3, 4] | {0, 1, 2, 3, 4} | 3:2, 4:2 |
| 3 | [3, 4] | (none new from 2) | |
| 4 | [4, 5] | {0–5} | 5:3 |
| 5 | [5] | | |
| 6 | [] | done | |

**Visit order:** `0, 1, 2, 3, 4, 5` — level by level. **Distances:** `[0, 1, 1, 2, 2, 3]`.

**The BFS tree edges** (parent of each vertex): `0 → {1, 2}, 1 → {3, 4}, 4 → 5`.

**Note:** there are two shortest paths to vertex 5 (via 3 and via 4), both length 3. BFS finds one — the one reached first in BFS order.

---

## Worked example 7.2 — DFS trace on the same graph

**DFS from 0 (recursive, neighbors in sorted order):**

**Trace:**

| Step | Stack (call) | Action | Visited |
|:---:|---|---|---|
| 1 | dfs(0) | visit 0 | {0} |
| 2 | dfs(1) | visit 1 | {0, 1} |
| 3 | dfs(3) | visit 3 (from 1) | {0, 1, 3} |
| 4 | dfs(5) | visit 5 (from 3) | {0, 1, 3, 5} |
| 5 | | neighbor 4 (from 5) | |
| 6 | dfs(4) | visit 4 | {0, 1, 3, 4, 5} |
| 7 | | neighbor 0 (visited), 1 (visited), 2 (unvisited) | |
| 8 | dfs(2) | visit 2 | {0, 1, 2, 3, 4, 5} |
| 9 | | back up | |

**Visit order:** `0, 1, 3, 5, 4, 2` — **not** the same as BFS. DFS dives deep first.

**The DFS tree edges:** `0 → 1, 1 → 3, 3 → 5, 5 → 4, 4 → 2`. A **path-shaped** tree — depth 5.

**Contrast with BFS tree depth: 3.**

---

## 7.5 Shortest paths — Dijkstra's algorithm

### 7.5.1 The problem

Given a weighted graph with **non-negative** edge weights, find the shortest path from a source vertex `s` to every other vertex.

**"Shortest"** means minimal **sum of edge weights** — not edge count. BFS solves this only when all weights are equal (unweighted = weight 1 each).

### 7.5.2 The algorithm

```java
int[] dijkstra(List<List<Edge>> adj, int source) {
    int n = adj.size();
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[source] = 0;
    
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[1] - b[1]);
    pq.add(new int[]{source, 0});
    
    boolean[] settled = new boolean[n];
    
    while (!pq.isEmpty()) {
        int[] cur = pq.poll();
        int u = cur[0];
        if (settled[u]) continue;
        settled[u] = true;
        
        for (Edge e : adj.get(u)) {
            int v = e.to, w = e.weight;
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.add(new int[]{v, dist[v]});
            }
        }
    }
    return dist;
}
```

**Idea:**

1. Start with `dist[s] = 0`, all others infinity.
2. Repeatedly pick the unvisited vertex with the smallest tentative distance.
3. **Relax** its outgoing edges: if `dist[u] + w(u,v) < dist[v]`, update `dist[v]`.
4. Repeat until all vertices settled.

**The priority queue** (Chapter 5 heap) makes picking the minimum vertex O(log V).

### 7.5.3 Correctness

**Invariant:** once a vertex `u` is settled (popped from PQ), its `dist[u]` is final.

**Proof by contradiction.** Suppose `u` is settled with tentative distance `D`, but there's a shorter path `P` of length `L < D`. Along `P`, at least one vertex is unsettled. Let `x` be the first unsettled vertex on `P`, with predecessor `p` (settled). By induction, `dist[p]` is final. When `p` was settled, `dist[x]` was updated to `dist[p] + w(p,x) ≤ L < D`. So `x`'s tentative distance is < D. Since the PQ pops minimum, `x` (or some vertex with even smaller tentative distance) would have been popped before `u` — contradicting that `u` was popped now. Therefore `D = L`. ✓

**Requires non-negative weights** — the invariant breaks with negative edges (a path through a settled vertex might be improved later).

### 7.5.4 Complexity

| Implementation | Complexity |
|---|---|
| Binary heap PQ | **O((V + E) log V)** |
| Fibonacci heap | O(E + V log V) |
| Array (for dense) | O(V²) |

**For sparse graphs** (E ≈ V), binary heap gives ~O(V log V). **For dense graphs** (E ≈ V²), the array version O(V²) wins.

### Worked example 7.3 — Dijkstra trace

**Graph (directed weighted, 6 vertices):**
```
        0
       / \
      4   1
     /    \
    1 —— 2 —— 3
      3  1   2
        \  /
         5
```

More formally:

Edges: `0→1 (4), 0→2 (1), 1→3 (1), 2→1 (3), 2→3 (5), 3→4 (2), 2→4 (6)`.

**Dijkstra from 0:**

Init: `dist = [0, ∞, ∞, ∞, ∞]`, PQ = `{(0, 0)}`.

| Step | Pop `(u, d)` | Settled set | `dist` (0..4) |
|:---:|---|---|---|
| 1 | (0, 0) | {0} | [0, 4, 1, ∞, ∞] |
| 2 | (2, 1) | {0, 2} | [0, 4, 1, 6, 7] — relax 2→1 (4), 2→3 (5), 2→4 (7) |
| 3 | (1, 4) | {0, 1, 2} | [0, 4, 1, 5, 7] — relax 1→3 (5 < 6) |
| 4 | (3, 5) | {0, 1, 2, 3} | [0, 4, 1, 5, 7] — 3→4 (7, no update) |
| 5 | (4, 7) | {0, 1, 2, 3, 4} | done |

**Final distances from 0:**
- `dist[0] = 0`
- `dist[1] = 4` (direct edge 0→1, weight 4)
- `dist[2] = 1` (direct edge 0→2)
- `dist[3] = 5` (path 0→2→1→3, total 1+3+1=5)
- `dist[4] = 7` (path 0→2→4, total 1+6=7)

**Verify by hand:** shortest path to 3 is 0→2→1→3 (5), not 0→1→3 (5) — actually equal here. Shortest to 4 is 0→2→4 (7) or 0→2→1→3→4 (1+3+1+2=7) — again equal.

---

## 7.6 Bellman–Ford — negative edges

### 7.6.1 The problem

Dijkstra assumes **non-negative weights**. If the graph has negative edges, Dijkstra's invariant breaks — it may settle a vertex too early.

**Example.** Graph `s → u` (weight 1), `s → v` (weight 4), `u → v` (weight -3). Dijkstra settles `u` first (dist 1), then `v` at dist 4 (via direct edge) — but the path `s → u → v` has cost `1 + (-3) = -2`, shorter. Dijkstra misses this because it settled `u` before considering the negative edge.

### 7.6.2 Bellman–Ford algorithm

```java
int[] bellmanFord(int n, List<int[]> edges, int source) {
    // edges: [u, v, w]
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[source] = 0;
    
    for (int i = 0; i < n - 1; i++) {
        for (int[] e : edges) {
            int u = e[0], v = e[1], w = e[2];
            if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
            }
        }
    }
    
    // Check for negative-weight cycles
    for (int[] e : edges) {
        int u = e[0], v = e[1], w = e[2];
        if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v]) {
            throw new RuntimeException("Negative cycle detected");
        }
    }
    return dist;
}
```

**Idea:** relax every edge `V - 1` times. Why `V - 1`?

**The shortest path has at most V - 1 edges.** (If it had V edges, it would visit V+1 vertices — contradiction, since a shortest path visits each vertex at most once.) After `k` full passes, `dist[v]` is the shortest path using at most `k` edges. After `V-1` passes, all shortest paths are found.

**Negative cycle detection:** if a further pass improves any distance, there must be a negative cycle (a cycle whose total weight is negative) — no finite shortest path exists.

### 7.6.3 Complexity

**O(V · E)** — V passes × E edge relaxations.

| Graph | Cost |
|---|---|
| Sparse (E ≈ V) | O(V²) |
| Dense (E ≈ V²) | O(V³) |

**Much slower than Dijkstra** — but handles negative edges and detects negative cycles.

### Worked example 7.4 — Bellman–Ford with a negative edge

**Graph:** `s=0 → 1 (4), 0 → 2 (5), 1 → 2 (-3)`.

**Bellman–Ford from 0:**

**Init:** `dist = [0, ∞, ∞]`.

**Pass 1 (over all edges):**
- Edge (0, 1, 4): `dist[1] = min(∞, 0 + 4) = 4`.
- Edge (0, 2, 5): `dist[2] = min(∞, 0 + 5) = 5`.
- Edge (1, 2, -3): `dist[2] = min(5, 4 + (-3)) = 1`.

**Pass 2 (over all edges):**
- Edge (0, 1, 4): `dist[1] = 4` (no change).
- Edge (0, 2, 5): `dist[2] = min(1, 5) = 1` (no change).
- Edge (1, 2, -3): `dist[2] = min(1, 4 - 3) = 1` (no change).

**Pass 3 (over all edges):** no changes → converged.

**Final:** `dist = [0, 4, 1]`.

**Correct:** `dist[2] = 1` (via 0 → 1 → 2, cost 4 + (-3) = 1), not 5 (direct). **Dijkstra would have returned 5** because it would settle node 2 at cost 5 before discovering the negative edge.

**Verify — Dijkstra on this graph:** settle 0 (dist 0), then 1 (dist 4, since direct edge weight 4 < ... nothing else), then 2. When we relax 0 → 2, dist[2] = 5. When we later relax 1 → 2 (weight -3), we'd update dist[2] to 1 — **but only if 1 is popped from the PQ before 2**. If PQ pops 2 first (dist 5 < ∞), then 2 is settled at 5, and the -3 edge updates never help. **Dijkstra returns 5 — wrong.** ✓ (This confirms Bellman–Ford is needed.)

---

## 7.7 Minimum Spanning Tree (MST)

### 7.7.1 The problem

Given a **connected, undirected, weighted graph**, find a subset of edges that connects all vertices with **minimum total weight**.

A **spanning tree** has exactly `V - 1` edges, connects all vertices, contains no cycles.

**MST applications:**

- **Network design** — laying cable / fiber to connect all homes with minimum total cost.
- **Road networks** — building roads to connect all towns with minimum total length.
- **Clustering** — MST edges connect similar items; cutting MST edges gives clusters.
- **Approximation algorithms** — the MST gives a 2-approximation to the Traveling Salesman Problem.

### 7.7.2 The cut property — the underlying theorem

**Cut.** A partition of `V` into two non-empty sets `S` and `V - S`.

**Cut property.** For any cut `(S, V - S)`, the **minimum-weight edge crossing the cut** belongs to some MST.

**Why this is the key.** Both Prim's and Kruskal's algorithms are grounded in this property — they greedily select edges that are minimum-weight crossing some cut.

### 7.7.3 Kruskal's algorithm

**Idea:** sort all edges by weight; add them to the MST one at a time, skipping edges that would create a cycle. **Uses Union-Find** for fast cycle detection.

```java
class UnionFind {
    int[] parent, rank;
    UnionFind(int n) {
        parent = new int[n];
        rank = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
    }
    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);   // path compression
        return parent[x];
    }
    boolean union(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;   // already in same set
        if (rank[rx] < rank[ry]) parent[rx] = ry;
        else if (rank[rx] > rank[ry]) parent[ry] = rx;
        else { parent[ry] = rx; rank[rx]++; }
        return true;
    }
}

List<int[]> kruskal(int n, List<int[]> edges) {
    // edges: [u, v, w]
    edges.sort((a, b) -> a[2] - b[2]);
    UnionFind uf = new UnionFind(n);
    List<int[]> mst = new ArrayList<>();
    for (int[] e : edges) {
        if (uf.union(e[0], e[1])) {
            mst.add(e);
            if (mst.size() == n - 1) break;
        }
    }
    return mst;
}
```

**Cost:** O(E log E) for sorting + O(E α(V)) for Union-Find operations. **O(E log E) total** — dominated by sorting.

**Sparse graphs:** O(E log E) ≈ O(V log V) — competitive.
**Dense graphs:** O(V² log V) — Prim's with an array is faster.

### Worked example 7.5 — Kruskal's trace

**Graph (6 vertices, 9 edges):**

| Edge | Weight |
|---|:---:|
| (0,1) | 6 |
| (0,2) | 5 |
| (1,2) | 1 |
| (1,3) | 5 |
| (2,3) | 5 |
| (2,4) | 6 |
| (3,4) | 2 |
| (3,5) | 4 |
| (4,5) | 6 |

**Sort edges by weight:**

1. `(1,2)` — 1
2. `(3,4)` — 2
3. `(3,5)` — 4
4. `(0,2)` — 5
5. `(1,3)` — 5
6. `(2,3)` — 5
7. `(0,1)` — 6
8. `(2,4)` — 6
9. `(4,5)` — 6

**Process:**

| Edge | Add? | MST edges | Total weight |
|:---:|:---:|---|---|
| (1,2), 1 | ✓ | {(1,2)} | 1 |
| (3,4), 2 | ✓ | {(1,2), (3,4)} | 3 |
| (3,5), 4 | ✓ | {(1,2), (3,4), (3,5)} | 7 |
| (0,2), 5 | ✓ | {(1,2), (3,4), (3,5), (0,2)} | 12 |
| (1,3), 5 | ✓ (connects {1,2,0} to {3,4,5}) | {(1,2), (3,4), (3,5), (0,2), (1,3)} | 17 |
| (2,3), 5 | ✗ (creates cycle 1-2-3-1) | — | — |
| (0,1), 6 | ✗ (cycle) | — | — |
| ... | | | |

**MST edges: 5** = V - 1 = 6 - 1. ✓

**Total weight: 17.**

**Verify it's a tree:** `0–2, 1–2, 1–3, 3–4, 3–5`. All 6 vertices connected, 5 edges, no cycles. ✓

### 7.7.4 Prim's algorithm

**Idea:** grow the MST from a start vertex, always adding the cheapest edge connecting a vertex **inside** the current tree to one **outside**.

```java
int prim(List<List<Edge>> adj, int start) {
    int n = adj.size();
    boolean[] inMST = new boolean[n];
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[1] - b[1]);
    pq.add(new int[]{start, 0});
    int totalWeight = 0;
    
    while (!pq.isEmpty()) {
        int[] cur = pq.poll();
        int u = cur[0], w = cur[1];
        if (inMST[u]) continue;
        inMST[u] = true;
        totalWeight += w;
        
        for (Edge e : adj.get(u)) {
            if (!inMST[e.to]) pq.add(new int[]{e.to, e.weight});
        }
    }
    return totalWeight;
}
```

**Cost:** O((V + E) log V) with binary heap — same as Dijkstra.

**For dense graphs:** array-based version O(V²).

**Advantage over Kruskal:** grows the MST from a specific start — useful when edges are not known in advance (e.g., a physical network you're exploring).

### Worked example 7.6 — Prim's on the same graph

**Start at vertex 0.**

| Step | Tree vertices | Min edge to outside | Add |
|:---:|---|---|---|
| 1 | {0} | (0,2, 5) | 2 (weight 5) |
| 2 | {0,2} | (1,2, 1) | 1 (weight 1) |
| 3 | {0,1,2} | (1,3, 5) | 3 (weight 5) |
| 4 | {0,1,2,3} | (3,4, 2) | 4 (weight 2) |
| 5 | {0–4} | (3,5, 4) | 5 (weight 4) |
| done | all | — | — |

**Total: 5 + 1 + 5 + 2 + 4 = 17.** Same as Kruskal. ✓

**MST is not unique** — when there are ties in edge weights, multiple MSTs may exist. But **total weight is unique** (all MSTs have the same total).

---

## 7.8 Which shortest-path algorithm?

| Algorithm | Handles negative weights? | Detects negative cycles? | Time complexity | Best for |
|---|:---:|:---:|:---:|---|
| **BFS** | n/a (unweighted) | n/a | O(V + E) | Unweighted graphs |
| **Dijkstra** | **No** | No | O((V+E) log V) | Non-negative weights, single source |
| **Bellman–Ford** | **Yes** | **Yes** | O(VE) | Negative edges, negative-cycle detection |
| **Floyd–Warshall** | Yes | Yes | O(V³) | All-pairs shortest paths (small graphs) |

**Flowchart:**

```
Weights negative? → No → Unweighted? → Yes → BFS
                  |     → No → Single source? → Yes → Dijkstra
                  |                            → No → Floyd-Warshall
                  → Yes → Single source? → Yes → Bellman-Ford
                                       → No → Floyd-Warshall (with negative cycle detection)
```

---

## Chapter summary

1. **Graph** = vertices + edges. Directed vs. undirected; weighted vs. unweighted; sparse vs. dense.
2. **Adjacency matrix:** O(n²) memory, O(1) edge lookup — best for dense graphs.
3. **Adjacency list:** O(n + m) memory, O(degree) edge lookup — best for sparse graphs (**most real graphs**).
4. **BFS** — level-by-level, uses a **queue**, O(V + E). Finds shortest paths in unweighted graphs.
5. **DFS** — dives deep, uses a **stack** (or recursion), O(V + E). Basis for cycle detection, topological sort, connected components.
6. **Dijkstra** — single-source shortest paths on **non-negative** weights, uses a priority queue. O((V + E) log V).
7. **Bellman–Ford** — handles **negative** weights, detects **negative cycles**. O(VE) — slower but more general.
8. **MST** — minimum total weight to connect all vertices. **Kruskal** (edge-sort + union-find) and **Prim** (grow from vertex) both compute MST in O(E log E) or O((V+E) log V).
9. **Cut property** — the minimum edge across any cut is in some MST. This theorem justifies the greedy strategies of both Kruskal and Prim.

---

## Quiz

### A. Multiple choice

1. The **degree** of a vertex in an undirected graph is:

   (a) its index (b) **the number of edges touching it** (c) its weight (d) the length of its path to root

2. Adjacency list memory is:

   (a) O(n) (b) **O(n + m)** (c) O(n²) (d) O(2ⁿ)

3. BFS uses which data structure?

   (a) stack (b) **queue** (c) heap (d) hash table

4. DFS uses which data structure?

   (a) **stack (or recursion)** (b) queue (c) heap (d) hash table

5. BFS finds shortest paths in:

   (a) weighted graphs (b) **unweighted graphs** (c) only DAGs (d) only trees

6. Dijkstra requires:

   (a) non-negative weights (b) **non-negative weights** (c) negative weights (d) no weights

7. Bellman–Ford can detect:

   (a) any cycle (b) **negative-weight cycles** (c) self-loops (d) bipartite structure

8. Kruskal's MST algorithm uses:

   (a) BFS (b) DFS (c) **Union-Find** (d) hash table

9. Prim's MST algorithm uses:

   (a) **a priority queue** (b) Union-Find (c) a stack (d) hash table

10. The cut property says:

    (a) every edge is in some MST (b) **the minimum edge crossing any cut is in some MST**
    (c) MST weight is unique (d) MST has exactly V edges

11. Dijkstra's complexity with binary heap is:

    (a) O(V) (b) O(E) (c) **O((V+E) log V)** (d) O(V²)

12. The number of edges in a spanning tree of V vertices is:

    (a) 0 (b) **V - 1** (c) V (d) V + 1

### B. Fill in the blank

1. A graph with n vertices has at most **\_\_\_\_\_** edges if undirected and unweighted.

2. Adjacency matrix memory is **O(\_\_\_\_\_**).

3. Adjacency list memory is **O(\_\_\_\_\_**).

4. BFS visits vertices in **\_\_\_\_\_** order from the start.

5. DFS visits vertices in **\_\_\_\_\_** order.

6. **Dijkstra** requires **\_\_\_\_\_** edge weights.

7. **Bellman–Ford** runs in **O(\_\_\_\_\_**).

8. Kruskal's algorithm sorts edges by **\_\_\_\_\_**.

9. The **cut property** guarantees the minimum edge across a cut is in **\_\_\_\_\_** MST.

10. A spanning tree of V vertices has **\_\_\_\_\_** edges.

### C. Define / explain

1. **Define** a graph. Distinguish directed vs. undirected, weighted vs. unweighted, sparse vs. dense.

2. **Compare** adjacency matrix and adjacency list on: memory, edge lookup, neighbor iteration. Give a scenario where each is preferred.

3. **Describe** BFS. What property makes it find shortest paths in unweighted graphs?

4. **Describe** DFS. What three problem classes does it naturally solve (cycle, topo-sort, components)?

5. **State** Dijkstra's algorithm. Why does it require non-negative weights?

6. **Compare** Dijkstra and Bellman–Ford. Give a scenario where only Bellman–Ford gives correct answers.

7. **State** the cut property. Use it to justify why Kruskal's algorithm produces a minimum spanning tree.

8. **Compare** Kruskal and Prim. When is each preferred?

9. **Trace** BFS on the graph in Example 7.1 from vertex 3. What is the visit order? The distances?

10. **Trace** Dijkstra on the graph in Example 7.3 from vertex 2. What are the shortest distances?

---

## Answer key

??? success "A. Multiple choice"
    1. **(b)** number of edges touching it.
    2. **(b)** O(n + m).
    3. **(b)** queue.
    4. **(a)** stack (or recursion).
    5. **(b)** unweighted graphs.
    6. **(a) / (b)** non-negative weights (both answers equivalent; (b) is the same as (a)).
    7. **(b)** negative-weight cycles.
    8. **(c)** Union-Find.
    9. **(a)** priority queue.
    10. **(b)** minimum edge across cut is in some MST.
    11. **(c)** O((V + E) log V).
    12. **(b)** V - 1.

??? success "B. Fill in the blank"
    1. $\frac{n(n-1)}{2}$ (undirected, no self-loops) or $n(n-1)$ (directed)
    2. $n^2$
    3. $n + m$
    4. level / breadth-first
    5. depth-first
    6. non-negative
    7. $V \cdot E$
    8. weight
    9. some
    10. $V - 1$

??? success "C. Define / explain — model answers"
    1. *Graph:* $G = (V, E)$ — a set of vertices and a set of edges (pairs of vertices). *Directed vs. undirected:* directed edges have orientation $(u, v) \ne (v, u)$; undirected edges are symmetric $\{u, v\} = \{v, u\}$. *Weighted vs. unweighted:* weighted edges carry a numeric cost; unweighted edges all cost 1. *Sparse vs. dense:* sparse has $E \approx O(V)$; dense has $E \approx O(V^2)$. Most real graphs are sparse.
    2. *Matrix vs. list:* (a) **memory** — matrix O(n²), list O(n + m). (b) **edge lookup** — matrix O(1), list O(degree). (c) **neighbor iteration** — matrix O(n) (must scan row), list O(degree). *Preferred:* matrix for dense graphs or frequent edge lookups; list for sparse graphs (the common case) or when iterating neighbors frequently.
    3. *BFS:* use a FIFO queue, mark start visited, enqueue. Dequeue one vertex, visit its unvisited neighbors (mark visited, enqueue). *Why shortest paths in unweighted graphs:* BFS processes vertices in non-decreasing distance. When it first reaches vertex `v` from `u`, `dist[v] = dist[u] + 1`. If a shorter path existed, that path's intermediate vertex would have been reached earlier — contradicting FIFO order. So the first assignment is optimal.
    4. *DFS:* visit vertex, then recursively DFS each unvisited neighbor, backtracking when stuck. *Problems:* (i) **cycle detection** — if we visit an already-visited ancestor, cycle exists; (ii) **topological sort** — reverse postorder of DFS on a DAG gives topological order; (iii) **connected components** — a new DFS start reveals a new component.
    5. *Dijkstra:* start with `dist[source] = 0`, all ∞. Use a priority queue. Repeatedly pop the unsettled vertex with the smallest tentative distance; relax its outgoing edges. *Requires non-negative:* the invariant "once popped, distance is final" relies on the fact that extending a path via a non-negative edge cannot make it shorter. With negative edges, a path through a settled vertex could later be improved — invalidating the invariant.
    6. *Dijkstra vs. Bellman–Ford:* Dijkstra is **faster** O((V+E) log V but **requires non-negative weights**. Bellman–Ford is **slower** O(VE) but **handles negative edges** and detects negative cycles. *Example where Dijkstra fails:* `s→u (1), s→v (4), u→v (-3)`. Dijkstra settles `v` at 4 (via direct edge), missing the shorter path `s→u→v (1-3=-2)`. Bellman–Ford correctly gives `dist[v] = -2`.
    7. *Cut property:* for any partition `(S, V-S)` of vertices, the minimum-weight edge crossing the cut belongs to at least one MST. *Kruskal justification:* at each step, we add the minimum-weight edge that doesn't create a cycle. Consider the moment the algorithm adds edge `e = (u, v)`. Define `S` = the connected component containing `u` in the current MST (before adding `e`). `e` is the minimum edge crossing `(S, V-S)` in the remaining edges (because Kruskal processes in sorted order). By the cut property, `e` is in some MST of the graph. By induction over Kruskal's steps, the final tree is an MST.
    8. *Kruskal vs. Prim:* Kruskal — sort edges, then add if not creating cycle, using Union-Find. O(E log E). Best for **sparse** graphs (few edges). *Prim* — grow tree from a vertex using a priority queue. O((V+E) log V). Best for **dense** graphs (array-based Prim O(V²) beats Kruskal's O(E log E) = O(V² log V)). *Also:* Kruskal gives an **edge-list** MST — good when edges are known upfront. Prim grows from a **vertex** — good when the graph is explored dynamically.
    9. *BFS from 3:* neighbors of 3 = {1, 5}. Visit 3, then 1 and 5. From 1: {0, 3, 4}, so 0 and 4. From 5: {3, 4} — 4 already visited. From 0: {1, 2}, so 2. From 4: {1, 2, 5}, 2 already visited. *Visit order:* 3, 1, 5, 0, 4, 2. *Distances:* `dist[3]=0, dist[1]=1, dist[5]=1, dist[0]=2, dist[4]=2, dist[2]=3`.
    10. *Dijkstra from 2:* `dist = [∞, ∞, 0, ∞, ∞]`. Pop 2 (dist 0): relax 2→1 (3), 2→3 (5), 2→4 (6). PQ: (1, 3), (3, 5), (4, 6). Pop 1 (3): relax 1→3 (1+3=4 < 5, update to 4). PQ: (3, 4), (4, 6). Pop 3 (4): relax 3→4 (2+4=6, no change). PQ: (4, 6). Pop 4 (6): no outgoing edges. *Final:* `dist = [∞, 3, 0, 4, 6]`. Node 0 is unreachable from 2 (∞). ✓

---

## Further reading

- **CLRS**, Chapters 22 (BFS/DFS), 23 (MST), 24 (Shortest paths).
- **Weiss**, Chapter 9 (Graph algorithms).
- **Lafore**, Chapter 14 (Weighted graphs).
- **Sedgewick & Wayne**, Chapters 4.1–4.4 (Undirected, directed, MST, shortest paths).
- **Skiena**, *The Algorithm Design Manual*, Chapters 5–6 — practical guide with examples.
- **VisuAlgo** — [visualgo.net](https://visualgo.net/en/dfsbfs) — animated BFS, DFS, Dijkstra, MST.
- **cp-algorithms.com** — [cp-algorithms.com/graph/breadth-first-search.html](https://cp-algorithms.com/graph/breadth-first-search.html) — dense reference for competitive programming.
- **"Dijkstra's algorithm"** — [en.wikipedia.org/wiki/Dijkstra%27s_algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm) — the definitive history and pseudocode.

---

## Suggested tutorial problems

1. **Adjacency matrix vs. list.** Build both for the graph in Example 7.1. Compare memory (bytes) for `n = 6, m = 7`.

2. **BFS trace.** Run BFS on the graph of Example 7.1 from vertex 4. Record visit order and distances.

3. **DFS trace.** Run DFS on the same graph from vertex 2 (recursive, neighbors sorted ascending). Record visit order and DFS tree edges.

4. **Connected components.** Given a graph with two disjoint components, modify BFS to count the number of connected components in O(V + E).

5. **Bipartite check.** Modify BFS or DFS to check whether a graph is bipartite. What coloring invariant must hold?

6. **Cycle detection (undirected).** Modify DFS to detect a cycle in an undirected graph. What's the key observation about "back edges"?

7. **Cycle detection (directed).** Modify DFS to detect a cycle in a directed graph. Why does the undirected approach fail? (Hint: 3-color DFS.)

8. **Topological sort.** Given a DAG, implement topological sort using (a) DFS postorder reversal; (b) Kahn's algorithm (BFS on in-degrees). Verify both give the same order (up to ties).

9. **Dijkstra trace.** Run Dijkstra on the graph of Example 7.3 from vertex 2. Record all iterations.

10. **Dijkstra fails with negative edges.** Construct a 3-vertex graph where Dijkstra gives the wrong answer but Bellman–Ford gives the right one.

11. **Bellman–Ford trace.** Run Bellman–Ford on the graph of Worked example 7.4 with a 4th vertex and edge `2→3 (-1)`. Detect the resulting negative cycle.

12. **Kruskal trace.** Run Kruskal on the graph of Example 7.5. Show the union-find state after each edge.

13. **Prim trace.** Run Prim on the same graph from vertex 3. Compare the final MST total weight to Kruskal's (must be the same).

14. **MST uniqueness.** When is the MST of a graph unique? Give a graph where two different MSTs exist and identify both.

15. **Road network.** Design a graph representation for a road network (cities = vertices, roads = weighted edges). Which representation fits? Which algorithm finds the shortest route? Which finds the cheapest road-construction plan?

16. **Friend network.** A social network has 100 million users. You want to find the shortest path from Alice to Bob. Estimate the memory for adjacency matrix vs. adjacency list. Which representation fits on a modern server?