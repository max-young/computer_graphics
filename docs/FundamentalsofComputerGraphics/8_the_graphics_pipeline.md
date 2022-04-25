<!-- TOC -->

- [_8.1 Rasterization光栅化](#_81-rasterization光栅化)
  - [_8.1.1 Line Drawing](#_811-line-drawing)
  - [_8.1.2 Triangle Rasterization三角形光栅化](#_812-triangle-rasterization三角形光栅化)
  - [_8.1.3 Clipping剪裁](#_813-clipping剪裁)
- [_8.2 Operations Before and After Rasterization](#_82-operations-before-and-after-rasterization)
  - [_8.2.1 Simple 2D Drawing](#_821-simple-2d-drawing)
  - [_8.2.2 A Minimal 3D Pipeline](#_822-a-minimal-3d-pipeline)
  - [_8.2.3 Using a z-Buffer for Hidden Surfaces](#_823-using-a-z-buffer-for-hidden-surfaces)
  - [_8.2.4 Per-vertex Shading逐个三角形着色](#_824-per-vertex-shading逐个三角形着色)
  - [_8.2.5 Per-fragment Shading逐个像素着色](#_825-per-fragment-shading逐个像素着色)
  - [_8.2.6 Texture Mapping纹理映射](#_826-texture-mapping纹理映射)
  - [_8.2.7 Shading Frequency着色频率](#_827-shading-frequency着色频率)
- [_8.3 Simple Antialiasing简单反锯齿(反走样)](#_83-simple-antialiasing简单反锯齿反走样)

<!-- /TOC -->

**The Graphics Pipeline**

上一章节讲到了各种转换, 将现实坐标系里的对象经过积层转换到屏幕显示空间坐标系  
这一章就要讲到如何显示到屏幕上, 这就是object-order rendering(渲染)  

例如, 屏幕是有一个个像素组成的, 可以理解为一个个小方格, 现实中的一个三角形怎么现实在屏幕上呢?  
就需要将这个三角形所占的像素找出来, 然后给这些像素赋值, 就把三角形显示出来了.  

将屏幕或者image上的像素用几何图形occupy圈占出来的过程就是rasterization(光栅化)  
rasterization依赖praphics pipeline(图形管线)过程, 这个过程包括一系列操作, 起于object对象, 终于像素pixel更新

番外:  
pixel就是picture element的缩写  
rasterization是德语图像化的意思

rendering有很多方法, 本章介绍基本的一种, 原理都是相通的  
rendering的过程包括:  
<img src="./_images/fundamentals_of_computer_graphics/graphics_pipeline.png" width=50%>  
application指的是存储vertices顶点集的文件  
经过vertex processing得到primitive(暂且译作多边形)  
经过rasterization stage得到fragment(pixel covered by the primitive)(多边形以及其圈占的像素)  
然后经过fragment processing和blending stage就可以显示了(着色等操作)  
<img src="./_images/fundamentals_of_computer_graphics/pipeline.png">  
在第四章讲到了shading着色, 我们可以对三角形进行着色, 也可以对单个像素进行着色  
对三角形着色就发生在vertex processing阶段, 对像素着色就发生在fragment processing阶段  
GPU(图形处理器)就是图形管线pipeline的硬件实现  

<a id="markdown-_81-rasterization光栅化" name="_81-rasterization光栅化"></a>
### _8.1 Rasterization光栅化

rasterization是rendering的核心, 也是graphics pipeline的核心  
rasterization有两项工作:  
1. 找出primitive覆盖的pixel
2. 给pixel赋值  

rasterization的输出是一系列fragment, fragment就是primitive覆盖的一组pixel

<a id="markdown-_811-line-drawing" name="_811-line-drawing"></a>
#### _8.1.1 Line Drawing

我们应该怎么在屏幕上画一条线呢?  
这就需要找到这条线所经过的像素

画线依赖这条线的equation方程式, 方程式有两种形式: implicit和parameter  
我们采用implicit方程式

**Drawing Using Implicit Line Equation**

在2.5.2章节里, 我们根据两个端点$(x_0, y_0)$和$(x_1, y_1)$就能得到这条线的implicit equation:
$$f(x, y) \equiv (y_0 - y_1)x + (x_1 - x_0)y + x_0y_1 - x_1y_0 = 0$$

我们假设$x_0 < x_1$, 我们得到这条线的slope:
$$m = \frac{y_1 - y_0}{x_1 - x_0}$$
我们假设$m \in (0, 1]$, 也就是说这条线往右上角的角度小于等于45度  
这样假设是为了接下来解释方便, 其他情况这是这种情况的变种

接下来的要做的工作是要通过这个方程式来找到这条线经过的pixel  
我们知道pixel是一个个小方格, 如果我们将line经过的小方格都找出来, 就会发现这些小方格组成的形状是不规则的, 而且比较粗, 因为存在一列下有两个pixel的情况.  

这里我们采用midpoint algorithm的算法, 找到一条不间断且最thin瘦的线, 也就是说需要连续(对角的两个pixel也是连续的), 且每列只选择一个像素.  
我们从一个像素开始画下一个像素的时候, 有两个选择, 要么画右侧的一个像素, 要么画右上角的一个像素, preseudo代码就是:
```
y = y0
for x = x0 to x1 do
  draw (x, y)
  if (some condition) then
    y = y + 1
```
这个some condition怎么确定呢?  
上面说到画下一个像素时有两个选择, 用坐标表示就是$(x + 1, y)$或者$(x + 1, y + 1)$, 他们的中点就是$(x + 1, y + 0.5)$  
如果$(x, y)$和这个中点$(x + 1, y + 0.5)$的连线, 在要画的线下面, 那么我们就选择右上角的像素$(x + 1, y + 1)$, 如果在线上面, 那么就选择右侧的像素$(x + 1, y)$  
那么怎么判断这条中线是否在要画的线的上面还是下面呢? euqtion就起到作用了.  
我们看equation的一项$(x_1 - x_0)y$, 因为我们假设$x_1 > x_0$, 所以如果x不变, y变大, 那么$f(x, y)$应该大于0 , 所以如果某个点(m, n)满足$f(m, n) > 0$, 那么这个点在这条线的上方  
所以上面的condition就可以变成:
```
if f(x + 1, y + 0.5) < 0 then:
  y = y + 1
```
这个midpoint algorithm就完成了  
我们还可以做优化, 因为$f(x, y)$equation满足:
$$
\begin{aligned}
f(x+1, y) &= f(x, y) + (y_0 - y_1) \\
f(x+1, y+1) &= f(x, y) + (y_0 - y_1) + (x_1 - x_0) \\
\end{aligned}
$$
所以midpoint algorithm算法可以优化成
```
y = y0
d = f(x0 + 1, y0 + 0.5)
for x = x0 to x1 do
  draw (x, y)
  if d < 0 then
    y = y + 1
    d = d + (x1 - x0) + (y0 - y1)
  else
    d = d + (y0 - y1)
```

<a id="markdown-_812-triangle-rasterization三角形光栅化" name="_812-triangle-rasterization三角形光栅化"></a>
#### _8.1.2 Triangle Rasterization三角形光栅化

在2.7章节里, 我们讲到三角形的barycentric coordinates, 一个点在三角形的barycentric坐标系下可以表示为:  
$$p = \alpha a + \beta b + \gamma c$$
假设已知三角形的三个定点的颜色$c_0, c_1, c_2$, 我们要对三角形内的一个点着色, 我们可以这样计算其颜色:
$$c = \alpha c_0 + \beta c_1 + \gamma c_2$$

接下来的问题是, 我们怎么判断一个像素是否在三角形内, 我们规定, 如果像素的中点在三角形内, 就认为这个像素应该被三角形框定  
那么怎么判断像素的中点在三角形内呢? 还是2.7章节, 我们根据这个像素中点的坐标, 就能求得$\alpha \beta \gamma$  
如果求得这三个数都大于等于0小于等于1, 那么这个像素中点就在三角形内(2.7章节里, 这三个变量由此特性)  

所以, brute-force algorithm就可以这样写, 对屏幕上的所有像素进行遍历:
```
for all x do
  for all y do
    if (α ∊ [0, 1], β ∊ [0, 1], ɣ ∊ [0, 1]) then
      c = αc0 + βc1 + ɣc2
      drawpixel(x, y) with color c
```

显然这个算法的效率不高, 对每个三角形都要遍历所有像素, 我们可以做一个优化, 只对相关的像素做遍历, 我们找到定点的xy边界即可  
里面的遍历算法也可做一下优化, 采用2.7章节的知识:  
<img src="./_images/triangle_rendering.png" width=50%>

**Dealing with Pixels on Triangle Edges**

如何处理边缘问题, 两个相邻三角形, 共用一条边, 应该怎么处理这条变呢?  
如果这条边的像素不做处理, 那就会留下孔洞  
如果做两次处理, 颜色就会重叠  
我们用下面的方法来决定这条边的像素应该用哪个三角形来处理:  
定义一个屏幕外的点, 这个点和三角形的另外一个顶点如果在这条公共边的同侧, 那么这条公共边就属于这个三角形  
判断是否在同侧, 就是用屏幕外点到公共边的距离乘以顶点到公共边的距离, 如果大于0, 就在同一侧, 请注意, 距离可以是负数, 这也是2.7章节的知识  
所以, preseudo code可以写作:  
<img src="./_images/triangle_edge.png" width=50%>  
注: 截图前面少了两行:
$$
x_{min} = floor(x_i) \\
x_{max} = ceiling(x_i))
$$

<a id="markdown-_813-clipping剪裁" name="_813-clipping剪裁"></a>
#### _8.1.3 Clipping剪裁

如果一个对象的一半在视点的后方, 也就是说z坐标是正数, 会怎样呢?  
在7.3章节里, perpsective transform将camera坐标转换到orthographic坐标  
x, y会经过缩放, z也会有一个转换:
$$
z^{\prime} = n + f - \frac{n}{z}f
$$
如果这个对象在视线前方, 那么z的范围应该在n+f范围内.  
如果在视线后面, 那么z是正直, n是负值, 转换后的z轴就超出n+f了  
所以我们需要clipping

clipping一般用一个六边形来剪切, 将六边形外的对象切掉  
有两种方法

### _8.2 Operations Before and After Rasterization

在光栅化之前除了要准备顶点向量等数据, 还需要准备色彩等数据  
光栅化之后还需要进行着色等操作

#### _8.2.1 Simple 2D Drawing

2D Drawing 在fragment stage不需要做任何事情.  
primitive可以画上同一个颜色, 也可以内插颜色, rasterizer光栅化器提供了相应的接口

#### _8.2.2 A Minimal 3D Pipeline

3D Drawing相比2D多了一些矩阵转换  
棘手的问题是对象之间的重叠咬合药如何处理.  
我们需要遵循back-to-front顺序原则, painter's algorithm似乎能解决这个问题, 先画背景, 再画前景.  
但是并不能解决咬合的问题, 比如两个三角形交叉, 一个三角形的一部分在背景, 一部分在前景, 我们就不能先画一个再画另外一个.  
或者三个三角形循环重叠, 也不能解决. 而且painter's algorithm效率不高.

#### _8.2.3 Using a z-Buffer for Hidden Surfaces

painter's algorithm很少使用, 取而代之的是另外一种简单高效的hidden surface removal algorithm: z-buffer algorithm  
算法原理很简单: 跟踪每个图像上的像素位置迄今为止所画的最近距离的fragment的像素, 之后这个位置所画的像素如果距离更远, 则丢弃, 如果更近, 则更新追踪.  
跟踪的是最近距离加上其颜色等属性, 称之为z-value或者depth, z-buffer是指z-value所组成的一些列网格, 这个算法应用于blending阶段.

**Precision Issues**

为了更高的效率, z-value距离值取正整数(float有更高的开销)  
在实际工作中, 值需要尽可能的小, 但是又要有足够的精读来区分不同的像素.  
假设我们用整数range B{0, 1, ..., B-1}来表示z-value的范围  
那么对应的z轴的值的最大差$\Delta z = (f-n)/B$
为了节省空间, B用bite表示, 那么$\Delta z = (f-n)/2^b$  
为了提高效率, 那么尽量使$\Delta z$更小, 所以要么增大b, 要么缩小f-n  
也就是增大f, 减小n, 书中有证明过程

#### _8.2.4 Per-vertex Shading逐个三角形着色

图形管线将三角形配上颜色然后输出图像.  
第四章提到了shading着色, 需要照射方向、观测方向、法线来进行计算.  
着色的方式有几种.  
一种是对三角形的顶点进行着色, 这种方式也被称为Gouraud Shading.  
着色用到的坐标系可选择world space或者eye space这样的正交坐标系, 因为计算需要用到几个向量之间的角度.

per-vertex shading的缺点是细节表现不足, 三角形内部的颜色只能通过内插得到(内插可用重心坐标系实现, 2.7章节讲到了)  
当然, 如果三角形足够小, 细节表现也是不错的.  
per-vertex shading在管线的vertex processing阶段进行

#### _8.2.5 Per-fragment Shading逐个像素着色

为了解决per-vertex shading的不足, 我们可以把着色过程放到fragment stage阶段进行, 做per-fragment shading  
per-fragment shading是对单个像素着色, 也被称为phong shading  
我们需要知道单个像素的相关信息(计算用到的几个向量、参数等信息), 才能进行着色.  
这些信息需要在vertex stage到fragment stage的过程中传递下去

#### _8.2.6 Texture Mapping纹理映射

texture纹理应用在着色过程中, 加上额外的信息, 避免图像看起来同质、人工.  
我们对一个球体着色时, 这个球可能不是一个均匀的同质的球, 而是有花纹、纹理, 球上的点的特质(颜色、感光度等等)时不一样的, 从而计算出来的颜色也不一样  
对应shading model, 就是每个点的漫反射系数、高光系数等是不一样的. (颜色其实就是这些特质的反应)  
纹理属性和每个点有对应关系, 可以用纹理坐标系来表现(texture coordinate)  
texture mapping就是找到这种关系, 然后进行着色.

#### _8.2.7 Shading Frequency着色频率

上面提到的着色方法对应不用的着色频率, 不同场景应用不同的着色方法和频率(开销和效果的平衡)  
大尺寸、细节较少的情况可以使用低频率的per-vertex shading, 尽管pxcel可能很多  
如果细节较多、纹理多样, 则可以采用per-fragment shading  
例如在游戏里, 用hardware pipeline做per-fragment shading  
在photorealistic renderman system真实感渲染器系统里可以用per-vertex shading, 因为primitive被切分成了很多细小的多边形, 大小和像素差不多.

<a id="markdown-_83-simple-antialiasing简单反锯齿反走样" name="_83-simple-antialiasing简单反锯齿反走样"></a>
### _8.3 Simple Antialiasing简单反锯齿(反走样)

和光线追踪一样, 在进行光栅化时, 我们会判断一个像素是否在三角形内, 直线可能是跨过像素, 这个像素部分在内, 这就造成了锯齿(走样)  
实际上标准光栅化过程也被称为走样(锯齿)光栅化(aliased rasterization)

如何去掉这些锯齿(antialiasing)呢? 完美的解决方法时对pixel进行切分, 允许部分被primitive覆盖, 但实际操作中不太能实现, blurring(模糊)是有效的.  
我们可以把一个像素的值, 和周围正方形范围的像素值做平均(box filtering), 这样就实现了blurring, 这样就看起来平滑了, 起到减轻走样的效果

如何实现box filtering呢? supersampling超采样    
我们要获得256X256的图片, 图片上的一条线的宽度是1.2pixels  
我们对原始图像做1024X1024的采样, 线的宽度是4.8pixels  
我们用4X4pixels的一个框去1024X1024的图片去采样(本来应该是单个像素采样), 然后对4X4个像素的值取平均, 就得到了256X256上的单个像素的值  
这样就实现了低采样率, 实现了box filtering

超采样的开销很大. 由于导致锯齿是因为primitive的边, 而不是着色的突然变化. 所以我们可以从着色频率来着手, 提高着色频率, 而且理想情况下我们只需对一种颜色做处理.  
(可以理解为, 三角形是走样的, 但是着色可能没到三角形的精度, 我们让着色不走样就可以了, 三角形走样不影响效果, 这样开销就小了)  
如果是per-vertex shading, 那自然最好, 三角形内部用内插实现平滑, 没有锯齿.  
如果是per-fragment shading, hardware pipeline会提前存储以及处理好的不同精度的blur结果, 然后应用到着色过程中, 不用实时计算.