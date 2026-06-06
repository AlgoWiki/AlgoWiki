---
categories: String data structures, Algorithm techniques
---

**Hashing** in competitive programming almost always means *polynomial string
hashing*: a technique that maps a string (or any sequence) to a small integer so
that two substrings can be compared in $O(1)$ after an $O(n)$ preprocessing step.
It underlies fast pattern matching, palindrome detection, suffix-array construction
shortcuts, and many other string algorithms.

## Description

Fix a *base* $p$ and a *modulus* $m$.  The **polynomial hash** of a string
$s = s_0 s_1 \dots s_{n-1}$ is

$$
H(s) = \sum_{i=0}^{n-1} s_i \cdot p^{i} \pmod{m}.
$$

(Some sources use $p^{n-1-i}$ instead, reversing the exponent; either convention
works as long as it is applied consistently.)

### Prefix hashes and substring queries

Store the *prefix hashes* $h[k] = H(s_0 \dots s_{k-1})$ and *prefix powers*
$\mathit{pw}[k] = p^k \bmod m$.  Then the hash of any substring $s[l\,..\,r]$
(0-indexed, inclusive) can be extracted in $O(1)$:

$$
H(s[l\,..\,r]) = h[r+1] - h[l] \cdot p^{r-l+1} \pmod{m}.
$$

This follows from the definition: $h[r+1]$ contains the contribution of
$s_l, \dots, s_r$ shifted up by a factor of $p^l$, so subtracting $h[l] \cdot p^{r-l+1}$ cancels the prefix exactly.

~~~ {.cpp}
struct HashStr {
    static const long long MOD1 = 1e9 + 7, MOD2 = 1e9 + 9;
    static const long long BASE1 = 131,     BASE2 = 137;
    int n;
    vector<long long> h1, h2, pw1, pw2;

    HashStr(const string& s) : n(s.size()),
            h1(n+1,0), h2(n+1,0), pw1(n+1,1), pw2(n+1,1) {
        for (int i = 0; i < n; i++) {
            h1[i+1] = (h1[i] * BASE1 + s[i]) % MOD1;
            h2[i+1] = (h2[i] * BASE2 + s[i]) % MOD2;
            pw1[i+1] = pw1[i] * BASE1 % MOD1;
            pw2[i+1] = pw2[i] * BASE2 % MOD2;
        }
    }

    // Hash of s[l..r] (0-indexed, inclusive).
    pair<long long,long long> get(int l, int r) const {
        long long v1 = (h1[r+1] - h1[l] * pw1[r-l+1] % MOD1 + MOD1 * 2) % MOD1;
        long long v2 = (h2[r+1] - h2[l] * pw2[r-l+1] % MOD2 + MOD2 * 2) % MOD2;
        return {v1, v2};
    }
};
~~~

Two substrings are considered equal when their hash pairs match.  The `+ MOD * 2`
before the final `%` prevents negative results from the subtraction.

### Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Preprocessing | $O(n)$ | $O(n)$ |
| Substring hash | $O(1)$ | — |
| Equality check | $O(1)$ | — |

With a single hash of modulus $m$, the probability of a false positive between
any two unequal substrings is at most $1/m \approx 10^{-9}$.  After $Q$
comparisons, the collision probability grows to roughly $Q/m$, which matters when
$Q$ is large (e.g. $Q = n^2$ distinct pairs).  [Double hashing](#double-hashing)
brings it down to $Q/m^2 \approx 10^{-18}$.

## Applications

- **Pattern matching.** Compare the hash of each length-$|p|$ window of text $t$
  against the hash of pattern $p$ in $O(|t| + |p|)$ — the [Rabin-Karp algorithm]().
- **Palindrome detection.** Hash both the string and its reverse; a substring
  $s[l\,..\,r]$ is a palindrome when its forward hash equals the reversed string's
  hash over the mirrored interval.  Combine with [binary search]() to find the
  longest palindromic substring at each center in $O(n \log n)$.
- **Counting distinct substrings.** Insert all $O(n^2)$ substring hashes into a
  hash set; the set size is the answer.  Compare against [Suffix array]() and
  [Suffix automaton](), which solve this in $O(n \log n)$ or $O(n)$.
- **Suffix array construction shortcut.** The [DC3 / skew algorithm]() and several
  practical SA builders use polynomial hashing internally to compare suffixes
  during radix sort phases.
- **Comparing circular shifts.** Hash concatenation $s + s$ lets you extract the
  hash of any rotation in $O(1)$, enabling $O(n \log n)$ lexicographic sorting of
  all rotations (useful for [Lyndon factorization]() and [Burrows-Wheeler transform]()).

## Variants

### Double hashing

Use two independent (base, modulus) pairs and treat the hash as a pair
$\bigl(H_1(s),\, H_2(s)\bigr)$.  The probability of a false positive drops to
$1/(m_1 m_2) \approx 10^{-18}$, making collisions practically impossible even
in adversarial settings.  The `HashStr` struct above already does this.

### Anti-hack randomization

In online judges that allow adversarial test generation (e.g. Codeforces hacks),
a fixed base is a liability: an adversary can compute strings that collide under
any fixed modular polynomial hash.  The standard defense is to choose the base
randomly at runtime:

~~~ {.cpp}
mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());
const long long BASE = uniform_int_distribution<long long>(200, 1e9)(rng);
~~~

Pick `BASE` once at program start (not per query) so all hashes remain consistent
within one run.  A large prime modulus (e.g. $10^9+7$) or a Mersenne prime
($2^{61}-1$, which enables fast modular reduction) is preferable over the
`unsigned long long` overflow trick — the latter is known to be crackable.

### 2D hashing

For grid problems, extend to two dimensions:

$$
H(A) = \sum_{i,j} A[i][j] \cdot p^i \cdot q^j \pmod{m}.
$$

Precompute a 2D prefix-hash table; any rectangular subgrid hash can then be
retrieved in $O(1)$ using the 2D analogue of the 1D prefix formula.  Used in
problems that ask for the number of distinct submatrices or pattern matching in a
grid.

### Rabin-Karp pattern matching

~~~ {.cpp}
// Returns all start positions (0-indexed) where pat occurs in text.
vector<int> rabin_karp(const string& text, const string& pat) {
    int n = text.size(), m = pat.size();
    if (m > n) return {};
    HashStr HT(text), HP(pat);
    auto ph = HP.get(0, m - 1);
    vector<int> res;
    for (int i = 0; i + m <= n; i++)
        if (HT.get(i, i + m - 1) == ph) res.push_back(i);
    return res;
}
~~~

This runs in $O(n + m)$ on average (and worst-case with double hashing, since
false positives requiring a character-by-character confirmation become negligible).
For multiple patterns use an [Aho-Corasick automaton]() instead.

## Problems

### Pattern matching

- [String Matching](https://cses.fi/problemset/task/1753) (CSES)

<details>
<summary>Solution sketch — String Matching</summary>

Build `HashStr` for the text and for the pattern.  Slide a window of the pattern's
length across the text, comparing hash pairs in $O(1)$ each.  Total time $O(n+m)$.
Watch out: the naive `string::find` in C++ is already $O(nm)$ in the worst case,
so hashing (or [KMP](Knuth-Morris-Pratt algorithm)) is needed for large inputs.

</details>

- [Chasing Subs](https://open.kattis.com/problems/chasingsubs) (Kattis)
- [String Hashing](https://open.kattis.com/problems/hashing) (Kattis) — implement the hash spec exactly; tests understanding of modular arithmetic.
- [NAJPF — Pattern Find](https://www.spoj.com/problems/NAJPF/) (SPOJ) — count occurrences and list positions.

### Palindromes

- [Palindrome Queries](https://cses.fi/problemset/task/2420) (CSES)

<details>
<summary>Solution sketch — Palindrome Queries</summary>

Build `HashStr` for $s$ and for $\text{rev}(s)$.  To check whether $s[l\,..\,r]$
is a palindrome, compare `H.get(l, r)` with `HR.get(n-1-r, n-1-l)` where `HR`
is the reversed string's `HashStr`.  Each query is $O(1)$ after $O(n)$ setup.
Point updates (changing a character) require recomputing prefix hashes, which is
$O(n)$ per update — use a [segment tree]() with lazy hashing for $O(\log n)$
updates if there are many.

</details>

### Distinct substrings and counting

- [Counting Distinct Strings](https://cses.fi/problemset/task/2105) (CSES — "Distinct Substrings" variant)

<details>
<summary>Solution sketch — Counting Distinct Strings</summary>

For each possible substring length $\ell = 1, \dots, n$, collect all $n-\ell+1$
substring hash pairs into an `unordered_set` (or sort + deduplicate).  Sum the
distinct counts across all lengths.  Time $O(n^2)$ — acceptable for $n \le 2000$.
For $n$ up to $10^5$ use a [Suffix array]() with the LCP array instead.

</details>

- [Animal Classification](https://open.kattis.com/problems/animal) (Kattis)

### Hashing with binary search (longest palindrome / common prefix)

- [Finding Borders](https://cses.fi/problemset/task/1732) (CSES)

<details>
<summary>Solution sketch — Finding Borders</summary>

A *border* of $s$ is a non-empty proper prefix that is also a suffix.  For each
length $k$, check whether `H.get(0, k-1) == H.get(n-k, n-1)` in $O(1)$.
Print every $k$ for which this holds.  Total time $O(n)$.

</details>

- [Longest Common Substring](https://cses.fi/problemset/task/1075) (CSES)

<details>
<summary>Solution sketch — Longest Common Substring</summary>

Binary search on the answer length $\ell$.  For a given $\ell$, hash all
length-$\ell$ substrings of the first string into a set, then check whether any
length-$\ell$ substring of the second string appears in that set.  Total time
$O(n \log n)$.

</details>

### Harder applications

- [Bear and Prime Numbers](https://codeforces.com/problemset/problem/385/C) (Codeforces) — rolling hash idea applied to number sequences.
- [Competitive Programming String Problems](https://codeforces.com/problemset?tags=hashing) — Codeforces tag for hashing problems.

## See also

- [Rabin-Karp algorithm]() — the canonical pattern-matching algorithm built on rolling hashes
- [Suffix array]() — $O(n \log n)$ construction; replaces hashing when exact answers are needed
- [Suffix automaton]() — $O(n)$ structure that counts distinct substrings exactly
- [Knuth-Morris-Pratt algorithm]() — deterministic $O(n+m)$ pattern matching, no false positives
- [Aho-Corasick automaton]() — multi-pattern matching; complement to hashing for many patterns
- [Z-function]() — another linear-time tool for string periodicity and matching
- [Perfect hashing]() — theoretical construction guaranteeing zero collisions

## External links

- [String Hashing (cp-algorithms)](https://cp-algorithms.com/string/string-hashing.html)
- [Rabin-Karp Algorithm (cp-algorithms)](https://cp-algorithms.com/string/rabin-karp.html)
- [Anti-hash tests (Codeforces blog)](https://codeforces.com/blog/entry/60442)
- [Hashing and Collisions — analysis (Codeforces blog)](https://codeforces.com/blog/entry/100027)
- [String Hashing (USACO Guide)](https://usaco.guide/gold/hashing)
