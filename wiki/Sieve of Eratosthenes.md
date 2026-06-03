---
categories: Number theory, Algorithm techniques
---

The **Sieve of Eratosthenes** is a classic algorithm for finding all prime
numbers up to a given limit $n$. Instead of testing each number for primality
individually, it works the other way around: it starts by assuming every number
is prime, and then repeatedly *crosses out* the multiples of each prime it
finds. Whatever survives is exactly the set of primes. It runs in
$O(n \log \log n)$ time, which is fast enough to sieve up to $10^7$–$10^8$
comfortably within typical contest limits, and it is the workhorse behind a huge
number of [number-theoretic](Number theory) tasks in competitive programming.

The sieve is the prototypical example of the more general [Sieve]() technique:
computing a value for *every* integer in a range by letting each prime (or each
small factor) contribute to all of its multiples.

## Description

We keep a boolean array `is_prime[0..n]`, initially all `true` except for $0$
and $1$, which are not prime. We then scan $i = 2, 3, 4, \dots$ in increasing
order. When we reach an $i$ that is still marked prime, it genuinely *is* prime
(every smaller factor would already have crossed it out), so we record it and
mark all of its multiples as composite.

A key optimization is to start crossing out from $i^2$ rather than $2i$: every
smaller multiple $k \cdot i$ with $k < i$ has a prime factor smaller than $i$
and was therefore already eliminated. A direct consequence is that once
$i^2 > n$ there is nothing left to cross out, so the outer loop only needs to
run while $i \le \sqrt{n}$.

~~~ {.cpp}
const int N = 1000000;
bitset<N + 1> is_prime;
vector<int> primes;

void sieve() {
    is_prime.set();                 // assume everything is prime
    is_prime[0] = is_prime[1] = 0;
    for (int i = 2; i <= N; i++) {
        if (is_prime[i]) {
            primes.push_back(i);
            for (long long j = (long long)i * i; j <= N; j += i)
                is_prime[j] = 0;    // cross out multiples of i
        }
    }
}
~~~

Using a `bitset` (or `vector<bool>`) keeps the memory footprint to one *bit* per
number, which both saves space and improves cache behaviour. Note the cast to
`long long` when computing `i * i`: for $n$ near $2^{31}$ the product overflows a
32-bit `int`.

### Complexity

The work done is proportional to the number of crossing-out operations, which is
$$\sum_{p \le n,\ p \text{ prime}} \frac{n}{p} = n \sum_{p \le n} \frac{1}{p} = n \log \log n + O(n),$$
by Mertens' theorem on the sum of reciprocals of primes. Hence the running time
is $O(n \log \log n)$ — only marginally above linear. The memory usage is $O(n)$
(one bit per number with a `bitset`).

## Applications

- **Listing or counting primes.** The most direct use: enumerate every prime up
  to $n$, or answer "how many primes are $\le x$?" with a prefix-sum over the
  sieve.
- **Fast repeated primality tests.** After one $O(n \log \log n)$ precomputation,
  each query "is $x$ prime?" for $x \le n$ is answered in $O(1)$.
- **Fast factorization.** By storing the *smallest prime factor* of every number
  (see [below](#smallest-prime-factor-and-fast-factorization)), any $x \le n$ can
  be factored in $O(\log x)$ time — far faster than $O(\sqrt{x})$ trial division
  when many numbers must be factored.
- **Computing multiplicative functions.** Functions such as
  [Euler's totient](Euler's totient function) $\varphi$, the
  [Möbius function](Möbius inversion formula) $\mu$, the divisor count $\tau$ and
  divisor sum $\sigma$ can all be tabulated for every integer up to $n$ during a
  single sieve (see [Sieving multiplicative functions](#sieving-multiplicative-functions)).
  These tables are the backbone of [Möbius inversion](Möbius inversion formula)
  and [inclusion–exclusion](Inclusion-exclusion principle) counting arguments.

## Variants

### Smallest prime factor and fast factorization

A small change turns the sieve into a tool for *factorizing* numbers quickly.
Instead of a boolean flag, store for each number the smallest prime that divides
it. The **linear sieve** (also called the *Euler sieve*) does this in genuinely
linear $O(n)$ time by guaranteeing that every composite is crossed out exactly
once — by its smallest prime factor.

~~~ {.cpp}
const int N = 1000000;
int spf[N + 1];          // spf[x] = smallest prime factor of x
vector<int> primes;

void linear_sieve() {
    for (int i = 2; i <= N; i++) {
        if (spf[i] == 0) {              // i is prime
            spf[i] = i;
            primes.push_back(i);
        }
        for (int p : primes) {
            if (p > spf[i] || (long long)i * p > N) break;
            spf[i * p] = p;
        }
    }
}
~~~

The crucial line is the loop break: we only mark $i \cdot p$ for primes
$p \le \mathrm{spf}(i)$, so each composite $m = i \cdot p$ is touched exactly
once, namely when $i = m / p$ and $p$ is the smallest prime factor of $m$. With
`spf` in hand, factorization is a short loop:

~~~ {.cpp}
vector<int> factorize(int x) {          // returns prime factors with multiplicity
    vector<int> f;
    while (x > 1) { f.push_back(spf[x]); x /= spf[x]; }
    return f;
}
~~~

In practice the plain `bitset` sieve is often *faster* than the linear sieve for
merely listing primes, because it touches memory more cache-friendly and avoids
the inner `vector` iteration; the linear sieve earns its keep when you also need
`spf` or a multiplicative function table.

### Segmented sieve

What if you need the primes in a high range such as $[10^9, 10^9 + 10^6]$? An
array of size $10^9$ is far too large, but the range itself is small. The
**segmented sieve** exploits the fact that every composite $\le H$ has a prime
factor $\le \sqrt{H}$. So we first sieve the primes up to $\sqrt{H}$ with the
ordinary sieve, then use them to cross out composites in the window $[L, H]$,
using an array indexed by offset $x - L$.

~~~ {.cpp}
// requires `primes` to contain all primes up to floor(sqrt(hi))
vector<char> segmented_sieve(long long lo, long long hi) {   // primes in [lo, hi]
    vector<char> is_prime(hi - lo + 1, 1);
    if (lo == 1) is_prime[0] = 0;
    for (int p : primes) {
        if ((long long)p * p > hi) break;
        long long start = max((long long)p * p, (lo + p - 1) / p * p);
        for (long long j = start; j <= hi; j += p)
            is_prime[j - lo] = 0;
    }
    return is_prime;
}
~~~

This runs in $O\big((H - L)\log\log H + \sqrt{H}\big)$ time using only
$O\big(\sqrt{H} + (H - L)\big)$ memory.

### Sieving multiplicative functions

A function $f$ is *multiplicative* if $f(ab) = f(a)f(b)$ whenever $\gcd(a,b)=1$.
The linear sieve can compute such a function for every integer up to $n$ in
$O(n)$ time, because when it marks a composite $i \cdot p$ it knows whether $p$
already divides $i$, which is exactly the information needed to extend $f$.

For example, [Euler's totient](Euler's totient function) $\varphi$ and the
[Möbius function](Möbius inversion formula) $\mu$:

~~~ {.cpp}
int phi[N + 1], mu[N + 1];
bool composite[N + 1];
vector<int> primes;

void multiplicative_sieve() {
    phi[1] = mu[1] = 1;
    for (int i = 2; i <= N; i++) {
        if (!composite[i]) { primes.push_back(i); phi[i] = i - 1; mu[i] = -1; }
        for (int p : primes) {
            if ((long long)i * p > N) break;
            composite[i * p] = true;
            if (i % p == 0) {              // p divides i: p is a repeated factor
                phi[i * p] = phi[i] * p;
                mu[i * p] = 0;
                break;
            } else {                       // p is a new prime factor
                phi[i * p] = phi[i] * (p - 1);
                mu[i * p] = -mu[i];
            }
        }
    }
}
~~~

The same skeleton tabulates the number of divisors $\tau$, the sum of divisors
$\sigma$, and many other functions — only the two update lines change.

## Problems

The list below is grouped by which idea the problem exercises. Several entries
include a short solution sketch behind a spoiler; it focuses only on how the
sieve is used, not on a full editorial.

### Basic sieve

- [Prime Sieve](https://open.kattis.com/problems/primesieve)
- [Summation of primes (Project Euler 10)](https://projecteuler.net/problem=10)

### Fast factorization with the smallest prime factor

- [Counting Divisors](https://cses.fi/problemset/task/1713)
- [Sherlock and his girlfriend](https://codeforces.com/problemset/problem/776/B)

<details>
<summary>Solution sketch — Counting Divisors</summary>

Precompute `spf` once with a linear sieve up to $10^6$. For each query value $x$,
factorize it in $O(\log x)$ using `spf`, collect the exponents
$e_1, \dots, e_k$ of its distinct primes, and output
$\tau(x) = \prod (e_i + 1)$. Total time $O(n_{\max} + q \log x)$, far better than
$O(q\sqrt{x})$ trial division when there are many queries.

</details>

<details>
<summary>Solution sketch — Sherlock and his girlfriend</summary>

Two numbers where one is a prime multiple of the other must get different
colours, so the answer is $1$ only when there is a single item, and otherwise
$2$. A valid $2$-colouring is to colour each item by the *parity of the number of
prime factors of its value* (with multiplicity); adjacent values in the
constraint differ by exactly one prime factor and so always disagree. The `spf`
sieve gives both the factor count and the primality test cheaply.

</details>

### Counting / prefix tricks over the sieve

- [Bear and Prime Numbers](https://codeforces.com/problemset/problem/385/C)

<details>
<summary>Solution sketch — Bear and Prime Numbers</summary>

For every prime $p \le 10^7$, count how many array elements are divisible by $p$;
this is itself a sieve-style loop (for each prime, walk its multiples and add up
their frequencies). Build a prefix sum over primes indexed by value. A query
$[l, r]$ is then answered in $O(1)$ as the difference of two prefix sums.

</details>

### Multiplicative-function sieves

- [Common Divisors](https://cses.fi/problemset/task/1081)
- [Counting Coprime Pairs](https://cses.fi/problemset/task/2417)
- [Short Task](https://codeforces.com/problemset/problem/1512/G)

<details>
<summary>Solution sketch — Counting Coprime Pairs</summary>

Let $c_d$ be how many array elements are divisible by $d$ (a sieve-style count).
The number of coprime pairs is obtained by [Möbius inversion](Möbius inversion formula):
$$\#\{(i,j): \gcd(a_i,a_j)=1\} = \sum_{d \ge 1} \mu(d)\binom{c_d}{2},$$
where $\mu$ comes from the multiplicative sieve. The whole solution is
$O(A \log A)$ where $A$ is the maximum value.

</details>

### Segmented sieve

- [Prime Generator (PRIME1)](https://www.spoj.com/problems/PRIME1/)

## See also

- [Sieve]() — the general "let each factor contribute to its multiples" technique
- [Möbius inversion formula]() — pairs naturally with a sieved $\mu$ table
- [Inclusion-exclusion principle]() — the counting principle underlying $\mu$
- [Primitive root modulo n]() — another number-theoretic routine often built on a prime sieve

## External links

- [Sieve of Eratosthenes (Wikipedia)](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes)
- [Sieve of Eratosthenes (cp-algorithms)](https://cp-algorithms.com/algebra/sieve-of-eratosthenes.html)
- [Linear sieve (cp-algorithms)](https://cp-algorithms.com/algebra/prime-sieve-linear.html)
- [Sieve Methods : Prime, Divisor, Euler Phi etc. (Codeforces)](http://codeforces.com/blog/entry/8989)
