---
categories: Graph algorithms, Graph theory
...

In the minimum Steiner tree problem, we are given an edge-weighted graph $G = (V, E, w)$ and a subset $S \subseteq V$ of required vertices. A Steiner tree is a tree in $G$ that spans all vertices of $S$, meaning that they are all in the same component. The objective of the problem is to find a Steiner tree of minimum weight. The decision version of the problem is [NP-complete](NP-completeness). However, it is [fixed-parameter tractable](Fixed-parameter tractability) when parameterized by the size of $S$, as is witnessed by the dynamic programming algorithm shown below.

## Algorithms

### Complete search
Time complexity is $O\left(\binom{n}{k-2} k^2 \log(k)\right)$, where $n=|V|$ and $k=|S|$.

### Dynamic programming
Time complexity is $O(n3^k + n^22^k)$.


## Problems
* [Jailbreak](https://open.kattis.com/problems/jailbreak) [^1]
* [Shovelling Snow](https://open.kattis.com/problems/shovelling)
* [Ticket to Ride](http://www.csc.kth.se/contest/nwerc/2006/problems/nwerc06.pdf)
* [Hexagonal Parcels](http://contest.felk.cvut.cz/07cerc/solved/h/)

## See also
* [Minimum spanning tree]()
* [NP-complete]()

## External links
* [The Steiner Problem in Graphs](http://sci-hub.cc/10.1002/net.3230010302)

[^1]: <http://2013.bapc.eu/docs/bapc2013juryslides.pdf>