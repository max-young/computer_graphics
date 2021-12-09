<!-- TOC -->

- [_4.1 The Basic ray-tracing Algorithm](#_41-the-basic-ray-tracing-algorithm)
- [_4.2 Perspective透视](#_42-perspective透视)
- [_4.3 Computing Viewing Rays](#_43-computing-viewing-rays)
  - [_4.3.1 Orthographic Views正交视图](#_431-orthographic-views正交视图)
  - [_4.3.2 Perspective Views透视视图](#_432-perspective-views透视视图)
- [_4.4 Ray-Object Intersection光线和对象相交](#_44-ray-object-intersection光线和对象相交)
  - [_4.4.1 Ray-Sphere Intersection光线和球相交](#_441-ray-sphere-intersection光线和球相交)
  - [_4.4.2 Ray-Triangle Intersection](#_442-ray-triangle-intersection)
  - [_4.4.3 Ray-Polygon Intersection](#_443-ray-polygon-intersection)
  - [_4.4.4 Intersecting a Group of Objects](#_444-intersecting-a-group-of-objects)
- [_4.5 Shading](#_45-shading)
  - [_4.5.1 Lambertian Shading](#_451-lambertian-shading)
  - [_4.5.2 Blinn-Phong Shading](#_452-blinn-phong-shading)
  - [_4.5.3 Anbient Shading](#_453-anbient-shading)
  - [_4.5.4 Multiple Point Lights](#_454-multiple-point-lights)
- [_4.6 A Ray-Tracing Program](#_46-a-ray-tracing-program)
  - [_4.6.1 Object-Oriented Design for a Ray-Tracing Program面向对象的光线追踪程序](#_461-object-oriented-design-for-a-ray-tracing-program面向对象的光线追踪程序)
- [_4.7 Shadows阴影](#_47-shadows阴影)
- [_4.8 Ideal Specular Reflection理想镜面反射](#_48-ideal-specular-reflection理想镜面反射)
- [_4.9 Historical Notes](#_49-historical-notes)
- [Frequently Asked Questions](#frequently-asked-questions)

<!-- /TOC -->

**Ray Tracing光线追踪**

计算机图形学最重要的任务之一是rendering渲染3D对象  
通俗的说是将在某一个点观测到的3D对象显示在2D图像上.  
渲染过程的输入是3D对象, 输出是像素pixel.  
渲染有两种方法: object-order rendering, image-order rendering  
object-order rendering是对每个对象做loop循环处理  
image-order rendering是对每个pixel做loop循环处理  
image-order rendering更加简单和灵活, 通常会耗时, 但是会生成更好的图像  

<a id="markdown-_41-the-basic-ray-tracing-algorithm" name="_41-the-basic-ray-tracing-algorithm"></a>
### _4.1 The Basic ray-tracing Algorithm

object通过pixel被看到. 这需要viewing ray视线和物体之间的intersection  
有必要解释一下shade这个词, 字面意识阴影  
在这里准确的意思是: 物体在视线前方, 挡住了视线, 那么这个物体is shaded  
光线追踪包括3个过程:  
1. ray generation光线生成: pixel的生成依赖光源(观测点)的位置和角度
2. ray intersection: 这个过程能找到光线前方的对象
3. shading: 基于ray intersection的结果计算pixel color

image-order rendering就是对每个pixel循环执行上面这个过程

<a id="markdown-_42-perspective透视" name="_42-perspective透视"></a>
### _4.2 Perspective透视

将3D对象用2D展示在几百年前就被艺术家们研究, 比如绘画, 摄影  
标准的方法是用linear perspective线性透视: 实际对象是直线, 在2D图像上也是直线  
鱼眼相机就不是linear perspective  

最简单的投影是parallel projection平行投影, 就是将3D物体沿着统一的一条线路(视线)移到2D平面上, 通俗的说就是压缩到一个平面上.  
如果这条线路是垂直于2D平面的, 我们称之为orthographic正交. 不垂直的话称之为oblique倾斜  
这种投影的决定因素是视线view direction和平面plane(的倾斜角度) 

平行投影也是有弊端的, 比如他和我们的观感是不一样的, 我们人眼观测是近大远小, 两条铁轨实际是平行不相交的, 但是我们人眼看在远处它们是相交的.  
但如果是用平行投影, 压缩到平面上, 那还是平行的, 人的观感会觉得铁轨的距离越来越大了.  
这是因为人眼观测不是只有单条视线viewing direction, 而是通过一个视点viewpoint为基点放射出去的.   
这种投影方式叫perspective projection, 这种投影方式的决定因素是视点viewpoint和平面plane的角度和其离视线的距离

### _4.3 Computing Viewing Rays

ray的基本生成, 是根据视点和图像平面, 也就是试点喝图像平面上的点(像素)的连线, 就是光线(视线).  

<img src="./_images/ray.png" width=20%>

ray可以用起点和方向来表示, 再加上一个时间变量, 就得到了ray的equation表示:
$$p(t) = e + t(s - e)$$

位置和坐标在坐标系下才有意义  
根据ray的定义方式, 我们可以定义camera frame相机坐标系, 这个坐标系是正交坐标系, 符合右手定则, 如下图所示:

<img src="./_images/camera_frame.png" width=30%>

请注意: v指向上方, w指向后方. 对应实际的相机就是这样的:

<img src="./_images/camera.png" width=10%>

#### _4.3.1 Orthographic Views正交视图

<img src="./_images/view.png" width=30%>

我们把图像平面的四个边界分别定义成l, r, b, t  
坐标系原点在图像平面的中心(orthographic view的坐标原点在中心, perspective view的坐标原点往图像平面做垂线, 交点是中心)  
l和r是u方向, $l < 0 < r$, b和t是v方向, $b < 0 < t$  
假设图像平面是由$n_x \times n_y$个像素组成, 那么对于第$(i, j)$个像素(从左下角开始从0开始算), 我们可以计算出, 这个像素中心对应的坐标:
$$
u = l + \frac{r-l}{n_x}\times(i+0.5) \\
\ \\
v = b + \frac{t-b}{n_y}\times(j+0.5) \\
$$
对于orthographic view, 视线方向就是:  
ray.origin是e加上u和v方向上的坐标值  
ray.direction就是$-w$

#### _4.3.2 Perspective Views透视视图

透视试图的光线起点不在图像平面上, 而是在viewpoint视点上, 它和图像平面有一定的距离, 这个距离称之为image plane distance(图像平面距离), 或者focal length(焦距)  
这样ray就是:
ray.direction: $(u, v, -d)$    
ray.origin: $e$

### _4.4 Ray-Object Intersection光线和对象相交

光线的定义是: $e + td$  
我们要找到光线和对象相交的时间t, $t_0 < t < t_1$  
$t_0 = 0, t_1 = +\infty$

#### _4.4.1 Ray-Sphere Intersection光线和球相交

光线的定义是: $p(t) = e + td$  
球的定义是: $f(p) = 0$, 如果有向量的形式表示就是:
$$(p-c)\cdot(p-c) - R^2 = 0$$
$c$是球心, $R$是半径  
如果光线与球相交, 那么交点必须既在光线上也在球面上, 那么:
$$
\begin{aligned}
&f(p(t)) = 0 \\
&f(e + td) = 0 \\
&(e + td - c)\cdot(e + td - c) - R^2 = 0
\end{aligned}
$$
这是一个二次多项式, 我们能计算出两个$t$值了  
如果两个t相等, 说明光线与球相切.  
如果都大于0, 说明有两个交点  
如果一个大于0, 一个小于0, 说明光线的起点$e$在球里面.  
如果都小于0, 那么光线射向球的反方向.  
如果无解, 那么不相交.

#### _4.4.2 Ray-Triangle Intersection

光线与三角形相交的问题, 可以用重心坐标来解决, 当然这只是其中一种解决方法.  
这种方法可以分为两步, 第一步求出光线与三角形所在的平面的交点, 第二步判断这个交点是否在三角形内.  

三角形所处的平面可以用barycentric coordinate重心坐标来表示, 三角形的三个顶点是$a, b, c$, 平面可以表示为:  
$$f(p) = a + \beta(b - a) + \gamma(c - a)$$  
这实际上就是
$$f(p) = \alpha a + \beta b + \gamma c$$
$$\alpha = 1 - \beta - \gamma$$  
同样的, 光线和平面的交点, 必须在光线上, 也必须在平面上:
$$e + td = a + \beta(b - a) + \gamma(c - a)$$  
如果这个交点在三角形内, 那么必须满足$\beta > 0, \gamma > 0, \beta + \gamma < 1$  
那么我们求出上面equation里的三个未知量$t, \beta, \gamma$就可以了, 上面这个等式可以展开为$x, y, z$三个方向上的三个等式:
$$
\begin{aligned}
x_e + tx_d = x_a + \beta(x_b - x_a) + \gamma(x_c - x_a) \\
y_e + ty_d = y_a + \beta(y_b - y_a) + \gamma(y_c - y_a) \\
z_e + tz_d = z_a + \beta(z_b - z_a) + \gamma(z_c - z_a) \\
\end{aligned}
$$
用矩阵来表示:
$$
  \begin{bmatrix}
  x_a - x_b & x_c - x_a & x_d \\
  y_a - y_b & y_c - y_a & y_d \\
  z_a - z_b & z_c - z_a & z_d \\
  \end{bmatrix}
  \begin{bmatrix}
  \beta \\ \gamma \\t
  \end{bmatrix} = 
  \begin{bmatrix}
  x_a - x_e \\ y_a - y_e \\ z_a - z_e
  \end{bmatrix}
$$
这个矩阵等式可以用cramer's rule克莱姆法则来解  
解出$t, \beta, \gamma$后, 我们就可以做判断了:

<img src="./_images/ray-triangle.png" width=30%>

#### _4.4.3 Ray-Polygon Intersection

假设m个点$p_1, ..., p_m$构成一个polygon, 所有点都在一个平面上, 那么这个平面有一个法线n, 那么这个平面上的所有点和$p_1$的连线都和发现n垂直:
$$(p - p_1)\cdot n = 0$$
和计算光线和三角形是否相交的方法一样, 我们先计算出光线和多边形所处的平面的交点, 然后再判断交点是否在多边形内.  
交点必须满足平面的等式, 也必须满足光线的等式, 那么:
$$(e + td - p_1)\cdot n = 0$$
$$ t = \frac{(p_1 - e)\cdot n}{d\cdot n}$$
这样我们就能计算出交点了.  

如何判断交点是否在多边形内呢? 假设交点在多边形内, 如果我们从交点在平面内发射任意一条光线, 那么这条光线和多边形的交点必然是奇数, 很奇妙对不对?  
我们再简单化一点, 我们把多边形和这个交点都投影在xy平面上, 从交点发射一条沿着x轴的光线, 那么我们只需判断多边形的所有边是否和这条光线相交, 并汇总数量即可.  
这样判断就很简单了, 如果交点的y值在一条边的两个顶点的y值范围内, 那么这条光线就和这条边相交.  

另外的一个问题, 如果这个多边形投影到xy平面是一条直线呢? 我们就要投影到yz或者zx平面了, 如何判断呢?  
哪个轴上的值最大, 则去掉哪个轴  
(此处没有理解清楚, 世纪应用时再看)

还有一种处理方法是把多边形分成三角形

#### _4.4.4 Intersecting a Group of Objects

对于一组对象, 我们可以判断光线与每个对象是否相交, 然后找到最小的时间t
### _4.5 Shading

pixel的value通过shading model计算得出  
本节介绍几种基本的shading model, 更高级的model在第十章介绍  
大多数shading model都是根据光线反射(light reflection)的过程来设计  
光线反射的过程表现为光线照射到物体表面, 然后反射部分光线到camera  
光线反射又几个重要参数:  
- 光线方向向量l: 照射点指向光源的单位向量
- 观测方向v: 照射点指向观测点的单位向量  
- 表面法线n: 照射点垂直于照射面的单位向量  
- 表面属性: 包括color颜色、shininess(光泽、感光度、吸收光线的强度属性)  

<img src="./_images/shading_model.png" width=20%>  

#### _4.5.1 Lambertian Shading

这个model是根据Lambert在18世纪的观测理论得出:  
照射点从光源得到的能量和照射角度相关  
如果是垂直于光照表面照射, 那么得到全部能量  
如果正切(平行)于光照表面, 则不获得能量  
如果是以一个夹角$\theta$照射(照射方向和光照表面法线的夹角), 则获取全部能量乘以$\cos \theta$

Lambert Model的公示是:
$$L = k_dImax(0, n\cdot l)$$
L是光照后的pixel color  
$k_d$是diffuse coefficient漫反射系数(或者叫surface color)  
$I$是光源强度  
$n$和$l$是单位向量, $n\cdot l$就是$\cos \theta$  
这个等式适用于颜色三通道RGB  
pixel value的红色部分就等于漫反射系数的红色部分乘以红色光源强度乘以$n\cdot l$, 蓝绿色同理  
向量$l$通过光源向量减去照射点向量得到  
不要忘记$nlv$都是单位向量

对于I光源强度, 从光源照射到物体表面, 还需要考虑到衰减, 我们用光源强度除以一个系数  
显然这个系数光源到照射点长度相关, 我们定义此系数为这个长度的平方, 可以通过照射向量和自己的点乘得到. 

#### _4.5.2 Blinn-Phong Shading

上一节讲到的Lambert Shading Model只解释了其中的一种光照情况.    
一个光源照射一个物体会有三种情况: 高光specular highlights、漫反射diffuse reflection、环境光照ambient lighting.  
高光和观测角度相关, 如果观测角度和照射角度相等, 也就是以发现对称, 我们就会看到特别亮(刺眼), 越接近这个对称角度就越刺眼, 这就是specular highlights  
Lambert Shading Model解释了第二种情况-漫反射, 和观测角度无关  
环境光照是指物体相对于光源的背面, 不受到光源的直接照射, 但是我们依然能够看到这一部分, 因为它受到了光源照射到其他位置后反射的光照.

对于specular highlights, 看这张图:  
<img src="./_images/blinn-phong.png">  
观测角度和照射角度与法线的对成方向越接近, 高光就越亮, 所以我们可以和漫反射模型一样, 用一个角度来计算强度  
但是这个角度计算起来比较麻烦, 我们可以用另外一个角度来替代: 法线和照射方向观测方向的中间方向的夹角, 这个夹角和之前是等价的, 而且计算更方便, 从而我们得到specular highlights的计算公式:
$$
\begin{aligned}
h &= \frac{v+l}{\left\|v+l\right\|} \\
L &= k_sImax(o, n \cdot h)^p
\end{aligned}
$$
$k_s$是高光系数, 为什么有一个p指数呢? 因为高光衰减特别快, 只在对称的很小的区域比较亮, 加上指数后, 这个曲线就会变得窄  
高光和漫反射叠加之后:
$$
L = k_dImax(0, n\cdot l) + k_sImax(o, n \cdot h)^p
$$

#### _4.5.3 Anbient Shading

上面已经解释了环境光照, 不受到光源直接照射的区域, 我们依然能够看见, 因为它接受了四面八方的反射光  
这种情况我们很难定义清楚, 所以我们用很简单的模型来定义:
$$L = k_aI_a$$
环境系数乘以环境光照强度  
这样, 一个区域的完整着色模型就是这三种光照的叠加:
$$
L = k_aI_a + k_dImax(0, n\cdot l) + k_sImax(o, n \cdot h)^p
$$

#### _4.5.4 Multiple Point Lights

如果有多个光源呢? 我们对模型进行叠加superposition:
$$
L = k_aI_a + \sum_{i=1}^{N}[k_dI_imax(0, n\cdot l_i) + k_sI_imax(o, n \cdot h_i)^p]
$$


### _4.6 A Ray-Tracing Program

<img src="/_images/ray_tracing_program.png" width=30%>

是否hit the project可以用4.4.4章节的知识来解决  
hit之后可以获取对象的引用, 或者其属性, 然后来进行着色

#### _4.6.1 Object-Oriented Design for a Ray-Tracing Program面向对象的光线追踪程序

我们对object(surface)创建一个类
```
class surface
  virtual box hit(ray e + td, real t0, real t1, hit-record rec)
  virtual box bounding-box()
```
hit函数判断光线是否和surface相交, 相交的时间t在t0和t1之间, 相交的记录记录在rec里, 例如相交的时间t  
bounding-box是surface的最下包围盒, 例如对于一个球体:
```
box sphere::bounding-box()
  vector3 min = center - vector3(radius, radius, radius)
  vector3 max = center + vector3(radius, radius, radius)
  return box(min, max)
```

### _4.7 Shadows阴影

我们在视点看某场景下的某个对象, 如果视线和对象相交, 那么我们就能看到对象.  
如果我们从对象处看光源, 如果能看到光源, 那么说明这个对象在光源照射下, 假设看不到光源, 也就是说视线和这个场景下的某个对象相交了, 那么这个对象就处在阴影下.  
所以为代码可以扩展一下:

<img src="./_images/shadow.png" width=30%>

总结一下, 如果从视点出发的射线和对象相交, 那么就会有环境光照ambient lighting  
如果从对象出发往光源方向的视线与场景下的对象都不相交, 那么这个对象就在光源的照射下, 那么我们再加上diffuse lighting漫反射和specular lighting高光  
注意: 在计算对象出发往光源防线的视线时, 起始时间是从$\epsilon$开始的, 这个值是自定义的一个很小的数, 是为了避免数值精度引起的计算误差

### _4.8 Ideal Specular Reflection理想镜面反射

ideal specular reflection也被称为mirror reflection  

<img src="./_images/ideal_specular_reflection.png" width=15%></br>
<img src="./_images/ideal_specular_reflection1.png" width=15%>
  
我们可以计算出反射向量:
$$r = d - 2(d\cdot n)n$$

我们从视线d看surface, 看到的颜色应该是和在surface看镜面反射方向看到的颜色是一样的. 但是光会衰减, 我们需要乘以一个系数来转换, 系数就是$raycolor(p+sr, \epsilon, \infty)$, 另外, 可能有很多光线照射到surface, 那么:
$$color c = c + k_mraycolor(p+sr, \epsilon, \infty)$$
k_m是RGB三通道颜色

这里涉及到一个理解, 光源并不一定是灯泡、太阳, 任何物体都是光源, 我们看到一个物体, 这个物体反射了太阳光到我们眼睛里, 这个物体也是光源, 所以在一个场景下, 光源会经过很多次(甚至是无限多次)的反射, 这就相当复杂了.  
我们会定一个反射数量的阈值, 例如我们规定只反射5次, 来解决这个问题

### _4.9 Historical Notes

real-time ray tracing越来越普遍

### Frequently Asked Questions

- ray tracing 为什么不再需要透视矩阵转换了

  在第七章里, 我们把现实世界的坐标经过旋转平移转换到视角坐标, 然后还要经过投影变换, 转换到canonical view coordinate, 然后再经过z-buffer判断显示什么, 然后成像  
  ray tracing是从视角(摄像机)出发, 方向是二维图像上的像素, 然后照射到对象上, 计算出二维图像上应该显示什么, 实际上是模拟了现实的观测, 相当于投影转换的逆  
