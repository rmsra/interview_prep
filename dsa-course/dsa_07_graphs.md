# Graphs

**Courses:** **DSA** · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**DSA modules:** [Index](dsa_00_README_start_here.md) · [C++ Setup](dsa_01_cpp_setup_and_best_practices.md) · [Arrays/Hashing/2Ptr/Stack](dsa_02_arrays_hashing_two_pointers_stack.md) · [Binary Search/Linked List](dsa_03_binary_search_and_linked_list.md) · [Recursion/Sorting](dsa_04_recursion_and_sorting.md) · [Trees/Tries/Union-Find](dsa_05_trees_tries_union_find.md) · [Heap/Backtracking](dsa_06_heap_priority_queue_backtracking.md) · **Graphs** · [DP](dsa_08_dynamic_programming.md) · [Bit Manipulation](dsa_09_bit_manipulation.md) · [Cisco Problem Bank](dsa_10_cisco_problem_bank.md) · [Intervals](dsa_11_intervals.md)

---

## Intro to Graphs

- Represented as: **adjacency matrix** (grid problems are secretly graphs — each cell is a node), **adjacency list** (explicit node/edge lists — most common), or **edge list** (used by Kruskal's).
- Directed vs. undirected, weighted vs. unweighted — always confirm which before coding; it changes the traversal and which algorithm applies.

## Adjacency List

```cpp
std::vector<std::vector<int>> adj(n);   // unweighted
adj[u].push_back(v);
adj[v].push_back(u);                     // omit for directed graphs

std::vector<std::vector<std::pair<int,int>>> wadj(n);  // weighted: {neighbor, weight}
wadj[u].push_back({v, w});
```
```python
from collections import defaultdict
adj = defaultdict(list)
adj[u].append(v)
adj[v].append(u)   # omit for directed

wadj = defaultdict(list)   # weighted
wadj[u].append((v, w))
```
- Space: O(V + E) — the default choice unless the graph is dense (E close to V²), where a matrix can be simpler to index.

## Matrix DFS

**Trigger:** grid problems — "number of islands," "flood fill," "max area of island."

```cpp
void dfs(std::vector<std::vector<int>>& grid, int r, int c, std::vector<std::vector<bool>>& visited) {
    int R = grid.size(), C = grid[0].size();
    if (r < 0 || r >= R || c < 0 || c >= C || visited[r][c] || grid[r][c] == 0) return;
    visited[r][c] = true;
    dfs(grid, r+1, c, visited);
    dfs(grid, r-1, c, visited);
    dfs(grid, r, c+1, visited);
    dfs(grid, r, c-1, visited);
}
```
- Same shape as backtracking's tree maze, but here you typically **don't** un-mark `visited` — you're counting/marking connected components, not searching for a single path, so nodes should stay visited once processed.

## Matrix BFS

**Trigger:** shortest path in an unweighted grid ("shortest path from start to end," "rotting oranges" — multi-source BFS).

```cpp
std::queue<std::pair<int,int>> q;
std::vector<std::vector<bool>> visited(R, std::vector<bool>(C, false));
q.push({sr, sc});
visited[sr][sc] = true;
int steps = 0;
int dr[] = {-1,1,0,0}, dc[] = {0,0,-1,1};
while (!q.empty()) {
    int sz = q.size();
    for (int i = 0; i < sz; i++) {
        auto [r, c] = q.front(); q.pop();
        // process (r, c) — this is `steps` away from source
        for (int d = 0; d < 4; d++) {
            int nr = r + dr[d], nc = c + dc[d];
            if (nr>=0 && nr<R && nc>=0 && nc<C && !visited[nr][nc] && grid[nr][nc] != /*wall*/1) {
                visited[nr][nc] = true;
                q.push({nr, nc});
            }
        }
    }
    steps++;
}
```
- **BFS = shortest path in unweighted graphs.** This is the single most important fact in this file — if a problem asks for shortest/minimum steps and the graph is unweighted, it's BFS, not DFS. DFS finds *a* path, not the shortest one.
- **Gotcha:** mark `visited` at push time, not at pop time — marking at pop time lets the same node get enqueued multiple times before it's processed, wasting time and in cyclic graphs risking infinite growth of the queue.

---

## Weighted Graph Algorithms (Tier 7 — good to have, lower priority given your timeline)

### Dijkstra's Algorithm

**Trigger:** shortest path in a graph with **non-negative** weights.

```cpp
std::vector<int> dijkstra(int n, int src, std::vector<std::vector<std::pair<int,int>>>& adj) {
    std::vector<int> dist(n, INT_MAX);
    dist[src] = 0;
    std::priority_queue<std::pair<int,int>, std::vector<std::pair<int,int>>, std::greater<>> pq; // {dist, node}
    pq.push({0, src});
    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (d > dist[u]) continue;   // stale entry — skip
        for (auto [v, w] : adj[u]) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;
}
```
- Greedy: always expand the closest unvisited node next. O((V + E) log V) with a binary heap.
- **Does not work with negative edge weights** (would need Bellman-Ford instead — not in your list, but worth naming if asked "what if weights can be negative?").

### Prim's Algorithm (Minimum Spanning Tree)

**Trigger:** MST, "connect all nodes with minimum total edge weight," starting from a node.

- Same greedy shape as Dijkstra, but track "minimum edge weight to reach an unvisited node" instead of "minimum distance from source." Grow the tree one cheapest edge at a time.
```cpp
// sketch — structurally near-identical to dijkstra above, swap dist[u]+w for just w
```

### Kruskal's Algorithm (Minimum Spanning Tree)

**Trigger:** MST — the edge-list-first alternative to Prim's.

```cpp
// edges: vector<{weight, u, v}>, sorted ascending by weight
std::sort(edges.begin(), edges.end());
UnionFind uf(n);
int mstWeight = 0;
for (auto& [w, u, v] : edges) {
    if (uf.unite(u, v)) mstWeight += w;   // unite() returns false if already connected (would form a cycle)
}
```
- Sort all edges by weight, then greedily add each edge if it doesn't create a cycle (checked via Union-Find). O(E log E) dominated by the sort.
- **When to pick Prim's vs. Kruskal's:** Prim's is better for dense graphs (adjacency matrix representation); Kruskal's is simpler when you already have an edge list and is the natural pairing with Union-Find, which you already know from the trees file.

### Topological Sort

**Trigger:** DAG (directed acyclic graph), "task scheduling with dependencies," "course prerequisites."

```cpp
// Kahn's algorithm (BFS-based) — also naturally detects cycles
std::vector<int> indegree(n, 0);
for (auto& edges_u : adj) for (int v : edges_u) indegree[v]++;
std::queue<int> q;
for (int i = 0; i < n; i++) if (indegree[i] == 0) q.push(i);
std::vector<int> order;
while (!q.empty()) {
    int u = q.front(); q.pop();
    order.push_back(u);
    for (int v : adj[u]) if (--indegree[v] == 0) q.push(v);
}
bool hasCycle = (int)order.size() != n;  // if not all nodes got processed, there's a cycle
```
- Kahn's algorithm: repeatedly remove nodes with in-degree 0. If you can't process all n nodes, the graph has a cycle — this is the standard way "course schedule / detect cycle in a DAG" problems get solved, and it reuses the BFS pattern you already know.
- Alternative: DFS-based topological sort (post-order, then reverse) — know Kahn's as your default; it's easier to get right live and doubles as cycle detection for free.

**Practice signal for this file:** grid → matrix DFS/BFS. "Shortest path, unweighted" → BFS. "Shortest path, weighted, non-negative" → Dijkstra. "Minimum cost to connect everything" → Prim's/Kruskal's. "Ordering with dependencies" → topological sort.

---

## Cisco Practice Problems

Full methodology in `dsa_10_cisco_problem_bank.md`. 🔥 = asked in the last 3–6 months.

- 🔥🔥🔥 HARD — [Shortest Path in a Grid with Obstacles Elimination](https://leetcode.com/problems/shortest-path-in-a-grid-with-obstacles-elimination) — BFS on a grid, state twist — asked in the last 30 days
- MEDIUM — [Power Grid Maintenance](https://leetcode.com/problems/power-grid-maintenance) — graph + Union-Find
- MEDIUM — [Number of Islands](https://leetcode.com/problems/number-of-islands) — matrix DFS/BFS
- MEDIUM — [Course Schedule](https://leetcode.com/problems/course-schedule) — topological sort
- HARD — [Word Ladder](https://leetcode.com/problems/word-ladder) — BFS
- MEDIUM — [Snakes and Ladders](https://leetcode.com/problems/snakes-and-ladders) — BFS
- MEDIUM — [Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops) — shortest path with a stop limit

---

## Related Modules

- [DSA 05 — Union-Find (used by Kruskal's)](dsa_05_trees_tries_union_find.md)
- [Networking 03 — Routing (OSPF runs Dijkstra's)](../networking-course/networking_03_ip_routing_and_switching.md)
