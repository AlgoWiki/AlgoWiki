---
categories: Algorithm techniques, Data structures, String data structures
---

**Hashing** is the technique of mapping a complex object — a string, a set, a
game board, a tree — to a small integer so that equality checks become $O(1)$.
In competitive programming it appears in at least four distinct flavours:
[polynomial string hashing](#polynomial-string-hashing) for fast substring
comparison, [set/multiset hashing](#setmultiset-hashing-via-random-assignment)
for order-independent equality, [Zobrist hashing](#zobrist-hashing) for
incrementally-updated composite states, and
[tree hashing](#tree-hashing-and-isomorphism) for structural isomorphism.
Each works by assigning random values to primitive components and combining them
with cheap operations (multiplication, XOR, addition); a collision — two distinct
objects with the same hash — occurs with low probability and can be made
negligible by using two independent hash functions simultaneously.

## Polynomial string hashing

Fix a *base* $p$ and a large prime *modulus* $m$.  The **polynomial hash** of a
string $s = s_0 s_1 \cdots s_{n-1}$ is

$$
H(s) = \sum_{i=0}^{n-1} s_i \cdot p^{i} \pmod{m}.
$$

Store prefix hashes $h[k] = H(s_0 \cdots s_{k-1})$ and prefix powers
$\mathit{pw}[k] = p^k \bmod m$.  Any substring $s[l\,..\,r]$ (0-indexed,
inclusive) can then be hashed in $O(1)$:

$$
H(s[l\,..\,r]) = h[r+1] - h[l] \cdot p^{r-l+1} \pmod{m}.
$$

Using two independent (base, modulus) pairs — **double hashing** — reduces the
false-positive probability from $\approx 1/m \approx 10^{-9}$ to
$\approx 1/(m_1 m_2) \approx 10^{-18}$, which is safe even for $n^2$ comparisons.

~~~ {.cpp}
struct HashStr {
    static const long long M1=1000000007LL, M2=1000000009LL, B1=131, B2=137;
    int n;
    vector<long long> h1, h2, pw1, pw2;

    HashStr(const string& s) : n(s.size()), h1(n+1), h2(n+1), pw1(n+1,1), pw2(n+1,1) {
        for (int i = 0; i < n; i++) {
            h1[i+1] = (h1[i]*B1 + s[i]) % M1;
            h2[i+1] = (h2[i]*B2 + s[i]) % M2;
            pw1[i+1] = pw1[i]*B1 % M1;
            pw2[i+1] = pw2[i]*B2 % M2;
        }
    }

    // Hash of s[l..r] (0-indexed, inclusive).
    pair<long long,long long> get(int l, int r) const {
        return {(h1[r+1] - h1[l]*pw1[r-l+1]%M1 + M1*2) % M1,
                (h2[r+1] - h2[l]*pw2[r-l+1]%M2 + M2*2) % M2};
    }
};
~~~

The `+ M*2` before the final `%` prevents negative results from the subtraction.

### Anti-hack: randomize the base at runtime

On Codeforces an adversary can craft anti-hash tests for any *fixed* base.
The defence is to pick the base randomly at program start:

~~~ {.cpp}
mt19937_64 rng(chrono::steady_clock::now().time_since_epoch().count());
const long long B = uniform_int_distribution<long long>(200, 1e9)(rng);
~~~

Use this `B` in place of the compile-time constants above.

### Applications of string hashing

- **Pattern matching ([Rabin-Karp algorithm]()).** Compare the hash of each
  length-$|p|$ window of text $t$ against the hash of pattern $p$ in
  $O(|t| + |p|)$ time.
- **Palindrome detection.** Build a `HashStr` for both $s$ and its reverse.
  Substring $s[l\,..\,r]$ is a palindrome when its forward hash equals the
  reversed string's hash over the mirrored interval.  Binary-searching the
  longest palindromic extent at each centre costs $O(n \log n)$.
- **Longest common substring.** Binary search on length $\ell$; for each $\ell$
  insert all length-$\ell$ hashes of one string into a set and probe with the
  other.  $O(n \log n)$ total.
- **Counting distinct substrings.** Insert all $O(n^2)$ substring hashes into an
  `unordered_set`; its size is the answer.  For large $n$ a [Suffix array]() or
  [Suffix automaton]() is faster.
- **Circular shift comparison.** Hash of rotation $k$ of $s$ equals
  `H(s+s).get(k, k+n-1)`; sorting all rotations takes $O(n \log n)$.

### 2D string hashing

For grids, extend to two dimensions:

$$
H(A) = \sum_{i,j} A[i][j] \cdot p^i \cdot q^j \pmod{m}.
$$

A 2D prefix-hash table lets any rectangular subgrid hash be retrieved in $O(1)$,
enabling $O(nm)$-time pattern matching in a grid.

## Set/multiset hashing via random assignment

Assign each distinct element $x$ a random 64-bit value $r(x)$.  The
**XOR hash** of a set $S$ is

$$
H_{\oplus}(S) = \bigoplus_{x \in S} r(x).
$$

Because XOR is commutative and associative, $H_{\oplus}$ is order-independent;
two sets are equal iff their XOR hashes agree (with overwhelming probability).
Insertions and deletions are $O(1)$: just XOR the element's random value in or out.

For **multisets**, XOR fails because $x \oplus x = 0$ — a double occurrence
cancels.  Use a *sum* instead:

$$
H_+(S) = \sum_{x \in S} r(x) \pmod{2^{64}}.
$$

Subtraction undoes an element.  Keep both $H_\oplus$ and $H_+$ together for
stronger guarantees when needed.

~~~ {.cpp}
mt19937_64 rng(chrono::steady_clock::now().time_since_epoch().count());

// Global random value table — all SetHash objects must share this.
unordered_map<int,uint64_t> rand_val;
uint64_t elem_hash(int x) {
    auto [it, ins] = rand_val.emplace(x, 0);
    if (ins) it->second = rng();
    return it->second;
}

struct SetHash {
    uint64_t h = 0;       // XOR hash; swap for sum if multiset needed
    void insert(int x) { h ^= elem_hash(x); }
    void erase (int x) { h ^= elem_hash(x); }
};
~~~

### Applications of set hashing

- **Sliding-window anagram detection.** Maintain a `SetHash` for the current
  window and one for the pattern.  Each step is $O(1)$: XOR out the departing
  element, XOR in the arriving one, compare hashes.  Total $O(n)$.
- **Checking if two arrays contain the same elements.** Compute the XOR hash of
  both in $O(n)$; equal hashes imply equal sets.
- **Subset-sum equality.** Hash each candidate subset; collect in a hash map.
  Avoids sorting or canonical forms.

## Zobrist hashing

**Zobrist hashing** (Zobrist 1970) is a specialisation of random-assignment
hashing for *composite states* that change incrementally — most famously
chess board positions, but also any state that can be described as a set of
active features.

Precompute a table of independent random 64-bit values: one per
(position, feature) pair.  The hash of a state is the XOR of the values of all
currently active features.  When the state changes, XOR out the old feature
value and XOR in the new one — $O(1)$ per move regardless of board size.

~~~ {.cpp}
// Example: 4x4 board, pieces 0=empty 1=X 2=O.
// In practice initialise once at program start.
uint64_t ZT[ROWS][COLS][NUM_PIECES];
void init_zobrist() {
    mt19937_64 rng(chrono::steady_clock::now().time_since_epoch().count());
    for (auto& row : ZT) for (auto& col : row) for (auto& v : col) v = rng();
}

struct Board {
    int cell[ROWS][COLS] = {};
    uint64_t h = 0;

    // Place piece p at (r,c), replacing whatever was there.
    void place(int r, int c, int p) {
        h ^= ZT[r][c][cell[r][c]];  // remove old piece
        cell[r][c] = p;
        h ^= ZT[r][c][p];           // add new piece
    }
};
~~~

Moves are trivially undone by calling `place` again with the previous piece
(since XOR is self-inverse).

### Applications of Zobrist hashing

- **Transposition tables in game-tree search.** Store evaluated board positions
  in a hash map keyed by Zobrist hash.  When the same position is reached by
  different move sequences, the evaluation is reused — the key speed-up in
  minimax / alpha-beta.
- **Cycle detection in abstract games.** Check whether the current board hash has
  appeared before (e.g. detecting superko in Go).
- **Set membership tracking.** Any problem that can be expressed as "which items
  are currently active" benefits from Zobrist: flip an item's random value to add
  or remove it.

## Tree hashing and isomorphism

Two rooted trees are **isomorphic** if one can be relabelled to match the other.
Hashing provides an $O(n \log n)$ randomised check.

Assign hash $1$ to the empty tree.  For a node $v$ with children
$c_1, \dots, c_k$, compute each child's hash recursively, **sort** the list
(children are unordered), then map the sorted tuple to a canonical integer:

~~~ {.cpp}
// tree_canon maps sorted child-hash vectors to unique integer IDs.
map<vector<int>,int> tree_canon;
int next_id = 1;

int tree_hash(int v, int par, const vector<vector<int>>& adj) {
    vector<int> child_hashes;
    for (int u : adj[v])
        if (u != par) child_hashes.push_back(tree_hash(u, v, adj));
    sort(child_hashes.begin(), child_hashes.end());
    auto [it, ins] = tree_canon.emplace(child_hashes, 0);
    if (ins) it->second = next_id++;
    return it->second;
}
~~~

Two rooted trees are isomorphic iff their root hashes agree.  The canonical ID
assignment makes this collision-free (it is essentially AHU canonicalization, not
a probabilistic hash).

For **unrooted trees**, find the [centroid](Centroid decomposition) of each tree
($O(n)$), root both there, then compare hashes.  If a tree has two centroids
(always adjacent), try both pairings.

### Applications of tree hashing

- **Tree isomorphism.** Determine whether two given trees have the same structure.
- **Counting distinct subtree shapes.** Hash every subtree and count unique IDs.
- **Canonical tree representations.** Assign a globally unique label to each tree
  shape, enabling set/map operations on trees.

## Safe hash maps: the splitmix64 trick

`std::unordered_map` uses a hash of the key to select a bucket.  For integer keys
the default hash is often the identity, which an adversary can exploit by feeding
keys that all land in the same bucket — degrading performance to $O(n)$ per
operation.  The defence is a quality custom hash function.

**splitmix64** (Vigna 2015) is a fast, high-quality bit mixer with excellent
avalanche behaviour:

~~~ {.cpp}
struct SplitMix {
    static uint64_t mix(uint64_t x) {
        x += 0x9e3779b97f4a7c15ULL;
        x = (x ^ (x >> 30)) * 0xbf58476d1ce4e5b9ULL;
        x = (x ^ (x >> 27)) * 0x94d049bb133111ebULL;
        return x ^ (x >> 31);
    }
    // A different random seed each run foils precomputed collision inputs.
    static const uint64_t SEED;
    size_t operator()(uint64_t x) const { return mix(x ^ SEED); }
};
const uint64_t SplitMix::SEED =
    chrono::steady_clock::now().time_since_epoch().count();

// Usage: unordered_map<long long, int, SplitMix> mp;
~~~

The XOR with a runtime-random `SEED` ensures that the hash function varies
between runs, so an adversary who hard-codes a collision set in a test case cannot
rely on it working.  Use this whenever `unordered_map` or `unordered_set` appears
in a Codeforces submission.

## Problems

### Polynomial string hashing

- [String Matching](https://cses.fi/problemset/task/1753) (CSES)

<details>
<summary>Solution sketch — String Matching</summary>

Build `HashStr` for both text and pattern.  Slide a window of the pattern's length
across the text and compare hash pairs in $O(1)$ per position.  Total time
$O(n + m)$.  The naive `string::find` is $O(nm)$ in the worst case and will TLE.

</details>

- [Palindrome Queries](https://cses.fi/problemset/task/2420) (CSES)

<details>
<summary>Solution sketch — Palindrome Queries</summary>

Build `HashStr` for $s$ and for $\text{rev}(s)$.  A substring $s[l\,..\,r]$ is a
palindrome when `HS.get(l,r) == HR.get(n-1-r, n-1-l)`, checked in $O(1)$.
Point updates require rebuilding the prefix arrays in $O(n)$ each; for
frequent updates use a [segment tree]() that stores hash and power per node,
supporting $O(\log n)$ updates and $O(\log n)$ substring hash queries.

</details>

- [Finding Borders](https://cses.fi/problemset/task/1732) (CSES)
- [Longest Common Substring](https://cses.fi/problemset/task/1075) (CSES)

<details>
<summary>Solution sketch — Longest Common Substring</summary>

Binary search on length $\ell$: for a candidate $\ell$, hash all length-$\ell$
substrings of string $A$ into an `unordered_set`, then check each length-$\ell$
substring of $B$ against the set.  $O(n \log n)$ total.

</details>

- [String Hashing](https://open.kattis.com/problems/hashing) (Kattis)
- [Chasing Subs](https://open.kattis.com/problems/chasingsubs) (Kattis)
- [NAJPF — Pattern Find](https://www.spoj.com/problems/NAJPF/) (SPOJ)

### Set/multiset hashing

- [Counting Distinct Substrings](https://cses.fi/problemset/task/2105) (CSES)

<details>
<summary>Solution sketch — Counting Distinct Substrings</summary>

For each pair $(l, r)$, compute the double hash of $s[l\,..\,r]$ and insert into
an `unordered_set<pair<long long,long long>>`.  The set size at the end is the
answer.  $O(n^2)$ time, fine for $n \le 2000$; for larger $n$ use a
[Suffix array]() with the LCP array.

</details>

- [Animal Classification](https://open.kattis.com/problems/animal) (Kattis) — sliding-window set equality.

### Zobrist hashing

- [Hashing](https://open.kattis.com/problems/hashing) (Kattis)
- Transposition-table problems in game AI (offline, not an online judge) — Zobrist is the standard tool for deduplicating positions in minimax search.

### Tree hashing / isomorphism

- [Tree Isomorphism I](https://cses.fi/problemset/task/1700) (CSES) — rooted trees.
- [Tree Isomorphism II](https://cses.fi/problemset/task/1701) (CSES) — unrooted trees; find centroid, then apply rooted algorithm.

<details>
<summary>Solution sketch — Tree Isomorphism I & II</summary>

**Rooted (problem I):** run `tree_hash` from each given root.  Compare the two
integers; equal ↔ isomorphic.

**Unrooted (problem II):** find the [centroid](Centroid decomposition) of each
tree.  A tree has one or two centroids.  Root each tree at its centroid and
compare hashes.  If there are two centroids in either tree, try both candidate
roots.  $O(n \log n)$ from the sort inside `tree_hash`.

</details>

- [Counting Distinct Trees](https://cses.fi/problemset/task/1706) (CSES) — count non-isomorphic rooted trees on $n$ vertices (combine tree hashing with DP).

### Safe hash maps

- Any Codeforces problem where `unordered_map` with integer keys TLEs — wrap it
  with `SplitMix` as described above.  A classic forced example:
  [Codeforces: Maps and Segments](https://codeforces.com/problemset/problem/613/D).

## See also

- [Rabin-Karp algorithm]() — pattern matching via rolling polynomial hashes
- [Suffix array]() — $O(n \log n)$ deterministic alternative for string problems
- [Suffix automaton]() — $O(n)$ structure; counts distinct substrings exactly
- [Knuth-Morris-Pratt algorithm]() — deterministic $O(n+m)$ pattern matching
- [Aho-Corasick automaton]() — multi-pattern matching
- [Z-function]() — linear-time periodicity and prefix-match queries
- [Centroid decomposition]() — needed to reduce unrooted tree isomorphism to rooted
- [Perfect hashing]() — theoretical construction that guarantees zero collisions

## External links

- [String Hashing (cp-algorithms)](https://cp-algorithms.com/string/string-hashing.html)
- [Rabin-Karp Algorithm (cp-algorithms)](https://cp-algorithms.com/string/rabin-karp.html)
- [Blowing up unordered_map, and how to stop getting hacked (Codeforces)](https://codeforces.com/blog/entry/62393)
- [Hashing root trees (Codeforces)](https://codeforces.com/blog/entry/113465)
- [Anti-hash tests — analysis (Codeforces)](https://codeforces.com/blog/entry/60442)
- [Zobrist Hashing (Chess Programming Wiki)](https://www.chessprogramming.org/Zobrist_Hashing)
- [String Hashing (USACO Guide)](https://usaco.guide/gold/hashing)
