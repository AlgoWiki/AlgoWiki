---
categories: Dynamic programming, Algorithm techniques
---

The **knapsack problem** is a family of combinatorial optimisation problems: given a
set of items each with a *weight* and a *value*, and a knapsack of fixed capacity,
choose items to maximise total value without exceeding capacity. Despite being
[NP-complete]() in general — no algorithm can solve every instance in time polynomial
in the description length — all standard variants admit [dynamic programming]()
solutions running in $O(nW)$ time, where $n$ is the number of items and $W$ is the
capacity. Because $W$ appears linearly rather than logarithmically, this is only
*pseudo-polynomial*, but it is fast enough for the $n \leq 10^3$, $W \leq 10^6$
constraints typical of contests. Knapsack is one of the most frequently recurring
DP patterns in competitive programming, and understanding its variants — bounded,
unbounded, group, tree, fractional, and meet-in-the-middle — covers a large
fraction of the "selection under budget" problems that appear on every major judge.

## Description

### 0/1 knapsack

The standard formulation: $n$ items, item $i$ with integer weight $w_i > 0$ and
integer value $v_i \geq 0$, knapsack capacity $W$. Each item may be taken at most
once. Maximise total value subject to total weight $\leq W$.

Define $\mathrm{dp}[i][c]$ = maximum total value obtainable using a subset of the
first $i$ items with total weight $\leq c$. The recurrence considers two cases for
item $i$: skip it (inheriting $\mathrm{dp}[i-1][c]$) or take it (adding $v_i$ and
consuming $w_i$ capacity):

$$
\mathrm{dp}[i][c] = \max\!\bigl(\mathrm{dp}[i{-}1][c],\; \mathrm{dp}[i{-}1][c - w_i] + v_i\bigr),
$$

where the second option requires $c \geq w_i$. The 2-D table has $O(nW)$ entries
and can be filled row by row in $O(nW)$ time and $O(nW)$ space. Space reduces to
$O(W)$ by *rolling* the $i$-dimension: maintain only one 1-D array, and process
capacity **right to left** so that $\mathrm{dp}[c - w_i]$ still holds the
$(i-1)$-th row value when it is read. Scanning left to right instead would allow
item $i$ to be used multiple times — which is exactly the unbounded variant below.

~~~ {.cpp}
// 0/1 knapsack: n items with weights w[], values v[], capacity W
// dp[c] = max value achievable with total weight <= c
vector<int> dp(W + 1, 0);
for (int i = 0; i < n; i++)
    for (int c = W; c >= w[i]; c--)    // right-to-left: each item at most once
        dp[c] = max(dp[c], dp[c - w[i]] + v[i]);
// answer: dp[W]
~~~

### Complexity

Time $O(nW)$, space $O(W)$ with the rolling optimisation. The complexity is
*pseudo-polynomial*: $W$ is encoded in $O(\log W)$ bits, so $O(nW)$ is exponential
in the input length — fully consistent with knapsack being NP-complete. In practice,
contest problems keep $nW \leq 10^8$ or so, and the constant factor of the inner
loop is small.

## Applications

- **[Subset-sum]()** — deciding whether some subset has total weight exactly $T$ is
  0/1 knapsack with all values equal to 1 (or a boolean DP array). This is the
  canonical NP-complete decision problem, appearing as a special case of knapsack
  and as a subroutine in countless contest problems.
- **Coin change** — producing a target amount $T$ from denominations (each usable
  many times) is [unbounded knapsack](#unbounded-knapsack). Minimising the number of
  coins swaps max for min; counting the number of distinct multisets uses the
  counting formulation. These are among the most common DP problems in contests.
- **Budget-constrained scheduling** — selecting jobs, projects, or items under a
  time or cost budget to maximise profit maps directly onto 0/1 or bounded knapsack,
  depending on whether each option may be chosen once or a limited number of times.
- **LP relaxation** — the [fractional knapsack](#fractional-knapsack) variant is the
  [linear programming]() relaxation of the 0/1 problem. Its greedy optimum provides
  a tight upper bound on the integer solution, and branch-and-bound solvers use this
  bound at every node of their search tree.

## Variants

### Unbounded knapsack

Each item may be used **any number of times**. The only change from 0/1 is to scan
capacity **left to right**, so that when $\mathrm{dp}[c - w_i]$ is read it already
reflects the possibility of having used item $i$ earlier in this same pass:

~~~ {.cpp}
// Unbounded knapsack: each item usable unlimited times
vector<int> dp(W + 1, 0);
for (int i = 0; i < n; i++)
    for (int c = w[i]; c <= W; c++)    // left-to-right: reuse allowed
        dp[c] = max(dp[c], dp[c - w[i]] + v[i]);
~~~

Coin change — minimum coins to reach amount $T$ — is the same recurrence with min
and an initialisation of $+\infty$ (use `INT_MAX / 2` to avoid overflow):

~~~ {.cpp}
// Minimum coins: dp[c] = fewest coins to make amount c
vector<int> dp(W + 1, INT_MAX / 2);
dp[0] = 0;
for (int c = 1; c <= W; c++)
    for (int i = 0; i < n; i++)
        if (c >= w[i])
            dp[c] = min(dp[c], dp[c - w[i]] + 1);
~~~

### Counting subsets and the bitset trick

Replace max with addition to count the number of subsets achieving each total weight.
For pure **feasibility** ("is weight $T$ achievable?") the DP is boolean and each
update is a logical OR. When weights are integers, the bitwise representation of the
reachability vector lets an entire pass over one item's weight be done with a single
left-shift and OR, reducing the per-item inner loop from $O(T)$ to $O(T/64)$:

~~~ {.cpp}
// Subset-sum feasibility with bitset: O(n * T / 64)
// reachable[c] = 1 iff some subset of the first i items has total weight exactly c
bitset<MAXW> reachable;
reachable[0] = 1;
for (int i = 0; i < n; i++)
    reachable |= reachable << w[i];
// reachable[T] = 1 iff target T is achievable
~~~

The intuition: `reachable << w[i]` shifts every achievable weight up by $w_i$,
producing the set of weights reachable by adding one copy of item $i$ to any
previously reachable subset. OR-ing with the original covers both "take" and "skip".

### Fractional knapsack

Items may be **split**: taking fraction $x_i \in [0,1]$ of item $i$ contributes
weight $x_i w_i$ and value $x_i v_i$. This variant is solved greedily in
$O(n \log n)$: sort items by *value density* $v_i / w_i$ descending, fill the
knapsack greedily in that order, and split the last item to fill remaining capacity
exactly. The greedy works because taking more of the highest-density item is always
at least as good as taking any of a lower-density item. The fractional solution
gives the LP relaxation of the integer 0/1 problem and is always an upper bound on
the 0/1 optimum.

### Bounded knapsack

Item $i$ is available $c_i$ times. The naïve reduction to 0/1 knapsack — replicate
each item $c_i$ times — costs $O((\sum c_i) W)$, which is too slow when counts are
large.

**Binary grouping** brings this down to $O(nW \log C)$ where $C = \max c_i$
[^1][^2]. The idea is to represent the $c_i$ copies of item $i$ using a small
number of *virtual* 0/1 items: take groups of sizes $1, 2, 4, 8, \ldots$ (powers of
two) up to a remainder that makes the total exactly $c_i$. Because every integer in
$[0, c_i]$ can be written as a sum of a subset of these group sizes (this follows
from the standard binary representation argument), the virtual items can collectively
reproduce any valid count from $0$ to $c_i$. Running the standard 0/1 knapsack on
the $O(\log c_i)$ virtual items per original item is therefore exact.

~~~ {.cpp}
// Bounded knapsack via binary grouping: O(n W log C)
vector<int> dp(W + 1, 0);
for (int i = 0; i < n; i++) {
    int rem = cnt[i];
    for (int k = 1; rem > 0; k *= 2) {
        int take = min(k, rem);
        rem -= take;
        int wt = take * w[i], vl = take * v[i];
        for (int c = W; c >= wt; c--)
            dp[c] = max(dp[c], dp[c - wt] + vl);
    }
}
~~~

**Monotone deque** achieves a strictly $O(nW)$ solution — no log factor — by
recognising that, for a fixed item type $i$, the capacities can be partitioned into
$w_i$ independent residue classes modulo $w_i$, and within each class the DP update
reduces to a **sliding-window maximum**.

Fix residue $r \in \{0, 1, \ldots, w_i - 1\}$ and let $a_j = \mathrm{dp}[r + j w_i]$
denote the old DP value for the $j$-th capacity in this class. The update for
bounded knapsack says we can take $k \in [0, c_i]$ copies of item $i$, so the new
value is

$$
b_j = \max_{0 \leq k \leq \min(j,\, c_i)} \bigl(a_{j-k} + k\, v_i\bigr).
$$

Substituting $t = j - k$ (the slot we "pull from") and rearranging:

$$
b_j = \max_{j - c_i \leq t \leq j} \bigl(a_t + (j - t)\, v_i\bigr)
     = j\, v_i + \max_{j - c_i \leq t \leq j} \underbrace{\bigl(a_t - t\, v_i\bigr)}_{f(t)}.
$$

The quantity $f(t) = a_t - t\, v_i$ depends only on the old value $a_t$, so it can
be precomputed for each $t$. The maximum of $f$ over the window $[j - c_i, j]$ of
size $c_i + 1$ is a standard sliding-window maximum problem, solved in amortised
$O(1)$ per step with a monotone deque:

~~~ {.cpp}
// Bounded knapsack with monotone deque: O(n W)
vector<int> dp(W + 1, 0);
for (int i = 0; i < n; i++) {
    vector<int> old = dp;   // snapshot of dp before processing item i
    for (int r = 0; r < w[i] && r <= W; r++) {
        // process the residue class r: capacities r, r+w[i], r+2*w[i], ...
        deque<pair<int,int>> dq;   // (index j, f(j) = old[r + j*w[i]] - j*v[i])
        int maxj = (W - r) / w[i];
        for (int j = 0; j <= maxj; j++) {
            int idx = r + j * w[i];
            int fj  = old[idx] - j * v[i];
            // remove indices that have left the window [j - cnt[i], j]
            while (!dq.empty() && dq.front().first < j - cnt[i])
                dq.pop_front();
            // maintain decreasing f-values; dominated entries can never be max
            while (!dq.empty() && dq.back().second <= fj)
                dq.pop_back();
            dq.push_back({j, fj});
            // b_j = (max f in window) + j * v[i]
            dp[idx] = dq.front().second + j * v[i];
        }
    }
}
~~~

The `old` snapshot is necessary because $f(t)$ must use the DP values from *before*
item $i$ was processed. Each residue class of $w_i$ elements is handled in $O(W /
w_i)$ time; summing over all residue classes gives $O(W)$ per item, and $O(nW)$
total.

### 2D (multi-dimensional) knapsack

Items have two weight dimensions (e.g. weight and volume) with separate capacities
$W$ and $V$. The DP table extends to $\mathrm{dp}[c_1][c_2]$ and both dimensions
are scanned right to left, again ensuring each item is counted at most once:

~~~ {.cpp}
// 2D knapsack: items with weights w[], volumes vol[], values v[]
vector<vector<int>> dp(W + 1, vector<int>(V + 1, 0));
for (int i = 0; i < n; i++)
    for (int c1 = W; c1 >= w[i]; c1--)
        for (int c2 = V; c2 >= vol[i]; c2--)
            dp[c1][c2] = max(dp[c1][c2], dp[c1 - w[i]][c2 - vol[i]] + v[i]);
~~~

The table is $O(WV)$ in size and the total work is $O(nWV)$, so both dimensions
must be small. Adding a third dimension is straightforward but quickly becomes
memory-prohibitive.

### Group knapsack

Items are partitioned into groups $G_1, \ldots, G_k$; **at most one item** may be
selected from each group. The key insight is that this is structurally identical to
0/1 knapsack at the group level: process groups one at a time (outer loop), scan
capacity right to left (middle loop) so the old pre-group DP values are always used,
and try every item in the group (inner loop). Because the middle loop reads
`dp[c - wt]` from the old state, no two items from the same group can both
contribute to a single DP cell:

~~~ {.cpp}
// Group knapsack: at most one item selected per group
// groups[g] = list of (weight, value) pairs in group g
vector<int> dp(W + 1, 0);
for (auto& grp : groups) {
    for (int c = W; c >= 0; c--)
        for (auto [wt, vl] : grp)
            if (c >= wt)
                dp[c] = max(dp[c], dp[c - wt] + vl);
}
~~~

This models "buy at most one product from each shop" and configurations where
choices within a component are mutually exclusive.

### Tree (dependency) knapsack

If selecting item $v$ requires that its parent in a rooted tree is also selected,
the problem becomes *tree knapsack* (or *dependent knapsack*). The key structure is
that valid selections form connected subtrees containing the root of each selected
component. The standard approach is a post-order DFS, where
$\mathrm{dp}[v][c]$ = max value in $v$'s subtree using capacity $c$ (with
$\mathrm{dp}[v][c] = 0$ for $c < w_v$, since $v$ itself cannot be afforded).
Children are merged in one at a time using a knapsack-of-knapsacks merge; the inner
loop limit of $c - w_v$ ensures that the $w_v$ units needed for $v$ itself are
always reserved:

~~~ {.cpp}
// Tree knapsack: dp[v][c] = max value in subtree(v) using capacity c,
//                with the constraint that any descendant requires v.
int W;
vector<int> adj[MAXN];
int w[MAXN], val[MAXN];
vector<int> dp[MAXN];

void dfs(int v, int par) {
    dp[v].assign(W + 1, 0);
    for (int c = w[v]; c <= W; c++) dp[v][c] = val[v]; // only v selected

    for (int u : adj[v]) {
        if (u == par) continue;
        dfs(u, v);
        // merge subtree u into v's DP; right-to-left on c keeps v's cost reserved
        for (int c = W; c >= w[v]; c--)
            for (int k = 1; k <= c - w[v]; k++)
                dp[v][c] = max(dp[v][c], dp[v][c - k] + dp[u][k]);
    }
}
// answer: dp[root][W]
~~~

The naive complexity is $O(n^2 W)$, but bounding each subtree's DP array to
$\min(\text{subtree weight}, W) + 1$ entries reduces the total work to $O(nW)$:
every pair of nodes meets at the merge of their lowest common ancestor exactly once,
and the total number of cells merged over the whole tree is bounded by $O(nW)$.

An elegant $O(nW)$ alternative flattens the tree with a DFS-order Euler tour and
runs a 1-D DP where "take item $v$" advances one step into $v$'s subtree and "skip
$v$" jumps past the entire subtree — see the external links.

### Meet in the middle

When $n \leq 40$ but $W$ is too large for $O(nW)$ DP (e.g. weights up to $10^{12}$,
or real-valued), [meet in the middle]() cuts the exponent in half. The idea: split
items into two halves $A$ and $B$ of $\sim n/2$ each, enumerate all $2^{n/2}$
subsets of each half as (weight, value) pairs in $O(2^{n/2} \cdot n)$ time, then
match the two halves. After sorting $A$ by weight and computing a prefix maximum on
values, each $B$-subset can be matched to the best complement in $A$ with a single
binary search:

~~~ {.cpp}
using ll = long long;
using pll = pair<ll, ll>;

void enumerate(vector<pll>& items, vector<pll>& out) {
    int m = items.size();
    for (int mask = 0; mask < (1 << m); mask++) {
        ll w = 0, v = 0;
        for (int i = 0; i < m; i++)
            if (mask >> i & 1) { w += items[i].first; v += items[i].second; }
        out.push_back({w, v});
    }
}

ll knapsack_mitm(vector<pll> items, ll W) {
    int n = items.size(), half = n / 2;
    vector<pll> A, B;
    vector<pll> left(items.begin(), items.begin() + half);
    vector<pll> right(items.begin() + half, items.end());
    enumerate(left, A);
    enumerate(right, B);

    // Sort A by weight; prefix-max on value so A[i].second = best value
    // among all A-subsets with weight <= A[i].first
    sort(A.begin(), A.end());
    for (size_t i = 1; i < A.size(); i++)
        A[i].second = max(A[i].second, A[i - 1].second);

    ll ans = 0;
    for (auto [wb, vb] : B) {
        if (wb > W) continue;
        // find the best A-subset with weight <= W - wb
        auto it = upper_bound(A.begin(), A.end(), pll{W - wb, LLONG_MAX});
        if (it != A.begin())
            ans = max(ans, vb + prev(it)->second);
    }
    return ans;
}
~~~

Time $O(2^{n/2} \cdot n)$, space $O(2^{n/2})$. For $n = 40$ this is roughly
$4 \times 10^7$ operations, comfortably within contest limits.

## Problems

### 0/1 knapsack

- [Knapsack](https://open.kattis.com/problems/knapsack) (Kattis) — textbook
  formulation; output the max value and the selected item indices.
- [Book Shop](https://cses.fi/problemset/task/1158) (CSES) — maximise pages
  bought under a price budget; standard 0/1 knapsack.
- [Walrus Weights](https://open.kattis.com/problems/walrusweights) (Kattis) —
  split weights into two piles minimising the difference; run the boolean 0/1
  knapsack up to $S/2$ and scan for the nearest achievable weight.

<details>
<summary>Solution sketch — Walrus Weights</summary>

Let $S$ be the total weight of all items. We want a subset whose total is as close
to $S/2$ as possible. Run the 0/1 boolean knapsack to find all achievable sums up to
$\lfloor S/2 \rfloor$, then scan from $\lfloor S/2 \rfloor$ downward to find the
largest achievable $c^*$. The minimum absolute difference between pile totals is
$S - 2c^*$.

</details>

- [Roller Coaster Fun](https://open.kattis.com/problems/rollercoasterfun) (Kattis)

### Unbounded knapsack and coin change

- [Coin Combinations I](https://cses.fi/problemset/task/1635) (CSES) — count
  *ordered* ways to reach target sum using coin denominations (each reusable). Swap
  the loop order: outer over amounts, inner over coins — each amount is reachable
  from any previously seen coin choice.
- [Coin Combinations II](https://cses.fi/problemset/task/1636) (CSES) — count
  *unordered* ways; the standard unbounded knapsack counting formulation (outer over
  coins, inner over amounts left-to-right).

<details>
<summary>Solution sketch — Coin Combinations II</summary>

Counting unordered multisets requires avoiding permutation duplicates. The fix: the
outer loop iterates coin types, the inner loop iterates amounts left-to-right,
updating $\mathrm{dp}[c] \mathrel{+}= \mathrm{dp}[c - w_i]$. Each dp value
accumulates combinations whose largest coin index is the current one, ensuring each
multiset is counted exactly once. The left-to-right scan allows the same coin to
appear multiple times in a single pass (unbounded), while the outer coin loop fixes
which coin types are "allowed so far".

</details>

- [Buffed Buffet](https://open.kattis.com/problems/buffet) (Kattis) — buffet items
  are either discrete (bounded) or continuous (fractional); requires combining both
  knapsack types.

### Subset sum and counting

- [Money Sums](https://cses.fi/problemset/task/1745) (CSES) — find all achievable
  coin subset sums; run the boolean 0/1 knapsack, then collect and count the true
  entries.
- [Two Sets II](https://cses.fi/problemset/task/1093) (CSES) — count ways to
  partition $\{1, \ldots, n\}$ into two equal-sum subsets; counting 0/1 knapsack on
  target $S/2$ (answer is 0 if $S$ is odd).

### Bounded knapsack

- [Book Shop II](https://cses.fi/problemset/task/1159) (CSES) — same as Book Shop
  but each book has a limited print run $c_i$; binary grouping or monotone deque
  required.
- [Lucky Country](https://codeforces.com/problemset/problem/95/E) (CF 95E) —
  bounded knapsack after grouping items by value label; the number of distinct item
  types is small, making binary grouping efficient [^3].

<details>
<summary>Solution sketch — Lucky Country</summary>

Group items by their label value: each distinct value $v$ appears $c_v$ times,
giving a bounded knapsack instance. Since the number of distinct values is at most
$O(\sqrt{W})$ (if more distinct values existed, their total copies would exceed the
capacity), apply binary grouping on each group and run the resulting 0/1 knapsack in
$O(D W \log C)$ time, where $D$ is the number of distinct values.

</details>

### Group knapsack

- [Tug of War](https://boi.cses.fi/files/boi2015_day2.pdf) (BOI 2015 Day 2) —
  partition players into two teams of almost-equal size and total weight; feasibility
  for each candidate split is checked with a 2D DP tracking both count and weight,
  naturally a group-style knapsack.

### Tree (dependency) knapsack

- [Karen and Supermarket](https://codeforces.com/problemset/problem/815/C)
  (CF 815C) — $n$ goods with a tree of coupon dependencies; select at most $m$ goods
  at minimum cost, choosing whether to apply each coupon (using a coupon requires the
  parent coupon to also be used).

<details>
<summary>Solution sketch — Karen and Supermarket</summary>

Root the coupon tree at node 1. Define $\mathrm{dp}[v][j][s]$ = minimum cost to
purchase exactly $j$ goods from $v$'s subtree, where $s = 0$ means coupon $v$ is
not used and $s = 1$ means it is (discounting $v$'s own price by $d_v$). At a leaf
$v$: $\mathrm{dp}[v][0][0] = 0$, $\mathrm{dp}[v][1][0] = c_v$,
$\mathrm{dp}[v][1][1] = c_v - d_v$. Merging child $u$ into $v$ is a standard
$O(|subtree_v| \cdot |subtree_u|)$ convolution, giving $O(n^2)$ total work by the
tree-merge argument. The key constraint: the $s=1$ state for $v$ allows each child
to independently choose $s \in \{0, 1\}$, while $s=0$ for $v$ forces $s=0$ for all
of $v$'s descendants (no coupon chain without the root coupon).

</details>

### Meet in the middle

- [Subset Sum](https://cses.fi/problemset/task/1641) (CSES) — $n \leq 40$
  integers, find a subset summing to target $t$ (up to $10^9$); the total can reach
  $4 \times 10^{10}$, far too large for any array-indexed DP, so MITM is the
  intended approach.

<details>
<summary>Solution sketch — Subset Sum (CSES 1641)</summary>

Split the $n \leq 40$ integers into two halves $A$, $B$ of $\leq 20$ each.
Enumerate all $2^{20} \approx 10^6$ subset sums of each half, yielding two lists of
(weight, value) pairs. Sort one list, compute its prefix maximum on values, and for
each sum $s_B$ in the $B$-list binary-search for the largest $s_A \leq t - s_B$ in
the $A$-list. Total time $O(2^{n/2} \cdot n)$, space $O(2^{n/2})$.

</details>

## See also

- [Dynamic programming]() — the general technique underlying knapsack DP
- [Meet-in-the-middle]() — exponential halving for large-$n$ or large-$W$ variants
- [Divide and conquer optimization]() — another DP speedup for certain
  knapsack-shaped recurrences with concave or convex structure
- [NP-complete]() — knapsack is the canonical pseudo-polynomial NP-complete problem
- [Frobenius coin problem]() — closely related: which sums are achievable at all
  with given coin denominations?
- [Linear programming]() — the fractional knapsack is the LP relaxation of 0/1

## External links

- [Knapsack problem (Wikipedia)](https://en.wikipedia.org/wiki/Knapsack_problem)
- [Knapsack problems (cp-algorithms)](https://cp-algorithms.com/dynamic_programming/knapsack.html)
- [Knapsack, Subset Sum and (max,+) Convolution — CF blog](https://codeforces.com/blog/entry/98663)
- [A new perspective on knapsack on trees — CF blog](https://codeforces.com/blog/entry/145340)
- [Optimized solution for Knapsack — CF blog](https://codeforces.com/blog/entry/59606)
- [Integral bounded knapsack — Petr Mitrichev](http://petr-mitrichev.blogspot.com/2011/07/integral-bounded-knapsack-problem.html)
- [Integer 0-1 bounded knapsack — dhruvbird](http://dhruvbird.blogspot.is/2011/09/integer-01-bounded-knapsack-problem.html)

[^1]: <http://petr-mitrichev.blogspot.com/2011/07/integral-bounded-knapsack-problem.html>
[^2]: <http://dhruvbird.blogspot.is/2011/09/integer-01-bounded-knapsack-problem.html>
[^3]: <http://codeforces.com/blog/entry/2257>
