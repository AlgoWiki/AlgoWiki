---
categories: Graph algorithms
---

**Breadth-first search** (BFS) explores a graph level by level outward from a
source vertex, using a queue to visit all vertices at distance $d$ before any at
distance $d+1$. On an unweighted graph it therefore finds the shortest path (in
number of edges) from the source to every reachable vertex, in $O(V + E)$ time.
It is the natural companion to [depth-first search](Depth-first search tree),
which instead explores as deep as possible before backtracking.

## Description

BFS keeps a FIFO queue of "discovered but not yet processed" vertices and an array
recording, for each vertex, its distance from the source (which doubles as a
*visited* marker). Starting with only the source in the queue at distance $0$, it
repeatedly removes a vertex $u$ and, for each neighbour $v$ not yet discovered,
records `dist[v] = dist[u] + 1` and enqueues $v$.

Because vertices enter the queue in nondecreasing order of distance, each vertex
is discovered exactly once, by the shortest route — so `dist` holds the
fewest-edges distance from the source, and the discovery edges form a **BFS tree**
of shortest paths. Storing the predecessor of each vertex lets you reconstruct an
actual shortest path by walking back from the target.

## Implementation

With the graph as an adjacency list, BFS from source `s` over `n` vertices:

~~~ {.cpp}
vector<int> bfs(int s, const vector<vector<int>>& adj) {
    vector<int> dist(adj.size(), -1);            // -1 = unvisited
    queue<int> q;
    dist[s] = 0; q.push(s);
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int v : adj[u])
            if (dist[v] == -1) {                 // first time we reach v
                dist[v] = dist[u] + 1;
                q.push(v);
            }
    }
    return dist;
}
~~~

To recover paths, keep a `parent` array and set `parent[v] = u` alongside
`dist[v]`; the path to a vertex is then obtained by following `parent` back to the
source and reversing.

## Applications

- **Unweighted shortest paths.** The distances above are exactly the minimum
  number of edges from the source.
- **Connected components.** Running BFS from every not-yet-visited vertex labels
  each [connected component](Connected component); [union-find](Union-find data structure)
  is an alternative when edges arrive incrementally.
- **Bipartiteness / 2-colouring.** Colour each vertex by the parity of its BFS
  distance; an edge joining two equal colours proves the graph is not
  [bipartite](Bipartite graph).
- **Multi-source BFS.** Seeding the queue with several sources at distance $0$
  computes, for every vertex, the distance to the *nearest* source in one pass —
  handy for "spread from all of these cells at once" grid problems.
- **Implicit graphs.** BFS needs no explicit adjacency list: the neighbours of a
  state (a grid cell, a board configuration, …) can be generated on the fly.

## Variants

### 0-1 BFS
When every edge weight is $0$ or $1$, shortest paths can still be found in
$O(V + E)$ — faster than a general-purpose shortest-path algorithm — by replacing
the queue with a double-ended queue: relax each edge, pushing the endpoint to the
**front** of the deque for a weight-$0$ edge and the **back** for a weight-$1$
edge. This keeps the deque sorted by distance just as an ordinary BFS queue is.

~~~ {.cpp}
vector<long long> zeroOneBfs(int s, const vector<vector<pair<int,int>>>& adj) {
    vector<long long> dist(adj.size(), LLONG_MAX);   // adj: (neighbour, weight ∈ {0,1})
    deque<int> dq;
    dist[s] = 0; dq.push_front(s);
    while (!dq.empty()) {
        int u = dq.front(); dq.pop_front();
        for (auto [v, w] : adj[u])
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                if (w == 0) dq.push_front(v);
                else        dq.push_back(v);
            }
    }
    return dist;
}
~~~

## Problems
- [Message Route](https://cses.fi/problemset/task/1667)
- [Monsters](https://cses.fi/problemset/task/1194)
- [Erdős Numbers](https://open.kattis.com/problems/erdosnumbers)
- [Erratic Ants](https://open.kattis.com/problems/erraticants)
- [Dominoes 2](https://open.kattis.com/problems/dominoes2)
- [Beehives](https://open.kattis.com/problems/beehives2)

<details>
<summary>Solution sketch — Message Route</summary>

A plain unweighted shortest path with reconstruction: BFS from vertex $1$, keeping
a `parent` array. If the target is unreachable (`dist == -1`) print "IMPOSSIBLE";
otherwise the distance gives the number of vertices on the route, and following
`parent` back from the target (then reversing) gives the route itself, in
$O(V + E)$.

</details>

<details>
<summary>Solution sketch — Monsters</summary>

First run a **multi-source BFS** from all monsters at once to get, for every cell,
the time a monster first reaches it. Then BFS the player from the start, stepping
into a cell only if the player arrives strictly before any monster (player
distance < monster distance), reconstructing the escape path with a `parent` grid.
Both passes are $O(nm)$ on the grid.

</details>

## See also
- [Depth-first search tree]() — the other fundamental traversal
- [Bipartite graph]() — testable with a BFS 2-colouring
- [Union-find data structure]() — components under incremental edges

## External links
- [0-1 BFS [Tutorial]](http://codeforces.com/blog/entry/22276)
- [Breadth-first search (cp-algorithms)](https://cp-algorithms.com/graph/breadth-first-search.html)
- [0-1 BFS (cp-algorithms)](https://cp-algorithms.com/graph/01_bfs.html)
