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
  - [_11.3.4 Basic Texture Filtering with Mipmap用mipmaps进行基本纹理过滤](#_1134-basic-texture-filtering-with-mipmap用mipmaps进行基本纹理过滤)
  - [_11.3.5 Anisotropic Filtering各向异性过滤](#_1135-anisotropic-filtering各向异性过滤)
- [_11.4 Applications of Texture Mapping纹理映射的应用](#_114-applications-of-texture-mapping纹理映射的应用)
  - [_11.4.1 Controlling Shading Parameters控制着色参数](#_1141-controlling-shading-parameters控制着色参数)
  - [_11.4.2 Normal Maps and Bump Maps法线图和凹凸贴图](#_1142-normal-maps-and-bump-maps法线图和凹凸贴图)
  - [_11.4.3 Displacement Maps位移贴图](#_1143-displacement-maps位移贴图)
  - [_11.4.4 Shadow Maps阴影贴图](#_1144-shadow-maps阴影贴图)
  - [_11.4.5 Environment Maps环境贴图](#_1145-environment-maps环境贴图)
- [_11.5 Procedural 3D Textures程序化三维纹理](#_115-procedural-3d-textures程序化三维纹理)
  - [_11.5.1 3D Stripe Textures三维条纹纹理](#_1151-3d-stripe-textures三维条纹纹理)
  - [_11.5.2 Solid Noise实体噪声](#_1152-solid-noise实体噪声)
  - [_11.5.3 Turbulance扰动](#_1153-turbulance扰动)

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
```cpp
// u, v是[0, 1]之间的坐标
Color texture_lookup(Texture t, float u, float v) {
  int i = round(u * t.width() - 0.5)
  int j = round(v * t.height() - 0.5)
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
<img src="./_images/texture_coordinate_function.png" width=90%>  
第七章的各种变换, 找到了实际空间中的坐标和图像坐标的对应关系, 这里需要找到实际空间坐标和纹理坐标的对应关系. 从而这三者就能互相转换了:
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

<a id="markdown-_112-texture-coordinate-functions" name="_112-texture-coordinate-functions"></a>
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

在[2.5.8章节](docs/FundamentalsofComputerGraphics/2_miscellaneous_math?id=_258-3d-parametric-surface%e4%b8%89%e7%bb%b4%e5%8f%82%e6%95%b0%e8%a1%a8%e9%9d%a2)我们用两个参数来定义曲面, 我们可以用这两个参数来定义texture coordinate, 这样就实现了三维对象的surface和2D纹理的对应关系.  
这个方法需要曲面是参数化定义, 如果曲面是[隐式表达](docs/FundamentalsofComputerGraphics/2_miscellaneous_math?id=_253-3d-implicit-surfaces)呢? 或者这个曲面就是一个三角网, 该怎么映射纹理坐标系呢? 有两种方法, 一种是根据曲面上点的空间坐标通过几何计算得到纹理坐标, 另外一种是根据曲面三角网顶点的纹理坐标来内插其他点的坐标.

<a id="markdown-_1121-geometrically-determined-coordinates几何上确定的坐标系" name="_1121-geometrically-determined-coordinates几何上确定的坐标系"></a>
#### _11.2.1 Geometrically Determined Coordinates几何上确定的坐标系

geometrically determined coordinates适用于比较简单的场景.  
这是一张纹理图:  
<img src="./_images/texture_test_image.png">

**Planar Projection平面投影**

和第7章的[正交投影](docs/FundamentalsofComputerGraphics/7_viewing?id=_712-the-orthographic-projection-transformation正交投影变换)类似, 对三维空间坐标进行仿射变换(平移缩放)就可以对应到纹理坐标u, v
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
Planar projection不具有injective, 因为如果是一个封闭的对象, 那么这个对象的前面和后面的两个点, 对应的texture coordinate的坐标是一样的:  
<img src="_images/fundamentals_of_computer_graphics/planar_projection.png">
<img src="_images/fundamentals_of_computer_graphics/planar_projection1.png">  
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

球形坐标系在两极会造成较大的变形, 替代方案是cubemaps, 可以改善这种情况, 但是会失去一定的连续性.  
立方体有六个面, 对应六个texture map. 意味着对象的不同区域投影到不同的texture map, 投影就是正交投影.  
例如: 如果|x|>|y|, |x|>|z|, 那么根据x的正负, 分别投影到+x面或者-x面.  
OpenGL里有对应的六个面的转换方法, 这里不列公式了, 用到的时候看书吧.

<a id="markdown-_1122-interpolated-texture-coordinates内插纹理坐标系" name="_1122-interpolated-texture-coordinates内插纹理坐标系"></a>
#### _11.2.2 Interpolated Texture Coordinates内插纹理坐标系

三角网只有三角形顶点的坐标和属性, 三角形内部用内插的方式来获取坐标和属性.  
内插方法采用barycentric coordinate重心坐标, 中心坐标参照第二章节

texture mapping的质量和顶点相关, 例如顶点的密度等等

这种方式在某些区域依然有变形, 比如书中举的例子: 鼻翼, 地球两极. 皆因为小的三角形对应面积较大的texture space

<a id="markdown-_1123-tiling-wrapping-modes-and-texture-transformations平铺环绕模式-和纹理变换" name="_1123-tiling-wrapping-modes-and-texture-transformations平铺环绕模式-和纹理变换"></a>
#### _11.2.3 Tiling, Wrapping Modes, and Texture Transformations平铺、环绕模式, 和纹理变换

有时图像上的点对应的texture coordinate由于某种原因落到了terxture space的外面  
有时我们只对一小部分区域着色, 这样texture space有很大一部分区域是黑色的, 为了让这一小部分区域有更精细的细节, 我们要把texture space放大, 让这一小部分变成unit quare  
对于落在这部分区域之外的部分, 可以赋予一个背景色常量, 也可以用这部分区域的平均值作为背景色  

对于重复图形的图像, 比如地板、砖块墙, 应该只需要一块重复区域的纹理, 这样更节省空间. 详情待实际应用再看.

<a id="markdown-_1124-perspective-correct-interpolation透视校正内插" name="_1124-perspective-correct-interpolation透视校正内插"></a>
#### _11.2.4 Perspective Correct Interpolation透视校正内插

<a id="markdown-_1125-continuity-ans-seams连续性和接缝" name="_1125-continuity-ans-seams连续性和接缝"></a>
#### _11.2.5 Continuity ans Seams连续性和接缝

<a id="markdown-_113-antialiasing-texture-lookups反走样纹理查找" name="_113-antialiasing-texture-lookups反走样纹理查找"></a>
### _11.3 Antialiasing Texture Lookups反走样纹理查找

texture mapping的第二个基本问题是antialiasing  
由于纹理是表现细节的关键, 所以纹理是引起aliasing的关键来源  
纹理aliasing和光栅化aliasing一样, 都涉及到sample、blur、reconstruction等等

<a id="markdown-_1131-the-footprint-of-a-pixel" name="_1131-the-footprint-of-a-pixel"></a>
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

<a id="markdown-_1132-reconstruction重建" name="_1132-reconstruction重建"></a>
#### _11.3.2 Reconstruction重建

一个像素对应的footprint可能很大, 需要做平均  
也可能小于一个texels纹素的大小, 这样就不连续了, 这就需要reconstruction  
这和第9章节的reconstruction不太一样, 第9章需要重建的离散数据是规律采样的  
但是纹理是不规则的, 详情待用到时再研究

<a id="markdown-_1133-mipmapping" name="_1133-mipmapping"></a>
#### _11.3.3 MipMapping

上一节我们说到一个pixel对应的footprint可能大于一个texel, 为了antialiasing我们需要做平均.   
但是有时候可能是几千个texels, 实时计算效率就太低了. 我们可以提前计算和存储.  
这个idea称之为"MIP mapping"或者"mip-mapping"  

一个mipmap是指一系列texture, 它们是指的同一个图像的纹理, 但是分辨率从高到低各不相同.  
最高的分辨率的texture称为base level或者level0, level1是在两个维度降低一倍分辨率, 也就是level 0的2*2个texels变成level 1的1个texel, 依此类推.  
level k的一个texel相当于base level的$2^k \times 2^k$个texels  

这些不同分辨率的texture层层堆叠, 我们形象的称之为image pyramid图像金字塔.

<a id="markdown-_1134-basic-texture-filtering-with-mipmap用mipmaps进行基本纹理过滤" name="_1134-basic-texture-filtering-with-mipmap用mipmaps进行基本纹理过滤"></a>
#### _11.3.4 Basic Texture Filtering with Mipmap用mipmaps进行基本纹理过滤

我们又了mipmaps, 如果我们计算出图像上的某个pixel对应原始texture上的$D \times D$个texels, 我们就能对应上mipmaps的level $k = log_2D$的一个texel.  
这是理想状况, 实际情况是pixel对应的texels不一定是规矩的正方形. 就算是正方形, $log_2D$也不一定是整数, 要知道, 我们存储的mipmaps都是整数level  
怎么办呢? 两种方法, 取最接近的level k(可能会产生seams接缝), 或者取最接近的两个level, 然后做差值(相比第一种更加smooth)

对于不是正方形的情况呢? 我们在11.3.1footprint章节里说到, footprint是一个平行四边形, 我们也有两种方法做处理, 一种是通过面积做处理, 另一种是根据四边形的长边来做处理, 我们说长边long edge这种方法:  
$$D = max\left\{\left\|u_x\right\|, \left\|u_y\right\|\right\}$$
preseudo是:
```
// u, v是pixel中心对应的texture坐标, J是texels矩阵
Color mipmap_sample_trilinear(Texture mip[], float u, float v, matrix J)
{
  // 最大宽度long edge
  D = max_column_norm(J)
  k = log2(D)
  k0 = floor(k); k1 = k0 + 1
  // interpolation参数
  a = k1 - k; b = 1 - a
  // 涉及到上一节reconstruction的内容, 根据u, v计算texel值
  c0 = tex_sample_bilinear(mip[k0], u, v)
  c1 = tex_sample_bilinear(mip[k1], u, v)
  // interpolation
  return a * c0 + b * c1
}
```
如果footprint是完美的正方形区域, 且宽度是2的指数, 那么mipmap的antialiasing很完美  
但是如果不规整, 那就会有缺陷, 下图能看到效果:  
<img src="./_images/mipmap_antialiasing.png">  
其中最后一个图Anisotropic filtering各项异性过滤是另外一种计算方法, 如下:

<a id="markdown-_1135-anisotropic-filtering各向异性过滤" name="_1135-anisotropic-filtering各向异性过滤"></a>
#### _11.3.5 Anisotropic Filtering各向异性过滤

这个计算方法是根据短边来匹配mipmap, 然后再沿着长边做平均. 这里没有详述.

<a id="markdown-_114-applications-of-texture-mapping纹理映射的应用" name="_114-applications-of-texture-mapping纹理映射的应用"></a>
### _11.4 Applications of Texture Mapping纹理映射的应用

"textures are a very general tool with applications limited only by what theprogrammer can think up"  
纹理是只有程序员才能理解的通用工具??

<a id="markdown-_1141-controlling-shading-parameters控制着色参数" name="_1141-controlling-shading-parameters控制着色参数"></a>
#### _11.4.1 Controlling Shading Parameters控制着色参数

在第2章节光线追踪和第10章节着色里说到漫反射计算  
在计算时有一个参数, 这个参数就可以通过纹理来获得, 来实现不同的光照纹理效果, 而不只是均质的黑白色

<a id="markdown-_1142-normal-maps-and-bump-maps法线图和凹凸贴图" name="_1142-normal-maps-and-bump-maps法线图和凹凸贴图"></a>
#### _11.4.2 Normal Maps and Bump Maps法线图和凹凸贴图

上面说的纹理存储的都是颜色值, 颜色值只是一个属性, 我们把这个属性换成别的呢?  
实际的物体表面不会像理想的几何模型, 它是凹凸不平的, 尽管有些物体已经做到了非常光滑, 但实际情况不会达到理想状态.  
所以我们把这个属性换成normal法线, 就能表现表现实际的凹凸不平的情况.  
我们把texture存储的RGB颜色值换成normal法线的向量.

但是normal map和表面是对应的, 如果表面出现变化, 那么normal map也就用不了了.  
我们可以用bump maps凹凸贴图来替代normal maps, 或者说normal可以根据bump得到.    
bump maps存储了一个texels的相对理想模型的高度, 比如理想模型是一个圆, 某个texels纹素可能凹进去低于圆的高度, 也可能凸出高于圆的高度.
我们存储这个相对高度, 这样, 我们就能计算出法线, 实现光影效果.

<a id="markdown-_1143-displacement-maps位移贴图" name="_1143-displacement-maps位移贴图"></a>
#### _11.4.3 Displacement Maps位移贴图

在GAMES101那门课里有一张这样的图, 形象的说明了normal maps、bump maps和displacement maps的区别:  
<img src="./_images/displacment_maps.png" width=50%>  
normal maps和bump maps不改变表面, 只改变法线, 相当于实现了一个光影的trick  
而displacement maps和bump maps一样,也是存储了texels的相对高度  
但是它真的改变了表面, 让表面上的点沿法线移动相对高度  
从图中可以看到, displacement maps更加真实

<a id="markdown-_1144-shadow-maps阴影贴图" name="_1144-shadow-maps阴影贴图"></a>
#### _11.4.4 Shadow Maps阴影贴图

和depth map类似, 只不过depth map是从视点出发, shadow是从光源出发.  
depth map存储的是从camera出发的射线, 最先照射到的表面.  
而shadow map则是从一个点光源出发的射线, 最先照射到的表面.  

所以渲染需要做两个pass, 第一个pass还是用z-buffer来判断一个点是否能被camera看到, 需不需要渲染.  
如果camera能看到, 则进行第二个pass, 看这个点和光源的距离和shadow map中的深度做比较, 如果相等, 则判定其被照亮, 如果大于, 则被遮挡, 处在阴影中.  

但是因为精度的关系, 我们做不到判断两个值相等, 所以引入了*shadow bias*, 如果小于这个值, 则判断其是同一个表面, 被光源照亮.  

另外, 在阴影的边缘会有锯齿, 为了消除这种现象, 需要对周围进行采样, 得到一个0-1的值, 而不是非0(遮挡)即1(照亮)的结果, 这就是*percentage closer filtering*, 在[real-time rendering](docs/RealTimeRendering/7_shadows?id=_75-percentage-closer-filtering)里有更详细的介绍.

<a id="markdown-_1145-environment-maps环境贴图" name="_1145-environment-maps环境贴图"></a>
#### _11.4.5 Environment Maps环境贴图

环境光照用纹理图来实现

<a id="markdown-_115-procedural-3d-textures程序化三维纹理" name="_115-procedural-3d-textures程序化三维纹理"></a>
### _11.5 Procedural 3D Textures程序化三维纹理

<img src="./_images/3D_textures.png" width=50%>  

三维纹理会消耗大量内存, 有些情况下我们可以不用存储这么大的纹理, 而通过程序化计算得到

<a id="markdown-_1151-3d-stripe-textures三维条纹纹理" name="_1151-3d-stripe-textures三维条纹纹理"></a>
#### _11.5.1 3D Stripe Textures三维条纹纹理

用两个颜色实现不同宽度的条纹, 已经渐变条纹, 书中写了三个函数

<a id="markdown-_1152-solid-noise实体噪声" name="_1152-solid-noise实体噪声"></a>
#### _11.5.2 Solid Noise实体噪声

条纹是比较规则的形状, 我们要实现类似蛋纹(瓷器裂纹等各种随机但是连续的花纹)的斑驳纹理, 可以通过solid noise, 也被称为Perlin noise(已发明者命名)  

<a id="markdown-_1153-turbulance扰动" name="_1153-turbulance扰动"></a>
#### _11.5.3 Turbulance扰动

一些自然的纹理在相同的纹理中包含一些特征随机的变化. 例如规整的条纹加上扰动, 变成奇怪的线条, 但是整体而言还是条纹.