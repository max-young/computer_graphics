<!-- TOC -->

- [_11.1 Looking Up Texture Values](#_111-looking-up-texture-values)
  - [_11.2 Texture Coordinate Functions](#_112-texture-coordinate-functions)
  - [_11.2.1 Geometrically Determined Coordinates几何上确定的坐标系](#_1121-geometrically-determined-coordinates几何上确定的坐标系)
  - [_11.2.2 Interpolated Texture Coordinates内插纹理坐标系](#_1122-interpolated-texture-coordinates内插纹理坐标系)
  - [_11.2.3 Tiling, Wrapping Modes, and Texture Transformations平铺、环绕模式, 和纹理变换](#_1123-tiling-wrapping-modes-and-texture-transformations平铺环绕模式-和纹理变换)
  - [_11.2.4 Perspective Correct Interpolation透视校正内插](#_1124-perspective-correct-interpolation透视校正内插)
  - [_11.2.5 Continuity ans Seams连续性和接缝](#_1125-continuity-ans-seams连续性和接缝)
- [_11.3 Antialiasing Texture Lookups反走样纹理查找](#_113-antialiasing-texture-lookups反走样纹理查找)
  - [_11.3.1 The Footprint of a Pixel](#_1131-the-footprint-of-a-pixel)
  - [_11.3.2 Reconstruction重建](#_1132-reconstruction重建)
  - [_11.3.3 MipMapping](#_1133-mipmapping)

<!-- /TOC -->

**Texture Mapping纹理映射**

我们观察世界时, 会发现太多的细节, 木头有纹理, 皮肤有皱纹, 衣服有编织结构, 就算是塑料和金属, 也有凹凸不平, 它们并不是光滑规整的模型.  

在图形学里我们引入texture的概念, 我们通过texture来呈现这些细节而不改变对象的形状.  
具体实现就是texture mapping纹理映射: 用texture map、 texture image、 texture来存储对象表面细节, 然后映射到对象表面.

纹理可以呈现细节、阴影、 反射. 难点是纹理映射过程中可能会造成变形、重叠等问题. 因为texture是二维的, 映射到三维物体表面, 涉及到投影等等问题.

<a id="markdown-_111-looking-up-texture-values" name="_111-looking-up-texture-values"></a>
### _11.1 Looking Up Texture Values

我们用一个房间的木质地板来做例子.  
我们需要找到地板上的点对应的纹理, 然后用纹理作为参数去做shader着色.  
根据图像上的点去纹理匹配的过程叫texture lookup, 找到的结果就是texture sampling.  
这个过程的代码大概是这样:  
```C++
Color texture_lookup(Texture y, float u, float v) {
  int i = round(u * t.width() - 0.5)
  int j = round(v * t.height() = 0.5)
  return t.get_pixel(i, j)
}

Color shade_surface_point(Surface s, Point p, Texture t) {
  Vector normal = s.get_normal(p)
  (u, v) = s.get_texcoord(p)
  Color diffuse_color = texture_lookup(u, v)
  // compute shading using diffuse_color and normal
  // return shading result
}
```
我们需要找到图像上的点和texture上的点的对应关系: texture coordinate function  
<img src="./_images/texture_coordinate_function.png" width=60%>  
第七章的各种变换, 找到了实际空间中的坐标和图像坐标的对应关系, 这里需要找到世纪空间坐标和纹理坐标的对应关系. 从而这三者就能互相转换了:
$$
\begin{aligned}
\phi &: S \to T \\
&: (x, y, z) \mapsto (u, v) 
\end{aligned}
$$

T称为texture space, 是一个单位方形区域$(u, v) \in [0, 1]^2$  
$\phi$转换和$\pi$一样都是三维空间到二维空间的转换

texture mapping有两个主要的问题需要解决:
1. 解决对应关系texture coordinate function
   我们可以想象得到这不是一次方函数那么简单, 可能会涉及到非常复杂的投影等问题
2. 不要有太多走样aliasing
   书中举了地板的例子, 我们可以这么理解  
   一个房间的地板的纹理图是均匀的, 远近的地板的纹理是一样的, 但是我们拍摄图像时有远近, 近处的一个像素覆盖了一个地板, 远处的一个像素可能覆盖了几块地板, 采样频率不一样, 这样就导致了aliasing

#### _11.2 Texture Coordinate Functions

这个问题并不只是图形学的问题, 对于地图绘图来说, 这个问题存在了几百年. 对三维的地球绘图, 在有限的图纸上保证尽可能的不失真.

texture coordinate map是平衡各种问题的解决办法. 这些问题包括:
- Bijectivity双向性
  我们希望表面的所有点对应texture space不同的点
- Size distortion尺寸失真
  整个纹理的比例大致是一样的. 对象表面的距离比例对应纹理表面的距离比例应该大致时一致的.
- Shape distortion形状失真
  对象表面如果是圆形, 在纹理上也应该大致是圆形
- Continuity连续性
  对象是连续的, 纹理上也应该是连续的. 纹理应避免不连续的情况.

在2.5.8章节我们用两个参数来定义曲面, 我们可以用这两个参数来定义texture coordinate, 这样就实现了对象表面和纹理的对应关系.  
如果不依靠曲面的参数化表达, 怎么定义纹理坐标系呢? 有两种方法, 一种是根据曲面上点的空间坐标, 另外一种是根据曲面三角网的顶点坐标来内插其他点的坐标. 看下面的章节解释.

#### _11.2.1 Geometrically Determined Coordinates几何上确定的坐标系

geometrically determined coordinates适用于比较简单的场景.  
我们用这个这个图像来说明, 上面的数字可以代表u,v坐标, 网格线可以看到变形的情况  
<img src="./_images/texture_test_image.png">

**Planar Projection平面投影**

和第7章的投影变换(正交投影、透视投影)类似, 用矩阵变换就可以转换得到u, v
$$
\phi(x, y, z) = (u, v)
$$
$$
\left[
\begin{matrix}
  u \\
  v \\
  \star \\
  1
\end{matrix}
\right] = M_t 
\left[
\begin{matrix}
  x \\
  y \\
  z \\
  1
\end{matrix}
\right]
$$
注: z坐标用星号是因为我们不关心z坐标  
Planar projection不具有injective, 因为如果是一个封闭的对象, 那么这个对象的前面和后面的两个点, 对应的texture coordinate的坐标是一样的  
但是它在shadow mapping会用到, 在第11.4.4章节会讲到

**Spherical Coordinates球面坐标**

对于一个近似球体的对象, 我们可以用球面坐标来进行转换纹理坐标.  
和地球的地图类似, 我们可以完整的表达地球上的点的坐标, 但是在两极变形较大.  

转换之后的球面坐标也是三维坐标: $(\rho, \theta, \phi)$, 我们舍弃$\rho$, 把$\theta, \phi$控制在[0, 1]范围  
在第2.5.8章节里讲到了曲面的参数表达, 我们得到:
$$\phi(x, y, z) = ([\pi + atan2(y, x)]/2\pi, [\pi - acos(z/||x||)]/\pi)$$
球面坐标是bijective的, 除了两极

**Cylindrical Coordinates圆柱坐标**

对于圆柱形的对象, 我们可以使用圆柱坐标, 用一个圆柱形套在对象外面做投影:
$$\phi(x, y, z) = (\frac{1}{2\pi}[\pi+atan2(y, x)]/2\pi, \frac{1}{2}[1+\pi])$$

**cubemaps立方体地图**

球星坐标系在两极会造成较大的变形, 提到方案是cubemaps, 可以改善这种情况, 但是会失去一定的连续性.  
立方体有六个面, 对应六个texture map. 意味着对象的不同区域投影到不同的texture map, 投影就是正交投影.  
例如: 如果|x|>|y|, |x|>|z|, 那么根据x的正负, 分别投影到+x面或者-x面.  
OpenGL里有对应的六个面的转换方法, 这里不列公式了, 用到的时候看书吧.

#### _11.2.2 Interpolated Texture Coordinates内插纹理坐标系

只存储三角形顶点的坐标和属性, 三角形内部用内插的方式来获取坐标和属性  
内插方法采用barycentric coordinate重心坐标, 中心坐标参照第二章节

rexture mapping的质量和顶点相关, 例如顶点的密度等等

这种方式在某些区域依然有变形, 比如书中举的例子: 鼻翼, 地球两极. 皆因为小的三角形对应面积较大的texture space

#### _11.2.3 Tiling, Wrapping Modes, and Texture Transformations平铺、环绕模式, 和纹理变换

有时图像上的点对应的texture coordinate由于某种原因落到了terxture space的外面  
有时我们只对一小部分区域着色, 这样texture space有很大一部分区域是黑色的, 为了让这一小部分区域有更精细的细节, 我们要把texture space放大, 让这一小部分变成unit quare  
对于落在这部分区域之外的部分, 可以赋予一个背景色常量, 也可以用这部分区域的平均值作为背景色  

对于重复图形的图像, 比如地板、砖块墙, 应该只需要一块重复区域的纹理, 这样更节省空间. 详情待实际应用再看.

#### _11.2.4 Perspective Correct Interpolation透视校正内插

#### _11.2.5 Continuity ans Seams连续性和接缝

### _11.3 Antialiasing Texture Lookups反走样纹理查找

texture mapping的第二个基本问题是antialiasing  
由于纹理是表现细节的关键, 所以纹理是引起aliasing的关键来源  
纹理aliasing和光栅化aliasing一样, 都涉及到sample、blur、reconstruction等等

#### _11.3.1 The Footprint of a Pixel

我们图像上一个pixel对应的纹理区域称为texture space footprint of the pixel  
<img src="./_images/footprint_of_pixel.png" width=50%>  
从图上可以看到图像上一个pixel对应的texture space的区域大小是不一样的.  
注意: 这里有个理解问题, texture的坐标系和图像的坐标系是没关系的, 他们对实际对象经过不同的转换得到  
上一章节, 对象经过$\pi$转换得到图像, 经过$\phi$转换得到texture space  
我们定义$\psi = \phi\cdot\pi^{-1}$, 这样图像上的像素经过$\psi$转换就得到了texture footprint  

antialiasing的方法是计算均值(模糊处理blur), 但是这是一个复杂的工作  
footprint大小各异, 还可能不是连续的  
$\psi$经过了两次转换, 所以pixel对应的footprint的大小形状和这两次转换都相关,  
也就是和视角以及texture的投影方法都相关  

这个复杂的问题怎么解决呢? approximation近似  
一个pixel的中心是x, 这个pixel对应texture space上一个平行四边形  
书中有详细解释, 但是没看懂, 用到时再详细研究

#### _11.3.2 Reconstruction重建

一个像素对应的footprint可能很大, 需要做平均  
也可能小于一个texels纹素的大小, 这样就不连续了, 这就需要reconstruction  
这和第9章节的reconstruction不太一样, 第9章需要重建的离散数据是规律采样的  
但是纹理是不规则的, 详情待用到时再研究

#### _11.3.3 MipMapping
