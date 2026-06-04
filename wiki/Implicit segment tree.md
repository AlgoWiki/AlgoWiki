---
categories: Data structures
---

An **implicit segment tree** — also called a **dynamic** or **sparse** segment
tree — is a [segment tree](Segment tree) built over a huge index range in which
nodes are created only when they are first touched. Instead of allocating $O(C)$
nodes for a coordinate range of size $C$, it allocates $O(q \log C)$ nodes for $q$
updates, which makes it practical to index directly into ranges as large as
$10^9$ or even $10^{18}$ without [coordinate compression](Coordinate compression).

## Description

The structure is an ordinary [segment tree](Segment tree) over $[0, C)$, with one
change: a node's children are not allocated up front. When an update descends into
a child that does not exist yet, that child is created on the spot; subtrees that
are never visited are never built. A query treats a missing child as empty and
returns the identity element for it.

Because each update follows a single root-to-leaf path, it creates at most
$O(\log C)$ new nodes, so after $q$ updates the tree holds $O(q \log C)$ nodes and
every operation still runs in $O(\log C)$ time. This is exactly the trade-off that
makes the structure useful: memory scales with the number of operations, not with
the size of the coordinate space.

This is closely related to the [persistent segment tree](Persistent segment tree),
which also allocates nodes lazily — there, a fresh path of nodes is created per
*version*, whereas here a node is created once, the first time it is needed.

## Implementation

The cleanest way to avoid pointer bookkeeping is a node pool (a `vector`) where
each node stores the indices of its children, with `-1` meaning "not created".
The version below supports point add and range sum over $[0, N)$.

~~~ {.cpp}
struct DynamicSeg {                      // point add, range sum over [0, N)
    struct Node { long long sum = 0; int lc = -1, rc = -1; };
    vector<Node> t;
    long long N;

    DynamicSeg(long long N) : N(N) { t.push_back(Node()); }   // node 0 = root
    int newNode() { t.push_back(Node()); return (int)t.size() - 1; }

    void update(int node, long long lo, long long hi, long long i, long long v) {
        if (lo == hi) { t[node].sum += v; return; }
        long long mid = lo + (hi - lo) / 2;
        if (i <= mid) {
            if (t[node].lc < 0) { int c = newNode(); t[node].lc = c; }
            update(t[node].lc, lo, mid, i, v);
        } else {
            if (t[node].rc < 0) { int c = newNode(); t[node].rc = c; }
            update(t[node].rc, mid + 1, hi, i, v);
        }
        long long s = 0;
        if (t[node].lc >= 0) s += t[t[node].lc].sum;
        if (t[node].rc >= 0) s += t[t[node].rc].sum;
        t[node].sum = s;
    }

    long long query(int node, long long lo, long long hi, long long l, long long r) {
        if (node < 0 || r < lo || hi < l) return 0;       // missing/disjoint: identity
        if (l <= lo && hi <= r) return t[node].sum;
        long long mid = lo + (hi - lo) / 2;
        return query(t[node].lc, lo, mid, l, r)
             + query(t[node].rc, mid + 1, hi, l, r);
    }

    void update(long long i, long long v) { update(0, 0, N - 1, i, v); }
    long long query(long long l, long long r) { return query(0, 0, N - 1, l, r); }
};
~~~

Note that children are stored as integer indices, not pointers, so growing the
`vector` (which may reallocate) never invalidates anything. Lazy propagation can
be added just as in an ordinary segment tree, allocating children before pushing a
tag down — this gives, for example, range-assign + range-sum over a range of size
$10^9$.

## Implicit vs. coordinate compression

When every coordinate is known in advance, [coordinate
compression](Coordinate compression) followed by a plain [segment
tree](Segment tree) is usually preferable: it has a smaller constant factor and
lower memory. An implicit segment tree wins when the coordinates are *not* known
ahead of time — most importantly for **online** queries, where compression is
impossible — or when it simply keeps the code shorter.

## Segment tree merging

Implicit segment trees can be **merged**: given the roots of two dynamic segment
trees over the same coordinate range, a recursive merge that reuses nodes runs in
time proportional to the number of nodes that overlap. Summed over a whole
process (for instance, merging children's trees up a rooted tree), the total cost
is $O(n \log C)$. This "segment tree merging" is a standard way to aggregate
information from subtrees.

### Problems
- [Lomsat gelral](https://codeforces.com/problemset/problem/600/E)
- [Dominant Indices](https://codeforces.com/problemset/problem/1009/F)

## Problems
- [Physical Education Lessons](https://codeforces.com/problemset/problem/915/E)

<details>
<summary>Solution sketch — Physical Education Lessons</summary>

Days are numbered up to $n \le 10^9$, and each query assigns a whole range of days
to "working" or "non-working" and then asks for the current number of working
days. Build an implicit segment tree over $[1, n]$ with lazy range-assign and a
count of working days; only the $O(q \log n)$ nodes actually touched are ever
allocated, so the $10^9$ range is no obstacle. Each query is $O(\log n)$.

</details>

## See also
- [Segment tree]() — the underlying structure
- [Persistent segment tree]() — also allocates nodes lazily, one path per version
- [Coordinate compression]() — the usual offline alternative

## External links
- [Dynamic / implicit segment trees (cp-algorithms)](https://cp-algorithms.com/data_structures/segment_tree.html#dynamic-segment-tree)
