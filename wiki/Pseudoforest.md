---
categories: Graph theory
---

A **pseudoforest** is an [undirected graph](Undirected graph) in which every [connected component](Connected component) has at most one [cycle](Cycle (graph theory)).

> **Intuition**: If a connected undirected graph on $n$ vertices has $n-1$ edges, then it is a [tree](Tree). If a connected undirected graph on $n$ vertices has $n$ edges, then it has exactly one cycle (i.e. an underlying tree, with an additional edge which closes the cycle). Hence, a pseudoforest consists of connected components, where each component on $k$ vertices consists of either $k-1$ or $k$ edges.

## Directed pseudoforests
The concept of a pseudoforest can also be extended to include [directed graphs](Directed graph). A **directed pseudoforest** is a directed graph in which each vertex has at most one outgoing edge; that is, it has [outdegree](Vertex degree) at most one. In the special case where each vertex has outdegree exactly one, the graph is called a [functional graph](Functional graph).

## Problems
- [Room Assignments](https://open.kattis.com/problems/roomassignments) [^3]
- [Tug of War](https://boi.cses.fi/files/boi2015_day2.pdf) [^1] [^2]

## See also
- [Functional graph]()

## External links
- [Pseudoforest](https://en.wikipedia.org/wiki/Pseudoforest)

[^1]: <http://codeforces.com/blog/entry/17628?#comment-225251>
[^2]: <http://web.archive.org/web/20160829100520/http://www.boi2015.mimuw.edu.pl/305_Tasks>
[^3]: <https://archive.algo.is/icpc/nwerc/2009/solOutline.pdf>
