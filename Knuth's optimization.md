## Applicability
Knuth's optimization applies when the dynamic programming recurrence is approximately of the form
$$ \mathrm{dp}[i][j] = \min_{i<k<j} \left\{\mathrm{dp}[i][k] + \mathrm{dp}[k][j] \right\} + C[i][j], $$
where $A[i][j-1] \leq A[i][j] \leq A[i+1][j]$. Here $A[i][j]$ is the smallest index $i < k^\star < j$ that minimizes $\mathrm{dp}[i][k^\star] + \mathrm{dp}[k^\star][j]$, i.e.
$$ A[i][j] = \mathrm{argmin}_{i<k<j} \left\{\mathrm{dp}[i][k] + \mathrm{dp}[k][j] \right\}. $$

A sufficient (but not necessary) condition for $A[i][j-1] \leq A[i][j] \leq A[i+1][j]$ to hold is that $C[a][c] + C[b][d] \leq C[a][d] + C[b][c]$ and $C[b][c] \leq C[a][d]$ where $a \leq b \leq c \leq d$.

The naive way of computing this recurrence with dynamic programming takes $O(n^3)$ time, but only takes $O(n^2)$ time with Knuth's optimization.

## Problems

* [Optimal Binary Search Tree](https://uva.onlinejudge.org/external/103/10304.pdf)
* [Guardians of the Lunatics](https://www.hackerrank.com/contests/ioi-2014-practice-contest-2/challenges/guardians-lunatics-ioi14) [^1]
* [Order-Preserving Codes](http://codeforces.com/gym/100212)
* [Breaking Strings](http://www.spoj.com/problems/BRKSTRNG/)

## See also
* [Dynamic programming optimization]()

## External links
* [Dynamic Programming Optimizations](http://codeforces.com/blog/entry/8219)
* [ZOJ 2860 Breaking Strings](https://apps.topcoder.com/forums/?module=Thread&threadID=579321&start=0&mc=17#823126)
* [Commonly used DP state domains](https://apps.topcoder.com/forums/?module=Thread&threadID=697369&start=0&mc=22#1327577)

[^1]: <https://www.hackerrank.com/contests/ioi-2014-practice-contest-2/challenges/guardians-lunatics-ioi14/editorial>
