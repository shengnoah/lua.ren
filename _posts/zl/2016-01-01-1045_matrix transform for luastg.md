---
layout: post
title: matrix transform for luastg 
tags: [lua文章]
categories: [topic]
---
<p><del>Another time copy from css (first attempt is ease function)</del></p>

<h2 id="theory">Theory</h2>

<p>From css3 animation, we have matrix, matrix3d method to describe transform of a image affinely.</p>

<p>For 2d image:
0. translate, skew and scale according to (0, 0)</p>

<script type="math/tex; mode=display">% <![CDATA[
begin{bmatrix} x \ y \ 1 end{bmatrix}  
begin{bmatrix}
	text{scaleX} & text{skewX} & text{translateX} \
	text{skewY} & text{scaleY} & text{translateY} \
	0 & 0 & 1
 end{bmatrix} %]]></script>

<ol>
  <li>rotation according to (0, 0)</li>
</ol>

<script type="math/tex; mode=display">% <![CDATA[
begin{bmatrix} 
	cos{theta} & -sin{theta} & 0 \
	sin{theta} & cos{theta} & 0 \
	0 & 0 & 1
 end{bmatrix} %]]></script>

<p>Matrix is kinda linear transformation, so rotation matrix ($R$) + sketching matrix ($S$) = $T(=RS)$</p>

<p>If we have matrix $A$ (invertiable), we can use eigenvalue decomposition to tween, $A=VDV^{-1}$, if separated into 10 steps, $T = VD^{1/10}V^{-1}$</p>

<p>Also for matrix 3d on 2d image:</p>

<p>o. translate, skew and scale according to (0, 0, 0)</p>

<script type="math/tex; mode=display">% <![CDATA[
begin{bmatrix} x \ y \ 1 end{bmatrix}  
begin{bmatrix}
	text{scaleX} & text{skewX} & text{translateX} \
	text{skewY} & text{scaleY} & text{translateY} \
	text{skewZ} & text{translateZ} & text{scaleZ} \
	0 & 0 & 1
 end{bmatrix} %]]></script>

<p>o.rotation according to (x, y, z) <a href="http://blog.csdn.net/harryhare/article/details/9195053">rotation</a> <a href="https://zh.wikipedia.org/wiki/%E6%97%8B%E8%BD%AC%E7%9F%A9%E9%98%B5">wiki</a></p>

<script type="math/tex; mode=display">% <![CDATA[
mathcal{M}(hat{mathbf{v}},theta) = begin{bmatrix}
   cos theta + (1 - cos theta) x^2
 & (1 - cos theta) x y - (sin theta) z 
 & (1 - cos theta) x z + (sin theta) y & 0 
\
   (1 - cos theta) y x + (sin theta) z 
 & cos theta + (1 - cos theta) y^2
 & (1 - cos theta) y z - (sin theta) x & 0
\
   (1 - cos theta) z x - (sin theta) y
 & (1 - cos theta) z y + (sin theta) x
 & cos theta + (1 - cos theta) z^2 & 0
 \
 0 & 0 & 0 & 1
end{bmatrix} %]]></script>

<p>Similarily, we can have eigenvalue decomposiiton to make tweening animation. One point is remained that for 3d object projected to xy-plane, we should make parameter equation for each surface.</p>

<p>Finally we can use projection of surface to create beautiful bullets.</p>

<h2 id="realization">Realization</h2>

<p>Use pyqt as gui, matplotlib.pylab, mpl_toolkits.mplot3d as drawer.
To simplify this problem, we only draw edges of convex polyhedron on xy-plane.</p>

<h3 id="place-points-at-anticlockwise-direction-and-add-edges">Place points at anticlockwise direction and add edges</h3>

<p><a href="http://www.cnblogs.com/loveclumsybaby/p/3420795.html">anticlockwise sort</a></p>

<p>Calculate the mass center, $vec{OA}, vec{OB}$ as vector from mass center to point A, B. If $vec{OA} times vec{OB} &lt; 0$, then B is anticlockwise to A. Quick sort the whole point set. Add edges AB in the edges set</p>

<h3 id="solve-the-convex">Solve the convex</h3>

<p>Accordint to anticlockwise points set and edges set, judge one edge AB and one point C. If the $CBA &lt; 90^{circ}$ then it’s concave, error.</p>

<h2 id="example">Example</h2>

<p>Remain to be done</p>


                <hr/>

                
                
                
                <div class="ds-share" style="text-align: right" data-thread-key="/2016/05/13/LuaSTG_Matrix_Transform" data-title="Matrix Transform for LuaSTG" data-url="http://airtnp.github.io/2016/05/13/LuaSTG_Matrix_Transform/" data-images="http://airtnp.github.io/img/post-matrix.jpg" data-content="Another time copy from css (first attempt is ease function)

Theory

From css... | Airtnp 的博客 | Airtnp Blog ">
                    
                <hr/>
                </div>