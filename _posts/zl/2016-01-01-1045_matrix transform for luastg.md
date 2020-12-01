---
layout: post
title: matrix transform for luastg 
tags: [lua文章]
categories: [topic]
---
~~Another time copy from css (first attempt is ease function)~~

## Theory

From css3 animation, we have matrix, matrix3d method to describe transform of
a image affinely.

For 2d image: 0\. translate, skew and scale according to (0, 0)

  1. rotation according to (0, 0)

Matrix is kinda linear transformation, so rotation matrix ($R$) + sketching
matrix ($S$) = $T(=RS)$

If we have matrix $A$ (invertiable), we can use eigenvalue decomposition to
tween, $A=VDV^{-1}$, if separated into 10 steps, $T = VD^{1/10}V^{-1}$

Also for matrix 3d on 2d image:

o. translate, skew and scale according to (0, 0, 0)

o.rotation according to (x, y, z)
[rotation](http://blog.csdn.net/harryhare/article/details/9195053)
[wiki](https://zh.wikipedia.org/wiki/%E6%97%8B%E8%BD%AC%E7%9F%A9%E9%98%B5)

Similarily, we can have eigenvalue decomposiiton to make tweening animation.
One point is remained that for 3d object projected to xy-plane, we should make
parameter equation for each surface.

Finally we can use projection of surface to create beautiful bullets.

## Realization

Use pyqt as gui, matplotlib.pylab, mpl_toolkits.mplot3d as drawer. To simplify
this problem, we only draw edges of convex polyhedron on xy-plane.

### Place points at anticlockwise direction and add edges

[anticlockwise sort](http://www.cnblogs.com/loveclumsybaby/p/3420795.html)

Calculate the mass center, $vec{OA}, vec{OB}$ as vector from mass center to
point A, B. If $vec{OA} times vec{OB} < 0$, then B is anticlockwise to A.
Quick sort the whole point set. Add edges AB in the edges set

### Solve the convex

Accordint to anticlockwise points set and edges set, judge one edge AB and one
point C. If the $CBA < 90^{circ}$ then it’s concave, error.

## Example

Remain to be done

* * *

* * *