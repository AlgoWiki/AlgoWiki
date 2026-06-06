---
categories: Algorithm techniques
---

Imagine you want to find the longest contiguous subarray of an array whose sum
is at most some limit $S$. The brute-force approach tries every pair of
endpoints — $O(n^2)$ pairs — and recomputes the sum each time, for a total of
$O(n^3)$ or $O(n^2)$ work. The **sliding window technique** cuts this to $O(n)$
by keeping track of a "window" $[l, r]$ and moving its two endpoints forward
only, never backward. When you slide the right end in, one new element enters
the window; when you slide the left end out, one element leaves. Because each
element enters and leaves at most once, the total work is linear.

The technique applies whenever the answer depends on a contiguous segment of the
input and you can update whatever you track about that segment cheaply as it
grows or shrinks from one end. Common examples: sum, distinct element count,
character frequencies, minimum, maximum, and median.

## Description

Think of the window as a view that slides across the array from left to right.
At any moment it covers the range of indices from `l` (inclusive) to `r`
(inclusive). You maintain some state — a running sum, a frequency map, a
monotone deque — that describes exactly the elements inside `[l, r]`. As `r`
advances, a new element enters from the right; as `l` advances, an element
leaves from the left.

The reason this is efficient is simple: each index is added to the window
exactly once and removed at most once, so the total number of state updates
across the entire pass is at most $2n$.

### Variable-size windows

The most common pattern lets the window grow until it breaks a constraint, then
shrinks from the left until the constraint is restored:

1. Advance `r` and add `a[r]` to the state.
2. While the window is invalid, advance `l` and remove `a[l]` from the state.
3. Now `[l, r]` is the longest valid window ending at `r`. Record it.

This works when the constraint is **monotone**: making the window larger can
only make it harder to satisfy, and making it smaller can only help. For
example, if all values are non-negative, the window sum only goes up as `r`
increases, and only goes down as `l` increases — so the shrinking step always
eventually restores validity.

~~~ {.cpp}
int longest_sum_at_most(const vector<int>& a, long long S) {
    int n = (int)a.size();
    int best = 0;
    int l = 0;
    long long sum = 0;

    for (int r = 0; r < n; r++) {
        sum += a[r];
        while (sum > S) {
            sum -= a[l];
            l++;
        }
        best = max(best, r - l + 1);
    }
    return best;
}
~~~

If the values can be negative, the sum no longer behaves monotonically —
shrinking the window might increase the sum — and a different tool is needed,
such as [Prefix sums]() or [Dynamic programming]().

**Counting instead of maximizing.** Sometimes you want to count every valid
subarray, not just the longest. Once the left endpoint has been pushed to the
rightmost valid position for a given `r`, every starting point in `[l, r]` also
gives a valid subarray ending at `r`. So instead of recording the length, add
`r - l + 1` to the answer.

Here is that idea applied to counting subarrays with at most `K` distinct
values:

~~~ {.cpp}
long long count_at_most_k_distinct(const vector<int>& a, int K) {
    unordered_map<int, int> freq;
    long long ans = 0;
    int l = 0, distinct = 0;

    for (int r = 0; r < (int)a.size(); r++) {
        if (freq[a[r]]++ == 0) distinct++;

        while (distinct > K) {
            if (--freq[a[l]] == 0) {
                freq.erase(a[l]);
                distinct--;
            }
            l++;
        }

        ans += r - l + 1;
    }
    return ans;
}
~~~

An "exactly `K` distinct values" count follows from `at_most(K) - at_most(K - 1)`.

### Fixed-size windows

When the window length is fixed at `k`, the left endpoint is always `r - k + 1`,
so there is no shrinking loop — just add the incoming element on the right and
remove the outgoing element on the left:

~~~ {.cpp}
vector<long long> fixed_window_sums(const vector<int>& a, int k) {
    vector<long long> ans;
    long long sum = 0;

    for (int r = 0; r < (int)a.size(); r++) {
        sum += a[r];
        if (r >= k) sum -= a[r - k];
        if (r + 1 >= k) ans.push_back(sum);
    }
    return ans;
}
~~~

Fixed windows also model streaming problems: after processing index `r`, the
state represents the last `k` elements seen.

### Sliding minimum and maximum

Finding the minimum (or maximum) of every window of size `k` seems to require
$O(k)$ work per window, giving $O(nk)$ total. A **monotone deque** reduces this
to $O(n)$ by keeping only the indices that could still be the minimum of some
future window.

The key observation: if `a[j] >= a[r]` and `j < r`, then `a[j]` can never be
the minimum of any window that contains `a[r]`, because `a[r]` is at least as
small and will outlast `a[j]` in the window. So we can discard `a[j]`
immediately when `a[r]` arrives.

The deque therefore stores indices in order, with values increasing from front
to back. The front is always the minimum of the current window.

~~~ {.cpp}
vector<int> sliding_minimum(const vector<int>& a, int k) {
    deque<int> dq;
    vector<int> ans;

    for (int r = 0; r < (int)a.size(); r++) {
        while (!dq.empty() && a[dq.back()] >= a[r]) dq.pop_back();
        dq.push_back(r);

        if (dq.front() <= r - k) dq.pop_front();
        if (r + 1 >= k) ans.push_back(a[dq.front()]);
    }
    return ans;
}
~~~

For maximums, flip the comparison to `<=`. The same deque idea appears in
[dynamic programming](Dynamic programming) optimizations where a transition
needs the best value over a range of previous states.

### Complexity

Each element is added to the window once and removed at most once, so:

- If each add and remove operation costs $O(1)$ (a running sum, a counter, a
  deque), the total time is $O(n)$.
- If each update costs $O(\log n)$ (an ordered set, a [Fenwick tree](), two
  multisets), the total time is $O(n \log n)$.
- Memory is $O(s)$ where $s$ is the size of the state, for example the number
  of distinct values currently in the window.

## Applications

- **Longest or shortest subarray under a constraint.** Any monotone constraint
  ("sum ≤ S for non-negative values", "at most K distinct elements", "no
  repeated characters") can be handled with a variable-size window.
- **Counting valid subarrays.** Once the window is maximally shrunk for a given
  right endpoint, all starting points from `l` to `r` yield valid subarrays,
  so you can count them in $O(1)$ rather than enumerating them.
- **Running statistics over the last `k` elements.** Sums, XOR, frequencies,
  minimums, maximums, medians, and modes are all computable with a fixed-size
  window and the right supporting structure.
- **String matching and anagram detection.** Keep a character-frequency array
  for the current window and compare it to the target's frequencies. Two arrays
  of 26 entries can be compared in $O(1)$ by maintaining a count of matching
  positions.
- **DP transitions over a bounded range.** When a recurrence looks up the best
  of the previous $k$ states, a monotone deque turns the $O(k)$ lookup per
  state into $O(1)$.

## Variants

### Two pointers vs. sliding window

Sliding window is a specialization of [Two pointers]() where both pointers
move in the same direction and bound a contiguous range. Two-pointer problems
where the pointers start at opposite ends of the array (for example, finding a
pair that sums to a target in a sorted array) are related, but they do not
maintain a moving subarray and are not usually called sliding windows.

If your solution has a left index, a right index, a state for everything between
them, and both indices only ever increase, it is a sliding-window solution.

### Jumping the left endpoint directly

For uniqueness constraints, tracking the most recent position of each element
lets you jump `l` in one step instead of advancing it one position at a time.
This is useful when the only reason to move `l` forward is that a duplicate
appeared:

~~~ {.cpp}
int longest_all_distinct(const vector<int>& a) {
    unordered_map<int, int> last;
    int l = 0, best = 0;

    for (int r = 0; r < (int)a.size(); r++) {
        auto it = last.find(a[r]);
        if (it != last.end()) l = max(l, it->second + 1);
        last[a[r]] = r;
        best = max(best, r - l + 1);
    }
    return best;
}
~~~

The `max` guard is important: if an earlier duplicate occurred before the
current window, its stored position must not push `l` backward.

### Ordered windows

Some statistics cannot be maintained with a simple counter or a deque. Medians
and the minimum total cost to make all window elements equal both require
knowing the order of elements inside the window.

A classic approach keeps the lower half of the window in a `multiset` (or
max-heap) and the upper half in another `multiset` (or min-heap), and rebalances
after each insertion and deletion. The boundary between the two halves is the
median. Each slide operation is $O(\log k)$.

For problems over a compressed or small domain, a [Fenwick tree]() or
[Segment tree]() storing frequencies can answer "$k$-th smallest" queries by
binary search, often faster in practice than multisets after
[Coordinate compression]().

### Non-invertible operations

A window sum is easy to update because subtraction is the inverse of addition.
But some aggregates — minimum, maximum, GCD, bitwise OR, bitwise AND — have no
inverse. You cannot un-apply them when an element leaves the window. Options:

- **Monotone deque** for minimum or maximum (see above).
- **Bit counts** for bitwise OR and AND over integer values: maintain a count
  of how many window elements have each bit set; OR is nonzero iff the count
  is nonzero, AND is set iff the count equals the window size.
- **Two-stacks queue** for any associative aggregate with $O(1)$ amortized
  cost per operation (described below).
- **[Segment tree]()** for arbitrary aggregates, or when the window endpoints
  are not both moving monotonically forward.

The **two-stacks queue** idea: split the conceptual queue into two stacks. New
elements are pushed onto the back stack; each stack also stores the running
aggregate of its elements. When the front stack is empty and a pop is needed,
reverse the back stack into the front stack — each element then carries the
aggregate of everything at or above it. Because every element crosses the
boundary at most once, the amortized cost per push/pop/query is $O(1)$.

~~~ {.cpp}
template<typename T>
struct MinQueue {
    vector<pair<T,T>> front_st, back_st;  // (value, running_min)

    void push(T x) {
        T m = back_st.empty() ? x : min(x, back_st.back().second);
        back_st.push_back({x, m});
    }

    void pop() {
        if (front_st.empty()) {
            while (!back_st.empty()) {
                T x = back_st.back().first;
                back_st.pop_back();
                T m = front_st.empty() ? x : min(x, front_st.back().second);
                front_st.push_back({x, m});
            }
        }
        front_st.pop_back();
    }

    T query() const {
        T res = numeric_limits<T>::max();
        if (!front_st.empty()) res = min(res, front_st.back().second);
        if (!back_st.empty()) res = min(res, back_st.back().second);
        return res;
    }
};
~~~

Replace `min` with `max`, `gcd`, `|`, or `&` to handle other non-invertible
aggregates.

### DP transitions over a sliding range

[Dynamic programming](Dynamic programming) transitions of the form

$$
dp[i] = \min_{j \in [i-k,\, i-1]} dp[j] + \text{cost}(i)
$$

appear in problems like minimum-cost jumping, shortest paths on a DAG with
bounded edge lengths, and various knapsack variants. A naive implementation
scans all $k$ predecessors for each $i$, giving $O(nk)$ total. A monotone
deque reduces this to $O(n)$.

The deque stores candidate predecessor indices in increasing order of their
$dp$ value. When computing $dp[i]$:
1. Remove indices from the front that are now outside the window ($< i - k$).
2. The front holds the index with the smallest $dp$ value — use it.
3. Remove indices from the back with $dp$ value $\geq dp[i]$ (they will never
   be chosen over $i$ by any future state), then push $i$.

~~~ {.cpp}
// dp[i] = min cost to reach position i; can jump from any j in [i-k, i-1]
long long min_cost_jump(const vector<int>& cost, int k) {
    int n = cost.size();
    vector<long long> dp(n, LLONG_MAX / 2);
    dp[0] = cost[0];
    deque<int> dq;  // indices, dp[front] is minimum
    dq.push_back(0);

    for (int i = 1; i < n; i++) {
        while (!dq.empty() && dq.front() < i - k) dq.pop_front();
        dp[i] = dp[dq.front()] + cost[i];
        while (!dq.empty() && dp[dq.back()] >= dp[i]) dq.pop_back();
        dq.push_back(i);
    }
    return dp[n - 1];
}
~~~

## Problems

### Basic variable windows

- [Books](https://codeforces.com/problemset/problem/279/B) (Codeforces): longest
  contiguous sequence with total reading time at most `t`.
- [Subarray Sums I](https://cses.fi/problemset/task/1660/) (CSES): count
  positive-sum subarrays equal to a target.
- [Playlist](https://cses.fi/problemset/task/1141/) (CSES): longest subarray
  with all distinct values.
- [Unique Snowflakes](https://open.kattis.com/problems/snowflakes) (Kattis):
  longest contiguous package with no repeated snowflake id.

<details>
<summary>Solution sketch — Books</summary>

All reading times are positive, so the window sum only grows as `r` advances.
Scan `r` left to right, adding each book to the sum. Whenever the sum exceeds
`t`, remove books from the left until it fits again. After that shrinking step,
`r - l + 1` is the longest valid window ending at `r`, and the answer is the
maximum over all `r`.

</details>

<details>
<summary>Solution sketch — Unique Snowflakes</summary>

Keep a frequency map of snowflake ids in the current window. When `a[r]`
appears for the second time, advance `l` until it is gone. Alternatively, store
each id's most recent index and jump `l` directly to one past that position —
no need to remove elements one by one. The answer is the maximum window length
seen.

</details>

### Counting by valid starts

- [Distinct Values Subarrays II](https://cses.fi/problemset/task/2428/) (CSES):
  count subarrays with at most `k` distinct values.
- [Sliding Window Distinct Values](https://cses.fi/problemset/task/3222/) (CSES):
  report the number of distinct values in every fixed-size window.

<details>
<summary>Solution sketch — Distinct Values Subarrays II</summary>

Use a frequency map. For each `r`, shrink `l` until the window has at most `k`
distinct values. Every starting index in `[l, r]` then gives a valid subarray
ending at `r`, so add `r - l + 1` to the answer.

</details>

### Monotone queues and ordered windows

- [Sliding Window Minimum](https://cses.fi/problemset/task/3221/) (CSES): xor
  all fixed-window minimums, with generated input large enough to require
  linear time.
- [Sliding Window Median](https://cses.fi/problemset/task/1076/) (CSES):
  maintain a median for each fixed-size window.
- [Sliding Window Cost](https://cses.fi/problemset/task/1077/) (CSES): maintain
  the sum of distances to the median in each fixed-size window.
- [Sliding Window Mode](https://cses.fi/problemset/task/3224/) (CSES): maintain
  the smallest most-frequent value in each fixed-size window.

<details>
<summary>Solution sketch — Sliding Window Minimum</summary>

Use the monotone deque described above. For each new index `r`, pop all
candidates from the back with value ≥ `a[r]` (they are dominated), then push
`r`. Pop expired candidates (index ≤ `r - k`) from the front. The front holds
the minimum.

</details>

<details>
<summary>Solution sketch — Sliding Window Cost</summary>

The cheapest target to move all window elements toward is the median. Keep the
lower half in a `multiset` (largest element = median) and the upper half in
another `multiset`. Maintain the sum of each half. After each slide, rebalance
the two halves so they differ in size by at most one. The cost is then
`median * lower_size - lower_sum + upper_sum - median * upper_size`.

</details>

### DP with sliding-range transitions

- [Subarray Sums II](https://cses.fi/problemset/task/1661/) (CSES): count
  subarrays with a given sum (values can be negative — prefix sums plus a map,
  but illuminates the boundary between sliding windows and prefix techniques).
- [Dice Combinations](https://cses.fi/problemset/task/1633/) (CSES): count ways
  to form sum $n$ using dice faces 1–6; the transition sums the last 6 DP states.
- [Elevator Rides](https://cses.fi/problemset/task/1653/) (CSES, harder): bitmask
  DP where the knapsack transition over a fixed-size window applies.
- [Frog Jump](https://cses.fi/problemset/task/1628/) (CSES): minimum-cost jump
  with bounded jump length, the direct template for the deque DP above.
- [Deque DP](https://codeforces.com/problemset/problem/436/E) (Codeforces): DP
  optimized with a sliding minimum over a deque.

<details>
<summary>Solution sketch — Dice Combinations</summary>

Let $dp[i]$ be the number of ways to form sum $i$. The recurrence is

$$
dp[i] = \sum_{j=1}^{6} dp[i - j]
$$

This is the sum of a fixed window of width 6 in the DP array. Maintain a
running sum of `dp[i-1]` through `dp[i-6]`, adding `dp[i]` and subtracting
`dp[i-7]` at each step. Total time $O(n)$, no deque required since addition is
invertible.

</details>

<details>
<summary>Solution sketch — Frog Jump</summary>

Let $dp[i]$ be the minimum cost to reach stone $i$. The transition is

$$
dp[i] = \min_{j \in [\max(0,\, i-k),\, i-1]} dp[j] + |h[i] - h[j]|
$$

When the cost depends on $j$ (here through $|h[i] - h[j]|$), the standard
deque template does not directly apply because the objective is not separable.
One approach: split on whether $h[j] \le h[i]$ or $h[j] > h[i]$ and use a
separate deque tracking $dp[j] - h[j]$ or $dp[j] + h[j]$ for each case. For
simpler variants where the cost is independent of $j$, the standard monotone
deque suffices directly.

</details>

## See also

- [Two pointers]() — the broader endpoint-moving technique; sliding window is a special case.
- [Prefix sums]() — useful when values can be negative and a window shrink might increase the sum.
- [Deque]() — the standard data structure behind monotone minimum and maximum queues.
- [Segment tree]() — handles window statistics when endpoints are not monotone or when richer range queries are needed.
- [Coordinate compression]() — useful before maintaining window frequencies in a Fenwick tree or segment tree.
- [Mo's algorithm]() — reorders offline range queries so endpoints move slowly instead of only forward.

## External links

- [USACO Guide: Two Pointers](https://usaco.guide/silver/two-pointers)
- [USACO Guide: Sliding Window](https://usaco.guide/gold/sliding-window)
- [Codeforces EDU: Two Pointers Method](https://codeforces.com/blog/entry/87248)
- [Competitive Programmer's Handbook, section 8.1: Two Pointers](https://cses.fi/book/book.pdf)
- [Sliding Window Technique in Data Structures](https://techiedelight.quora.com/Sliding-Window-Technique-in-Data-Structures)
