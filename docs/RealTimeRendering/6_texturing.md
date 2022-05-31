<!-- TOC -->

- [_6.1 The Texturing Pipeline](#_61-the-texturing-pipeline)
  - [_6.1.1 The Projection Function](#_611-the-projection-function)
  - [_6.1.2 The Corresponder Function](#_612-the-corresponder-function)
  - [_6.1.3 Texture Values](#_613-texture-values)
- [_6.7 Bump Mapping](#_67-bump-mapping)
  - [_6.7.1 Blinn Methods](#_671-blinn-methods)
  - [_6.7.2 Normal Mapping](#_672-normal-mapping)
- [_6.8 Parallax Mapping视差映射](#_68-parallax-mapping视差映射)
  - [_6.8.1 Parallax Occlusion Mapping](#_681-parallax-occlusion-mapping)

<!-- /TOC -->

**Texturing**

surface的texture是指其外观.  
computer graphics的texturing是指利用图像、函数等数据来修改surface的外观

例如一个砖墙, 我们只需在集合上定义其为一个矩形. 然后用颜色texture来给他贴上颜色, 用粗糙度texture来区分砖和水泥, 用bump texture来改变接缝处水泥的法线, 实现其不平整, 用户displacement mapping来改变surface的高度, 实现更好的效果.

### _6.1 The Texturing Pipeline

texture的一个像素称为texel.  
roughness texture改变roughness值, 从而影响specular光照值.  
bump texture改变normal值, 从而影响normal mapping, 从而影响diffuse和specular光照值.

texuring用texture pipeline来描述.  

世界坐标系下的一个点的位置通过*projector*得到*texture coordinates*, 这个坐标可以从texture上获取对应的值. 这是一个*mapping*过程, 称之为*texture mappping*:  
<img src="_images/real_time_rendering/texture_pipeline.png">  
举个例子更好理解:  
<img src="_images/real_time_rendering/texture_pipeline_brick_wall.png">  
世界坐标系的一个点的三维坐标, 经过projection得到texture参照系下的二维坐标(值的范围是[0, 1]), 再通过corresponding function转换为texture对应的坐标(texel的列和行), 获取texel的值

#### _6.1.1 The Projection Function

首先我们要将一个点的三维坐标转换为texture坐标系下的二维坐标$(u, v)$. 需要用到project function.  
vertex的$(u, v)$坐标艺术家可以定义.

比如, 根据scene处在一个cube当中, 可以给model定义6个texture, 然后将空间坐标进行投影.  
或者, 不要投影, 有些model可以用曲面定义, 曲面的定义就包括$u, v$坐标, 例如球面、圆柱等等, 这就可以将曲面的参数和texture coordinate对应上.  
球面坐标系包括方位角$\phi$和极角$\theta$, 可以对应texture coordinate.  
圆柱可以用圆的方位以及轴的距离来对应texture coordinte.  
平面的话可以用正交投影.

在非交互式的渲染中, 渲染过程中执行projector function.  
在实时渲染中, 可能在建模过程中就会执行projector function, 把结果存储起来. 但也不总是这样, shading过程中执行会有更好的精度和灵活性. 另外, 一些渲染方法, 比如environment mapping, 会有自己特殊的projector function.

投影会导致边缘变形, 所以艺术家会将texture展开成平面, 使模型的各个区域对应的texture面积大小接近. 同时连通性也很重要, 不能出现裂隙.

有的texture coordinate是三维的, 另外一个参数是投影方向的深度. 还有四维的, 纹理可以随着时间或者观看距离而变化.

还有一种texture的输入是观测方向, 观测方向和法线的关系决定texture的值.

值得一提的是一维texture, 例如地形图的texture, 输入是高度, 输出是颜色.

无论什么样的texture coordinate, 背后的思想都是一样的: 用texture coordinate来插值获取texture的值. 再被插值之前, 需要用projector function转换获取.

#### _6.1.2 The Corresponder Function

corresponder function是将texture coordinate转换为texture-space location.  
function的一个例子是API. 另外一种类型是matrix transformation.

还有一些corresponder function会控制texture image的应用方式. texture coordinate的取值范围是[0, 1], 但是范围之外会发生什么呢? corresponder function来决定. 在OpenGL中, 这种function称为wrapping mode. 在DirectX里称为texture addressing mode. 取值范围之外的处理方式有以下几种:
1. wrap(DirectX), repeat(OpenGL), or tile: 在表面重复, 这是默认值.
2. mirror: 也是重复, 但是会连续做镜面翻转.
3. clamp(DirectX), clamp to edge(OpenGL): 限制值在[0, 1]之间. 如果超出, 则取1.
4. border(DirectX), clamp to border(OpenGL): 限制值在[0, 1]之间, 如果超出, 则取默认值.  

DirectX还有一种mirror once的模式, 只mirror repeat一次.  
看下面的图就一目了然了:  
<img src="_images/real_time_rendering/corresponder_function.png">  

纹理的重复平铺虽然很方便, 但是有时候会导致边缘接缝的错位等问题, 书中介绍了两种方法.

#### _6.1.3 Texture Values

texture value大都是从image中获取. 称为image texturing.  
还有一种方式是procedural texturing. 不从内存里获取, 而是经过函数计算得到texture value.

texture value可以是RGB颜色, 也可以是灰度值, 还有$RGB\alpha$, $\alpha$是透明度, 还有roughness, 还有法线值.

有的texture value需要做转换才能用, 比如法线以RGB格式存在texture image里面, 值的范围是[0, 1], 需要转换为法线的正常范围[-1, 1]才能用.

### _6.6 Alpha Mapping

我们设定一个值, 当纹理的alpha值小于这个数值时, 则不渲染这个像素(discard), 例如前景和背景叠加, 前景的一部分是透明的, 那么就可以这么实现.  
这样可以节省成本, 这一部分可以不要再做blend.

<a id="markdown-_67-bump-mapping" name="_67-bump-mapping"></a>
### _6.7 Bump Mapping

bump mapping可以表现微小的细节, 能实现很好的3D效果, 而不要增加几何元素.

对象上的细节可以分为: 覆盖很多pixel的macro-feature大特征, 覆盖较少pixel的meso-feature中特征, 小于一个pixel的微小特征micro-feature. 在动画和人机交互场景下, 这些分类可能是变化的, 例如镜头拉远或拉近.

宏观几何macrogeometry通过点和三角形来表现, 或者其他几何形状. 创建三维人物时, 头和胳膊就是在宏观尺度建模.  
microgeometry微观几何封装在shading model里面. 在pixel shader和texture里实现. 对象的roughness属于microgeometry的范畴.

人物脸上的皱纹、衣服的褶皱属于meso-geometry中观几何, 描述单个三角形渲染效率太低, 但是又不能忽视的细节.  
bump mapping技术就是为了解决meso-geometry的问题. 不同类型的bump mapping技术的区别在于如何表示细节特征.

blinn发现了surface的小细节, surface的normal并不是均匀的, 他将法线的扰动数据存储在阵列中, 几何并不发生改变, normal的改变会影响外观.  

normal的扰动必须参照local坐标系(为什么要这样? 为什么不直接存储world space下的扰动后的normal? 6.7.2会讲). 所以, 每个vertex的*tangent frame切线框架*, 或者称为*tangent-space basis*被存储起来. 这个frame可以把light transform到这个坐标系下, 从而计算扰动的normal对light的影响. 

tangent, bitangent, normal构成一个坐标系, 简称为TBN. 下图比较直观:  
<img src="_images/real_time_rendering/TBN.png">  

为了节省空间, 可以只存储tangent和bitangent, normal通过cross product计算得到. 

#### _6.7.1 Blinn Methods

一种方法是在texture上存储两个值$b_u, b_v$, 分别表示$u$和$v$方向的扰动. 这种texture称为*offset vector bump map*或者*offset map*

另外一种方法是heightfield. 只存储一个高度值, 相邻的两个高度值之间的差值代表扰动的方向. 

<img src="_images/real_time_rendering/bump_mapping_blinn_methods.png">

heightfield的示意如下:  
<img src="_images/real_time_rendering/heightfield.png">

#### _6.7.2 Normal Mapping

bump mapping最普遍的方法是直接存储normal map. 算法和结果在数学上和blinn method一样, 只是存储格式和shader计算不一样.

normal vector $(x, y, z)$的取值范围是[-1, 1], 如果存储都8-bit的image中, 需要将其映射到[0, 255]. 那么0代表-1, 128代表0, 255代表1. 所以[128, 128, 255]浅蓝色代表(0, 0, 1)一个平坦的surface. 如下图所示:  
<img src="_images/real_time_rendering/normal_map.png">  
这里我们说的是直接将world space下(或者object space)的扰动后的normal存储到texture image里面, 这样很直观, 但是一般不这么用, 因为扰动后的normal和space是绑定的, 限制了它的重用.

所以我们将normal的扰动参照于tangent space, 这样它就和surface绑定, 可以重复使用. 而且存储时可以压缩, 因为tangent space的z轴方向的值都是正的.

### _6.8 Parallax Mapping视差映射

bump mapping的一个缺点是它不改变几何形状, 例如砖墙的水泥缝, 不同的角度观看, 会有不一样的遮挡, 看到的缝的大小也不一样. 如果能将水泥缝真的凹进去, 效果会更好.  

parallax mapping登场, 它的思想是凹凸在不同的视角下有相对高度, 这样看到的东西也不一样. 水泥缝隙如果从侧面看, 将会被砖遮挡, 在解释一下, 墙的几何形体还是平面, 正常情况下不论从哪个角度看, 都能看到所有的砖和水泥缝隙, 但是经过parallax mapping, 看到的点被移动了, 从侧面看墙缝, 应该也能看到, 但是现在看到的是墙.

如何做到的? parallax mapping存储的也是heightfield, 相比于surface的高度. 看向一个点p, 通过p点对应的parallax texture的height, 以及视角在tangent space的水平方向的二维向量$v_{xy}$, 以及垂直方向的值$v_z$, 计算出另外一个点p_{adj}, 这就是应该看到的点:
$$p_{adj} = p + \frac{h\cdot v_{xy}}{v_z}$$
意思是将$p$往视线方向移动, 移动的距离和其高度成正比, 与视线的高度成反比:  
<img src="_images/real_time_rendering/parallax_mapping.png">  
左图, 一个平面, 视线看向p点, heightfield反应了实际的surface, 视线应该看到的是$p_{ideal}$这个点, 需要将p点往视线方向移动. 右图反应了这种算法. (好像有点不准确, 但是实现了近似)  

这种算法在视线和surface贴近时不太适用, 因为$v_z$无限小, 会导致偏移过大. 一种优化的方法是做一个限定, 只能移动h这么远, 将上面的equation改成:
$$p_{adj} = p + h\cdot v_{xy}$$
示意如下:  
<img src="_images/real_time_rendering/parallax_mapping1.png">  

#### _6.8.1 Parallax Occlusion Mapping

bump mapping改变了normal, parallax mapping近似实现了视角的问题, 但是没有完全解决heightfield遮挡的问题. 我们需要精确知道视线和heightfield的交点, 也就是实际看到的点.

我们可以在pixel shader里实现计算视线和heightfield的交点. (我猜应该是在glsl里实现)

这种算法称为*parallax occlusion mapping(POM)*或者*relief mapping*. 其基本原理如下图所示:  
<img src="_images/real_time_rendering/parallax_occlusion_mapping.png">  
视线在surface上投影, 以一定的interval在surface上采样, 然后在heightfield texture上找到对应的value组成一个点, 这些点构成一条线, 视线vector和这条线的交点就能近似得到视线和高度场的交点. 也就是视线实际看到的点.

这个方法有很多文献. 书中介绍了一些, 想要深入了解的话需要深入到文献中去.