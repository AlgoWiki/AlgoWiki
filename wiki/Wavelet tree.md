---
categories: Data structures
---

A **wavelet tree** is a static data structure on an integer sequence that answers
a rich family of range queries — $k$-th smallest, count of elements less than a
value, range frequency, range median, and more — each in $O(\log \sigma)$ time,
where $\sigma$ is the size of the value domain.  Building the tree takes
$O(n \log \sigma)$ time and space.  In competitive programming $\sigma$ is almost
always reduced to $n$ by [coordinate compression](), so both bounds become
$O(n \log n)$.

The wavelet tree is closely related to the [persistent segment tree](), which
answers the same queries with the same asymptotic complexity.  In practice the
wavelet tree is faster (better cache behaviour, no pointer overhead) and
significantly simpler to implement correctly.

## Description

Given an array $A[1 \ldots n]$ of integers drawn from the range $[\mathit{lo},
\mathit{hi}]$, a wavelet tree is a complete binary tree over the value range.
Each node covers a value sub-range $[\mathit{lo}, \mathit{hi}]$.  At every
internal node with $\mathit{mid} = \lfloor(\mathit{lo}+\mathit{hi})/2\rfloor$:

- Elements with value $\le \mathit{mid}$ are routed to the **left child**.
- Elements with value $> \mathit{mid}$ are routed to the **right child**.

The node stores a prefix-count array $\mathit{cnt}$, where $\mathit{cnt}[i]$ is
the number of the first $i$ elements (in the node's local ordering) that were
routed left.  With these counts every query can navigate the tree without
storing the values explicitly.

### Construction

~~~ {.cpp}
struct WaveletTree {
    int lo, hi;
    WaveletTree *left = nullptr, *right = nullptr;
    vector<int> cnt; // cnt[i] = # of first i elements routed to left child

    // Build over A[from, to) with value range [lo, hi].
    // The array is reordered in place during construction.
    void build(int *from, int *to, int lo, int hi) {
        this->lo = lo; this->hi = hi;
        if (from >= to || lo == hi) return;
        int mid = lo + (hi - lo) / 2;
        cnt.reserve(to - from + 1);
        cnt.push_back(0);
        for (auto it = from; it != to; ++it)
            cnt.push_back(cnt.back() + (*it <= mid));
        auto pivot = stable_partition(from, to,
                         [mid](int x){ return x <= mid; });
        left  = new WaveletTree(); left->build(from, pivot, lo, mid);
        right = new WaveletTree(); right->build(pivot, to, mid+1, hi);
    }
~~~

`stable_partition` physically splits the array so that left-routed elements
come first; the left child then builds over that prefix, the right child over
the remainder.

### Queries

All queries take a 1-indexed range $[l, r]$.  At each node, two values
summarize the range:

$$
\mathit{lb} = \mathit{cnt}[l-1], \quad \mathit{rb} = \mathit{cnt}[r]
$$

$\mathit{rb} - \mathit{lb}$ elements of $[l,r]$ went left; the rest went
right.  In the left child the range maps to $[\mathit{lb}+1, \mathit{rb}]$; in
the right child it maps to $[l - \mathit{lb},\; r - \mathit{rb}]$.

**$k$-th smallest** (`kth(l, r, k)`): At each node count how many elements in
$[l,r]$ went left.  If $k$ does not exceed that count, recurse left; otherwise
subtract it from $k$ and recurse right.  At a leaf the value is determined.

~~~ {.cpp}
    // k-th smallest element in A[l..r] (1-indexed, k is 1-based)
    int kth(int l, int r, int k) {
        if (lo == hi) return lo;
        int lb = cnt[l-1], rb = cnt[r];
        int inLeft = rb - lb;
        if (k <= inLeft) return left->kth(lb+1, rb, k);
        return right->kth(l-lb, r-rb, k-inLeft);
    }
~~~

**Count less than** (`countLess(l, r, v)`): returns $|\{i \in [l,r] : A[i] < v\}|$.
If $v$ is entirely outside $[\mathit{lo}, \mathit{hi}]$ the answer is trivial.
Otherwise, if $v \le \mathit{mid}$ the right subtree can contribute nothing
(all its values are $> \mathit{mid} \ge v$), so we recurse left only.  If
$v > \mathit{mid}$ all left elements are $< v$, so they all count, plus a
recursive call into the right subtree.

~~~ {.cpp}
    // # elements in A[l..r] with value < v
    int countLess(int l, int r, int v) {
        if (v <= lo) return 0;
        if (v > hi)  return r - l + 1;
        int mid = lo + (hi - lo) / 2;
        int lb = cnt[l-1], rb = cnt[r];
        if (v <= mid) return left->countLess(lb+1, rb, v);
        return (rb - lb) + right->countLess(l-lb, r-rb, v);
    }
};
~~~

**Range frequency** of value $v$ in $[l,r]$: `countLess(l, r, v+1) - countLess(l, r, v)`.

**Range median**: `kth(l, r, (r - l + 2) / 2)`.

### Complexity

Each query descends exactly one root-to-leaf path of the value-range tree,
which has height $\lceil\log_2 \sigma\rceil$.  Each step is $O(1)$.  Hence
every query is $O(\log \sigma)$.

Building involves one `stable_partition` pass at every level of the tree.
Each element participates in exactly one pass per level, giving $O(n \log
\sigma)$ total time.  The $\mathit{cnt}$ arrays across all nodes at any single
level together hold exactly $n+1$ integers, so total space is also $O(n \log
\sigma)$.

## Applications

- **Range order statistics** — finding the $k$-th smallest element in a
  subarray; counting how many elements fall in a value range $[a, b]$ (as
  `countLess(l, r, b+1) - countLess(l, r, a)`).  See [Order statistics tree]()
  for the single-element analogue.
- **Sliding window median** — for a window of fixed size $k$ the median is
  `kth(i, i+k-1, (k+1)/2)`, answered in $O(\log \sigma)$ per window position
  instead of the $O(\log n)$ per insertion of a [balanced BST]() approach (with
  a better constant and no pointer overhead).
- **Range count-distinct** — number of distinct values in $A[l \ldots r]$.
  Define $\mathit{prev}[i]$ = last position before $i$ with the same value (0
  if none).  Build the wavelet tree on the $\mathit{prev}$ array.  Then the
  count of distinct values in $[l, r]$ equals the count of positions $i \in
  [l,r]$ with $\mathit{prev}[i] < l$, which is exactly `countLess(l, r, l)` on
  the $\mathit{prev}$ wavelet tree — answered in $O(\log n)$.
- **Predecessor / successor in a range** — the largest value $\le v$ in
  $A[l \ldots r]$ can be found with a binary-search variant of `countLess`:
  if there is any element $\le v$, compute $k =$ `countLess(l, r, v+1)` and
  return `kth(l, r, k)`.
- **Bitvector rank/select** — at each tree level the $\mathit{cnt}$ array is a
  rank structure; the wavelet tree generalises this to multi-valued alphabets,
  which underpins many [suffix array]() and compressed index applications.

## Variants

### Coordinate compression

When values can be up to $10^9$, map them to $[1, n]$ before building the tree:

~~~ {.cpp}
// Returns a wavelet tree over A[0..n-1], compressed to [1, n].
// sorted_vals receives the sorted unique values so kth results can be decoded.
WaveletTree* buildCompressed(vector<int> &A, vector<int> &sorted_vals) {
    sorted_vals = A;
    sort(sorted_vals.begin(), sorted_vals.end());
    sorted_vals.erase(unique(sorted_vals.begin(), sorted_vals.end()),
                      sorted_vals.end());
    int sigma = sorted_vals.size();
    vector<int> C = A;
    for (int &x : C)
        x = lower_bound(sorted_vals.begin(), sorted_vals.end(), x)
            - sorted_vals.begin() + 1;
    WaveletTree *wt = new WaveletTree();
    wt->build(C.data(), C.data() + C.size(), 1, sigma);
    return wt;
}
// Decode: original value = sorted_vals[ wt->kth(l, r, k) - 1 ]
// countLess for original threshold x:
//   threshold = lower_bound(sorted_vals, x) - sorted_vals.begin() + 1
//   wt->countLess(l, r, threshold)
~~~

### Array-based (flat) implementation

The pointer-based tree above allocates a node per split.  A flat layout stores
all $\mathit{cnt}$ arrays level by level in a single vector, which improves
cache performance and removes allocation overhead:

~~~ {.cpp}
struct WaveletFlat {
    int n, lo, hi, levels;
    vector<vector<int>> cnt; // cnt[d][i] at depth d

    WaveletFlat(vector<int> A, int lo, int hi) : n(A.size()), lo(lo), hi(hi) {
        levels = 1;
        while ((1 << levels) < hi - lo + 1) levels++;
        cnt.assign(levels + 1, vector<int>(n + 1, 0));
        vector<int> cur = A, nxt(n);
        for (int d = 0; d < levels; d++) {
            int range_lo = lo, range_hi = hi;
            // midpoint for the root at this level needs per-node tracking;
            // a simpler approach flattens by bit from MSB to LSB:
            int bit = levels - 1 - d;
            for (int i = 0; i < n; i++)
                cnt[d][i+1] = cnt[d][i] + !((A[i] - lo) >> bit & 1);
            // partition stably by bit `bit`
            int li = 0, ri = cnt[d][n];
            for (int i = 0; i < n; i++) {
                if (!((A[i] - lo) >> bit & 1)) nxt[li++] = A[i];
                else                           nxt[ri++] = A[i];
            }
            swap(A, nxt);
        }
    }
};
~~~

The flat implementation avoids `new` entirely and is typically 2–3× faster in
practice.

### Range sum augmentation

To answer "sum of the $k$ smallest elements in $A[l \ldots r]$", augment each
node with a parallel `sum` prefix array alongside `cnt`, accumulating the
values of elements that go left.  A `sumKSmallest(l, r, k)` query mirrors
`kth`: when $k$ elements fit in the left subtree, return the left subtree's
answer; otherwise add the full left sum and recurse right for the remainder.
The $k$-th smallest value times its count plus sums of smaller elements can
all be computed without changing the $O(\log \sigma)$ per-query bound.

## Problems

### $k$-th order statistics

- [K-th Number (SPOJ MKTHNUM)](https://www.spoj.com/problems/MKTHNUM/)
- [Sliding Median (CSES 1076)](https://cses.fi/problemset/task/1076)

<details>
<summary>Solution sketch — K-th Number (MKTHNUM)</summary>

This is the canonical wavelet tree problem: given $A[1 \ldots n]$ and $m$
queries $(l, r, k)$, output the $k$-th smallest value in $A[l \ldots r]$.

Coordinate-compress the values to $[1, n]$, build the wavelet tree, and for
each query call `kth(l, r, k)` in $O(\log n)$.  Total time $O((n + m)\log n)$.

</details>

<details>
<summary>Solution sketch — Sliding Median (CSES 1076)</summary>

For a sliding window of width $k$, the median position is $p = (k+1)/2$.
After building the wavelet tree on the full array, each window $[i, i+k-1]$
contributes one query `kth(i, i+k-1, p)`.  Each query is $O(\log n)$, giving
$O(n \log n)$ total — the same as a [balanced BST]() sliding window but with a
much smaller constant.

</details>

### Range counting

- [K Query (SPOJ KQUERY)](https://www.spoj.com/problems/KQUERY/)

<details>
<summary>Solution sketch — K Query (KQUERY)</summary>

Each query asks: how many elements in $A[i \ldots j]$ are **greater than** $k$?

This is $(j - i + 1) - \mathtt{countLess}(i,\, j,\, k+1)$.  After
coordinate-compressing, `countLess(i, j, threshold)` where `threshold` is the
rank of $k+1$ in the sorted values runs in $O(\log n)$ per query.

</details>

### Count-distinct and range partitioning

- [Till I Collapse (Codeforces 786C)](https://codeforces.com/contest/786/problem/C)

<details>
<summary>Solution sketch — Till I Collapse (CF 786C)</summary>

For each $k = 1 \ldots n$, compute $f(k)$: the number of groups produced by
greedily splitting $A$ into maximal-length contiguous segments each with $\le k$
distinct values.

Build a wavelet tree on the **previous-occurrence array** $\mathit{prev}$,
where $\mathit{prev}[i]$ is the last index $< i$ with $A[\mathit{prev}[i]] =
A[i]$ (or $0$ if none).  Then the number of distinct values in $A[l \ldots r]$
is `countLess(l, r, l)` on this tree — a single $O(\log n)$ call.

For a fixed $k$, simulate the greedy: starting at $l = 1$, binary search for
the rightmost $r$ such that `countDistinct(l, r)` $\le k$, advance $l = r+1$,
and increment the group counter.  Each greedy step costs $O(\log^2 n)$, and
there are at most $\lceil n/k \rceil$ steps per $k$, giving an overall
$O(n \log^2 n \sum_{k=1}^{n} 1/k) = O(n \log^3 n)$ bound when done offline
in the right order — or $O(n \sqrt{n} \log n)$ with the standard
divide-and-conquer on $k$.

</details>

## See also

- [Persistent segment tree]() — same asymptotic complexity; wavelet tree is the
  offline, space-efficient alternative
- [Merge sort tree]() — stores a sorted list at each segment-tree node; answers
  the same queries in $O(\log^2 n)$ but is simpler to extend to updates
- [Segment tree]() — the underlying range-decomposition idea
- [Order statistics tree]() — handles point insertions/deletions but lacks
  arbitrary range queries
- [Coordinate compression]() — essential preprocessing step when values exceed
  $n$

## External links

- [Wavelet Trees for Competitive Programming (IOI 2016 paper)](https://ioinformatics.org/journal/v10_2016_19_37.pdf)
- [Introduction to a New Data Structure: Wavelet Trees (Rachit Jain)](http://rachitiitr.blogspot.com/2017/06/wavelet-trees-wavelet-trees-editorial.html)
- [Wavelet Tree Implementation – Array Based (Codeforces blog)](https://codeforces.com/blog/entry/113196)
- [Wavelet Tree (Codeforces blog entry 52854)](https://codeforces.com/blog/entry/52854)
- [Wavelet Tree — USACO Guide](https://usaco.guide/adv/wavelet)
