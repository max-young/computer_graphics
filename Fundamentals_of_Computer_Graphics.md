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

<a id="markdown-notes" name="notes"></a>
### Notes