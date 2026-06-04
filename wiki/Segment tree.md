---
categories: Data structures
---

A **segment tree** is a data structure that stores an array of $n$ elements and
answers *range queries* — such as the sum, minimum, maximum, or gcd over any
contiguous subarray — while also supporting updates, all in $O(\log n)$ time per
operation. It is one of the most versatile structures in competitive programming:
almost any associative aggregate can be maintained, and with [lazy
propagation](#lazy-propagation) it supports range *updates* as well.

## Description

The segment tree is a binary tree built over the index range $[0, n-1]$. The root
covers the whole range; every internal node splits its range in half between its
two children; and the leaves correspond to individual array elements. Each node
stores the aggregate (say, the sum) of its range, computed by *combining* the
aggregates of its two children. Because the range is halved at each level, the
tree has height $O(\log n)$ and uses $O(n)$ nodes.

Two operations follow directly:

- **Point update**: to change one element, update its leaf and recompute the
  aggregate of each ancestor on the path back to the root — $O(\log n)$ nodes.
- **Range query**: a query range $[l, r]$ decomposes into $O(\log n)$ canonical
  nodes whose ranges are fully contained in $[l, r]$; combine their aggregates.
  Recursively: if a node's range is disjoint from $[l, r]$ return the identity
  element, if it is fully inside return the stored aggregate, otherwise recurse
  into both children.

The combining operation only needs to be **associative**, and must have an
identity element to return for empty ranges. Sum (identity $0$), min (identity
$+\infty$), max ($-\infty$), gcd ($0$), and matrix product (the identity matrix)
all qualify, so the same skeleton handles a wide range of problems by swapping out
the combine function.

## Implementation

A common representation stores the tree in an array `t` of size $4n$, where node
`1` is the root and node `k` has children `2k` and `2k+1`. The version below
maintains range sums; change the three combine lines (and the query identity) to
get min, max, gcd, and so on.

~~~ {.cpp}
struct SegTree {
    int n;
    vector<long long> t;                      // 1-indexed; size 4n

    SegTree(const vector<long long>& a) : n(a.size()), t(4 * n) {
        build(1, 0, n - 1, a);
    }

    void build(int node, int lo, int hi, const vector<long long>& a) {
        if (lo == hi) { t[node] = a[lo]; return; }
        int mid = (lo + hi) / 2;
        build(2*node,     lo,      mid, a);
        build(2*node + 1, mid + 1, hi,  a);
        t[node] = t[2*node] + t[2*node + 1];          // combine
    }

    void update(int node, int lo, int hi, int i, long long val) {
        if (lo == hi) { t[node] = val; return; }
        int mid = (lo + hi) / 2;
        if (i <= mid) update(2*node,     lo,      mid, i, val);
        else          update(2*node + 1, mid + 1, hi,  i, val);
        t[node] = t[2*node] + t[2*node + 1];
    }

    long long query(int node, int lo, int hi, int l, int r) {
        if (r < lo || hi < l) return 0;               // disjoint: identity
        if (l <= lo && hi <= r) return t[node];        // fully inside
        int mid = (lo + hi) / 2;
        return query(2*node, lo, mid, l, r)
             + query(2*node + 1, mid + 1, hi, l, r);
    }

    // convenience wrappers over the whole array
    void update(int i, long long val) { update(1, 0, n - 1, i, val); }
    long long query(int l, int r)     { return query(1, 0, n - 1, l, r); }
};
~~~

There is also a well-known *iterative* bottom-up segment tree that is shorter and
faster by a constant factor (see [External links](#external-links)); the
recursive form above is easier to extend, especially to lazy propagation.

## Lazy propagation

To support **range updates** (e.g. "add $v$ to every element in $[l, r]$") in
$O(\log n)$, we cannot afford to touch every affected leaf. Instead, when an
update fully covers a node's range we apply it to that node's aggregate and record
a *pending* update — a **lazy tag** — without descending further. The tag is only
*pushed down* to the children when a later query or update needs to enter that
node. Each node thus carries both its aggregate and any update not yet propagated
to its subtree.

The version below supports range-add and range-sum:

~~~ {.cpp}
struct LazySeg {
    int n;
    vector<long long> t, lazy;

    LazySeg(const vector<long long>& a) : n(a.size()), t(4*n), lazy(4*n, 0) {
        build(1, 0, n - 1, a);
    }

    void build(int node, int lo, int hi, const vector<long long>& a) {
        if (lo == hi) { t[node] = a[lo]; return; }
        int mid = (lo + hi) / 2;
        build(2*node, lo, mid, a);
        build(2*node + 1, mid + 1, hi, a);
        t[node] = t[2*node] + t[2*node + 1];
    }

    void apply(int node, int lo, int hi, long long add) {     // add to a whole range
        t[node]   += (long long)(hi - lo + 1) * add;
        lazy[node] += add;
    }

    void push(int node, int lo, int hi) {                     // propagate to children
        if (lazy[node]) {
            int mid = (lo + hi) / 2;
            apply(2*node,     lo,      mid, lazy[node]);
            apply(2*node + 1, mid + 1, hi,  lazy[node]);
            lazy[node] = 0;
        }
    }

    void update(int node, int lo, int hi, int l, int r, long long add) {
        if (r < lo || hi < l) return;
        if (l <= lo && hi <= r) { apply(node, lo, hi, add); return; }
        push(node, lo, hi);
        int mid = (lo + hi) / 2;
        update(2*node, lo, mid, l, r, add);
        update(2*node + 1, mid + 1, hi, l, r, add);
        t[node] = t[2*node] + t[2*node + 1];
    }

    long long query(int node, int lo, int hi, int l, int r) {
        if (r < lo || hi < l) return 0;
        if (l <= lo && hi <= r) return t[node];
        push(node, lo, hi);
        int mid = (lo + hi) / 2;
        return query(2*node, lo, mid, l, r)
             + query(2*node + 1, mid + 1, hi, l, r);
    }

    void update(int l, int r, long long add) { update(1, 0, n - 1, l, r, add); }
    long long query(int l, int r)            { return query(1, 0, n - 1, l, r); }
};
~~~

Combining different kinds of range update (e.g. range-assign together with
range-add) requires defining how two pending tags compose, which is the main
subtlety in writing a lazy segment tree.

### Problems
- [JuQueen](http://gcpc.nwerc.eu/problemset_2014.pdf)
- [Sum of Squares with Segment Tree](http://www.spoj.com/problems/SEGSQRSS/)

## Applications and variants

- **Coordinate compression.** When indices are large but few are used, map them to
  $[0, n)$ first (see [coordinate compression](Coordinate compression)).
- **Large or unbounded index ranges.** An [implicit segment tree](Implicit segment tree)
  (a.k.a. dynamic/sparse) creates nodes only as they are touched, indexing ranges
  up to $10^{18}$ without coordinate compression — and supports online queries.
- **Hard range updates.** [Segment tree beats](Segment tree beats) handles updates
  such as range $\min$/$\max$ ($a_i \leftarrow \min(a_i, x)$) that ordinary lazy
  propagation cannot.
- **[Persistence](#persistence).** Keeping every past version cheaply.
- **Merge sort tree.** Storing a sorted list at each node answers questions like
  "how many values in $[l, r]$ are $\le x$"; see [merge sort tree](Merge sort tree).
  A [wavelet tree](Wavelet tree) answers similar queries more compactly.
- **Segment tree on a tree.** Combined with [heavy-light
  decomposition](Heavy-light decomposition), path queries on a tree reduce to a
  handful of segment-tree range queries.
- **Higher dimensions.** A 2D segment tree (a segment tree of segment trees)
  handles rectangle queries.

## Persistence
See [Persistent segment tree]().

## Problems
- [Dynamic Range Sum Queries](https://cses.fi/problemset/task/1648)
- [Dynamic Range Minimum Queries](https://cses.fi/problemset/task/1649)
- [Range Update Queries](https://cses.fi/problemset/task/1651)
- [GCD 2010](http://acm.timus.ru/problem.aspx?space=1&num=1846)
- [Movie Collection](https://open.kattis.com/problems/moviecollection)

<details>
<summary>Solution sketch — Dynamic Range Sum / Minimum Queries</summary>

Textbook point-update, range-query segment trees: use the `SegTree` above with
the combine being addition for the sum version, and `min` (identity $+\infty$) for
the minimum version. Each of the $q$ operations is $O(\log n)$.

</details>

<details>
<summary>Solution sketch — Range Update Queries</summary>

Here updates add a value to a whole range while queries read a single position —
the dual of the basic structure. Either use the `LazySeg` above directly (range
add, then a point query is just `query(i, i)`), or keep a segment tree over the
*difference array* so that a range add becomes two point updates and a point read
becomes a prefix-sum query.

</details>

## External links
- [Algorithm Gym :: Everything About Segment Trees](http://codeforces.com/blog/entry/15890)
- [Efficient and easy segment trees](http://codeforces.com/blog/entry/18051)
- [An efficient way to strengthen up your segment tree](http://codeforces.com/blog/entry/13703)
- [Segment tree with insertion and deletion operators](http://codeforces.com/blog/entry/12285)
- [How does a 2D segment tree work?](https://www.quora.com/How-does-a-2D-segment-tree-work)
- [Segment Trees](https://x.algo.is/despin)

## See also
- [Implicit segment tree]() — dynamic/sparse, for huge index ranges
- [Segment tree beats]() — range min/max updates that defy ordinary lazy propagation
- [Persistent segment tree]()
- [Merge sort tree]()
- [Wavelet tree]()
- [Heavy-light decomposition]()
- [Coordinate compression]()
