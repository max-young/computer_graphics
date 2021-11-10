<!-- TOC -->

- [_15.1 Curves](#_151-curves)
  - [_15.1.1 Parameterizations and Reparameterizations](#_1511-parameterizations-and-reparameterizations)
  - [_15.1.2 Piecewise Parametric Representations分片参数表示法](#_1512-piecewise-parametric-representations分片参数表示法)
  - [_15.1.3 Splines](#_1513-splines)
- [_15.2 Curve Properties曲线特征](#_152-curve-properties曲线特征)
  - [_15.2.1 Continuity](#_1521-continuity)
- [_15.3 Polynomial Pieces多项式分片](#_153-polynomial-pieces多项式分片)
  - [_15.3.1 Polynomial Notation多项式记号](#_1531-polynomial-notation多项式记号)
  - [_15.3.2 A Line Segment线段](#_1532-a-line-segment线段)
  - [_15.3.3 Beyond Line Segment](#_1533-beyond-line-segment)
  - [_15.3.4 Basis Matrices for Cubics立方基础矩阵](#_1534-basis-matrices-for-cubics立方基础矩阵)
  - [_15.3.5 Blending Functions混合函数](#_1535-blending-functions混合函数)
  - [Interpolating Polynomials插值多项式](#interpolating-polynomials插值多项式)
- [_15.4 Putting Pieces Together](#_154-putting-pieces-together)
  - [_15.4.1 Knots节点](#_1541-knots节点)
  - [_15.4.2 Using Independent Pieces](#_1542-using-independent-pieces)
  - [_15.4.3 Putting Segments Together](#_1543-putting-segments-together)
- [Cubics](#cubics)

<!-- /TOC -->

**Curves曲线**

<a id="markdown-_151-curves" name="_151-curves"></a>
### _15.1 Curves

curve又两种定义:
1. 在n维空间里连续的图像
2. 一维空间和n维空间的对应关系图

什么意思呢? 我们那现实中的例子来说明, 我们用笔画线, 那么这条线就是curve.  
我们可以说这条线是二维空间里连续的无限的点.  
也可以说是时间和点集的对应关系.  
在本章我们取第一种定义

曲线的点是无限的. 除了两个端点, 每个点都有两个相邻的点. 除了这条曲线两端是无限长的, 或者曲线是环状的.

我们需要在数学上定义曲线. 有些曲线有特定的名称: line segments(线段), circle(圆), elliptical(椭圆)  
还有很多不规则的曲线我们无法这么描述, 这些曲线称为free-form curve  

有三种定义曲线的方法:  
1. Implicit curve隐式曲线
例如二维曲线:
$$f(x, y) = 0$$
2. Parametric curve参数曲线
例如第二种定义, 时间和点的对应关系:
$$(x, y) = f(t)$$
某个时间点t对应的二维向量(x, y)
3. Generative or procedural curve  
还不太理解, 大概是说描述曲线的生成过程

例如一个半径为1的圆, 用implict form可以表示为:
$$af(x, y) = x^2 + y^2 - 1 = 0$$
用parametric form可以表示为:
$$(x, y) = f(t) = (\cos t, \sin t), t \in [0, 2\pi)$$

有些曲线可以用第一种方式表达, 有些方式用第二种方式表达很难.  
但是第二种方式更直观, 更容易采样.  
每种方式都有自己的优缺点, 在图形学里我们聚焦在第二种parametric form.

<a id="markdown-_1511-parameterizations-and-reparameterizations" name="_1511-parameterizations-and-reparameterizations"></a>
#### _15.1.1 Parameterizations and Reparameterizations

根据Parametric form的定义, 我们能确定一个时间段内, 某个时间点t对应的点的位置.  
为了方便起见, 我们定义这个时间段为单位时间, 也就是0到1, 这种定义方式我们把参数记作u    

但是实际情况可能不是单位时间, 不用担心, 我们可以转换, 例如一段曲线的时间t是a到b,那么:
$$g(u) = a + (b - a)u$$
这样, 之前用f(t)来表示的曲线, 可以用一个新的parametric form来表示:
$$f_2(u) = f(g(u))$$
这种转换称为reparametrization, 我们可以转换到任意时间段.

有一个问题是, 我们拿笔画线的速度可能不是匀速的, 可能前半部分慢, 后半部分快.  
这样, 我们无法确定某个时间点对应的位置. u = 0.5时, 对应的位置不一定在曲线的中点.  

我们假设速度时均匀的, 这种表达方式更为natural.  
如果速度时匀速的, 那么我们就可以实现用时间来计算曲线的长度了. 所以我们称之为arc-length parameterization, 参数用s表示.

想想导数的概念, 导数的其中一种含义就是表示速率. 画曲线的速度是不变的, 也就是这种方式表示的曲线的导数是不变的:
$$\left|\frac{df(s)}{ds}\right|^2=c$$
我们计算0到v的曲线长度就是:
$$s = \int_{0}^{v}\left|\frac{df(t)}{dt}\right|^2dt$$

最后总结一下:  
我们用参数u来表示单位间隔的**free parameters**  
s来表示**arc-length free parameters**

#### _15.1.2 Piecewise Parametric Representations分片参数表示法

<img src='./_images/piecewise_parametric_representations.png' width=50%>  

对于规则的形状, 比如直线、圆, 我们很容易用parametric representation来表示  
对于复杂的图形, 就不太容易了, 例如上图的1和2. 我们可能得用分片的形式来表达:

$$
f(u) = 
\left\{
  \begin{array}{lr}
  f_1(2u)&\ if\ u \le 0.5, \\
  f_2(2u-1)&\ if\ u \gt 0.5
  \end{array}
\right.
$$

对于图形2, 我门可以分两个片段来表示, 一个是线段, 一个是圆弧.  
我们还可以只有一种类型的图形-线段, 来表示, 如图3, 虽然不太精确, 但是能表示大致的形状.  
如果分便足够细, 也是能够完美的表示图2的图形的.  
我们更倾向于用类型足够少的方式来表示图形. 

这涉及到权衡. 如同效率和精度的权衡一样.

#### _15.1.3 Splines

在计算机出现之前, 人们利用金属片来画曲线, 把金属片弯曲成想要的形状, 因为金属足够硬, 所以弯曲后足够平滑.  
我们把这种金属片称之为spline

数学家根据这个启发, 用spline来表示piecewise polynomial function分片多项式函数, 表示曲线时其非常有用. 

### _15.2 Curve Properties曲线特征

要描述一条曲线, 就要知道曲线的特征. 比如描述圆, 我们需要定义其圆心位置和半径, 描述椭圆, 需要定义其中心位置、长轴、长短轴比例.  
对于复杂的曲线, 怎么确定其特征呢?

假如我们在大雾天的铁轨上, 我们能知道我们所处的铁轨位置是直线还是曲线, 是不是起始点. 这些特性我们称之为*local properties*  
但是仅仅知道local properties还无法定义铁轨这条曲线, 因为我们无法确定铁轨是否是环线, 是否交叉, 有多长. 这些属性称之为*global properties*

对几何对象local property的研究, 称之为*differential geometry(差分集合)*, local property对于描述曲线非常重要, 其主要包括下面这些特性:
- continuity连续性
- 位置
- 方向
- 曲率(导数)

如果曲线包括某个点, 我们称这条曲线interpolate这个点.  
某个点v, 对于parametric representation的曲线, $f(t) = v$  
我们称interpolation的位置是t的value, the site.

#### _15.2.1 Continuity

在上面的_15.1.2章节里, 我们用piecewise parametric representation来表示曲线.  
对于两个pieces, 如果$f_1(1) \ne f_2(0)$, 说明这两个片段是断开的, 不连续的. 根据曲线的定义, 这种情况不能称之为曲线.  
如果两者相等, 那我们说这个曲线在这个位置是连续的continuity

我们引入导数这个概念, 导数是分阶的, 一阶导数的导数就是二阶导数.  
如果两个片段的连接处一阶导数相等, 即$f_1^{\prime}(1) = f_2^{\prime}(0)$, 那我们称之为$C^1$连续, 依此类推  
对于上面的这种情况, 我们称之为$C^0$连续  
下面的图比较直观:  
<img src='./_images/curve_continuity.png' width=50%>  

对于上面的定义, 两个分片的速率是一样的, 称之为parametric continuity参数连续  
如果速率是不一样呢? 我们称之为geometric continuity. 如图中的右侧, 两个片段又相同的方向, 不同的大小.  
数学上这么表示:  
$C^1$连续是:
$$f_1^{\prime}(1) = f_2^{\prime}(0)$$
$G^1$连续是:
$$f_1^{\prime}(1) = kf_2^{\prime}(0)$$

如果一个曲线是$C^n$连续, 那么它一定是$G^n$连续

### _15.3 Polynomial Pieces多项式分片

在图形学里, 最广泛的曲线表示法是用多项式分片, 每个分片用基本的元素表示.

#### _15.3.1 Polynomial Notation多项式记号

多项式是这样的格式:
$$f(t) = a_0 + a_1t + a_2t^2 + ... + a_nt^n$$
$a_i$是polynomial多项式的coefficients系数, $n$是polynomial多项式的degree阶  
我们还可以简单表示为:
$$f(t) = \sum_{i=0}^{n}a_it^i$$
这种形式称之为polynomial的canonical form经典形式  
还可以表示为:
$$f(t) = \sum_{i=0}^{n}c_ib_i(t)$$
$b_i(t)$是多项式, $t^i$就是$b_i(t)$的一种形式

例如, 我们可以对一个二维曲线用两个多项式来表示, 一个多项式表示x的变化, 另外一个表示y的变化.  
也可以用一个多项式, 来表示(x, y)的变化

#### _15.3.2 A Line Segment线段

如果我们知道一条线段的两个端点$p_0$和$p_1$, 那么我们可以这样表示这条线段:  
$$f(u) = (1-u)p_0 + up_1$$
这里我们可以理解为两个多项式的表达, 分别代表x和y的变化

我们换一个思路, 用多项式来表达线段:
$$f(u) = a_0 + ua_1$$
我们把u写成一个向量, $a_i$也写成一个向量:
$$u = [0, u, u^2, ...]$$
$$a = [a_0, a_1, ...]$$
那么:
$$f(u) = u.a$$
我们需要求得$a_0$和$a_1$, 才能定义这条线段

那么对于两个端点$p_0$和$p_1$  
$$
p_0 = f(0) = [1, 0][a_0, a_1] \\
p_1 = f(1) = [1, 1][a_0, a_1]
$$
这样, $a_0 = p_0$, $a_1 = p_1 - p_0$, 从而:

**Matrix Form for Polynomials多项式的矩阵形式**

我们将上面两个等式可以写作:

$$
\begin{bmatrix}
p_0 \\ p_1
\end{bmatrix}
= \begin{bmatrix}
1 & 0 \\ 1 & 1
\end{bmatrix}
\begin{bmatrix}
a_0 \\ a_1
\end{bmatrix}
$$
简单记作:
$$p = Ca$$
我们把矩阵C的逆记作B, 那么我们就能得到$a = C^{-1}p = Bp$, 这样:

$$f(u) = uBp$$

我们称C为**constraint matrix(约束矩阵)**, B为**basis matrix(基本矩阵)**

举个实例, 假如我们知道线段的中点$p_0$和结束端点$p_1$, 那么:

$$
p_0 = f(0.5) = a_0 + 0.5a_1 \\
p_1 = f(1) = a_0 + a_1
$$

这样C就等于:
$$
\begin{bmatrix}
1 & 0.5 \\ 1 & 1
\end{bmatrix}
$$
我们就得到
$$
B = C^{-1} = \begin{bmatrix}
2 & -1 \\ -2 & 2
\end{bmatrix} 
$$
p是
$\begin{bmatrix}p_0x \\ p_1x\end{bmatrix}$以及$\begin{bmatrix}p_0y \\ p_1y\end{bmatrix}$  
这样我们就能用x和y方向上的两个多项式来表示这个线段了

#### _15.3.3 Beyond Line Segment

上一节line segment的多项式表达提供了很好的范例, 我们可以据此推导出高阶的多项式表达曲线.
例如2阶多项式:
$$f(u) = a_0 + ua_1 + u^2a_2$$
加入我们知道这个曲线上的起点$p_0$, 中点$p_1$, 终点$p_2$, 那么:
$$
p_0 = f(0) = a_0 + 0a_1 + 0^2a_2 \\
p_1 = f(0.5) = a_0 + 0.5a_1 + 0.5^2a_2 \\
p_2 = f(1) = a_0 + 1a_1 + 1^2a_2
$$
从而
$$
C = \begin{bmatrix}
1 & 0 & 0 \\ 1 & 0.5 & 0.25 \\ 1 & 1 & 1
\end{bmatrix}
$$
从而我们计算出
$$
B = C^-1 = \begin{bmatrix}
1 & 0 & 0 \\ -3 & 4 & -1 \\ 2 & -4 & 2
\end{bmatrix}
$$
从而我们能得到这个二阶多项式的参数[a_0, a_1, a_2]

我们再看看导数. 对于二阶表达式的曲线, 一阶导数表示了其行进方向, 二阶导数表示了其方向变化的速率. $a^n$的导数是$na^{n-1}$(参见维基百科), 那么对于二阶多项式:
$$
f(u) = a_0 + ua_1 + u^2a_2 \\
f^{\prime}(u) = \frac{df}{du} = a_1 + 2ua_2 \\
f^{\prime\prime}(u) = \frac{d^2f}{du^2} = \frac{df^{\prime}u}{du} = 2a_2 \\
$$
我们可以归纳为:
$$
f^{\prime}(u) = \sum_{i=1}^{n}iu^{i-1}a_i \\
f^{\prime\prime}(u) = \sum_{i=2}^{n}i(i-1)u^{i-2}a_i
$$
这样, 假如我们知道中点的位置、一阶导数、二阶导数, 那么:
$$
\begin{aligned}
&f(0.5) = a_0 + &0.5&a_1 + &0.5^2&a_2 \\
&f^{\prime}(0.5) = &&a_1 + &2*0.5&a_2 \\
&f^{\prime\prime}(0.5) = &&&2&a_2
\end{aligned}
$$
这样
$$
C = \begin{bmatrix}
1 & 0.5 & 0.25 \\ 0 & 1 & 1 \\ 0 & 0 & 2
\end{bmatrix}
$$
从而我们得到
$$
B = C^{-1} = \begin{bmatrix}
1 & -0.5 & 0.125 \\ 0 & 1 & -0.5 \\ 0 & 0 & 0.5
\end{bmatrix}
$$
从而我们可以得到$a = Bp$, 中点的位置、一阶导数、二阶导数都可以用(x, y)的形式表示.

#### _15.3.4 Basis Matrices for Cubics立方基础矩阵

对于三阶多项式曲线, 如果我们知道起始点的位置和一阶导数, 那么:
$$
f(0) = a_0 + 0^1a_1 + 0^2a_2 + 0^3a_3 \\
f^{\prime}(0) = a_1 + 2\times0^1a_2 + 3\times0^2a_3 \\
f(1) = a_0 + 1^1a_1 + 1^2a_2 + 1^3a_3 \\
f^{\prime}(0) = 1\times1^0a_1 + 2\times1^1a_2 + 3\times1^2a_3 \\
$$
我们就得到
$$
C= \begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
1 & 1 & 1 & 1 \\
0 & 1 & 2 & 3
\end{bmatrix}
$$
从而得到
$$
B = C^{-1} = \begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
-3 & -2 & 3 & -1 \\
2 & 1 & -2 & 1
\end{bmatrix}
$$
从而得到这个三次方多项式表达的曲线

#### _15.3.5 Blending Functions混合函数

通过上面的章节, 我们知道了basis matrix B, $f(u) = uBp$, 我们把$uB$记作:
$$b(u) = uB$$
$b(u)$称之为blending function, 因为这个函数通过blending控制点(p)从而定义了曲线  
我们把$f(u)$抽象为:
$$f(u) = \sum_{i=0}^{n}b_i(u)p_i$$
这个等式的意义也在于此.

#### Interpolating Polynomials插值多项式

通过上面的章节我们了解到, 一个n阶多项式表示的曲线, 参数的个数是n+1.  
我们需要n+1个控制点来推到定义这条曲线.  
假设有$[p_0, p_1, ..., p_n]$个点, 以及其对应的参数值(时间点)$[t_0, t_1, ..., t_n]$  
那么我们就能得到$(n+1)\times(n+1)$的basis matrix

后面涉及到lagrange form, 看不懂  
参照[拉格朗日插值](https://zh.wikipedia.org/wiki/%E6%8B%89%E6%A0%BC%E6%9C%97%E6%97%A5%E6%8F%92%E5%80%BC%E6%B3%95)  
以及[《数值分析》](https://book.douban.com/subject/26600495/)

### _15.4 Putting Pieces Together

上一章节我们了解到曲线片段的多项式表达. 现在我们看看怎么把这些片段组合在一起 

#### _15.4.1 Knots节点

<img src="./_images/knots.png" width=50%>  

如上图, 两个直线片段, 三个点$p_1, p_2, p_3$, 如何将其组合在一起呢?  
三个点分别是两个直线片段的起始点, 这些点称之为knots  

有两种方法进行组合, 其实上一章节已经说到了  
$$
f(u) = 
\left\{
  \begin{array}{lr}
  f_1(2u)&\ if\ u \le 0.5, \\
  f_2(2u-1)&\ if\ u \gt 0.5
  \end{array}
\right.
$$

另外一种方法:
$$
f(u) = p_1b_1(u) + p_2b_2(u) + p_3b_3(u)
$$
这种方法好像是来表示多项式曲线的, 但是这两条线段也是曲线.  
$$
b_1(u) = 
\left\{
  \begin{array}{lr}
  1-2u&\ if\ 0\le u \lt 0.5, \\
  0&\ otherwise
  \end{array}
\right.
$$
$b_2(u)$和$b_3(u)$类似, 如图所示  
如何求这几个$b_i(u)$呢? 不知道

#### _15.4.2 Using Independent Pieces

分段组合时, 争端曲线的参数时单位参数, 每个分段参数也要是单位参数. 这就需要做转换. 

#### _15.4.3 Putting Segments Together

对于两条line segment, 有三种方式将它们连接在一起, 从易到难分别是:  
1. shared-point schema: 共用端点
2. dependency schema: 复制前一个线段的终点赋值给后一个线段的起点
3. 用显式等式定义连接关系

这里重点说明前两种方式.
第一种方式最简单最推荐. 但是如果我们要用线段上的起点和其他点(例如中点)作为控制点, 那么就必须用到第二种方式了.  
计算出终点, 然后赋值给下一个线段的起点.  
这就涉及到locality了.  
第一种方式, 如果我们改变其中一个控制点, 只会影响到相邻的两条线段, 这称为具有locality, 只影响局部.  
第二种方式, 如果改变其中一个控制点, 就会传递影响整个曲线, 我们称为lack of locality.  
如下图所示:  
<img src="./_images/putting_segments_together.png" width=50%>

### Cubics

在用分片多项式来表示曲线式, 我们一般使用三次多项式, 因为:
- 三次多项式支持$C^2$ continuity, $C^2$ continuity满足绝大部分需求
- Cubic curves provide the minimum-curvatureinterpolants to a set of points. 三次多项式提供最小曲率内插
- 三次多项式在首尾俱有很好的位置和导数对称性
- 三次多项式在计算和平滑之间有很好的平衡

三次多项式的canonical form经典形式是:
$$f(u) = a_0 + a_1u + a_2u^2 + 1_3u^3$$

曲线表达完美的形式应该具有下面的特征:
1. 每个片段都是三次方
2. 曲线经过控制点
3. 具有local control特性
4. 曲线具有$C^2$ continuity

但是没有一种表达能同时具有这4种特性.  
之后讲到的几种表达里:  
B-splines具有local control特性, 也具有$C^2$ continuity特性, 但是不interpolate控制点.
Cardinal splines和Catmull-Rom splines interpolate控制点, 具有local control特性, 但是不是$C^2$ continuity.  
natural cubics interpolcate控制点, 也具有$C^2$ Continuity, 但是不能local control



