---
categories: Algorithms
...

## Discrete Fourier transform in two dimensions
To compute the discrete Fourier transform of a two-dimensional array, first compute the one-dimensional discrete Fourier transform of each row, replacing the contents of the row with the result, and then do the same for the columns. [^1]

> TODO: Can this be generalized to higher dimensions?

## Problems
- [Just A Few More Triangles!](https://open.kattis.com/problems/moretriangles)
- [A+B Problem](https://open.kattis.com/problems/aplusb)
- [Tile Cutting](https://icpc.kattis.com/problems/tiles)
- [Another Fibonacci](https://www.codechef.com/JUNE15/problems/MOREFB)
- [Cipher Message 3](http://codeforces.com/gym/100285)
- [Matchings](https://open.kattis.com/problems/matchings)
- [Fuzzy Search](http://codeforces.com/contest/528/problem/D)
- [K-Inversions](https://open.kattis.com/problems/kinversions)
- [Power sets of power sets](https://projecteuler.net/problem=553)

## See also
- [Number theoretic transform]()
- [Fast Hadamard transform]()
- [Divide and conquer]()

## External links
- [Integer and polynomial multiplication](http://www.cs.princeton.edu/~wayne/kleinberg-tardos/pdf/05DivideAndConquerII.pdf)
- [Avoiding FFT precision issues (Russian)](http://codeforces.com/blog/entry/17130?locale=ru#comment-219836)
- [FFT and overflows](http://codeforces.com/blog/entry/47758)
- [MIPT fall training camp 2013, FFT contest](http://it-edu.mipt.ru/sites/default/files/131110b.pdf)
- [Polynomial Algorithms](http://docplayer.net/25594945-Lecture-4-polynomial-algorithms.html)


[^1]: <http://paulbourke.net/miscellaneous/dft/>