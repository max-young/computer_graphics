<!-- TOC -->

- [Preface](#preface)
- [_1. Introduction](#_1-introduction)
  - [_1.1 Graphics Areas图形学领域](#_11-graphics-areas图形学领域)
  - [_1.2 Major Applications主要用途](#_12-major-applications主要用途)
  - [_1.3 Graphics APIs图形学接口](#_13-graphics-apis图形学接口)
  - [_1.4 Graphics Pipeline图形管道](#_14-graphics-pipeline图形管道)
  - [_1.5 Numerical Issues数值问题](#_15-numerical-issues数值问题)
  - [_1.6 Efficiency效率](#_16-efficiency效率)
  - [_1.7 Designing and Coding Graphics Programs](#_17-designing-and-coding-graphics-programs)
    - [_1.7.1 Class Design](#_171-class-design)
    - [_1.7.2 Float vs. Double](#_172-float-vs-double)
    - [_1.7.3 Debugging Graphics Programs](#_173-debugging-graphics-programs)
  - [Notes](#notes)
- [_2 Miscellaneous Math多样的数学](#_2-miscellaneous-math多样的数学)
  - [_2.1 Sets and Mappings](#_21-sets-and-mappings)
    - [_2.1.1 Inverse Mappings逆向映射](#_211-inverse-mappings逆向映射)
    - [_2.1.2 Intervals区间](#_212-intervals区间)
    - [_2.1.3 Logarithms对数](#_213-logarithms对数)
  - [_2.2 Solving Quadratic Equations求解二次方程](#_22-solving-quadratic-equations求解二次方程)
  - [_2.3 Trigonometry三角几何](#_23-trigonometry三角几何)
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
  - [_2.6 Linear Interpolation线性插值](#_26-linear-interpolation线性插值)
  - [_2.7 Triangles三角形](#_27-triangles三角形)
  - [Frequently Asked Questions](#frequently-asked-questions)
  - [notes](#notes-1)
  - [Exercises](#exercises)

<!-- /TOC -->

<a id="markdown-preface" name="preface"></a>
## Preface

第2-8章是核心的核心, 讲述了通过基本的ray tracing(光线跟踪)和rasterization(光栅化)将图像显示在显示器上.

首先介绍ray tracing, 它是生成3D图像的最简单的方法

之后是对一些内容的介绍性描述, such as sampling theory, texture mapping, spatial data structures, and splines

15章以后是一些贡献文章

这本书的封面是一只老虎, 为什么选老虎当封面, 有一段演讲很有趣, 去翻书吧.

<a id="markdown-_1-introduction" name="_1-introduction"></a>
## _1. Introduction

computer graphics的定义:  
The term computer graphics describes any use of computers to create and manipulate images

这本书介绍了算法和数学工具, 它们可以用来创建所有的图像: 逼真的视觉效果、内容丰富的插图、精美的电脑动画

Graphics图形可以是二维或者三维的; image图像可以完成生成、也可以在通过照片处理得到

这本书是关于基本的算法和数学, 它们用来合成三维对象和场景.

<a id="markdown-_11-graphics-areas图形学领域" name="_11-graphics-areas图形学领域"></a>
### _1.1 Graphics Areas图形学领域

计算机图形学的核心领域包括:
- Modeling建模
  建模能存储在计算机里的对形状和外观的数学特征表达
- Rendering渲染
  渲染是根据3D计算机模型来创建阴影图像, 这个词来源于艺术领域
- Animation动画
  动画是通过连续的图形来生成动态“幻觉”的技术. 

计算机图形学还包括其他领域, 但是否是核心, 看法不一:
- User interaction用户交互
- virtual reality虚拟现实
- Virtualization可视化
- Image processing图像处理
- 3D scanning 3D扫描
- Computational photography计算摄影

<a id="markdown-_12-major-applications主要用途" name="_12-major-applications主要用途"></a>
### _1.2 Major Applications主要用途
- Video game视频游戏
- Cartoons卡通
- Visual effects视觉效果  
  视觉效果几乎用到了计算机图形学所有类型的结束. 在现在电影里, 几乎都会将拍摄的前景与数字合成的背景相结合. 很多电影还会用3D建模和3D动画来合成幻觉、 对象、甚至任务, 观众难辨真假
- Animated films动画电影  
  用到了很对visual effects的技术, 但是不以真实图像为创作目标
- CAD/CAM计算机辅助设计/计算机辅助制造  
  computer aided design and computer aided manufacturing. 例如, 很多机械零件是用计算机3D建模工具设计, 然后在计算机控制的机床设备上自动生产
- Simulation模拟  
  模拟可以理解为精确的视频游戏. 比如可以模拟飞行来训练初级飞行员. 这项技术可以应用到其他花费较高或者具有一定危险性的行业训练中.
- Medical imaging医学成像  
  例如可以将CT数据合成为shaded images(阴影图像), 帮助医生获取更多医学数据
- Information virsualization信息可视化

应用领域涵盖娱乐、工业、民用等领域

<a id="markdown-_13-graphics-apis图形学接口" name="_13-graphics-apis图形学接口"></a>
### _1.3 Graphics APIs图形学接口

API的经典定义:
> An applicaiton program interface(API) is a standard collection of functions to perform a set of related operation, and a graphics API is a set of functions that perform basic operations such as drawing images and 3D surfaces into windows on the screen.

图形程序需要有两类接口: 可视化输出接口, 用户输入接口.

接口有两种范式:
1. 方法的集成, 由语言包实现, 例如java等
2. Direct3D、OpenGL, 由C++等实现的软件
  
<a id="markdown-_14-graphics-pipeline图形管道" name="_14-graphics-pipeline图形管道"></a>
### _1.4 Graphics Pipeline图形管道

什么是图形管道? 每台计算机都有强大的3D图形管道. 这是一个特殊的子系统, 可以高校的绘制3D图像. 

基本的操作是绘制共享定点的三角形并加上阴影, 使之呈现3D效果.

<a id="markdown-_15-numerical-issues数值问题" name="_15-numerical-issues数值问题"></a>
### _1.5 Numerical Issues数值问题

很多图形程序只是3D数值代码.数值至关重要.

几乎所有现代计算机都符合IEEE浮点标准(IEEE Standards Association, 1985), 下面是对于图形学来说, IEEE的一些重要特征:
- Infinity $\infty$
  this is a valid number that is larger than all other valid number
- Minus Infinity $-\infty$
  this is a valid number that is smaller than all other valid number
- Not a number(NaN)  
  invalid number that arise from an operation, such as zero divided by zero

IEEE标准对上面三个值的操作有下面的定义:  
a是实数  

$$
\begin{aligned}
+a/(+\infty) &= +0\\
-a/(+\infty) &= -0\\
+a/(-\infty) &= -0\\
-a/(-\infty) &= +0\\
\infty + \infty &= \infty\\
\infty - \infty &= NaN\\
\infty \times \infty &= \infty\\
\infty / \infty &= NaN\\
\infty / a &= \infty\\
\infty / 0 &= \infty\\
0 / 0 &= NaN
\end{aligned}
$$

请注意, 在上面$\infty$除以0不是NaN, 0除以0才是NaN, a除以0也不是NaN  
另外+0、-0在这里是有意义的

$$
\begin{aligned}
+a / +0 &= +\infty\newline
-a / +0 &= -\infty
\end{aligned}
$$

这个特性给我们带来了很多方便, 例如:
$$a = \frac{1}{\frac{1}{b} + \frac{1}{c}}$$
如果不遵循IEEE不标准, 就需要检查b和c是否为0, 否则就会出现异常.  

另外一种情况:
```pseudocode
a = f(x)
if (a>0) then
  do something
```
如果`f(x)`返回NaN或者$\infty$, 那么也会抛出异常.  
但是如果遵循IEEE标准, 如果a = NaN或者$-\infty$, 则是false  
如果$a = +\infty$, 则是true
遵循这个标准让程序变得更小、更健壮、更有效率

<a id="markdown-_16-efficiency效率" name="_16-efficiency效率"></a>
### _1.6 Efficiency效率

在可见的未来, 对于效率, 我们应更加关注内存占用, 多于操作次数. 因为内存的发展速度跟不上处理器的运算速度.

使代码快速运行的方法:
1. 以最直接的方式编写代码. 根据需要即时计算中间结果, 而不是存储它们.
2. 以优化模式编译
3. 用分析工具定位瓶颈
4. 检查数据结构, 找到优化方法. 使数据单元大小与缓存匹配(?)
5. 如果瓶颈使数值计算问题, 你可能需要重写代码.

最重要的是上面的第一点. 很多"优化""使代码阅读性变差, 但效率并没有提升. 在前期, 更应该把时间花在解决bug和功能开发上

<a id="markdown-_17-designing-and-coding-graphics-programs" name="_17-designing-and-coding-graphics-programs"></a>
### _1.7 Designing and Coding Graphics Programs

一些通用策略在图形编程中很有用

#### _1.7.1 Class Design

在图形编程中, 涉及到集合实体(vectors向量、matrics矩阵)和图形实体(RGB颜色、images图像), 它们可以为class设计提供便利.  
locations位置和displacements位移是否需要用不同的类来表达, 看法不一. 为了举例方便, 我们使两者使用同样的类.  

我们可以设计下面这些基本的类:
- vector2  
  2D向量, 包括x、y分量(component), 存储在array里, 还应实现加法减法等运算
- vector3  
  3D向量
- hvector  
  包括4个分量的同构向量(homogeneous), 姑且认为是4D向量
- rgb  
  包括3个分享的RGB颜色, 也需要实现加法减法等操作
- transform
  4*4的矩阵, 在第六章介绍
- image  
  能够输出的元素是RGB像素的2D array
  
#### _1.7.2 Float vs. Double

为了效率, 现代架构建议降低内存使用以及保持一致的内存访问. 这意味着用单精度数据更好.  
但是为了避免数值问题, 使用双精度更好.  
根据实际情况权衡  

> 建议使用double进行几何计算、float进行颜色计算. 对于三角网等占用大量内存的数据, 建议用float, 但是当通过成员函数访问数据时, 建议转换为double

> 我主张使用浮点数进行所有计算，直到找到证据证明在代码的特定部分中需要双精度。

#### _1.7.3 Debugging Graphics Programs 

在图形学中有用的debug策略
- The scientific Method  
  一种取代传统debug的方法通常非常有用. 但是看起来好像有些“傻瓜”, 而且通常在我们的编程生涯初期, 都被教育不要这样做, 那这是一种什么样的debug方法呢?  
  我们创建一个图像, 观察它有什么问题  
  我们假设导致问题的原因, 并测试  
  例如, 在光线追踪程序里很典型的shadow acne阴影痤疮问题, 我们可以假设这些点是被错误的标记了称了阴影. 为了证明这个假设, 我们可以关闭阴影检查并重新编译.  
  通过这种方法避免了浪费大量的时间在逐步检查变量值上面
- Images as coded Debugging Output  
  调试过程中输出图像, 来验证假设
- Using a Debugger  
  The scientific Method看似以来直觉和经验, 无法解决所有的问题.
  我们可以用条件断点, 在特定输入下运行程序, 并在假设的引起错误的值的运行位置, set a trap, 例如:
  ```
  if x == 126 and y == 247 then
    print "blarg!"
  ```
  一些debugger有条件断点的功能, 就不需要想上面这样手动操作了.  
- Data Visualization for Debugging  
  我们经常无法理解程序在做什么, 因为在程序出问题之前会计算出大量中间数据.  
  这样我们就需要图和图解来帮助我们理解.  
  比如在光线追踪程序里, 我们可以编写代码来可视化光线, 这样就能理解像素是由哪些光线生成的. 

<a id="markdown-notes" name="notes"></a>
### Notes

本章对于软件工程的讨论, 收到了Effective C++ se- ries (Meyers, 1995, 1997)、Extreme Programming movement (Beck & Andres, 2004)、The Practice of Programming (Kernighan & Pike, 1999)的影响.    
有许多计算机图形学大会, 包括 ACM SIGGRAPH 和 SIGGRAPH Asia, Graphics Interface, the Game Developers Conference (GDC), Eurographics, Pacific Graphics, High Performance Graphics, the Eurographics Symposium on Rendering, and IEEE VisWeek. 去网上搜索吧.

## _2 Miscellaneous Math多样的数学

一些图形工作只是将数学直接转换为代码. 数学越清晰, 代码越简洁. 本章只提及图形学相关的数学知识作为参考.

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

#### _2.1.1 Inverse Mappings逆向映射

如果一个function: $f: A \mapsto B$, 那么可能存在inverse function: $f^-1: B \mapsto A$  
另一种表示方法是: $b = f(a)$, $f^-1(b) = a$  
这个定义需要满足这样的条件: target B里的所有元素b都是domain中的某个元素a的image(换句话说, range和target相等), 并且a是唯一的.  
那么我们称这样的mappings或者functions为*bijections(双射)*  
形象一点的描述: 一群骑手, 一群马, 每个骑手都骑一匹马, 每匹马都有骑手骑.  

举两个例子:
- bijection: $f:\R \mapsto \R, f(x) = x^3$
- not bijection: $sqr: \R \mapsto \R, sqr(x) = x^2$, 因为target中的元素对应两个domain中的元素

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

### _2.3 Trigonometry三角几何

**Angles**

两条half_lines(direction)组成的角度angle如何表示呢?  
交点为圆心画一个单位圆(半径为1), 两条线切的圆弧长度可以用来表示角度, 一般用较短的圆弧来表示角度.  
这种表示方法的angle的单位是**radians**  
两条线切的圆弧有长的一段和短的一段, 两个角度加起来就是单位圆的周长$2\pi$

还有另外一种表示方法叫**degrees**, 半圆的角度用radians表示是$\pi$radians, 用degrees表示是180degrees, 符号是$180^{\circ}$

两种表示方法的换算关系是:
$$degrees = \frac{180}{\pi}radians$$
$$radians = \frac{\pi}{180}degrees$$

**Trigonometrics Functions三角函数**

勾股定理的证明  
以及sin, cos, tan, cot等的定义

**Userful Identities**

sin, cos, tan, cot的一些等式

### _2.4 Vectors向量

vector描述了长度和方向. 如果两个vector的长度和方向相同, 即时我们想象它们在不同的位置, 它们依然是相等的.  
vector一般用粗体字母表示, 比如$\bm{a}$, vector的长度可以这么表示: $\parallel\bm{a}\parallel$  
unit vector的长度是1, zero vector的长度是0, 方向没有定义  

vector可用来表示offset(偏移)

#### _2.4.1 Vector Operations

vector的加法和减法, 画图就清晰明了了

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

#### _2.4.5 Orthonormal Bases and Coordinate Frames正交基和坐标系

orthonormal basis就是一个三维坐标系, 三个轴向量的长度都是1, 方向遵守右手螺旋定则

carnonical orthonormal basis可以理解为主坐标系, 它的原点$o$称之为origin location, 这个坐标系也可称之为global coordinate system  
与之对应的是, 在global coordinate system里还可以设定一个子坐标系, 例如大地坐标是是主坐标系, 我们可以确定飞机在主坐标系的位置  
但是对于飞行员来说, 可能还需要一个原点是飞机, 航向是x轴的坐标系  
大地坐标系和飞机坐标系构成了a frame of reference, 或者说coordinate frame  
子坐标系称为local coordinate system

第6章会有详细讲述, 我对本章的理解没有很透测, 暂且搁置

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

#### _2.4.7 Constructing a Basis from Two Vectors

已知两个vectors $\bm{a}$和$\bm{b}$, 构建u-v-w basis  
我们以$\bm{a}$为w:  
$$\bm{w} = \frac{\bm{a}}{\parallel \bm{a} \parallel}$$
然后以$\bm{w}$和$\bm{b}$的cross product, 也就是他们的垂线作为$\bm{u}$:
$$\bm{u} = \frac{\bm{b} \times \bm{w}}{\parallel \bm{b} \times \bm{w} \parallel}$$
然后:
$$\bm{v} = \bm{w} \times \bm{u}$$

#### Squaring Up a Basis修复

假如我们计算向量时发生错误, 我们可以用上面的方法重建basis, 但是这不是最好的方法, 在5.4.1章节里有更精确的方法SVD
### _2.5 Curves and Surfaces曲线和曲面

### _2.6 Linear Interpolation线性插值

### _2.7 Triangles三角形

### Frequently Asked Questions

### notes

### Exercises