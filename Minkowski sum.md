---
toc: no
categories: Geometry
...


The *Minkowski sum* of two sets of position vectors $A$ and $B$ in Euclidean space is formed by adding each vector in $A$ to each vector in $B$, i.e., the set

$$A + B = \{\mathbf{a}+\mathbf{b}\,|\,\mathbf{a}\in A,\ \mathbf{b}\in B\}.$$

Analogously, the *Minkowski difference* is defined as
$$A - B = \{\mathbf{a}-\mathbf{b}\,|\,\mathbf{a}\in A,\ \mathbf{b}\in B\}.$$


## Properties
For Minkowski addition, the *zero set* ${0}$, containing only the zero vector $0$, is an identity element: For every subset $S$, of a vector space
$$S + {0} = S;$$

The empty set is important in Minkowski addition, because the empty set annihilates every other subset: for every subset, $S$, of a vector space, its sum with the empty set is empty: $S + \emptyset = \emptyset$.

Minkowski addition behaves well with respect to the operation of taking [convex hulls](Convex hull), as shown by the following proposition:

* For all non-empty subsets $S_1$ and $S_2$ of a real vector-space, the convex hull of their Minkowski sum is the Minkowski sum of their convex hulls $$\mathrm{Conv}(S_1 + S_2) = \mathrm{Conv}(S_1) + \mathrm{Conv}(S_2).$$

This result holds more generally for each finite collection of non-empty sets
$$\mathrm{Conv}(\Sigma S_n) = \Sigma \mathrm{Conv}(S_n).$$

If $S$ is a convex set then also $\mu S+\lambda S$ is a convex set; furthermore
$\mu S+\lambda S=(\mu+\lambda)S$ for every $\mu,\lambda \geq 0$.
Conversely, if this "distributive property" holds for all non-negative real numbers, $\mu, \lambda$, then the set is convex.

Minkowski sums act linearly on the perimeter of two-dimensional convex bodies: the perimeter of the sum equals the sum of perimeters.

## Algorithms for computing Minkowski sums

### Planar case

#### Two convex polygons in the plane
For two [convex polygons](Convex polygon) $P$ and $Q$ in the plane with $m$ and $n$ vertices, their Minkowski sum is a convex polygon with at most $m + n$ vertices and may be computed in time $O (m + n)$ by a very simple procedure, which may be informally described as follows. Assume that the edges of a polygon are given and the direction, say, counterclockwise, along the polygon boundary. Then it is easily seen that these edges of the convex polygon are ordered by polar angle. Let us merge the ordered sequences of the directed edges from $P$ and $Q$ into a single ordered sequence $S$. Imagine that these edges are solid arrows which can be moved freely while keeping them parallel to their original direction. Assemble these arrows in the order of the sequence $S$ by attaching the tail of the next arrow to the head of the previous arrow. It turns out that the resulting polygonal chain will in fact be a convex polygon which is the Minkowski sum of $P$ and $Q$.

#### Other
If one polygon is convex and another one is not, the complexity of their Minkowski sum is $O(nm)$. If both of them are nonconvex, their Minkowski sum complexity is $O((mn)^2)$.


## Problems
* [Minkowski Sums](https://projecteuler.net/problem=228)
* [Kiddie Pool](https://code.google.com/codejam/contest/8234486/dashboard#s=p1&a=3) [^1]
* [Board game](http://amppz.ii.uni.wroc.pl/files/zadania_en.pdf)
* [Fireworks](https://www.docdroid.net/eviUqH5/fireworks-en.pdf.html) [^2]
* [Bridge Building](http://opencup.ru/files/ocf/gp13/problems1-e.pdf) [^3]
* [Mogohu Ree Idol](http://codeforces.com/contest/87/problem/E) [^4]
* [TrianglePainting](https://community.topcoder.com/stat?c=problem_statement&pm=13775) [^5]
* [Gears](http://codeforces.com/contest/497/problem/D) [^6]


## External links
* [Minkowski addition](https://en.wikipedia.org/wiki/Minkowski_addition)
* [Minkowski's addition of convex shapes](http://www.cut-the-knot.org/Curriculum/Geometry/PolyAddition.shtml)

[^1]: <http://codeforces.com/blog/entry/18204?#comment-231204>
[^2]: <http://codeforces.com/blog/entry/44657?#comment-293094>
[^3]: <http://codeforces.com/blog/entry/17483?#comment-223283>
[^4]: <http://codeforces.com/blog/entry/2121?locale=en>
[^5]: <http://codeforces.com/blog/entry/18241?#comment-231500>
[^6]: <http://codeforces.com/blog/entry/15208>
