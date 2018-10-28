## Pseudocode
~~~{.cpp}
void change_root(int old_root, int new_root) {
    // Update data structures as if `new_root` was being removed as a child from `old_root`
    // Update data structures as if `old_root` was being added as a child to `new_root`
}

void walk(int vertex, int parent) {
    // Data structures now reflect the tree as if `vertex` were the root

    for (child in adjacency_list[vertex]) {
        if (child == parent) continue;

        change_root(vertex, child);
        walk(child, vertex);
        change_root(child, vertex);
    }
}

// Initialize data structures with respect to the initial root `root`

walk(root, -1);
~~~


## Problems
- [Moving to Nuremberg](https://open.kattis.com/problems/nuremberg) [^1]
- [Kamp](https://open.kattis.com/problems/kamp)
- [Postman](http://amppz.ii.uni.wroc.pl/amppz2015/files/zadania_en.pdf) [^2] [^5]
- [JZPTREE](http://acm.hdu.edu.cn/showproblem.php?pid=4625) [^3][^4]

[^1]: <http://2009.nwerc.eu/results/solOutline.pdf>
[^2]: <http://amppz.ii.uni.wroc.pl/amppz2015/files/rozwiazania.pdf>
[^3]: <https://archive.algo.is/camps/mipt/2016/statement4a.pdf>
[^4]: <https://archive.algo.is/camps/mipt/2016/editorial4a.pdf>
[^5]: <https://archive.algo.is/camps/mipt/2015/day1/editorial.pdf>