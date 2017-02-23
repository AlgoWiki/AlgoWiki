---
categories: Algorithm techniques
...

Binary search is an algorithm that uses Divide and Conquer paradigm to obtain $O(log(n))$ complexity for finding an element in a set.

## Description
Suppose that you have an array $a$ of objects and a property in them. Also, suppose that the objects are ordered in such a way that there is an index $i$ in them such that for all $j>i$, $a_j$ satisfies the property, and for all $j\leq i$, $a_j$ doesn't satisfy the property. We can find this index in logarithmic time with the algorithm described in the following pseudo-code:

~~~ {.py}
hi=len(a)
lo=-1

while lo+1<hi:
    mi=(lo+hi)//2

    if P(a[mi]): hi=mi
    else: lo=mi
~~~

At completion, $lo$ will be the index $i$ we looked for. (-1 means that all elements in the array satisfy the property)

## Applications

The first application is finding elements in an array. If we have a sorted array, and we define the property $P$ as an element being greater than $k$. Due to the sorted property of the array, a binary search will allow to find the index of the last element that doesn't satisfy the property (i.e. the last element lesser than or equal to $k$) in logarithmic time.

However, there are other kinds of problems solvable by this algorithm. See this problem [Monkey](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=24&page=show_problem&problem=3183). You can notice that if you have a bigger strength factor than the minimum necessary, you will be able to reach the top. If you have a smaller strength factor, then you won't be able to reach the top. Then, we can binary search $k$, and do evaluations for $O(log(n))$ different $k$ 's.

## Problems
* [Pie](http://www.csc.kth.se/contest/nwerc/2006/problems/nwerc06.pdf)
* [Strip](http://codeforces.com/problemset/problem/487/B)
* [Monkey](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=24&page=show_problem&problem=3183)

## See also
* [Parallel binary search]()
* [Divide and conquer]()

