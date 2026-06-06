---
categories: Algorithm techniques
---

The **sliding window technique** maintains a contiguous range of an array or
string while its endpoints move only forward. It is the subarray version of
[Two pointers](): by updating the data kept for the current window when an
element enters or leaves, many brute-force $O(n^2)$ searches over all subarrays
become $O(n)$, or $O(n \log n)$ when the window needs an ordered
[data structure](Data structures).

The technique is useful when the answer depends on a contiguous segment and the
condition can be maintained incrementally: sums of non-negative numbers,
distinct elements, character counts, minimums, maximums, medians, and similar
quantities.

## Description

A window is an interval $[l, r]$ of indices, usually represented by two
inclusive endpoints or by a half-open interval $[l, r)$. The implementation
keeps a small state describing the elements currently inside the window. When
the right endpoint moves, one element is added to the state; when the left
endpoint moves, one element is removed.

The central invariant is:

- `add(i)` updates the state to include `a[i]`.
- `remove(i)` updates the state to exclude `a[i]`.
- `valid()` answers whether the current window satisfies the constraint.
- Both endpoints only increase.

Because each index is added at most once and removed at most once, the total
number of state updates is linear.

### Variable-size windows

The most common form grows the right endpoint and then shrinks the left endpoint
until the invariant is restored. For example, suppose all values are
non-negative and we want the longest subarray with sum at most `S`.

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

This works because the array values are non-negative: adding more elements can
only increase the sum, and removing elements from the left can only decrease it.
If negative values are allowed, the monotonicity disappears and a different tool
is usually needed, such as [Prefix sums](), a [balanced binary search tree](), or
[dynamic programming](Dynamic programming).

Counting all valid subarrays uses the same invariant. After the left endpoint
has been advanced until $[l, r]$ is valid, every suffix ending at `r` and
starting at an index in `[l, r]` is also valid for monotone constraints such as
"sum at most `S`" or "at most `K` distinct values". Therefore this step
contributes `r - l + 1` subarrays.

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

An "exactly `K`" distinct-elements count is often computed as
`at_most(K) - at_most(K - 1)`.

### Fixed-size windows

When the length is fixed, the left endpoint is determined by the right endpoint.
There is no inner `while` loop: add the new element, remove the element that just
fell out, and record the answer once the first full window is present.

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

Fixed windows are also the right model for stream-like tasks: after processing
position `r`, the maintained state describes the last `k` elements.

### Sliding minimum and maximum

For every fixed window minimum or maximum, recomputing the aggregate from
scratch costs $O(nk)$. A monotone [deque]() gives $O(n)$ time by storing only
candidate indices.

For window minimums, keep indices in increasing order of value. Before inserting
`r`, remove larger or equal values from the back: they can never become the
minimum while `a[r]` is still in the window. Remove indices that have fallen out
from the front. The front is then the minimum of the current window.

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

For maximums, reverse the comparison. This same idea appears inside optimizations
for [dynamic programming](Dynamic programming), where a transition asks for the
best value over a moving range.

### Complexity

If `add`, `remove`, and `valid` are $O(1)$, a sliding-window algorithm is
$O(n)$ time and $O(s)$ memory, where $s$ is the size of the maintained state
(for example, the number of distinct values in the current window). With an
ordered set, two multisets, or a [Fenwick tree](), each update is usually
$O(\log n)$, so the total time becomes $O(n \log n)$.

## Applications

- **Longest or shortest subarray under a monotone constraint.** Examples include
  "sum at most `S`" for non-negative arrays, "at most `K` distinct values", and
  "no repeated characters".
- **Counting subarrays.** Once a window ending at `r` is valid, all valid starts
  can often be counted in one step instead of enumerated.
- **Streaming statistics over the last `k` elements.** Sums, xor, frequency
  counts, minimums, maximums, medians, modes, and mex values are all fixed-window
  variants.
- **String matching and frequency constraints.** Maintain counts of characters
  in a substring and compare them with a target multiset, often together with
  [Hashing]() for faster equality checks.
- **Range-limited dynamic programming.** A monotone queue turns transitions of
  the form "minimum over the last `k` states" from $O(nk)$ into $O(n)$.

## Variants

### Two pointers versus sliding window

Sliding window is a special case of [Two pointers]() where the two pointers
bound a contiguous range and usually move in the same direction. Opposite-end
two-pointer algorithms, such as searching for a pair with a target sum in a
sorted array, are related but do not maintain a moving subarray.

The name matters less than the invariant: if the solution has a left boundary, a
right boundary, a state for the current segment, and both boundaries move only
forward, it is a sliding-window solution.

### Last occurrence instead of shrinking one step at a time

For uniqueness constraints, it is often cleaner to jump the left endpoint over
the previous occurrence of a duplicate. This avoids removing elements one by one
when the only state needed is the most recent position.

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

The `max` is important: an old occurrence before the current window must not
move `l` backwards.

### Ordered windows

Some fixed-window statistics cannot be updated with a single counter or deque.
For example, medians and costs to make all values equal need order statistics.
A standard solution keeps the lower half of the window in one multiset and the
upper half in another; the largest element of the lower half is the median.
Rebalancing after every insertion and deletion gives $O(\log k)$ per step.

For counts of values in a small or compressed domain, a [Fenwick tree]() or
[Segment tree]() can store frequencies and find the $k$-th element
by binary lifting. This is often faster than multisets after
[Coordinate compression]().

### Non-invertible operations

A simple window sum works because removing `a[l]` is easy. Some operations, such
as minimum, maximum, gcd, bitwise or, or bitwise and, do not have a direct
inverse. Common approaches are:

- Use a monotone deque for minimum or maximum.
- Keep bit counts for bitwise or or and over integer values.
- Use two stacks with aggregate values for a queue that supports amortized
  $O(1)$ aggregate queries.
- Use a [Segment tree]() or sparse table when the endpoints are not
  both moving forward.

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
<summary>Solution sketch - Books</summary>

All reading times are positive, so the window sum is monotone with respect to
the right endpoint. Add books while scanning `r`; whenever the sum exceeds `t`,
remove books from the left until it fits again. After the shrinking step,
`r - l + 1` is the best segment ending at `r`, and the maximum over all `r` is
the answer.

</details>

<details>
<summary>Solution sketch - Unique Snowflakes</summary>

Maintain a window with no duplicate snowflake ids. Either keep a frequency map
and shrink while the new id has frequency greater than one, or store each id's
last position and jump `l` to one past that position. The maximum window length
seen during the scan is the answer.

</details>

### Counting by valid starts

- [Distinct Values Subarrays II](https://cses.fi/problemset/task/2428/) (CSES):
  count subarrays with at most `k` distinct values.
- [Sliding Window Distinct Values](https://cses.fi/problemset/task/3222/) (CSES):
  report the number of distinct values in every fixed-size window.

<details>
<summary>Solution sketch - Distinct Values Subarrays II</summary>

Use a frequency map and keep the current window at most `k` distinct values.
After adding `a[r]`, shrink `l` until the condition holds. Then every subarray
ending at `r` and starting at one of `l, l+1, ..., r` is valid, so add
`r - l + 1` to the answer.

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
<summary>Solution sketch - Sliding Window Minimum</summary>

Keep a deque of candidate indices whose values are increasing. Before pushing a
new index, pop all candidates with value at least the new value; they are worse
and expire no later. Pop expired indices from the front. Once the first full
window exists, the value at the front is the minimum.

</details>

<details>
<summary>Solution sketch - Sliding Window Cost</summary>

For a fixed window, the optimal target value is a median. Keep the lower half
and upper half in two multisets, together with their sums. Rebalance so the
lower half contains the median. The cost is
`median * lower_size - lower_sum + upper_sum - median * upper_size`; update the
two multisets when the window slides.

</details>

## See also

- [Two pointers]() - the broader endpoint-moving technique.
- [Prefix sums]() - often replaces sliding windows when values can be negative.
- [Deque]() - the standard structure for monotone minimum and maximum queues.
- [Segment tree]() - handles window statistics when endpoints are not monotone or
  when richer range queries are needed.
- [Coordinate compression]() - useful before maintaining window frequencies in a
  Fenwick tree or segment tree.
- [Mo's algorithm]() - reorders offline range queries so endpoints move slowly
  instead of only forward.

## External links

- [USACO Guide: Two Pointers](https://usaco.guide/silver/two-pointers)
- [USACO Guide: Sliding Window](https://usaco.guide/gold/sliding-window)
- [Codeforces EDU: Two Pointers Method](https://codeforces.com/blog/entry/87248)
- [Competitive Programmer's Handbook, section 8.1: Two Pointers](https://cses.fi/book/book.pdf)
- [Sliding Window Technique in Data Structures](https://techiedelight.quora.com/Sliding-Window-Technique-in-Data-Structures)
