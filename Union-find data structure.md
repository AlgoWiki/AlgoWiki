---
categories: Data structures
...

## Problems
- [Disastrous Downfall](https://open.kattis.com/problems/downfall)
- [Šuma](https://open.kattis.com/problems/suma)
- [Almost Union-Find](https://open.kattis.com/problems/almostunionfind)
- [Ladice](https://open.kattis.com/problems/ladice)

## Edge deletion instead of insertion
The union-find data structure can sometimes be used to support edge deletion instead of edge insertion. The idea is simply to simulate the sequence of operations in reverse, and then deletions become insertions. This only works when the sequence of operations is known beforehand.

To support both insertions and deletions, see [Dynamic connectivity]().

### Problems
- [Islands](https://icpcarchive.ecs.baylor.edu/index.php?option=com_onlinejudge&Itemid=8&category=360&page=show_problem&problem=2628)
- [Escape from Enemy Territory](https://open.kattis.com/problems/enemyterritory)
- [Artwork](https://open.kattis.com/problems/artwork) [^1]

## Undo operation
The union-find data structure can be extended to support an undo operation. By repeatedly executing the operation it is possible to revert to an earlier state of the data structure, and then continue from there as if nothing had happened later. This is implemented by keeping a stack of all modifications to the underlying array. For example, if index $i$ in the underlying array currently has value $v$, and is then changed to $v'$, then the tuple $(i,v)$ is pushed to the stack (i.e. the index and the ''old'' value). To perform an undo operation, $(i,v)$ is popped from the stack, and the value at index $i$ is set to $v$, the old value.

### Problems
- [Chef and Graph Queries](https://www.codechef.com/MARCH14/problems/GERALD07)
- [Disconnected Graph](http://codeforces.com/gym/100551/problem/E)

[^1]: <http://ncpc.idi.ntnu.no/ncpc2016/ncpc2016slides.pdf>
