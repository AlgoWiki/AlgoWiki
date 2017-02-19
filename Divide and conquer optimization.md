## Applicability
The divide and conquer optimization applies when the dynamic programming recurrence is approximately of the form
$$ \mathrm{dp}[k][i] = \min_{j<i} \left\{\mathrm{dp}[k-1][j] + C[i][j] \right\}, $$
where $A[k][i] \leq A[k][i+1]$. Here $A[k][i]$ is the smallest index $j^\star < i$ that minimizes $\mathrm{dp}[k-1][j^\star] + C[i][j^\star]$, i.e.
$$ A[k][i] = \mathrm{argmin}_{j<i} \left\{\mathrm{dp}[k-1][j] + C[i][j] \right\}. $$

A sufficient (but not necessary) condition for $A[k][i] \leq A[k][i+1]$ to hold is that $C[a][c] + C[b][d] \leq C[a][d] + C[b][c]$ where $a < b < c < d$.

The naive way of computing this recurrence with dynamic programming takes $O(kn^2)$ time, but only takes $O(kn\log n)$ time with the divide and conquer optimization.

## Problems
* [Guardians of the Lunatics](https://www.hackerrank.com/contests/ioi-2014-practice-contest-2/challenges/guardians-lunatics-ioi14) [^1]
* [Branch Assignment](https://open.kattis.com/problems/branch)
* [Mining](https://www.hackerrank.com/contests/world-codesprint-5/challenges/mining)
* [Ciel and Gondolas](http://codeforces.com/contest/321/problem/E) [^2]
* [Leaves](http://www.spoj.com/problems/NKLEAVES/)

## See also
* [Dynamic programming optimization]()
* [Divide and conquer]()

## External links
* [Dynamic Programming Optimizations](http://codeforces.com/blog/entry/8219)

[^1]: <https://www.hackerrank.com/contests/ioi-2014-practice-contest-2/challenges/guardians-lunatics-ioi14/editorial>
[^2]: <http://codeforces.com/blog/entry/8192>
