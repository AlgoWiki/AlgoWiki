---
categories: Algorithm techniques
...

**Binary search** is an algorithm technique based on the [Divide and conquer]() paradigm.
The most common example of binary search is to find an element in a sorted array in logarithmic time.

## Description
Suppose you have an array $A$ of $n$ objects, and a predicate function $P$, such that for each element $a\in A$, $P(a)$ returns either $\mathrm{true}$ or $\mathrm{false}$. Thus $P$ tests some property of the elements in $A$. 

Furthermore, the array must satisfy the **binary search property**: there exists an index $i$ such that $P(a_j)$ is $\mathrm{true}$ for all $j\geq i$, and $P(a_j)$ is $\mathrm{false}$ for all $j < i$. When this holds, binary search can be used to find this index $i$, i.e. the index of the leftmost element $a$ such that $P(a)$ is true. The following pseudocode shows how.

~~~ {.cpp}
int lo = 0,
    hi = (int)A.size() - 1,
    res = -1;
while (lo <= hi) {
    int mid = (lo+hi)/2;
    if (P(A[mid])) {
        res = mid;
        hi = mid - 1;
    } else {
        lo = mid + 1;
    }
}
~~~

At completion, $\mathrm{res}$ will be the index of the leftmost element $a$ such that $P(a)$ is true (the index that we called $i$ above), or $-1$ if no element in the array satisfies the property. Since the value of $\mathrm{hi} - \mathrm{lo}$ decreases by roughly half on every iteration, the algorithm takes time $O(\log(n))$, where $n$ is the size of the input array $A$.

## Applications

Say we want to find a specific element $x$ in a sorted array, or report that is not in the array. To do that, we define the property
$$P(a) := \left\{\begin{array}{ll}
\mathrm{true} & \textrm{if } a \geq x \\
\mathrm{false} & \textrm{otherwise}
\end{array}\right.$$
Since the array is sorted, we can see that the binary search property is fulfilled: there is an index $i$ such that for all $j \geq i$, $a_j \geq x$ is true, and for all $j < i$, $a_j \geq x$ is false. Thus, using binary search we can find the index of the first element that satisfies the property in logarithmic time. If this element is equal to $x$, then we're done, otherwise (and in the case that $\mathrm{res} = -1$) we can report that $x$ is not in the array.

It is also very common to use binary search when finding the minimum or maximum solution, even if there is no explicit array involved. Consider the problem [The Monkey and the Oiled Bamboo](#Problems). Notice that strength satisfies the binary search property: if we have a bigger strength factor than the minimum necessary, we will be able to reach the top, but if we have a smaller strength factor, then we won't be able to reach the top. Thus we can binary search $k$, doing $O(\log(n))$ simulations of jumping to the top to evaluate the predicate $P$ for different values of $k$.

## Variants
Binary search can also be applied over real numbers instead of integers. In that case the technique is known as the [Bisection method]().

## Problems
* [The Monkey and the Oiled Bamboo](https://uva.onlinejudge.org/external/120/p12032.pdf)

## See also
* [Parallel binary search]()
* [Divide and conquer]()
