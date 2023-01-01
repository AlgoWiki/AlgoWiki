---
categories: Algorithm techniques, Data structures
---

Not to be confused with [convex hull](Convex hull), the **convex hull trick** is a [dynamic programming optimization](Dynamic programming optimization) technique. It can be thought of as a [data structure](Data structure) supporting the following operations:

- <tt>Insert(ax+b)</tt>: insert the line $ax+b$ into the data structure
- <tt>GetMin(x)</tt>: considering all the lines that have been inserted so far, return the one that has the minimum value at $x$

## Applicability
The convex hull trick applies when the dynamic programming recurrence is approximately of the form
$$ \mathrm{dp}[i] = \min_{j<i} \left\{\mathrm{dp}[j] + b[j]\times a[i]\right\}. $$
where $b[j]\geq b[j+1]$ and $a[i] \leq a[i+1]$. The naive way of computing this recurrence with dynamic programming takes $O(n^2)$ time, but only takes $O(n)$ time with the convex hull trick. The restrictions on $a$ and $b$ can be dropped at the expense of a more complex and slightly slower approach.

> TODO: Talk about dynamic vs. static variants

Sometimes the above form appears within a more complex recurrence, as is the case for
$$ \mathrm{dp}[k][i] = \min_{j<i} \left\{\mathrm{dp}[k-1][j] + b[j]\times a[i]\right\}. $$
The approach remains very similar, and in this case the convex hull trick brings the time complexity down to $O(kn)$ from $O(kn^2)$.
This latter form can also be computed quickly using the [divide and conquer optimization](Divide and conquer optimization).

## Problems
- [Commando](http://www.spoj.com/problems/APIO10A/)
- [Land Acquisition](http://tjsct.wikidot.com/usaco-mar08-gold)
- [Batch Scheduling](http://wcipeg.com/problem/ioi0221)
- [Covered Walkway](https://open.kattis.com/problems/coveredwalkway)
- [Branch Assignment](https://open.kattis.com/problems/branch)
- [Good Inflation](http://www.spoj.com/problems/GOODG/)
- [Cats Transport](http://codeforces.com/problemset/problem/311/B) [^1]
- [Cow School](http://poj.org/problem?id=3266) [^2]
- [Kalila and Dimna in the Logging Industry](http://codeforces.com/contest/319/problem/C) [^3]
- [Leaves](http://www.spoj.com/problems/NKLEAVES/)
- [Squared Ends](https://csacademy.com/contest/round-70/task/squared-ends/)

## See also
- [Dynamic programming optimization]()

## External links
- [Convex hull trick](http://wcipeg.com/wiki/Convex_hull_trick)
- [Effective Usage of C++ STL for quick and concise code writing in competitive programming](http://codeforces.com/blog/entry/11155#comment-162462)
- [Advanced dynamic programming techniques](https://apps.topcoder.com/forums/?module=Thread&threadID=608334&start=0&mc=14#1120736)


[^1]: <http://codeforces.com/blog/entry/7785>
[^2]: <http://people.cs.clemson.edu/~bcdean/cowschool.swf>
[^3]: <http://codeforces.com/blog/entry/8166>