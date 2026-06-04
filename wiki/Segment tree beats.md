---
categories: Data structures
---

**Segment tree beats** (also known as the *Ji Driver Segment Tree*) is a
[segment tree](Segment tree) technique for range updates that cannot be applied by
ordinary [lazy propagation](Segment tree#lazy-propagation), most famously the
range *chmin* update $a_i \leftarrow \min(a_i, x)$ for all $i$ in a range, while
still answering range-sum and range-max queries. The trick keeps a little extra
information at each node so that such an update can usually be applied in bulk, and
an amortized analysis shows the total cost stays near $O((n + q)\log n)$.

## The problem with plain lazy propagation

A range update is easy to make lazy when its effect on a node's aggregate depends
only on the node (range add, range assign, …). The update $a_i \leftarrow
\min(a_i, x)$ is not like that: how it changes a node's sum depends on *which*
elements in the node exceed $x$, information a single tag cannot capture. Segment
tree beats sidesteps this by recognising when the update *does* have a simple
effect, and only recursing when it genuinely must.

## Description

Each node stores, for its range:

- the **maximum** value `mx`,
- the **strict second maximum** `mx2` (the largest value strictly less than `mx`,
  or $-\infty$ if all values are equal),
- the **count** `cnt` of elements equal to `mx`,
- the **sum**.

To apply $a_i \leftarrow \min(a_i, x)$ over a node fully inside the update range,
compare $x$ against these:

- if $x \ge \texttt{mx}$ — every element is already $\le x$; do nothing (prune),
- if $\texttt{mx2} < x < \texttt{mx}$ — only the maximal elements are affected;
  they all drop to $x$, so subtract $(\texttt{mx} - x)\cdot\texttt{cnt}$ from the
  sum and set $\texttt{mx} = x$. This is the lazy case,
- if $x \le \texttt{mx2}$ — more than one distinct value is affected, so the tag
  is not enough; recurse into both children and merge.

The lazy "cap" stored on a node is pushed to its children exactly as in a normal
lazy segment tree (applying the parent's `mx` as a chmin to each child, which by
the invariant only touches each child's maximal elements).

The surprising part is the cost. Although a single update may recurse deeply, a
[potential argument](Amortized analysis) over the number of *distinct* values
present bounds the total work: each forced recursion permanently removes a
distinct value from some subtree. For range chmin with range-sum/max queries the
total is $O((n + q)\log n)$; once range *add* is mixed in (the full "beats"), the
bound is $O(n \log^2 n)$.

## Implementation

The version below supports range chmin, range-sum and range-max. Leaves start with
`mx2` $= -\infty$ and `cnt` $= 1$.

~~~ {.cpp}
const long long NEG = LLONG_MIN / 4;

struct Beats {                                   // range chmin, range-sum, range-max
    int n;
    vector<long long> mx, mx2, sum, cnt, A;

    Beats(const vector<long long>& a)
        : n(a.size()), mx(4*n), mx2(4*n), sum(4*n), cnt(4*n), A(a) { build(1, 0, n-1); }

    void pull(int k) {
        int l = 2*k, r = 2*k+1;
        sum[k] = sum[l] + sum[r];
        if (mx[l] == mx[r]) { mx[k]=mx[l]; cnt[k]=cnt[l]+cnt[r]; mx2[k]=max(mx2[l],mx2[r]); }
        else {
            int hi = (mx[l] > mx[r]) ? l : r, lo = (mx[l] > mx[r]) ? r : l;
            mx[k]=mx[hi]; cnt[k]=cnt[hi]; mx2[k]=max(mx2[hi], mx[lo]);
        }
    }
    void build(int k, int lo, int hi) {
        if (lo == hi) { mx[k]=sum[k]=A[lo]; mx2[k]=NEG; cnt[k]=1; return; }
        int mid=(lo+hi)/2; build(2*k,lo,mid); build(2*k+1,mid+1,hi); pull(k);
    }
    void applyMin(int k, long long x) {          // requires mx2[k] < x < mx[k]
        if (x >= mx[k]) return;
        sum[k] -= (mx[k]-x)*cnt[k];
        mx[k] = x;
    }
    void push(int k) { applyMin(2*k, mx[k]); applyMin(2*k+1, mx[k]); }

    void chmin(int k, int lo, int hi, int l, int r, long long x) {
        if (r<lo || hi<l || mx[k] <= x) return;                  // prune
        if (l<=lo && hi<=r && mx2[k] < x) { applyMin(k, x); return; }   // lazy case
        push(k); int mid=(lo+hi)/2;
        chmin(2*k,lo,mid,l,r,x); chmin(2*k+1,mid+1,hi,l,r,x); pull(k);
    }
    long long qsum(int k,int lo,int hi,int l,int r){
        if(r<lo||hi<l) return 0; if(l<=lo&&hi<=r) return sum[k];
        push(k); int mid=(lo+hi)/2; return qsum(2*k,lo,mid,l,r)+qsum(2*k+1,mid+1,hi,l,r);
    }
    long long qmax(int k,int lo,int hi,int l,int r){
        if(r<lo||hi<l) return NEG; if(l<=lo&&hi<=r) return mx[k];
        push(k); int mid=(lo+hi)/2; return max(qmax(2*k,lo,mid,l,r),qmax(2*k+1,mid+1,hi,l,r));
    }

    void chmin(int l,int r,long long x){ chmin(1,0,n-1,l,r,x); }
    long long qsum(int l,int r){ return qsum(1,0,n-1,l,r); }
    long long qmax(int l,int r){ return qmax(1,0,n-1,l,r); }
};
~~~

## Extensions

The same idea extends to a richer operation set, at the cost of more tags and more
cases:

- **Range chmax** $a_i \leftarrow \max(a_i, x)$ is symmetric — track the minimum,
  strict second minimum, and count of the minimum.
- **Range add** can be combined with chmin/chmax (this is the full "beats"); it
  raises the amortized bound to $O(n \log^2 n)$.
- **Historic maximum** ("the max value each position has ever held") is a
  well-known further extension.

## Problems
- [Gorgeous Sequence](http://acm.hdu.edu.cn/showproblem.php?pid=5306)
- [Range Chmin Chmax Add Range Sum](https://judge.yosupo.jp/problem/range_chmin_chmax_add_range_sum)

<details>
<summary>Solution sketch — Gorgeous Sequence</summary>

This is the original segment tree beats problem: support range chmin
$a_i \leftarrow \min(a_i, x)$ together with range-max and range-sum queries. Use
the `Beats` structure above directly — the three-way case analysis on
$x$ vs. `mx`/`mx2` is exactly what is needed, and the amortized
$O((n+q)\log n)$ bound is what makes it pass.

</details>

## See also
- [Segment tree]() — the base structure
- [Lazy propagation](Segment tree#lazy-propagation) — what beats generalizes
- [Implicit segment tree]() — another advanced segment-tree variant

## External links
- [Segment tree beats (Codeforces)](https://codeforces.com/blog/entry/57319)
