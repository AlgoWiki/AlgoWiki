---
categories: Combinatorics
...

## Cycle indices of some permutation groups

### Identity group $E_n$</sub>
This group contains one permutation that fixes every element (this must be a natural action).

$$ Z(E_n) = a_1^n.$$

### Cyclic group $C_n$
A [Cyclic group](), $C_n$ is the group of rotations of a regular $n$-gon, that is, $n$ elements equally spaced around a circle. This group has $\varphi(d)$ elements of order $d$ for each divisor $d$ of $n$, where $φ(d)$ is the [Euler φ-function](Euler's totient function), giving the number of natural numbers less than $d$ which are relatively prime to $d$. In the regular representation of $C_n$, a permutation of order $d$ has $n/d$ cycles of length $d$, thus

$$ Z(C_n) = \frac{1}{n} \sum_{d|n} \varphi(d) a_d^{n/d}.$$

### Dihedral group $D_n$
The [Dihedral group]() is like the cyclic group, but also includes reflections. In its natural action,

$$ Z(D_n) = \frac{1}{2} Z(C_n) +
\begin{cases}
\frac{1}{2} a_1 a_2^{(n-1)/2}, & n \mbox{ odd, } \\
\frac{1}{4}
\left( a_1^2 a_2^{(n-2)/2} + a_2^{n/2} \right), & n \mbox{ even.}
\end{cases}
$$

### The alternating group $A_n$
The cycle index of the [alternating group](Alternating group) in its natural action as a permutation group is

$$ Z(A_n) =
\sum_{j_1+2 j_2 + 3 j_3 + \cdots + n j_n = n}
\frac{1 + (-1)^{j_2+j_4+\cdots}}{\prod_{k=1}^n k^{j_k} j_k!} \prod_{k=1}^n a_k^{j_k}.
$$
The numerator is 2 for the even permutations, and 0 for the odd permutations. The 2 is needed because
$$\frac{1}{|A_n|}=\frac{2}{n!}.$$

### The symmetric group $S_n$

The cycle index of the [symmetric group](Symmetric group) $S_n$ in its natural action is given by the formula
$$ Z(S_n) = \sum_{j_1+2 j_2 + 3 j_3 + \cdots + n j_n = n} \frac{1}{\prod_{k=1}^n k^{j_k} j_k!} \prod_{k=1}^n a_k^{j_k}.$$

This formula is obtained by counting how many times a given permutation shape can occur. There are three steps: first partition the set of $n$ labels into subsets, where there are $j_k$ subsets of size $k$. Every such subset generates $k!/k$ cycles of length $k$. But we do not distinguish between cycles of the same size, i.e. they are permuted by $S_{j_k}$. This yields
$$
\frac{n!}{\prod_{k=1}^n (k!)^{j_k}}
\prod_{k=1}^n \left( \frac{k!}{k} \right)^{j_k}
\prod_{k=1}^n \frac{1}{j_k!}
=
\frac{n!}{\prod_{k=1}^n k^{j_k} j_k!}.$$

There is a useful recursive formula for the cycle index of the symmetric group.
Set $Z(S_0) = 1$ and consider the size $l$ of the cycle that contains $n$,
where $$\begin{matrix}1 \le l \le n.\end{matrix}$$
There are $$\begin{matrix}{n-1 \choose l-1}\end{matrix}$$ ways to choose the remaining $l-1$ elements of the cycle and every such choice generates
$$\begin{matrix}\frac{l!}{l}\end{matrix}$$ different cycles.

This yields the recurrence
$$ Z(S_n) = \frac{1}{n!} \sum_{g\in S_n} \prod_{k=1}^n a_k^{j_k(g)}
=
\frac{1}{n!}
\sum_{l=1}^n {n-1 \choose l-1} \; \frac{l!}{l} \; a_l \; (n-l)! \; Z(S_{n-l})
$$
or
$$
Z(S_n) = \frac{1}{n} \sum_{l=1}^n a_l \; Z(S_{n-l}).$$

## See also
* [Pólya enumeration theorem]()
* [Combinatorial species]()
* [Generating functions]()
