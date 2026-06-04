---
categories: Data structures
---

The **union-find** data structure — also called a **disjoint-set union** (DSU) —
maintains a collection of disjoint sets under two operations: *find*, which
returns a canonical representative of the set containing a given element, and
*union*, which merges the two sets containing two given elements. With the right
optimizations both operations run in almost-constant amortized time
$O(\alpha(n))$, where $\alpha$ is the inverse Ackermann function (at most $4$ for
any input that fits in the universe). It is the standard tool for tracking the
[connected components](Connected component) of a graph as edges are added, and is
the engine behind [Kruskal's algorithm](Minimum spanning tree) for minimum
spanning trees.

## Description

Each set is stored as a rooted tree, and every element points to a *parent*; the
root of a tree (an element that is its own parent) is the representative of its
set. Two elements are in the same set exactly when they have the same root.

- **find($x$)** follows parent pointers from $x$ up to the root.
- **union($a$, $b$)** finds the roots of $a$ and $b$ and, if they differ, makes
  one root a child of the other — merging the two trees into one.

On their own these trees can degenerate into long chains, making `find` cost
$O(n)$. Two classic optimizations keep them flat:

- **Union by size (or rank)**: always attach the smaller tree under the root of
  the larger one. This alone bounds the height by $O(\log n)$.
- **Path compression**: while doing a `find`, repoint the nodes along the way
  directly at the root, so later queries are faster.

Used together, the amortized cost per operation drops to $O(\alpha(n))$ —
effectively constant. It is cheap to track a little extra information on each
root, such as the size of its set or the total number of disjoint sets, updating
it only when two sets are merged.

## Implementation

The following keeps a `parent` and a `size` array, uses union by size and path
halving (a one-pass form of path compression), and maintains a live count of
components.

~~~ {.cpp}
struct UnionFind {
    vector<int> parent, sz;
    int components;

    UnionFind(int n) : parent(n), sz(n, 1), components(n) {
        iota(parent.begin(), parent.end(), 0);   // every element its own root
    }

    int find(int x) {                            // path halving
        while (parent[x] != x)
            x = parent[x] = parent[parent[x]];
        return x;
    }

    bool unite(int a, int b) {                   // union by size
        a = find(a); b = find(b);
        if (a == b) return false;                // already together
        if (sz[a] < sz[b]) swap(a, b);
        parent[b] = a;
        sz[a] += sz[b];
        components--;
        return true;
    }

    bool same(int a, int b) { return find(a) == find(b); }
    int  size(int x)        { return sz[find(x)]; }
};
~~~

`unite` returns `false` when the two elements were already in the same set, which
is exactly the information several algorithms need (for example, detecting that
an edge closes a cycle).

## Applications

- **Connected components under edge insertions.** Process edges one by one,
  calling `unite` on their endpoints; `components` then tracks the number of
  connected components, and `size` the size of each.
- **[Kruskal's algorithm](Minimum spanning tree).** Sort the edges by weight and
  add each edge whose endpoints are in different sets (`unite` returns `true`),
  skipping any edge that would create a cycle.
- **Cycle detection in an undirected graph.** An edge whose endpoints already
  share a root closes a cycle.
- **Maintaining equivalence classes.** Any "$a$ is equivalent to $b$" constraints
  can be merged and queried; the root acts as a [class representative](Class representative).
- **Offline connectivity queries.** Combined with sorting the queries, or with
  [parallel binary search](Parallel binary search), union-find answers many
  connectivity questions in one pass.

## Problems
- [Road Construction](https://cses.fi/problemset/task/1676)
- [Disastrous Downfall](https://open.kattis.com/problems/downfall)
- [Šuma](https://open.kattis.com/problems/suma)
- [Almost Union-Find](https://open.kattis.com/problems/almostunionfind)
- [Ladice](https://open.kattis.com/problems/ladice)

<details>
<summary>Solution sketch — Road Construction</summary>

Start with $n$ isolated nodes: $n$ components, each of size $1$. For each new
road, `unite` its endpoints; on a successful union decrement the component count
and set the new component's size to the sum of the two old sizes. Keep a running
maximum of all set sizes (it only ever changes to the size of a freshly merged
set). Print the component count and the current maximum size after each road, in
$O(m\,\alpha(n))$ total.

</details>

<details>
<summary>Solution sketch — Almost Union-Find</summary>

This problem also asks to *move* a single element to another set, which plain DSU
can't do (you cannot pull one node out of a tree). Trick: give each label an
indirection `where[label]` pointing at an internal node id, and store set `size`
and `sum` on each root. A *move* allocates a fresh internal node for the label
and repoints `where[label]` at it, leaving the old node behind in its set; union
and the size/sum bookkeeping are otherwise standard.

</details>

## Variants

### Weighted / parity union-find

By storing, for each node, its *relation* to its parent (rather than just the
parent), union-find can track relationships between elements, not merely whether
they are in the same set. The most common case is a **parity** (mod 2) relation,
used to test [bipartiteness](Bipartite graph): we want to record constraints of
the form "$a$ and $b$ are on opposite sides" and detect a contradiction (an odd
cycle).

~~~ {.cpp}
struct ParityDSU {
    vector<int> parent, rel;          // rel[x] = parity of x relative to parent[x]

    ParityDSU(int n) : parent(n), rel(n, 0) {
        iota(parent.begin(), parent.end(), 0);
    }

    pair<int,int> find(int x) {       // returns {root, parity of x to root}
        if (parent[x] == x) return {x, 0};
        auto [root, p] = find(parent[x]);
        parent[x] = root;
        rel[x] ^= p;
        return {root, rel[x]};
    }

    // enforce parity(a) xor parity(b) == d; returns false on contradiction
    bool unite(int a, int b, int d) {
        auto [ra, pa] = find(a);
        auto [rb, pb] = find(b);
        if (ra == rb) return (pa ^ pb) == d;
        parent[rb] = ra;
        rel[rb] = pa ^ pb ^ d;
        return true;
    }
};
~~~

The same idea generalizes from parity to relations modulo any $k$ (storing an
integer offset instead of a bit), which is what the problem below needs.

#### Problems
- [Food Chain](http://poj.org/problem?id=1182)

## Edge deletion instead of insertion
The union-find data structure can sometimes be used to support edge deletion instead of edge insertion. The idea is simply to simulate the sequence of operations in reverse, and then deletions become insertions. This only works when the sequence of operations is known beforehand.

To support both insertions and deletions, see [Dynamic connectivity]().

### Problems
- [Islands](https://archive.algo.is/icpc/cerc/2009/island.pdf)
- [Escape from Enemy Territory](https://open.kattis.com/problems/enemyterritory)
- [Artwork](https://open.kattis.com/problems/artwork) [^1]

## Undo operation
The union-find data structure can be extended to support an undo operation. By repeatedly executing the operation it is possible to revert to an earlier state of the data structure, and then continue from there as if nothing had happened later. This is implemented by keeping a stack of all modifications to the underlying array. For example, if index $i$ in the underlying array currently has value $v$, and is then changed to $v'$, then the tuple $(i,v)$ is pushed to the stack (i.e. the index and the ''old'' value). To perform an undo operation, $(i,v)$ is popped from the stack, and the value at index $i$ is set to $v$, the old value.

Note that undo is incompatible with path compression, since a single `find` may
rewrite arbitrarily many pointers. The undoable ("rollback") version therefore
uses **union by size/rank only**, giving $O(\log n)$ per operation. This rollback
DSU is the workhorse behind offline [dynamic connectivity]() and is frequently
paired with [parallel binary search]().

### Problems
- [Chef and Graph Queries](https://www.codechef.com/MARCH14/problems/GERALD07)
- [Disconnected Graph](http://codeforces.com/gym/100551/problem/E)

## See also
- [Minimum spanning tree]() — Kruskal's algorithm is built on union-find
- [Dynamic connectivity]() — supports both edge insertions and deletions
- [Parallel binary search]() — commonly paired with the rollback DSU
- [Class representative]() — the root serves as a set's representative

## External links
- [Disjoint Set Union (cp-algorithms)](https://cp-algorithms.com/data_structures/disjoint_set_union.html)

[^1]: <https://archive.algo.is/icpc/nwerc/ncpc/2016/ncpc2016slides.pdf>
