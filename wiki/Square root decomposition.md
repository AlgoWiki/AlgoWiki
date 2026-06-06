---
categories: Algorithm techniques, Data structures
---

**Square root decomposition** (also called **sqrt decomposition** or **block decomposition**) is a family of techniques that achieve $O(\sqrt{n})$ per operation — or $O((n + q)\sqrt{n})$ overall — by partitioning an array (or query set) into blocks of size $B \approx \sqrt{n}$ and maintaining a summary for each block. It is simpler to implement than a [Segment tree]() or [Fenwick tree]() and is often the only feasible approach for problems where the per-element operation is too expensive for logarithmic data structures.

## Description

The basic idea is to divide an array of $n$ elements into blocks of size $B$.
For each block $k$ we precompute a *summary* (e.g. block sum, block minimum). A
range query $[l, r]$ then decomposes into:

1. Up to two *partial blocks* at the two ends — touched element by element.
2. $O(n/B)$ *complete blocks* in the middle — answered in $O(1)$ each via the
   precomputed summary.

This costs $O(B + n/B)$ per query, minimized at $B = \sqrt{n}$ giving
$O(\sqrt{n})$.

### Range sum with point updates

The canonical implementation for range-sum queries with point updates:

~~~ {.cpp}
struct SqrtSum {
    int n, B;
    vector<long long> a, block;

    SqrtSum(vector<int>& v) : n(v.size()), B(max(1, (int)sqrt(n))),
                               a(v.begin(), v.end()), block(n / B + 1, 0) {
        for (int i = 0; i < n; i++) block[i / B] += a[i];
    }

    void update(int i, int val) {
        block[i / B] += val - a[i];
        a[i] = val;
    }

    long long query(int l, int r) {         // sum of a[l..r], inclusive
        long long res = 0;
        int bl = l / B, br = r / B;
        if (bl == br) {
            for (int i = l; i <= r; i++) res += a[i];
        } else {
            for (int i = l; i < (bl + 1) * B; i++) res += a[i];
            for (int b = bl + 1; b < br; b++) res += block[b];
            for (int i = br * B; i <= r; i++) res += a[i];
        }
        return res;
    }
};
~~~

Both `update` and `query` run in $O(\sqrt{n})$. Precomputation is $O(n)$.

### Range updates with range queries (lazy blocks)

When we need *range* updates (add $v$ to every element in $[l, r]$) and *range*
queries, we keep a *lazy* addend `lazy[b]` for each complete block and a
separate `block_sum[b]` for the sum of the raw `a[]` values within that block.
Partial-block updates touch elements directly; complete-block updates only
increment `lazy[b]`.

~~~ {.cpp}
struct SqrtRange {
    int n, B;
    vector<long long> a, lazy, block_sum;

    SqrtRange(int n) : n(n), B(max(1, (int)sqrt(n))),
                       a(n, 0), lazy(n / B + 1, 0), block_sum(n / B + 1, 0) {}

    // add val to a[l..r]
    void update(int l, int r, long long val) {
        int bl = l / B, br = r / B;
        if (bl == br) {
            for (int i = l; i <= r; i++) { a[i] += val; block_sum[bl] += val; }
        } else {
            for (int i = l; i < (bl+1)*B; i++) { a[i] += val; block_sum[bl] += val; }
            for (int b = bl+1; b < br; b++) lazy[b] += val;
            for (int i = br*B; i <= r; i++) { a[i] += val; block_sum[br] += val; }
        }
    }

    // sum of a[l..r]
    long long query(int l, int r) {
        long long res = 0;
        int bl = l / B, br = r / B;
        if (bl == br) {
            for (int i = l; i <= r; i++) res += a[i];
            res += lazy[bl] * (r - l + 1);
        } else {
            for (int i = l; i < (bl+1)*B; i++) res += a[i];
            res += lazy[bl] * ((bl+1)*B - l);
            for (int b = bl+1; b < br; b++) res += block_sum[b] + lazy[b] * B;
            for (int i = br*B; i <= r; i++) res += a[i];
            res += lazy[br] * (r - br*B + 1);
        }
        return res;
    }
};
~~~

For a *minimum* query with range assignment updates, store a per-block minimum
and replace the lazy addend with a lazy assignment flag; rebuild the block
minimum only when touching boundary elements.

### Complexity

$$
T_{\text{query}} = O(B + n/B) = O(\sqrt{n}) \quad \text{at } B = \sqrt{n}.
$$

For $q$ operations on $n$ elements: total time $O((n + q)\sqrt{n})$. In
practice, tuning $B$ (e.g. $B = \lceil\sqrt{n}\rceil$ or $B \approx 700$ for
$n = 10^5$) and cache effects can matter.

## Applications

- **Range queries with expensive per-element operations.** When the underlying
  operation doesn't easily compose (e.g. counting distinct values, XOR of
  sorted medians), a sqrt block can do it with $O(B)$ brute-force per partial
  block and a maintained summary per complete block.
- **Offline range queries — [Mo's algorithm]().** By sorting queries in a
  specific order derived from block assignments, all queries are answered in
  $O((n + q)\sqrt{n})$ total using two pointers that expand/shrink a sliding
  window.
- **Answering queries on static trees.** [Mo's algorithm on trees]() flattens a
  tree into an Euler-tour sequence and applies the same trick.
- **√-time updates on a [Segment tree]() / [Fenwick tree]().** Some
  problems mix arbitrary updates with structural queries; a sqrt layer can absorb
  recent updates in a buffer and flush them in bulk.
- **Persistent / functional arrays.** Sqrt decomposition over a version tree
  enables $O(\sqrt{n})$ rollbacks without a fully persistent structure.

## Variants

### Mo's algorithm (offline range queries)

[Mo's algorithm]() is a direct application of sqrt decomposition to *offline*
queries. Sort queries $[l_i, r_i]$ by (block of $l_i$, $r_i$ pairwise
direction-alternating) so that when processing them in order the total movement
of the two pointers is $O((n + q)\sqrt{n})$. Each step adds or removes one
element from the current window, requiring $O(1)$ work per element.

~~~ {.cpp}
struct Mo {
    int n, B;
    struct Query { int l, r, idx; };

    vector<Query> queries;
    Mo(int n, int q) : n(n), B(max(1, (int)sqrt(n))) { queries.reserve(q); }

    void add_query(int l, int r, int idx) { queries.push_back({l, r, idx}); }

    // add / remove must maintain the current answer for [curL, curR]
    template<class Add, class Rem, class Ans>
    vector<typename result_of<Ans()>::type>
    solve(Add add, Rem rem, Ans ans) {
        sort(queries.begin(), queries.end(), [&](auto& a, auto& b) {
            int ba = a.l / B, bb = b.l / B;
            if (ba != bb) return ba < bb;
            return (ba & 1) ? a.r > b.r : a.r < b.r;
        });
        using T = decltype(ans());
        vector<T> res(queries.size());
        int curL = 0, curR = -1;
        for (auto& q : queries) {
            while (curR < q.r) add(++curR);
            while (curL > q.l) add(--curL);
            while (curR > q.r) rem(curR--);
            while (curL < q.l) rem(curL++);
            res[q.idx] = ans();
        }
        return res;
    }
};
~~~

### Sqrt decomposition with rollback (Mo's algorithm with updates)

When queries also have point updates, a third parameter — time — is introduced.
Queries become triples $(l, r, t)$; the block size for the time axis is also
$O(\sqrt{q})$, giving total time $O(q^{5/3})$ or $O(n \cdot q^{2/3})$.

### Sqrt of a Segment tree (sqrt-bucket tree)

For problems that need both $O(\log n)$ queries *and* $O(1)$ updates per
position but are too complex for a standard segment tree, one can build a
two-level structure: the bottom level is plain arrays of size $\sqrt{n}$, and
the top level maintains block summaries. Updates touch one block in $O(\sqrt{n})$;
queries scan $O(\sqrt{n})$ blocks.

### Static range minimum with sparse table fallback

When updates are absent, a [Sparse table]() answers RMQ in $O(1)$ per query
after $O(n \log n)$ preprocessing, and is strictly better. Sqrt decomposition
is the go-to when updates are present and $O(\log n)$ per operation from a
segment tree is either too slow (because of large constants) or overkill.

## Problems

Problems are grouped by the idea each exercises. Selected entries include a
solution sketch focused on how sqrt decomposition is used.

### Basic range queries

- [Range Sum Queries II](https://cses.fi/problemset/task/1648) (CSES) — range
  sum with point updates; a warmup where a Fenwick tree is faster but sqrt works.
- [Range Minimum Queries](https://cses.fi/problemset/task/1649) (CSES) — range
  min with point updates.
- [XOR on Segment](https://codeforces.com/problemset/problem/242/E) (Codeforces
  242E) — range XOR update, range sum query.

<details>
<summary>Solution sketch — XOR on Segment</summary>

Maintain blocks of size $B \approx \sqrt{n}$. For a range-XOR update $[l, r]$
with value $v$: apply $v$ element-by-element to the two partial blocks; for
each complete block, instead of touching every element, store a lazy XOR
`lazy[b]`. The block sum becomes $\text{sum1}[b]$ (sum when no extra XOR) and
$\text{sum0}[b]$ (sum XORed by $v$); toggling the lazy bit swaps them in $O(1)$.
A range-sum query accumulates element sums (offset by `lazy`) for partial blocks
and `sum[b]` for complete blocks. Both operations are $O(\sqrt{n})$.

</details>

### Offline range queries (Mo's algorithm)

- [D-query](https://www.spoj.com/problems/DQUERY/) (SPOJ) — count distinct
  values in a range.
- [Powerful Array](https://codeforces.com/contest/86/problem/D) (Codeforces 86D)
  — sum of $c_v^2$ over all values $v$ in a range, where $c_v$ is the count.
- [Curious Cupid](https://open.kattis.com/problems/cupid) (Kattis)

<details>
<summary>Solution sketch — Powerful Array (CF 86D)</summary>

Maintain a frequency array `cnt[v]` and the current answer `ans`. When element
$v$ is added, `ans += 2*cnt[v] + 1`, then `cnt[v]++`; when removed, `cnt[v]--`,
then `ans -= 2*cnt[v] + 1`. Each add/remove is $O(1)$, so Mo's algorithm
answers all $q$ queries in $O((n+q)\sqrt{n})$ total.

</details>

<details>
<summary>Solution sketch — D-query</summary>

Maintain `cnt[v]` (count of value $v$ in current window) and `distinct` (number
of values with non-zero count). On add: if `cnt[v]` was $0$, increment
`distinct`. On remove: if `cnt[v]` drops to $0$, decrement `distinct`. Mo's
ordering makes the total pointer movement $O((n+q)\sqrt{n})$.

</details>

### Range updates + range queries (lazy sqrt)

- [Ynoi2015 - 苦痛](https://www.luogu.com.cn/problem/P4119) — range add, range
  sum of squares; showcase of the lazy-block technique.
- [Sqrt Queries](https://cses.fi/problemset/task/3076) (CSES) — range assign
  and range sum.

<details>
<summary>Solution sketch — Range assign, range sum</summary>

Each block stores a *lazy assignment* flag and the block sum. A range-assign
$[l, r] \leftarrow v$: for partial blocks, write the value element-by-element
and recompute the block sum ($O(B)$); for complete blocks, set `lazy[b] = v`
and `block_sum[b] = v * B` ($O(1)$ per block). A range-sum query: for partial
blocks, sum elements plus `lazy` if set; for complete blocks, add `block_sum[b]`
directly. Both are $O(\sqrt{n})$.

</details>

### Harder applications

- [Tree and Queries](https://codeforces.com/problemset/problem/375/D)
  (Codeforces 375D) — count values appearing $\ge k$ times in a subtree range
  (Mo's on trees).
- [Jeff and Removing Periods](https://codeforces.com/problemset/problem/351/D)
  (Codeforces 351D) — offline; maintain a bitset of "active" elements.
- [2D queries](https://www.spoj.com/problems/IOPC1207/) (SPOJ IOPC1207) — 2D
  prefix sums; sqrt decomposition reduces update cost.

## See also

- [Mo's algorithm]() — offline range queries via sqrt block ordering
- [Mo's algorithm on trees]() — applies Mo's to tree paths via Euler tours
- [Segment tree]() — $O(\log n)$ alternative when lazy propagation is feasible
- [Fenwick tree]() — simpler $O(\log n)$ structure for prefix-sum problems
- [Sparse table]() — $O(1)$ range minimum for static arrays

## External links

- [Sqrt decomposition (cp-algorithms)](https://cp-algorithms.com/data_structures/sqrt_decomposition.html)
- [MO's Algorithm (Query sqrt decomposition)](https://blog.anudeep2011.com/mos-algorithm/)
- [Sqrt decomposition tutorial (Codeforces)](https://codeforces.com/blog/entry/83248)
- [SQRT decomposition (e-maxx)](https://e-maxx.ru/algo/sqrt_decomposition)
