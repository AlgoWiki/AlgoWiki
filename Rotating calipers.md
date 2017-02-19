---
categories: Algorithm techniques,Geometry algorithms
...

<!-- TODO: Factor this out into something reusable -->
<div style="float:right; border: 1px solid #ddd; background-color: #eee; padding: 5px; margin: 5px;">
<div style="">![Calipers](/Calipers.gif){width=300px}</div>
<div style="color: #444">Calipers</div>
</div>

There are many applications of rotating calipers, but I think the most basic one is to find the diameter of a convex polygon. (Finding the diameter of an arbitrary polygon can be reduced to this problem by first taking the convex hull.) Conceptually it's very simple. We put the polygon between the two jaws of a caliper, and tighten. We then rotate the calipers a full circle around the polygon, while keeping the caliper tight at all times. The answer is then the maximum distance between the two jaws at any point during the procedure.

The rotating calipers technique is basically just an "angle-sweep-line" version of this. We have two parallel lines that are on either side of the polygon, representing the jaws of the caliper. Initially we could, for example, let one of the lines touch the leftmost point of the polygon and the other line touch the rightmost point of the polygon, and say they are completely vertical (i.e. at an angle $\pi/2$). We then simulate this procedure by rotating the two lines at the same speed, keeping track of which two points the two lines are touching, and then only stopping at the moments these points change.

This can then be easily generalized to many applications:

* Minimum rectangle enclosing a convex polygon: Now we have two calipers that are orthogonal to each other, and again we rotate them around the polygon. This corresponds to having four lines in the simulation, i.e. two pairs of parallel lines, which are then orthogonal to each other so that the interior forms a rectangle.
* Minimum distance between two convex polygons: Now one of the lines touches one of the polygons, and the other line touches the other polygon. Furthermore, we will initially let one of the lines touch the leftmost point of the corresponding polygon, while the other line touches the rightmost point of the corresponding polygon, and again the lines start out vertical.

See the links below for further applications and more detailed explanations.

For some example code you can look at my [implementation](https://github.com/SuprDewd/CompetitiveProgramming/blob/master/code/geometry/rotating_calipers.cpp). I have a class that represents a single jaw of the caliper (i.e. a single line), and then below (commented out) is how I use that class to find the diameter of an arbitrary polygon. Note that the code uses doubles and angles (as that was more intuitive for me at the time), but a more robust implementation would only use integers.

## Problems

* [Robert Hood](https://open.kattis.com/problems/roberthood)
* [Fossil in the Ice](http://www.spoj.com/problems/TFOSS/)
* [Trash Removal](https://uva.onlinejudge.org/external/11/p1111.pdf)
* [Fence Orthogonality](https://open.kattis.com/problems/fenceortho)
* [Board game](http://amppz.ii.uni.wroc.pl/files/zadania_en.pdf)
* [Smallest Enclosing Rectangle](https://uva.onlinejudge.org/external/123/12307.pdf)

## See also
* [Sweep line]()

## External links
* [Antipodal points](http://www.tcs.fudan.edu.cn/rudolf/Courses/Algorithms/Alg_ss_07w/Webprojects/Qinbo_diameter/2d_alg.htm)
* [Rotating Calipers: An Efficient, Multipurpose, Computational Tool](http://sdiwc.net/digital-library/web-admin/upload-pdf/00001036.pdfThe)
* [COMPUTATIONAL GEOMETRY WITH THE ROTATING CALIPERS](http://digitool.library.mcgill.ca/webclient/StreamGate?folder_id=0&dvs=1469220661104~207&usePid1=true&usePid2=true)
