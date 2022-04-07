<!-- TOC -->

- [_11.1 The Rendering Equation](#_111-the-rendering-equation)
- [_11.2 General Global Illumination](#_112-general-global-illumination)
  - [_11.2.1 Radiosity](#_1121-radiosity)
  - [_11.2.2 Ray Tracing](#_1122-ray-tracing)
- [_11.3 Ambient Occlusion](#_113-ambient-occlusion)
  - [_11.3.1 Ambient Occlusion Theory](#_1131-ambient-occlusion-theory)
  - [_11.3.2 Visibility and Obscurance](#_1132-visibility-and-obscurance)
  - [_11.3.3 Accounting for Interreflections](#_1133-accounting-for-interreflections)
  - [_11.3.4 Precomputed Ambient Occlusion](#_1134-precomputed-ambient-occlusion)
  - [_11.3.5 Dynamic Computatio of Ambient Occlusion](#_1135-dynamic-computatio-of-ambient-occlusion)

<!-- /TOC -->

**Global Illumination**

现在我们知道reflection equation:
$$L_o(p, v) = \int_{l \in \Omega}L_i(p, l)f(l, v)(l \cdot n)^+ dl$$

<a id="markdown-_111-the-rendering-equation" name="_111-the-rendering-equation"></a>
### _11.1 The Rendering Equation

reflection equation是rendering equation的特殊情况, rendering equation是这样的:
$$L_o(p, v) = L_e(p, v) + \int_{l \in \Omega}L_o(r(p, l), -l)f(l, v)(l \cdot n)^+dl$$
rendering equation相比于reflection equation有两处变化:  
一是多了$L_e(p, v)$, 代表从p点主动发出的radiance.  
二是$L_i(p, l)$变成了L_o(r(p, l), -l), 这个代表p的上一个点沿着l方向发射和反射的radiance, **这是一个recursive的过程**, 因为我们看到的p点反射过来的光是p点接收到的其他位置的光. $r(p, l)$是ray casting funtion, 代表上一个点.  

有一种标记法来表示复杂的光照弹射过程, E代表眼睛, L代表光源, S代表specular surface, D代表diffuse surface, $*$代表0 or more, $+$代表1 or more, $?$代表0 or 1, $|$代表or, $()$代表group.  
rendering equation就可以表示为$L(S|D)*E$

因为rendering equation是一个recursive的过程, 所以计算会很复杂, 但是real time rendering对效率要求很高. 一般有两种方法来提升效率:  
1. simplify: 比如假设光线的弹射表面都是diffuse surface.
2. precompute: 预计算

### _11.2 General Global Illumination

reflection equation的入射光是给定的, rendering equation的入射光是从别的地方发出和反射的.  
rendering equation有太多的计算量, 对real time rendering来说效率太低, 但是对于一些静态的场景我门可以提前计算好, 另外rendering equation有严谨的理论基础, 全局光照依赖于它, 需要在其基础之上进行计算和验证.

rendering equation有两种解法:  
1. finite element methods: 有限元方法, 比如radiosity算法
2. Monte Carlo methods: ray tracing是其中的一种  
第二种方法更为流行

#### _11.2.1 Radiosity

#### _11.2.2 Ray Tracing

这个就是采用monte carlo的方法做path tracing, GAMES101的作业里有实现.

### _11.3 Ambient Occlusion

global illumination指的是实现scene里多个surface之间的光线相互弹射的效果.  
上面讲的rendering equation需要大量的计算.  
本章讲一些替代的简单方法, 来实现一些global illumination的效果, 例如ambient occlusion(环境光遮蔽).

#### _11.3.1 Ambient Occlusion Theory

我们假定lambertian surface(往半球的各个方向均匀反射)的场景, 各个方向照射过来的环境光的radiance也是一个常值$L_A$, 那么我们能够计算得到反射的irridiance:
$$E(p, n) = \int_{l \in \Omega}L_A(n \cdot l)^+dl = \pi L_A$$
请注意, 这个equation假设p点的半球范围内都能被环境光照射到, 但是实际场景可能不是, 所以我们引入一个visibility的函数$v(p, l)$, 这样上面的等式就变成:
$$E(p, n) = L_A\int_{l \in \Omega}v(p, l)(n \cdot l)^+dl$$

我们把入射光的radiance去掉, 只留下visibility和cosine项, 并归一化, 得到ambient occlusion:
$$k_A(p) = \frac{1}{\pi}\int_{l \in \Omega}v(p, l)(n \cdot l)^+dl$$
$k_A(p)$等于0则代表p点完全被遮蔽, 等于1则半球范围内都能被环境光照射到.
这样irridiance就变成:
$$E(p, n) = k_A(p) \pi L_A$$

<img src="_images/real_time_rendering/ambient_occlusion.png">

看上面左图, 加上ambient occlusion之后, P0就会比P1暗, P1和P2被光照射的角度范围一样大, 但是光线和P1的法线更接近, 所以P1比P2更亮.

右图的三个方向指的是三个点可见范围的平均单位向量, 称之为*bent normal*, 可以用下面的等式计算得到:
$$n_{bent} = \frac{\int_{l \in \Omega}l\mathit{v}(l)(n \cdot l)^+dl}{\|\int_{l \in \Omega}l\mathit{v}(l)(n \cdot l)^+dl\|}$$

#### _11.3.2 Visibility and Obscurance

visibility function $\mathit{v}(l)$应该怎么判断呢?  
一种方法是从p点发射光线l, 如果l照射到了self object, 那么$v(l)$就是0, 否则就是1. 还有一种情况需要考虑, 物体一般都是放在某个平面上, 如果光线$l$照射到了这个平面, 也应该考虑在内, $v(l)$是0.  
这种方法在某些场景下是有问题的, 比如一个封闭的空间, 按照这个算法, 这个空间的表面的$v(l)$都是0, 显然这是不对的. 

另一种方法是用obscurance模糊的idea, 用distance mapping function $\rho(l)$来替代visibility function:
$$k_A = \frac{1}{\pi}\int_{l \in \Omega}\rho(l)(n \cdot l)^+dl$$

$\rho(l)$是设定一个距离$d_{max}$, 在这个距离内如果光线有碰撞, 则$\rho(l)$就是0, 否则, 如果超过这个距离, 不管有没有碰撞, $\rho(l)$都是1.  
这种算法不符合物理的, 但是效果不错, 在图形学里有很多这样的compromise.

#### _11.3.3 Accounting for Interreflections

根据上面ambient occlusion的算法得到的图, 会比完整的全局光照算法得到的图要暗.  
这是因为这个算法忽略了occlusion的反射光线, 将其的光照贡献算作0, but the reflected light by the occlusion will illuminate the shading point.  
一种方法是提高$k_A$的值, 来避免recursive计算这种反射的高开销.  
基于假设occlusion反射的radiance等于着色点发出的radiance, $k_A$做这样的调整:
$$k_A^{\prime} = \frac{k_A}{1-\rho_{ss}(1-k_A)}$$
$\rho_{ss}$是subsurface albedo.  
这样, 着色点发出的irridiance就变成:
$$E = \frac{\pi k_A}{1-\rho_{ss}(1-k_A)}L_i$$
这种方法能得到很好的效果, 但是很依赖于$\rho_{ss}$

#### _11.3.4 Precomputed Ambient Occlusion

Precomputing the ambient occlusion can accelerate the rendering process, this precompute process is called *baking*.

The most common way of precomputing ambient occlusion is via Monto Carlo methods.  
we pick N random directions and check the visibility, ambient occlusion can be computed by this equation:
$$k_A = \frac{1}{N}\sum_{i}^{N}\mathit{v}(l_i)(n \cdot l_i)^+$$
if calculate obscurance, then replace $\mathit{v}(l_i)$ with $\rho(l_i)$

This equation includes a cosine factor, which means directions close to the normal have a higher weight, so we can sample more directions which close to the normal. This sampling schema is called *Malley's method*.

ambient occlusion precomputation can performed on the CPU or GPU by library such as Embree or OptiX.

#### _11.3.5 Dynamic Computatio of Ambient Occlusion