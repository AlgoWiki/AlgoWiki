---
categories: Number theory
...

Given a list of positive integers $a_1,\ldots,a_n$, the *Frobenius coin problem* asks for the largest integer $N$ such that the equation
$$ x_1 a_1 + \cdots + x_n a_n = N $$
has no solutions where $x_i$ are nonnegative integers. This integer is denoted as $g(a_1,\ldots,a_n)$.

A related question asks whether a given $N$ has such a solution. A necessary (albeit not sufficient) condition is that $N$ is divisible by $G = \mathrm{gcd}(a_1,\ldots,a_n)$. However, it can be shown that, if $A = \max_i a_i$, any $N>A^2$ has such a solution iff. $N$ is divisible by $G$.[^need] Thus the Frobenius coin problem is only well defined when $G=1$.

These problems are [NP-Hard](), although they can be solved in pseudo-polynomial time $O(na_1\log(na_1))$ [^1] [^2]

## Identities for small $n$

### $n=2$
There is a closed-form solution for $n=2$:
$$ g(a_1,a_2) = a_1a_2 - a_1 - a_2 $$
Furthermore, the number of integers that don't have a solution is given by the formula:
$$ N(a_1,a_2) = (a_1-1)(a_2-1)/2 $$


### $n=3$

There is no known closed-form solution for $n=3$, although the following equality is known to hold:[^need]
$$ g(d\cdot a_1, d\cdot a_2, a_3) = d\cdot g(a_1,a_2,a_3) + a_3(d-1) $$

## Problems
* [Knapsack in a Globalized World](http://gcpc.nwerc.eu/problemset_2016.pdf)
* [Linear Combinations of Semiprimes](https://projecteuler.net/problem=278)
* [Sums](http://main.edu.pl/en/archive/oi/10/sum)
* [LongLongTripDiv1](https://community.topcoder.com/stat?c=problem_statement&pm=13090) [^3]

## See also
* [Bézout's identity](Bézout's identity)
* [Knapsack problem]()

## External links
* [Coin problem](https://en.wikipedia.org/wiki/Coin_problem)


[^1]: <http://gcpc.nwerc.eu/outlines_2016.pdf>
[^2]: <https://www.quora.com/What-are-some-competitive-programming-problems-with-really-elegant-solutions/answer/Michal-Fori%C5%A1ek>
[^3]: <https://apps.topcoder.com/wiki/display/tc/SRM+615>
[^need]: Citation needed!
