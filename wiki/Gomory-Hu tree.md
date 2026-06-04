---
categories: Graph algorithms, Graph theory, Combinatorial optimization
---

A **Gomory–Hu tree** (or *cut tree*) compactly represents the minimum cut between
*every* pair of vertices in an undirected graph with nonnegative edge capacities.
It is a weighted tree on the same vertex set with the property that, for any two
vertices $s$ and $t$, the [minimum $s$–$t$ cut](Minimum cut) in the original graph
equals the smallest-weight edge on the unique tree path between $s$ and $t$ — and
removing that edge splits the tree into exactly the two sides of a minimum cut.
Although there are $\binom{n}{2}$ pairs, only $n - 1$ distinct cut values can
occur, and the whole tree can be built with just $n - 1$
[maximum-flow](Maximum flow) computations.

## Properties

Let $T$ be a Gomory–Hu tree of $G$, with each edge labelled by a capacity. For
vertices $s$ and $t$, let $P(s,t)$ be the path between them in $T$. Then:

- The minimum $s$–$t$ cut value in $G$ is $\min_{e \in P(s,t)} w(e)$, the lightest
  edge on the path.
- Deleting that lightest edge partitions $T$ — and hence the vertices — into the
  two sides of an actual minimum $s$–$t$ cut of $G$.

So a single tree of $n - 1$ edges answers every pairwise minimum-cut query by a
path-minimum lookup. (The construction relies on the symmetry of undirected cuts;
it does not extend to directed graphs.)

## Construction (Gusfield's algorithm)

The original Gomory–Hu construction repeatedly contracts the graph. **Gusfield's
algorithm** is much simpler to implement and just as efficient: it performs
$n - 1$ maximum-flow computations on the *original* graph, never contracting.

It grows the tree as a parent array, initially making vertex $0$ the parent of
everyone. For each other vertex $s$, it computes a minimum cut between $s$ and its
current parent $t$, attaches $s$ to $t$ with that cut value, and then reroutes any
vertex that lies on $s$'s side of the cut and currently points at $t$ to point at
$s$ instead. A final rotation keeps the tree consistent when $t$'s own parent ends
up on $s$'s side.

It needs any [maximum-flow](Maximum flow) routine that can also report which
vertices are on the source side of the minimum cut (the set reachable from the
source in the residual graph). For an undirected edge of capacity $c$, both
residual directions start at $c$:

~~~ {.cpp}
struct Dinic {
    struct E { int to, rev; long long cap; };
    int n; vector<vector<E>> g; vector<int> level, it;
    Dinic(int n) : n(n), g(n), level(n), it(n) {}
    void addUndirected(int a, int b, long long c) {
        g[a].push_back({b, (int)g[b].size(), c});
        g[b].push_back({a, (int)g[a].size() - 1, c});      // reverse cap = c
    }
    bool bfs(int s, int t) {
        fill(level.begin(), level.end(), -1);
        queue<int> q; level[s] = 0; q.push(s);
        while (!q.empty()) { int u = q.front(); q.pop();
            for (auto& e : g[u]) if (e.cap > 0 && level[e.to] < 0) { level[e.to] = level[u]+1; q.push(e.to); } }
        return level[t] >= 0;
    }
    long long dfs(int u, int t, long long f) {
        if (u == t) return f;
        for (int& i = it[u]; i < (int)g[u].size(); i++) { E& e = g[u][i];
            if (e.cap > 0 && level[e.to] == level[u]+1) {
                long long d = dfs(e.to, t, min(f, e.cap));
                if (d > 0) { e.cap -= d; g[e.to][e.rev].cap += d; return d; } } }
        return 0;
    }
    long long maxflow(int s, int t) {
        long long fl = 0;
        while (bfs(s, t)) { fill(it.begin(), it.end(), 0);
            long long f; while ((f = dfs(s, t, LLONG_MAX)) > 0) fl += f; }
        return fl;
    }
    vector<char> sourceSide(int s) {                       // residual-reachable = source side
        vector<char> vis(n, 0); queue<int> q; vis[s] = 1; q.push(s);
        while (!q.empty()) { int u = q.front(); q.pop();
            for (auto& e : g[u]) if (e.cap > 0 && !vis[e.to]) { vis[e.to] = 1; q.push(e.to); } }
        return vis;
    }
};

struct Edge { int u, v; long long c; };

// Returns parent[] and weight[] of the Gomory-Hu tree (rooted at 0):
// the tree edges are (i, parent[i]) with capacity weight[i], for i = 1..n-1.
pair<vector<int>, vector<long long>> gomoryHu(int n, const vector<Edge>& edges) {
    vector<int> par(n, 0);
    vector<long long> w(n, 0);
    for (int s = 1; s < n; s++) {
        int t = par[s];
        Dinic d(n);
        for (auto& e : edges) d.addUndirected(e.u, e.v, e.c);
        long long f = d.maxflow(s, t);
        w[s] = f;
        auto X = d.sourceSide(s);                          // s's side of the min cut
        for (int i = 0; i < n; i++)
            if (i != s && X[i] && par[i] == t) par[i] = s;
        if (X[par[t]]) {                                   // keep the tree consistent
            par[s] = par[t]; par[t] = s;
            w[s] = w[t];     w[t] = f;
        }
    }
    return {par, w};
}
~~~

Each iteration runs one maximum flow, so the total cost is $n - 1$ max-flow
computations.

## Querying

To answer a minimum-cut query for a pair $(s, t)$, build the tree's adjacency from
the `parent`/`weight` arrays and take the minimum edge weight on the $s$–$t$ path
(any traversal works; for many queries, root the tree and use [binary
jumping](Binary jumping) to get the path minimum in $O(\log n)$). The two
components obtained by deleting that lightest edge are the two sides of a
corresponding minimum cut.

## Applications

- **All-pairs minimum cut.** The headline use: every pairwise minimum cut, stored
  in $O(n)$ space and answered by a path-minimum.
- **Global minimum cut.** The lightest edge of the Gomory–Hu tree is a global
  [minimum cut](Minimum cut) of the graph.
- **Problems needing many cut values.** Grouping or ordering vertices to optimize
  over pairwise cuts becomes a problem on a tree, which is usually far easier than
  on the original graph.

The Gomory–Hu tree is, in a sense, the cut analogue of a [minimum spanning
tree](Minimum spanning tree): it summarizes a quadratic amount of connectivity
information in a single spanning tree.

## Problems
- [Pumping Stations](https://codeforces.com/problemset/problem/343/E)

<details>
<summary>Solution sketch — Pumping Stations</summary>

Build the Gomory–Hu tree of the network. The quantity to maximize decomposes
nicely on the tree: process it like a tree DP / recursive ordering where, at each
step, the lightest tree edge determines a split, and the order within each side is
solved recursively. Reducing the all-pairs-cut structure to the $n-1$ tree edges
is what makes the optimization tractable.

</details>

## See also
- [Maximum flow]() — the subroutine; each tree edge is one max-flow computation
- [Minimum cut]() — what the tree's edges encode
- [Minimum spanning tree]() — the structural analogue for connectivity

## External links
- [Gomory–Hu tree (Wikipedia)](https://en.wikipedia.org/wiki/Gomory%E2%80%93Hu_tree)
