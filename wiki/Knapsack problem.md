---
categories: Dynamic programming, Algorithm techniques
---

The **knapsack problem** is a family of combinatorial optimisation problems: given a
set of items each with a *weight* and a *value*, and a knapsack of fixed capacity,
choose items to maximise total value without exceeding capacity. Despite being
[NP-complete]() in general, all standard variants admit [dynamic programming]()
solutions running in $O(nW)$ time — pseudo-polynomial in the capacity $W$ — which
is fast enough for the $n \leq 10^3$, $W \leq 10^6$ constraints typical of contests.
The knapsack problem is a cornerstone of competitive programming DP and a template
for a wide class of resource-allocation problems.

## Description

### 0/1 knapsack

$n$ items, item $i$ with integer weight $w_i > 0$ and value $v_i \geq 0$, knapsack
capacity $W$. Each item may be taken at most once. Maximise total value subject to
total weight $\leq W$.

Define $\mathrm{dp}[i][c]$ = max value using a subset of the first $i$ items with
total weight $\leq c$. The recurrence is

$$
\mathrm{dp}[i][c] = \max\!\bigl(\mathrm{dp}[i{-}1][c],\; \mathrm{dp}[i{-}1][c - w_i] + v_i\bigr),
$$

where the second option requires $c \geq w_i$. Rolling the $i$ dimension into a 1-D
array and scanning capacity **right to left** ensures each item is used at most once
— reading $\mathrm{dp}[c - w_i]$ before it is overwritten guarantees we see the
pre-$i$ value:

~~~ {.cpp}
// 0/1 knapsack: n items with weights w[], values v[], capacity W
// dp[c] = max value with total weight <= c
vector<int> dp(W + 1, 0);
for (int i = 0; i < n; i++)
    for (int c = W; c >= w[i]; c--)    // right-to-left: each item at most once
        dp[c] = max(dp[c], dp[c - w[i]] + v[i]);
// answer: dp[W]
~~~

### Complexity

Time $O(nW)$, space $O(W)$. This is *pseudo-polynomial*: the input has
$O(n \log W)$ bits, so $O(nW)$ is exponential in input length — consistent with
NP-completeness. Practical contest constraints keep the product small.

## Applications

- **[Subset-sum]()** — deciding whether a subset sums to exactly $T$ is 0/1
  knapsack with all values equal to 1 (or a boolean DP), and is the canonical
  NP-complete decision problem.
- **Coin change** — making a target amount from denominations (each usable
  many times) is [unbounded knapsack](#unbounded-knapsack); minimising coin count
  swaps max for min.
- **Budget-constrained scheduling** — select tasks or projects under a cost
  budget to maximise profit.
- **LP relaxation** — the [fractional knapsack](#fractional-knapsack) variant is
  the [linear programming]() relaxation of the 0/1 problem, and its greedy optimum
  is an upper bound used in branch-and-bound solvers.

## Variants

### Unbounded knapsack

Each item may be used **any number of times**. Scan capacity **left to right** so
that an item can be reselected in the same pass:

~~~ {.cpp}
// Unbounded knapsack: each item usable unlimited times
vector<int> dp(W + 1, 0);
for (int i = 0; i < n; i++)
    for (int c = w[i]; c <= W; c++)    // left-to-right: reuse allowed
        dp[c] = max(dp[c], dp[c - w[i]] + v[i]);
~~~

Coin change — minimum coins to reach amount $T$ — replaces max with min:

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

Replace max with addition to count the number of subsets achieving each total
weight. For **feasibility** ("is weight $T$ achievable?"), the DP is boolean and
each update is a logical OR. When all weights are integers, a `bitset` vectorises
this to $O(T/64)$ per item:

~~~ {.cpp}
// Subset-sum feasibility: O(n * T / 64)
// reachable[c] = 1 iff some subset has total weight exactly c
bitset<MAXW> reachable;
reachable[0] = 1;
for (int i = 0; i < n; i++)
    reachable |= reachable << w[i];
// reachable[T] = 1 iff target T is achievable
~~~

### Fractional knapsack

Items may be **split**: taking fraction $x_i \in [0,1]$ of item $i$ contributes
weight $x_i w_i$ and value $x_i v_i$. Solved greedily in $O(n \log n)$: sort items
by value-density $v_i / w_i$ descending, fill greedily, split the last item to
exactly fill remaining capacity. The fractional solution is also the LP relaxation
of the integer 0/1 problem.

### Bounded knapsack

Item $i$ has $c_i$ available copies. Expanding to $c_i$ identical 0/1 items costs
$O((\sum c_i) W)$.

**Binary grouping** reduces this to $O(nW \log C)$ where $C = \max c_i$: split the
$c_i$ copies into virtual 0/1 items of sizes $1, 2, 4, \ldots$ (powers of two) plus
a remainder. Any count from $0$ to $c_i$ is representable as a subset of these
groups, so running the standard 0/1 knapsack on the virtual items is exact.

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

**Monotone deque** achieves $O(nW)$: for each item type $i$, partition capacities
by residue mod $w_i$. Within each residue class the DP values form a sequence whose
update is a sliding-window maximum over a window of $c_i + 1$ entries, solvable
with a deque in amortised $O(1)$ per entry.

### 2D (multi-dimensional) knapsack

Items have two weight dimensions (e.g. weight and volume) with separate
capacities $W$ and $V$. Extend the table to $\mathrm{dp}[c_1][c_2]$ and scan both
dimensions right to left:

~~~ {.cpp}
// 2D knapsack: items with weights w[], volumes vol[], values v[]
vector<vector<int>> dp(W + 1, vector<int>(V + 1, 0));
for (int i = 0; i < n; i++)
    for (int c1 = W; c1 >= w[i]; c1--)
        for (int c2 = V; c2 >= vol[i]; c2--)
            dp[c1][c2] = max(dp[c1][c2], dp[c1 - w[i]][c2 - vol[i]] + v[i]);
~~~

Both dimensions must be small; the table is $O(WV)$.

### Group knapsack

Items are partitioned into groups $G_1, \ldots, G_k$; at most one item may be
selected from each group. Process groups one at a time; scan capacity right to left
(exactly as in 0/1 knapsack) and, for each capacity, try every item in the current
group. The right-to-left scan means the old DP values — from before the group is
processed — are always used, so at most one item per group is ever chosen:

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

This models "buy at most one product from each of several shops" and similar
partitioned-selection problems.

### Tree (dependency) knapsack

If selecting item $v$ requires that its parent in a rooted tree is also selected,
the problem becomes *tree knapsack* (or *dependent knapsack*). Use a post-order DFS,
maintaining $\mathrm{dp}[v][c]$ = max value in $v$'s subtree using capacity $c$,
where $\mathrm{dp}[v][c] = 0$ for $c < w_v$ (cannot afford $v$, so nothing is
selected) and $\mathrm{dp}[v][c] \geq v_v$ for $c \geq w_v$. Merging child
subtrees one at a time is a knapsack-of-knapsacks:

~~~ {.cpp}
// Tree knapsack: dp[v][c] = max value in subtree(v) using capacity c,
//                where selecting any descendant requires selecting v.
// Initialise all dp[v] to zero-vectors before calling dfs(root, -1).
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
        // merge subtree u; right-to-left ensures u's capacity is drawn from
        // the budget left after paying w[v], never allowing u without v
        for (int c = W; c >= w[v]; c--)
            for (int k = 1; k <= c - w[v]; k++)
                dp[v][c] = max(dp[v][c], dp[v][c - k] + dp[u][k]);
    }
}
// answer: dp[root][W]
~~~

The naive complexity is $O(n^2 W)$ per root call, but bounding each subtree's DP
array to $\min(\text{subtree weight}, W) + 1$ entries reduces total merge work to
$O(nW)$: each pair of nodes contributes to the merge at their lowest common
ancestor exactly once, and the total cells merged is bounded by $O(nW)$.

An elegant $O(nW)$ alternative flattens the tree with a DFS-order Euler tour, then
runs a 1-D DP where "take item $v$" advances one step into $v$'s subtree and "skip
$v$" jumps over the entire subtree — see the external links.

### Meet in the middle

When $n \leq 40$ but $W$ is too large for $O(nW)$ DP (or items have real-valued
weights), [meet in the middle]() halves the exponent. Split items into two halves
$A$, $B$ of $\sim n/2$ each, enumerate all $2^{n/2} \approx 10^6$ subsets of each
half as (weight, value) pairs, then for each $B$-subset find the best matching
$A$-subset via sort + binary search:

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
        // largest A-entry with weight <= W - wb
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

Let $S$ be the total weight. We want a subset with total as close to $S/2$ as
possible. Run the 0/1 boolean knapsack: $\mathrm{reachable}[c] = \text{true}$ if
some subset sums to exactly $c$. Scan from $\lfloor S/2 \rfloor$ downward to find
the largest achievable $c^*$. The minimum difference is $S - 2c^*$.

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

Counting unordered multisets requires avoiding permutation duplicates. The fix: outer
loop iterates coin types, inner loop amounts left-to-right, updating
$\mathrm{dp}[c] \mathrel{+}= \mathrm{dp}[c - w_i]$. Each dp value accumulates
combinations whose largest coin index is the current one, ensuring each multiset is
counted exactly once.

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
  types is small, making binary grouping efficient.

<details>
<summary>Solution sketch — Lucky Country</summary>

Group items by their label value: each distinct value $v$ appears $c_v$ times,
giving a bounded knapsack instance. Since the number of distinct values is at most
$O(\sqrt{W})$ (otherwise some value exceeds the capacity), apply binary grouping on
each group and run the resulting 0/1 knapsack in $O(D W \log C)$ where $D$ is the
number of distinct values.

</details>

### Group knapsack

- [Tug of War](https://boi.cses.fi/files/boi2015_day2.pdf) (BOI 2015 Day 2) —
  partition players into two teams of almost-equal size and weight; feasibility is
  checked with a 2D DP (count and total weight simultaneously), naturally a
  group-style knapsack.

### Tree (dependency) knapsack

- [Karen and Supermarket](https://codeforces.com/problemset/problem/815/C)
  (CF 815C) — $n$ goods with a tree of coupon dependencies; select at most $m$ goods
  at minimum cost, choosing whether to apply each coupon (requires parent's coupon).

<details>
<summary>Solution sketch — Karen and Supermarket</summary>

Root the coupon tree at node 1. Define
$\mathrm{dp}[v][j][s]$ = minimum cost to purchase exactly $j$ goods from
$v$'s subtree, where $s = 0$ means coupon $v$ is not used and $s = 1$ means it is.
Base (leaf $v$): $\mathrm{dp}[v][0][0] = 0$, $\mathrm{dp}[v][1][0] = c_v$,
$\mathrm{dp}[v][1][1] = c_v - d_v$. Merging child $u$ into $v$ is a standard
$O(|subtree_v| \cdot |subtree_u|)$ knapsack merge, giving $O(n^2)$ total by the
tree-merge argument. Since coupons can only be used when the parent's coupon is
used, the $s=1$ state for $v$ enables $s \in \{0,1\}$ for each child, while $s=0$
forces $s=0$ for all descendants.

</details>

### Meet in the middle

- [Subset Sum](https://cses.fi/problemset/task/1641) (CSES) — $n \leq 40$
  integers, find a subset summing to target $t$ (up to $10^9$); values are far too
  large for $O(nW)$ DP; MITM is the intended approach.

<details>
<summary>Solution sketch — Subset Sum (CSES 1641)</summary>

With $n \leq 40$ and values up to $10^9$, the target can reach $4 \times 10^{10}$,
making any array-indexed DP infeasible. Split items into halves $A$, $B$ of $\leq 20$
each. Enumerate all $2^{20}$ subset sums for each half, sort one list, and for each
sum $s_B$ in the $B$-list binary-search for $t - s_B$ in the $A$-list. Total time
$O(2^{n/2} \cdot n)$, space $O(2^{n/2})$.

</details>

## See also

- [Dynamic programming]() — the general technique underlying knapsack DP
- [Meet-in-the-middle]() — exponential halving for large-$n$ variants
- [Divide and conquer optimization]() — another DP speedup for knapsack-shaped
  recurrences with concave/convex structure
- [NP-complete]() — knapsack is the canonical pseudo-polynomial NP-complete problem
- [Frobenius coin problem]() — related: which sums are achievable at all with given
  coin denominations?
- [Linear programming]() — the fractional knapsack is the LP relaxation of 0/1

## External links

- [Knapsack problem (Wikipedia)](https://en.wikipedia.org/wiki/Knapsack_problem)
- [Knapsack problems (cp-algorithms)](https://cp-algorithms.com/dynamic_programming/knapsack.html)
- [Knapsack, Subset Sum and (max,+) Convolution — CF blog](https://codeforces.com/blog/entry/98663)
- [A new perspective on knapsack on trees — CF blog](https://codeforces.com/blog/entry/145340)
- [Optimized solution for Knapsack — CF blog](https://codeforces.com/blog/entry/59606)
- [Integral bounded knapsack (Petr Mitrichev)](http://petr-mitrichev.blogspot.com/2011/07/integral-bounded-knapsack-problem.html)
