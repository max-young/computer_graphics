<!-- TOC -->

- [_2 Miscellaneous Math多样的数学](#_2-miscellaneous-math多样的数学)
  - [_2.1 Sets and Mappings](#_21-sets-and-mappings)
    - [_2.1.1 Inverse Mappings逆向映射](#_211-inverse-mappings逆向映射)
    - [_2.1.2 Intervals区间](#_212-intervals区间)
    - [_2.1.3 Logarithms对数](#_213-logarithms对数)
  - [_2.2 Solving Quadratic Equations求解二次方程](#_22-solving-quadratic-equations求解二次方程)
  - [_2.3 Trigonometry三角几何](#_23-trigonometry三角几何)
    - [_2.3.1 Angles角度](#_231-angles角度)
    - [_2.3.2 Trigonometrics Functions三角函数](#_232-trigonometrics-functions三角函数)
    - [_2.3.3 Userful Identities](#_233-userful-identities)
  - [_2.4 Vectors向量](#_24-vectors向量)
    - [_2.4.1 Vector Operations](#_241-vector-operations)
    - [_2.4.2 Cartesian Coordinates of a Vector](#_242-cartesian-coordinates-of-a-vector)
    - [_2.4.3 Dot Product点积](#_243-dot-product点积)
    - [_2.4.4 Cross Product叉积](#_244-cross-product叉积)
    - [_2.4.5 Orthonormal Bases and Coordinate Frames正交基和坐标系](#_245-orthonormal-bases-and-coordinate-frames正交基和坐标系)
    - [_2.4.6 Constructing a Basis from a Single Vector根据一个向量来构建正交基](#_246-constructing-a-basis-from-a-single-vector根据一个向量来构建正交基)
    - [_2.4.7 Constructing a Basis from Two Vectors](#_247-constructing-a-basis-from-two-vectors)
    - [Squaring Up a Basis修复](#squaring-up-a-basis修复)
  - [_2.5 Curves and Surfaces曲线和曲面](#_25-curves-and-surfaces曲线和曲面)
    - [_2.5.1 2D Implicit Curves 2D隐式曲线](#_251-2d-implicit-curves-2d隐式曲线)
    - [_2.5.2 The 2D Gradient 2D梯度](#_252-the-2d-gradient-2d梯度)
      - [Implicit 2D lines隐式2D直线](#implicit-2d-lines隐式2d直线)
      - [Implicit Quadric Curves隐式二次曲线](#implicit-quadric-curves隐式二次曲线)
  - [_2.6 Linear Interpolation线性插值](#_26-linear-interpolation线性插值)
  - [_2.7 Triangles三角形](#_27-triangles三角形)
    - [_2.7.1 2D Triangles](#_271-2d-triangles)
  - [Frequently Asked Questions](#frequently-asked-questions)
  - [notes](#notes)
  - [Exercises](#exercises)

<!-- /TOC -->

<a id="markdown-_2-miscellaneous-math多样的数学" name="_2-miscellaneous-math多样的数学"></a>
## _2 Miscellaneous Math多样的数学

一些图形工作只是将数学直接转换为代码. 数学越清晰, 代码越简洁. 本章只提及图形学相关的数学知识作为参考.

<a id="markdown-_21-sets-and-mappings" name="_21-sets-and-mappings"></a>
### _2.1 Sets and Mappings

Mappings映射, 也被成为functions函数, 是数学和编程的基础.  
一个mapping映射是一个type映射到另外一个类型的type, 在这里我们需要定义set集合来包含这些type.  

一个对象属于一个集合, 可以表示为:
$$a\in S$$  
意思是"a is a member of set S"  
有两个集合$A$和$B$, 它们的Cartesian product笛卡尔积, 可以表示为$A\times B$, 它表示一个集合, 元素是所有可能的配对$(a, b)$, $a \in A$, $b \in B$  
$A \times A$可以简化为$A^2$, 这是2维的, 我们可以扩展为3维、多维  

一些特定的集合:
- $\R$: the real numbers
- $\R^+$: the nonnegative real numbers(includes zero)
- $\R^2$: 2D实数的配对
- $\R^n$: n维笛卡尔积
- $\Z$: the integers
- $S^2$: 球面上的3D笛卡尔积点的集合

mapping映射可以这么表示
$$f: \R \mapsto \Z$$
意思是"There is a function called f that takes a real number as input and maps it to an integer"  
箭头左侧的集合称之为domain, 右侧称之为target  
在计算机程序里可以表达为: "There  is a function called f which has one real argument and returns an integer"
$$integer\ f(real) \gets equivalent \to f:\R \mapsto \Z$$

$f(a)$成为$a$的image, "the image of the whole domain is called the range of the function"

<a id="markdown-_211-inverse-mappings逆向映射" name="_211-inverse-mappings逆向映射"></a>
#### _2.1.1 Inverse Mappings逆向映射

如果一个function: $f: A \mapsto B$, 那么可能存在inverse function: $f^-1: B \mapsto A$  
另一种表示方法是: $b = f(a)$, $f^-1(b) = a$  
这个定义需要满足这样的条件: target B里的所有元素b都是domain中的某个元素a的image(换句话说, range和target相等), 并且a是唯一的.  
那么我们称这样的mappings或者functions为*bijections(双射)*  
形象一点的描述: 一群骑手, 一群马, 每个骑手都骑一匹马, 每匹马都有骑手骑.  

举两个例子:
- bijection: $f:\R \mapsto \R, f(x) = x^3$
- not bijection: $sqr: \R \mapsto \R, sqr(x) = x^2$, 因为target中的元素对应两个domain中的元素

<a id="markdown-_212-intervals区间" name="_212-intervals区间"></a>
#### _2.1.2 Intervals区间

我们通常想指定函数只处理值在限定范围的实数, 我们称之为我们制定了interval区间  
比如interval区间是0和1之间的实数, 不包括0和1, 我们表示为$(0, 1)$  
因为不包括端点, 我们称之为open interval开区间, 与之对应的是closed interval闭区间, 依照上面的例子, 可以表示为$[0, 1]$  
也有半开区间, 只包含其中一个端点
我们可以应用到区间笛卡尔积, 比如表示单位立方体的点集, 可以用$X \in [0, 1]^3$, X是$(a, b, c)$, abc都属于$[0, 1]$这个区间

intervals和set操作结合非常有用, 举例说明:  
$[3,5) \cap [4, 6] = [4, 5)$  
$[3, 5) \cup [4, 6] = [3, 6]$  
$[3, 5) - [4, 6] = [3, 4)$  
$[4, 6] - [3, 5) = [5, 6]$

<a id="markdown-_213-logarithms对数" name="_213-logarithms对数"></a>
#### _2.1.3 Logarithms对数

每个对数都有一个base a, x的log base a可以写作$log_a x$, 指的是a要提高到x的指数(exponent), 例如:
$$ y = log_ax \iff a^y = x$$

根据这个定义, 我们可以得到:
$$
\begin{aligned}
a^{log_a(x)} &= x\\
log_a(a^x) &= x\\
log_a(xy) &= log_ax + log_ay\\
log_a(x/y) &= log_ax - log_ay\\
log_ax &= log_ab\ log_bx
\end{aligned}
$$

之后还讲到了微积分的特殊数字$e = 2.718...$
$$ln\ x \equiv log_ex$$
知识盲区, 以后再补

<a id="markdown-_22-solving-quadratic-equations求解二次方程" name="_22-solving-quadratic-equations求解二次方程"></a>
### _2.2 Solving Quadratic Equations求解二次方程

Quadratic Equation可以表示为:
$$Ax^2 + Bx + C = 0$$
这一节的图Figure2.5很有意思, 我们根据$y = Ax^2 + Bx + C$可以画出一个倒抛物线  
$Ax^2 + Bx + C = 0$的root(解/根)就是y=0的线(也就是x轴)和抛物线相交的点  
抛物线可以misses(错过), grazes(掠过), hits(撞击)x轴, 分别得到0、1、2个解

我们可以推导出:
$$x = \frac{-B \pm \sqrt{B^2 - 4AC}}{2A}$$
有几个root取决于$B^2 - 4AC$, 我们表示为:
$$D \equiv B^2 - 4AC$$
D成为这个二次方程的discriminant(判别式), 如果$D > 0$, 则有两个实数解, 如果$D = 0$, 则有1个实数解, 如果$D < 0$, 则没有实数解答

<a id="markdown-_23-trigonometry三角几何" name="_23-trigonometry三角几何"></a>
### _2.3 Trigonometry三角几何

#### _2.3.1 Angles角度

两条half_lines(direction)组成的角度angle如何表示呢?  
交点为圆心画一个单位圆(半径为1), 两条线切的圆弧长度可以用来表示角度, 一般用较短的圆弧来表示角度.  
这种表示方法的angle的单位是**radians**  
两条线切的圆弧有长的一段和短的一段, 两个角度加起来就是单位圆的周长$2\pi$

还有另外一种表示方法叫**degrees**, 半圆的角度用radians表示是$\pi$radians, 用degrees表示是180degrees, 符号是$180^{\circ}$

两种表示方法的换算关系是:
$$degrees = \frac{180}{\pi}radians$$
$$radians = \frac{\pi}{180}degrees$$

#### _2.3.2 Trigonometrics Functions三角函数

勾股定理的证明  
以及sin, cos, tan, cot等的定义

#### _2.3.3 Userful Identities

sin, cos, tan, cot的一些等式

<a id="markdown-_24-vectors向量" name="_24-vectors向量"></a>
### _2.4 Vectors向量

vector描述了长度和方向. 如果两个vector的长度和方向相同, 即时我们想象它们在不同的位置, 它们依然是相等的.  
vector一般用粗体字母表示, 比如$\bm{a}$, vector的长度可以这么表示: $\parallel\bm{a}\parallel$  
unit vector的长度是1, zero vector的长度是0, 方向没有定义  

vector可用来表示offset(偏移)

<a id="markdown-_241-vector-operations" name="_241-vector-operations"></a>
#### _2.4.1 Vector Operations

vector的加法和减法, 画图就清晰明了了

<a id="markdown-_242-cartesian-coordinates-of-a-vector" name="_242-cartesian-coordinates-of-a-vector"></a>
#### _2.4.2 Cartesian Coordinates of a Vector

根据上面的加法, 一个2D的vector可以是两个不平行的2D vector相加:
$$\bm{c} = a_c\bm{a} + b_c\bm{b}$$
$a_c$和$b_c$分别代表$\bm{a}$和$\bm{b}$的weight(或者是长度), $\bm{a}$和$\bm{b}$指basic vector, 意思是只有方向没有长度的vector. 为什么这么表示呢? 因为长度是和$\bm{c}$相关的, 看下面就理解了.

那么假如$\bm{a}$和$\bm{b}$是垂直相交的呢? 它们组成Cartesian coordinate system直角坐标系, vector就能表示为:
$$\bm{a} = x_a\bm{x} + y_a\bm{y}$$
$\bm{a}$的长度就是:
$$\parallel\bm{a}\parallel = \sqrt{x^{2}_{a}+y^{2}_{a}}$$
有了$x_a$和$y_a$, 方向也能确定, 所以向量可以用一个列矩阵column matrix表示:
$$
\begin{bmatrix}
x_a \\
y_a
\end{bmatrix}
$$
也可以行矩阵row matrix表示:
$$\bm{a}^T = \left[ x_a\ y_a \right]$$
这是2D vector的表示方法, 同样的, 我们也可以用这种方法表示3D, 4D等vector

<a id="markdown-_243-dot-product点积" name="_243-dot-product点积"></a>
#### _2.4.3 Dot Product点积

上面说的是两个vector的加减法, 那么可以相乘吗?  
两个vector相乘可以用点乘(dot product), 结果是一个scalar product(标量)
$$\bm{a}\cdot\bm{b} = \parallel\bm{a}\parallel\parallel\bm{b}\parallel \cos\phi$$
可以理解为a在b上的垂直(right angle)投影(projection)长度乘以另一个b的长度, **这是计算机图形学里最重要的公式之一**  
a在b上的垂直投影长度可以表示为:
$$\bm{a}\to\bm{b} = \parallel\bm{a}\parallel\cos\phi = \frac{\bm{a}\cdot\bm{b}}{\parallel\bm{b}\parallel}$$

dot product满足以下特性:  
$$
\bm{a}\cdot\bm{b} = \bm{b}\cdot\bm{a} \\
\bm{a}\cdot(\bm{b} + \bm{c}) = \bm{a}\cdot\bm{b} + \bm{a}\cdot\bm{c} \\
(k\bm{a})\cdot\bm{b} = \bm{a}\cdot(k\bm{b}) = k\bm{a}\cdot\bm{b}
$$

向量$\bm{a}$
$
\begin{bmatrix}
x_a \\
y_a
\end{bmatrix}
$
和向量$\bm{b}$
$
\begin{bmatrix}
x_b \\
y_b
\end{bmatrix}
$的dot product如何计算呢?  
$$
\begin{aligned}
\bm{a}\cdot\bm{b}&=(x_a\bm{x}+y_a\bm{y})\cdot(x_b\bm{x}+y_b\bm{y}) \\
&=x_ax_b(\bm{x}\cdot\bm{x}) + x_ay_b(\bm{x}\cdot\bm{y}) + y_ax_b(\bm{y}\cdot\bm{x}) + y_ay_b(\bm{y}\cdot\bm{y}) \\
&=x_ax_b + y_ay_b
\end{aligned}
$$
这里假定$\bm{x}\cdot\bm{x} = \bm{y}\cdot\bm{y} = 1$, $\bm{x}\cdot\bm{y} = 0$  
同样的, 对于3D vector:  
$$\bm{a}\cdot\bm{b} = x_ax_b + y_ay_b + z_az_b$$

<a id="markdown-_244-cross-product叉积" name="_244-cross-product叉积"></a>
#### _2.4.4 Cross Product叉积

Cross product只应用在三维向量领域, 因为两个向量的cross product的结果还是一个向量, 不像dot product的结果是一个scalar标量, 没有方向, 只有长度.  
而且cross product的结果向量的方向还不和原始的两个向量在一个平面上, 是垂直于这两个向量的平面的, 方向遵守右手螺旋定则  
也就是说$\bm{a} \times \bm{b}$得出的向量的方向, 我们可以用右手从$\bm{a}$旋转到$\bm{b}$, 大拇指的方向就是cross product结果向量的方向  
那么长度呢? 长度是$\bm{a}$和$\bm{b}$组成的平行四边形的面积(the area of parallelogram), 也就是:

$$\parallel \bm{a} \times \bm{b} \parallel = \parallel \bm{a} \parallel \parallel \bm{b} \parallel \sin\phi$$

我们假设一个三维坐标系, $\bm{x}$, $\bm{y}$, $\bm{z}$是三个坐标轴的三维向量, 也就是:

$$
\begin{aligned}
\bm{x} = (1, 0, 0), \\
\bm{y} = (0, 1, 0), \\
\bm{z} = (0, 0, 1),
\end{aligned}
$$

这样就好理解了, 他们的cross product分别是:

$$
\begin{aligned}
\bm{x} \times \bm{y} = + \bm{z} \\
\bm{y} \times \bm{x} = - \bm{z} \\
\bm{y} \times \bm{z} = + \bm{x} \\
\bm{z} \times \bm{y} = - \bm{x} \\
\bm{z} \times \bm{x} = + \bm{y} \\
\bm{x} \times \bm{z} = - \bm{y} \\
\end{aligned}
$$

另外, 因为$\sin\phi$的特性, $\bm{x} \times \bm{x} = 0$, 此外, cross product还有下面的特性:

$$
\begin{aligned}
\bm{a} \times (\bm{b} + \bm{c}) &= \bm{a} \times \bm{b} + \bm{a} \times \bm{c} \\
\bm{a} \times (k\bm{b}) &= k(\bm{a} \times \bm{b}) \\
\bm{a} \times \bm{b} &= - (\bm{b} \times \bm{a})
\end{aligned}
$$

在笛卡尔坐标系里, 我们可以得出:

$$
\begin{aligned}
\bm{a} \times \bm{b} &= (x_a\bm{x} + y_a\bm{y} + z_a\bm{z})\times(x_b\bm{x} + y_b\bm{y} + z_b\bm{z}) \\
&= x_ax_b\bm{x}\times\bm{x} + x_ay_b\bm{x}\times\bm{y} + x_az_b\bm{x}\times\bm{z} + y_ax_b\bm{y}\times\bm{x} + y_ay_b\bm{y}\times\bm{y} + y_az_b\bm{y}\times\bm{z} + z_ax_b\bm{z}\times\bm{x} + z_ay_b\bm{z}\times\bm{y} + z_az_b\bm{z}\times \bm{z} \\
&= x_ay_b\bm{z} - x_az_b\bm{y} - y_ax_b\bm{z} + y_az_b\bm{x} + z_ax_b\bm{y} - z_ay_b\bm{x} \\
&= (y_az_b - z_ay_b)\bm{x} + (z_ax_b - x_az_b)\bm{y} + (x_ay_b - y_ax_b)\bm{z}
\end{aligned}
$$

用坐标格式表示就是:
$$\bm{a} \times \bm{b} = (y_az_b - z_ay_b, z_ax_b - x_az_b, x_ay_b - y_ax_b)$$

<a id="markdown-_245-orthonormal-bases-and-coordinate-frames正交基和坐标系" name="_245-orthonormal-bases-and-coordinate-frames正交基和坐标系"></a>
#### _2.4.5 Orthonormal Bases and Coordinate Frames正交基和坐标系

orthonormal basis就是一个三维坐标系, 三个轴向量的长度都是1, 方向遵守右手螺旋定则

carnonical orthonormal basis可以理解为主坐标系, 它的原点$o$称之为origin location, 这个坐标系也可称之为global coordinate system  
与之对应的是, 在global coordinate system里还可以设定一个子坐标系, 例如大地坐标是是主坐标系, 我们可以确定飞机在主坐标系的位置  
但是对于飞行员来说, 可能还需要一个原点是飞机, 航向是x轴的坐标系  
大地坐标系和飞机坐标系构成了a frame of reference, 或者说coordinate frame  
子坐标系称为local coordinate system

第6章会有详细讲述, 我对本章的理解没有很透测, 暂且搁置

<a id="markdown-_246-constructing-a-basis-from-a-single-vector根据一个向量来构建正交基" name="_246-constructing-a-basis-from-a-single-vector根据一个向量来构建正交基"></a>
#### _2.4.6 Constructing a Basis from a Single Vector根据一个向量来构建正交基

如上一章节所述, 飞行员需要根据自己的航向这个单个向量, 来构建一个坐标系.  
假设这个向量是$\bm{a}$, 要构建一个u-v-w的basis  
首先以$\bm{a}$的单位向量作为$\bm{w}$:
$$\bm{w} = \frac{\bm{a}}{\parallel \bm{a} \parallel}$$
然后我们取任意一个和$\bm{a}$不同的向量$\bm{t}$, 以$\bm{w}$和$\bm{t}$的cross product作为$\bm{u}$, 也就是:
$$\bm{u} = \frac{\bm{t} \times \bm{w}}{\parallel \bm{t} \times \bm{w} \parallel}$$
但是问题是, 如何选择$\bm{t}$呢? 如果太接近$\bm{a}$, 得到的结果精度很低(原画如此, 我的理解是, 两条边的夹角很小, 构成的面也很小, 做垂线的精度就会比较低, 如果夹角比较大就不一样了)  
那么我们怎么取$\bm{t}$呢? 方法是将$\bm{w}$最小的一个坐标值变成1, 比如$\bm{w} = (1/\sqrt{2}, -1/\sqrt{2}, 0)$, 那么$\bm{t} = (1/\sqrt{2}, -1/\sqrt{2}, 1)$  
这样$\bm{w}$和$\bm{u}$都确定了, $\bm{v}$也就确定了:
$$\bm{v} = \bm{w} \times \bm{u}$$

<a id="markdown-_247-constructing-a-basis-from-two-vectors" name="_247-constructing-a-basis-from-two-vectors"></a>
#### _2.4.7 Constructing a Basis from Two Vectors

已知两个vectors $\bm{a}$和$\bm{b}$, 构建u-v-w basis  
我们以$\bm{a}$为w:  
$$\bm{w} = \frac{\bm{a}}{\parallel \bm{a} \parallel}$$
然后以$\bm{w}$和$\bm{b}$的cross product, 也就是他们的垂线作为$\bm{u}$:
$$\bm{u} = \frac{\bm{b} \times \bm{w}}{\parallel \bm{b} \times \bm{w} \parallel}$$
然后:
$$\bm{v} = \bm{w} \times \bm{u}$$

<a id="markdown-squaring-up-a-basis修复" name="squaring-up-a-basis修复"></a>
#### Squaring Up a Basis修复

假如我们计算向量时发生错误, 我们可以用上面的方法重建basis, 但是这不是最好的方法, 在5.4.1章节里有更精确的方法SVD
<a id="markdown-_25-curves-and-surfaces曲线和曲面" name="_25-curves-and-surfaces曲线和曲面"></a>
### _2.5 Curves and Surfaces曲线和曲面

这里需要了解导数derivative和梯度gradient的概念  

导数是指只有一个参数的曲线(也就是平面), 在某个点上曲线的值和参数之比, 无限趋近得到的一个数.  
可以理解为曲线在某个点的切线的倾斜度. 实际场景下, 运动的瞬时速度就是运动过程在某个时间点的导数.  

梯度可以理解为2D导数, 两个参数的曲线, 比如等高线, 参数是x, y坐标, 值是高度, 梯度和x方向的导数以及y方向的导数相关, 详细概念待学习

#### _2.5.1 2D Implicit Curves 2D隐式曲线

我们用这样的一个implicit equation隐式方程来表示2D曲线:
$$f(x, y) = 0$$
举一个例子:
$$f(x, y) = (x - x_c)^2 + (y - y_c)^2 - r^2$$
这就表示了一个以$(x_c, y_c)$为圆心, 半径为r的圆  
为什么说是implicit, 因为我们从这个方程得不到具体的$(x, y)$坐标  

我们还可以用向量来解释上面的方程, 假设向量$p = (x, y)$, $c = (x_c, y_c)$, 满足:
$$(p-c)\cdot(p-c) - r^2 = 0$$
c到p的向量dot product自身, 等于$r^2$  
其实这和上面的式子是等价的, 因为dot product自己就是自身的长度的平方  
根据dot product的特性, 我们可以这样简化表示:
$$\parallel p - c \parallel - r = 0$$

#### _2.5.2 The 2D Gradient 2D梯度

在本章的前言离说到了derivative和gradient的概念  
derivative导数是一个数字, gradient(2D derivative)的结果是一个向量  

我们可以用等高线来解释, 这样更形象  
一座山峰, $f(x, y) = 0$可以看作是一条某一高度的等高线, 这条曲线的gradient就是在某一个点处指向往上最陡的向量, 我们这么表示:
$$\nabla f(x, y) = \left(\frac{\partial f}{\partial x}, \frac{\partial f}{\partial y}\right)$$
这个式子表示什么意思呢? 我们回忆一下derivative导数的概念, $y=g(x)$的derivarive是:
$$\frac{d_y}{d_x} = \lim_{\Delta x \to 0}\frac{\Delta y}{\Delta x} = \lim_{\Delta x \to 0}\frac{g(x + \Delta x) - g(x)}{\Delta x}$$
对于梯度:
$$\frac{\partial f}{\partial x} = \lim_{\Delta x \to 0}\frac{f(x+\Delta x, y) - f(x, y)}{\Delta x}$$
我们将其称为partial derivative偏导数, 因为我们只变化x, y保持不变, 求得f的变化的极限  
书中有证明为什么这样规定梯度, 这里不再详述了  

这样, 我们就可以得到$f(x, y) = x^2 + y^2 - 1 = 0$的gradient是$(2x, 2y)$

##### Implicit 2D lines隐式2D直线

一条斜线:
$$y = mx +b$$
用$f(x, y)$来表示就是:
$$y - mx - b = 0$$  
因为对这个等式两边乘以一个不为0的数字k, 还是同一条线, 我们对这条线用更通用的表示:
$$Ax + By + C = 0$$

假设我们知道这条线上的两个点$(x_0, y_0)$, $x_1, y_1$, 我们能确定这3个数字吗?  
因为只有两个点, 两个方程, 3个未知数, 好像算不出来  
我们可能会问, 既然$kf(x)$和$f(x)$是同一条线, 那么我们让$k = 1/C$不就可以了吗? 就不用算3个数字了?   
但是我们怎么能保证C不等于0呢?

我们知道$Ax+By+C$的gradient vector是$(A, B)$, 这个向量是和这条直线垂直的, 也就是和这两个点的连线是垂直的  
这两个点的连线向量是$(x_1 - x_0, y_1 - y_0)$, 那么$(A, B) = (y_0 - y_1, x_1 - x_0)$, 垂直的话dot product等于0, 这样:
$$(y_0 - y_1)x + (x_1 - x_0)y + C = 0$$
因为$(x_0, y_0)$在这条线上, 代入这个式子, 算出来C的值, 从而得到:
$$(y_0 - y_1)x + (x_1 - x_0)y + x_0y_1 -x_1y_0 = 0$$

我们还可以算出来某个点$(a, b)$到这条线的垂直距离  
点到这条线的垂直距离应该是梯度向量距离的倍数, 因为梯度也是垂直于这条线的:
$$distance = k\sqrt{(A^2 + B^2)}$$
这个点可以表示为$(x, y) + k(A, B)$, 所以:
$$f(a, b) = f(x + kA, y + kB) = Ax + kA^2 + By + kB^2 + C = k(A^2 + B^2)$$
从而:
$$distance = \frac{f(a, b)}{\sqrt{A^2 + B^2}}$$

我们可以让gradient vector变成一个单位向量, 这样计算更方便:
$$f(x, y) = \frac{y_0 - y_1}{\sqrt{(x_1 - x_0)^2 + (y_0-y_1)^2}}x + \frac{x_1-x_0}{\sqrt{(x_1 - x_0)^2 + (y_0 - y_1)^2}}y + \frac{x_0y_1 - x_1y_0}{\sqrt{(x_1-x_0)^2 +(y_0 - y_1)^2}} = 0$$

##### Implicit Quadric Curves隐式二次曲线

假如x和y是平方, 那么就是曲线  
比如下面这个等式就是一个圆:
$$(x-x_c)^2 + (y-y_c)^2 - r^2 = 0$$
这个等式就是一个长半径是a, 短半径是b的椭圆:
$$\frac{(x-x_c)^2}{a^2} + \frac{(y-y_c)^2}{b^2} - 1 = 0$$

<a id="markdown-_26-linear-interpolation线性插值" name="_26-linear-interpolation线性插值"></a>
### _2.6 Linear Interpolation线性插值

<a id="markdown-_27-triangles三角形" name="_27-triangles三角形"></a>
### _2.7 Triangles三角形

三角形在2D和3D图像里起到非常重要的作用  
因为三角形是最基本的图形, 四边形可以拆成两个三角形, 三角形往下拆还是三角形  
三角形构成一个平面, 四边形可能不是  

#### _2.7.1 2D Triangles

一个三角形的面积, 就是四边形的一半, 在第5章里会讲到两个向量的determinant, 也就是四边形的面积, 这样三角形的面积是:
$$
\begin{aligned}
area &= \frac{1}{2}
\Big|
\begin{matrix}
x_b - x_a & x_c - x_a \\
y_b - y_a & y_c - y_a
\end{matrix}
\Big| \\
&= \frac{1}{2}(x_ay_b + x_by_c +x_cy_a -x_ay_c - x_by_a -x_cy_b)
\end{aligned}
$$
如果abc三个点构成逆时针方向, 则面积是正数, 否者是负数

在图形学里, 为了让颜色渲染更平滑, 消除锯齿现象, 我们需要用到*barycentric coordinate*  
就是一个三角形一个定点为origin, 两条边为basis, 构成的一个不垂直的坐标系.  
将正交坐标系下的点转化为这个坐标系下的坐标, 这样就能精确的平滑的只将三角形内的区域着色, 从而消除锯齿现象  

假设以a为origin, (b-a), (c-a)为basis, 那么这个坐标系下的点可表示为:
$$p = a + \beta(b-a) + \gamma(c-a)$$
$\beta$以b到ac的距离为单位, $\gamma$以c到ab的距离为单位  
也就是说, 通过b且与ac平行的线上的点, $\beta = 1$
通过c且与ab平行的线上的点, $\gamma = 1$
经过变换:
$$
\begin{aligned}
p &= (1-\beta-\gamma)a + \beta b + \gamma c \\
\alpha &\equiv 1 - \beta - \gamma \\
p(\alpha, \beta, \gamma) &= \alpha a + \beta b + \gamma c \\
\alpha + \beta + \gamma &= 1
\end{aligned}
$$

那么$\alpha, \beta, \gamma$如何求得呢?  
在上面的坐标系定义里, 我们知道, $\beta$就是一个点到ac的距离, 除以b到ac的距离的比值:
$$\beta = \frac{f_{ac}(x, y)}{f_{ac}(x_b, y_b)}$$
同理:
$$\gamma = \frac{f_{ab}(x, y)}{f_{ab}(x_c, y_c)}$$

一个点到一条线的距离, 我们在2.5章节里讲到了  
我们先根据两个点来将直线的implicit equation表示出来, 2.5章节已推导:
$$f_{ab}(x, y) \equiv (y_a - y_b)x + (x_b - x_a)y + x_ay_b - x_by_a = 0$$
这样:
$$\gamma = \frac{(y_a-y_b)x + (x_b-x_a)y + x_ay_b - x_by_a}{(y_a-y_b)x_c + (x_b-x_a)y_c +x_ay_b - x_by_a}$$
同理得到:
$$
\begin{aligned}
\beta &= \frac{(y_a-y_c)x + (x_c-x_a)y + x_ay_c - x_cy_a}{(y_a-y_c)x_b + (x_c-x_a)y_b +x_ay_c - x_cy_a} \\
\alpha &= 1-\beta-\gamma
\end{aligned}
$$


<a id="markdown-frequently-asked-questions" name="frequently-asked-questions"></a>
### Frequently Asked Questions

<a id="markdown-notes" name="notes"></a>
### notes

<a id="markdown-exercises" name="exercises"></a>
### Exercises