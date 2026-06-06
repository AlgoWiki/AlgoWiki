---
categories: Algorithm techniques
---

The **sliding window technique** is a method for computing properties of every
contiguous subarray (or substring) of a sequence in $O(n)$ total time, by
maintaining a window $[l, r]$ whose two endpoints each advance only to the right.
Because every element enters the window exactly once (when $r$ passes it) and
leaves at most once (when $l$ passes it), the total number of window operations is
$O(n)$, replacing the $O(n^2)$ cost of recomputing the property from scratch for
each subarray. The technique applies whenever the window's validity condition is
*monotone*: if the window $[l, r]$ is invalid (e.g. it contains a duplicate, or
its sum exceeds a budget), then any wider window $[l, r+1]$ is also invalid,
so it is safe to advance $l$ to restore validity.

## Description

### Fixed-size window

When the window size is fixed at $k$, the window slides one step at a time: remove
the leftmost element, add the new rightmost element, and update the tracked
statistic in $O(1)$. For a running **sum** this is immediate; for a running
**maximum or minimum** the monotone deque technique is needed (see
[below](#sliding-window-maximum)).

~~~ {.cpp}
// Maximum sum of any k consecutive elements
int max_window_sum(vector<int>& a, int k) {
    int n = a.size(), s = 0;
    for (int i = 0; i < k; i++) s += a[i];
    int best = s;
    for (int i = k; i < n; i++) {
        s += a[i] - a[i - k];
        best = max(best, s);
    }
    return best;
}
~~~

### Variable-size (shrinkable) window

When there is no fixed window size but instead a *constraint* the window must
satisfy, use two pointers $l \leq r$ that both advance only to the right. The
right pointer $r$ expands the window one element at a time. Whenever the window
violates the constraint, advance $l$ (shrink from the left) until validity is
restored. Both pointers together traverse the array at most twice, giving $O(n)$
total time (plus the cost of maintaining whatever data structure tracks the window
contents).

The general template is:

~~~ {.cpp}
// Variable-size window: process all maximal valid windows
int l = 0;
// window_state tracks whatever is needed to check validity
for (int r = 0; r < n; r++) {
    // expand: include a[r] in the window
    window_state.add(a[r]);
    // shrink until window is valid again
    while (!is_valid(window_state)) {
        window_state.remove(a[l]);
        l++;
    }
    // window [l, r] is the largest valid window ending at r
    best = max(best, r - l + 1);
}
~~~

A concrete example — finding the **longest subarray with all distinct elements**
— tracks a frequency map and shrinks whenever a duplicate appears:

~~~ {.cpp}
// Longest subarray with all distinct elements
int longest_all_distinct(vector<int>& a) {
    int n = a.size(), best = 0, l = 0;
    unordered_map<int,int> freq;
    for (int r = 0; r < n; r++) {
        freq[a[r]]++;
        while (freq[a[r]] > 1) {
            if (--freq[a[l]] == 0) freq.erase(a[l]);
            l++;
        }
        best = max(best, r - l + 1);
    }
    return best;
}
~~~

### Sliding window maximum

The fixed-size window maximum — the maximum of every contiguous subarray of
length $k$ — cannot be handled by a simple running update because removing the
leftmost element might remove the current maximum, and finding the new maximum
naïvely costs $O(k)$. The $O(n)$ solution uses a **monotone deque** (double-ended
queue) of indices whose corresponding values are kept in *decreasing* order. This
invariant ensures that the front of the deque always holds the index of the
maximum element in the current window:

- When processing index $i$, pop from the **back** any indices $j$ with
  $a[j] \leq a[i]$: those can never be the maximum of any future window, since
  $i$ is further right and at least as large.
- Pop from the **front** any index that has left the window ($\leq i - k$).
- The front of the deque is the index of the current window's maximum.

~~~ {.cpp}
// Maximum of every window of exactly k consecutive elements: O(n)
vector<int> sliding_window_max(vector<int>& a, int k) {
    int n = a.size();
    vector<int> result;
    deque<int> dq;   // indices; a[dq.front()] is the window maximum
    for (int i = 0; i < n; i++) {
        // Remove index that has left the window
        while (!dq.empty() && dq.front() <= i - k)
            dq.pop_front();
        // Maintain decreasing values: remove dominated indices from back
        while (!dq.empty() && a[dq.back()] <= a[i])
            dq.pop_back();
        dq.push_back(i);
        if (i >= k - 1)
            result.push_back(a[dq.front()]);
    }
    return result;
}
~~~

The sliding window *minimum* is symmetric: maintain an increasing deque instead.
Both run in $O(n)$ because each index is pushed and popped at most once.

## Applications

### Subarray statistics under a constraint

The most direct use: given a constraint on the window (e.g. sum $\leq S$, at most
$k$ distinct elements, at most $k$ occurrences of any value), find the
longest or shortest window satisfying it, or count the number of valid windows.
The frequency-map pattern from the description handles character or value
constraints in $O(n)$ with a hash map (or $O(n \log n)$ with an ordered map when
exact order matters).

To count subarrays with **at most $k$ distinct values**, accumulate the window
length whenever the window is valid — every right endpoint of the valid window
contributes `r - l + 1` new valid subarrays:

~~~ {.cpp}
// Count subarrays with at most k distinct values: O(n) expected
long long count_at_most_k_distinct(vector<int>& a, int k) {
    int n = a.size(), l = 0; long long cnt = 0;
    unordered_map<int,int> freq;
    for (int r = 0; r < n; r++) {
        freq[a[r]]++;
        while ((int)freq.size() > k) {
            if (--freq[a[l]] == 0) freq.erase(a[l]);
            l++;
        }
        cnt += r - l + 1;
    }
    return cnt;
}
~~~

(To count subarrays with **exactly $k$** distinct values, use
$f(k) - f(k-1)$ where $f(k)$ is the count above.)

### Length-constrained subarray sum

To maximise the sum of a subarray whose length is between $\mathrm{lo}$ and
$\mathrm{hi}$, combine [prefix sums]() with a sliding window minimum. Write
$\mathrm{sum}(l, r) = P[r+1] - P[l]$ where $P$ is the prefix-sum array. For a
fixed right endpoint $r$, the valid left endpoints are
$l \in [\max(0, r - \mathrm{hi} + 1),\; r - \mathrm{lo} + 1]$, so we want the
minimum $P[l]$ in this sliding window — a standard deque query:

~~~ {.cpp}
// Maximum subarray sum with length in [lo, hi]: O(n)
long long max_subarray_constrained(vector<int>& arr, int lo, int hi) {
    int n = arr.size();
    vector<long long> P(n + 1, 0);
    for (int i = 0; i < n; i++) P[i + 1] = P[i] + arr[i];

    long long ans = LLONG_MIN / 2;
    deque<int> dq;   // indices l; front = index with minimum P[l]
    for (int r = lo - 1; r < n; r++) {
        int l_new = r - lo + 1;   // new valid left endpoint entering the window
        while (!dq.empty() && P[dq.back()] >= P[l_new])
            dq.pop_back();
        dq.push_back(l_new);
        while (!dq.empty() && dq.front() < r - hi + 1)
            dq.pop_front();
        if (!dq.empty())
            ans = max(ans, P[r + 1] - P[dq.front()]);
    }
    return ans;
}
~~~

### Two pointers on sorted sequences

When two sorted sequences must be matched — for example, assigning apartments to
applicants where applicant $i$ accepts any apartment within $\pm k$ of their
desired size — advance a pointer through each sequence, skipping elements that are
too small and matching when in range. Both pointers only ever move forward, giving
$O(n + m)$ time after the $O(n \log n + m \log m)$ sort.

~~~ {.cpp}
// Match applicants a[] to apartments b[] within tolerance k; return match count
// Both arrays are sorted in ascending order on entry.
int match_within_tolerance(vector<int> a, vector<int> b, int k) {
    sort(a.begin(), a.end());
    sort(b.begin(), b.end());
    int count = 0, j = 0;
    for (int i = 0; i < (int)a.size() && j < (int)b.size(); ) {
        if      (b[j] < a[i] - k) j++;          // apartment too small
        else if (b[j] > a[i] + k) i++;          // apartment too large
        else                       { count++; i++; j++; }  // matched
    }
    return count;
}
~~~

### DP with sliding window maximum

Sliding window maximum is not only a standalone technique but an ingredient in
several DP optimisations. The bounded knapsack $O(nW)$ deque solution
([Knapsack problem]()) and certain 1-D DP recurrences of the form
$\mathrm{dp}[r] = \max_{l \in [r-k, r-1]} \mathrm{dp}[l] + \mathrm{cost}(l, r)$
use exactly this idea. See [Dynamic programming optimization]() for further
applications.

## Variants

### Circular arrays

On a circular sequence of $n$ elements, a contiguous window can wrap around.
The standard trick is to double the array (or just work with indices modulo $n$)
and run the sliding window on the doubled version, restricting window size to at
most $n$. This is the approach used in problems like
[The Best Vacation](https://codeforces.com/problemset/problem/1358/D) (CF 1358D),
where months wrap across year boundaries.

### Sliding window median

A deque cannot directly maintain the median because finding the new median after
a removal is non-trivial. Instead, maintain two multisets (a max-heap for the
lower half and a min-heap for the upper half) and rebalance after each insertion
and deletion. Each step costs $O(\log k)$, giving $O(n \log k)$ overall — much
better than the $O(nk)$ naïve approach. See [CSES 1076 (Sliding Window Median)](https://cses.fi/problemset/task/1076)
for a classic instance. The sliding window cost problem
([CSES 1077](https://cses.fi/problemset/task/1077)) similarly requires the
sliding median as a subroutine.

### Rolling hash for strings

To find all occurrences of a pattern of length $k$ inside a text, or to compare
all length-$k$ substrings pairwise, maintain a polynomial [hash]() of the window.
Advancing the window one character costs $O(1)$: subtract the outgoing character's
contribution and add the incoming one. See [Hashing]() for the details and for
Rabin–Karp string matching.

### Mo's algorithm

[Mo's algorithm]() is an offline technique that processes range queries by sorting
them in a specific order so that the query window moves $O(n \sqrt{n})$ steps in
total. It extends the sliding window idea to arbitrary query ranges rather than a
contiguous left-to-right scan.

## Problems

### Fixed-size window and sliding window maximum

- [Sliding Window Median](https://cses.fi/problemset/task/1076) (CSES 1076) — for
  each window of $k$ elements, output the median; requires two balanced multisets
  rather than a simple deque.
- [Sliding Window Cost](https://cses.fi/problemset/task/1077) (CSES 1077) — for
  each window of $k$ elements, find the minimum cost to make all elements equal;
  uses the sliding median as a subroutine.

### Variable-size window — distinct elements

- [Unique Snowflakes](https://open.kattis.com/problems/snowflakes) (Kattis) — find
  the longest contiguous subarray with all distinct values; the canonical sliding
  window problem.

<details>
<summary>Solution sketch — Unique Snowflakes</summary>

Maintain a frequency map of elements in the window $[l, r]$. Expand $r$ one step
at a time. Whenever `freq[a[r]] > 1`, advance $l$ (shrinking the window) until the
duplicate is gone. At each $r$, record $r - l + 1$ as a candidate answer. Each
element is added to and removed from the map at most once, so total time is $O(n)$
expected with an unordered map.

</details>

- [Subarray Distinct Values](https://cses.fi/problemset/task/2428) (CSES 2428) —
  count the number of subarrays with at most $k$ distinct values.

<details>
<summary>Solution sketch — Subarray Distinct Values</summary>

Run the `count_at_most_k_distinct` pattern: maintain a frequency map of the window,
shrink from the left whenever the number of distinct values exceeds $k$, and for
each valid $r$ add $r - l + 1$ to the answer (every subarray ending at $r$ with
left endpoint $\geq l$ is valid). Total time $O(n)$ expected.

</details>

### Two pointers on sorted sequences

- [Apartments](https://cses.fi/problemset/task/1084) (CSES 1084) — match
  applicants to apartments within a tolerance; two pointers on two sorted arrays.

<details>
<summary>Solution sketch — Apartments</summary>

Sort both arrays. Use pointers $i$ (applicants) and $j$ (apartments). If
$b[j] < a[i] - k$, apartment $j$ is too small for anyone remaining, so advance
$j$. If $b[j] > a[i] + k$, apartment $j$ is too large for applicant $i$ (and
since $a$ is sorted, for all smaller applicants too), so advance $i$ without a
match. Otherwise the pair matches: count it and advance both. Total time after
sort: $O(n + m)$.

</details>

### Length-constrained maximum subarray sum

- [Maximum Subarray Sum II](https://cses.fi/problemset/task/1644) (CSES 1644) —
  find the maximum sum contiguous subarray with length in a given range $[a, b]$.

<details>
<summary>Solution sketch — Maximum Subarray Sum II</summary>

Build the prefix-sum array $P$. For each right endpoint $r$ (the last index of the
subarray), the valid left endpoints are $l \in [r - b, r - a]$ (0-indexed).
We want to minimise $P[l]$ over this window. As $r$ advances from $a-1$ to $n-1$,
the window for valid $l$ slides one step right, so a monotone deque (maintaining
indices of increasing $P[l]$ values, front = minimum) answers each query in $O(1)$.
Total time $O(n)$.

</details>

### Circular and harder variants

- [The Best Vacation](https://codeforces.com/problemset/problem/1358/D) (CF 1358D)
  — calendar with $n$ months arranged in a circle, each with $a_i$ days and total
  value proportional to days selected; pick at most $x$ consecutive days to
  maximise value. Double the month array and run a sliding window whose total day
  count stays $\leq x$.

<details>
<summary>Solution sketch — The Best Vacation</summary>

Each month $i$ has $a_i$ days and value $a_i \cdot (\text{accumulated day index})$
— but actually the value is the sum of day numbers within selected months. After
doubling the month sequence, build prefix sums of $a_i$ (day counts) and of the
value contributions. Use a sliding window on the doubled array: advance $r$, shrink
$l$ from the front while the day sum $\sum_{i=l}^{r} a_i > x$, and record the
maximum value in $[l, r]$ at each step. The window size is bounded by $n$ (can't
wrap more than once). Total time $O(n)$.

</details>

## See also

- [Hashing]() — rolling hash is the standard $O(n)$ technique for fixed-size string
  windows
- [Mo's algorithm]() — offline generalisation of sliding window to arbitrary query
  intervals
- [Mo's algorithm on trees]() — extension to tree paths
- [Knapsack problem]() — the bounded knapsack deque optimisation is sliding window
  maximum applied to DP
- [Dynamic programming optimization]() — several DP recurrences are sped up by
  maintaining a sliding window maximum or minimum
- [Binary search]() — sometimes combined with sliding window: binary-search on the
  window size, then check feasibility with a single scan

## External links

- [Sliding window technique (cp-algorithms)](https://cp-algorithms.com/sequences/sliding-windows.html)
- [Sliding window — USACO Guide](https://usaco.guide/gold/sliding-window)
- [CSES Sliding Window editorial (Codeforces blog)](https://codeforces.com/blog/entry/143960)
- [Sliding window maximum / minimum (Codeforces blog)](https://codeforces.com/blog/entry/71687)
