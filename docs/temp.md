<!-- TOC -->

- [前置知识](#前置知识)
  - [the rendering equation](#the-rendering-equation)
- [Shadows](#shadows)
  - [_7.1 Planar Shadows投射到平面上的阴影](#_71-planar-shadows投射到平面上的阴影)
    - [_7.1.1 Projection Shadows](#_711-projection-shadows)
    - [_7.1.2 Soft Shadows](#_712-soft-shadows)
  - [_7.2 Shadows on Curved Surfaces](#_72-shadows-on-curved-surfaces)
  - [_7.3 Shadow Volumes](#_73-shadow-volumes)
  - [_7.4 Shadow Maps](#_74-shadow-maps)
    - [_7.4.1 Resolution Enhancement](#_741-resolution-enhancement)
  - [_7.5 Percentage-Closer Filtering](#_75-percentage-closer-filtering)
  - [_7.6 Percentage-Closer Soft Shadows](#_76-percentage-closer-soft-shadows)

<!-- /TOC -->

**shadows**

## 前置知识

<a id="markdown-the-rendering-equation" name="the-rendering-equation"></a>
### the rendering equation

在GAMES101里面, the rendering equation是这么写的:
$$L_o(p, \omega_o) = L_e(p, \omega_o) + \int_{all\ \omega_i}L_i(p, \omega_i)\rho(p, \omega_i, \omega_o)\cos\theta_id\omega_i$$
意思是在某个点p的出射方向$k_o$的radiance等于这个点自己发射的radiance加上所有反射的radiance的积分.  

在实时渲染里, 会加入一个visibility项, 代表p点是否能够看到$\omega_i$方向入射的光:
$$L_o(p, \omega_o) = L_e(p, \omega_o) + \int_{all\ \omega_i}L_i(p, \omega_i)\rho(p, \omega_i, \omega_o)\cos\theta_iV(p, \omega_i)d\omega_i$$

<a id="markdown-shadow" name="shadow"></a>
## Shadows

关于术语:

<img src="./_images/real_time_rendering/shadow.png" width=50%>

occluder指的是遮挡物  
umbra可以翻译成本影, 是指完全看不到光源的区域  
penumbra可以翻译成半影, 是指看到部分光源的区域, 越靠近本影, 看到的面光源的面积越小  
月亮的圆缺就是这样形成的, 可以帮助理解  
这种包含umbra和penumbrad的shadow就叫soft shadow

如果是点光源或者定向光源, 就不存在umbra和penumbra的问题, 只有umbra, 称之为hard shadow.  

shadow的sharp和soft, 取决于多种因素, 比如光源大小, occluder和receiver的距离等等, 如下图:

<img src="./_images/real_time_rendering/soft_shadow.png" width=70%>

右下角的箱子都是hard shadow, 人的下半身hard shadow居多, 上半身soft shadow居多. 树越靠近底部, hard shadow越多, 树枝则都是soft shadow的效果

### _7.1 Planar Shadows投射到平面上的阴影

#### _7.1.1 Projection Shadows

<img src="./_images/real_time_rendering/projection_shadows.png" width=70%>

一个点光源, 一个occulder, 一个平面的receiver  
我们可以计算出occluder上的一个点再receiver上的投影坐标, 计算方法详见书本, 原理是相似三角形.  
这样我们就能做渲染了, 需要注意的问题是阴影不能在receiver的下面, 已经不能超过receiver的边界. 解决办法是先绘制receiver.  
另外有一种场景, 光源在occluder和receiver的中间, 这种计算方法也可能会计算出shadow, 需要避免这种情况.

#### _7.1.2 Soft Shadows

对于面光源, 就涉及到soft shadow的问题了.  
一种方法是把面光源当成很多个点光源, 然后用点光源的算法做很多次采样, 然后将生成的图片重叠, 做成texture. 这样做的开销非常大.

在这个方法的基础上有一些优化的方法, 比如做filtering减少采样率, 或者用antialiasing的方法对阴影做模糊处理, 但是效果并不是很好, 并不符合物理规律.

### _7.2 Shadows on Curved Surfaces

上面讲的方法是将occluder投影到receiver上, 计算出occluder上的点投影到receiver上的坐标, 从而算出阴影.  

这里讲的方法是阴影纹理, 从光源出发看向场景得到的纹理. 渲染时计算出场景上的点在阴影纹理上的坐标然后渲染.  
这种方法有个问题是, 从viewpoint出发我们能看到occluder和receiver, 他们在阴影纹理上的坐标是一样的, 我们需要辨别.  

### _7.3 Shadow Volumes

<img src="./_images/real_time_rendering/shadow_volumes.png" width=70%>

一个point light和一个triangle occluder构成了一个锥体, 在锥体的上半部分处在光照下  
在下半部分(truncated pyramid)的volume里没有光照, 这部分称之为shadow volume  

所以我们判断一个点是否在阴影中, 我们可以判断其是否在shadow volume.  
应该怎么判断呢? 在fundamentals of computer graphics里我们学过. 我们从viewpoint看向这个点, 这条视线和shadow volume求交点, 如果没有交点, 那么就在volume之外, 如果只有一个交点, 则在volume内, 如果有两个交点则在volume外. 那么加入场景里有很多个shadow volumes, 那么如果交点是奇数, 那么就在volume内, 是偶数则在volume外.  
书中说的方法是进入volume则加一, 出来则减一, 最后count是1则在volume内, 是0则在volume外, 原理一样.  

shadow volumes的方法开销比较大, 一种优化方法是用buffer. 这里不再详述.  
但是开销依然是不可控的, 所以这个方法应用很少. 但是这个方法依然是可取的, 可能以后有更好的优化方法, 效率提升了, 应用会更广.

### _7.4 Shadow Maps

我们从light出发, 看向场景, 将场景上的点离light的距离记录下来, 得到shadow map, 或者称之为shadow depth map或者shadow buffer.

我们从viewpoint出发看向场景上的点, 也能计算出这个点到光源的距离, 如果大于阴影贴图上的距离, 那么说明这个点被遮挡了, 处在阴影中. 这种算法应用最为广泛.

这种算法有一个优化的点是不用做场景下所有区域的shadow map, 只需要做视线区域和光照区域重合的部分.

在GAMES202里讲到, 这种方法会产生自遮挡:

<img src="./_images/real_time_rendering/shadow1.png" width=60%>

原因是阴影贴图是有分辨率的, 在阴影贴图的一个像素里, 深度是一样的, 如图中所示, viewpoint出发看到的点和橙色的点在阴影贴图里处在同一个像素, 深度一样, 但是viewpoint看到的点计算出来的深度更大, 这就会错误的把这个视点算作在阴影里.

其中一个解决办法是引入一个bias, 计算视线看到的点和light的距离时减去bias: 
<img src="./_images/real_time_rendering/shadow_bias.png" width=60%>

左图, 三个彩色的圆点都在对象的表面, 灰色的线是shadow map的像素中心, 有颜色的叉是三个圆点最近的shadow map的像素中心, 也就是说会拿这三个点和light的距离和对应的有颜色的叉的shadow depth做比较. 如果我们不引入bias, 那么蓝色和橙色的两个点就会被错误的算作处在阴影之中.  
中图和右图是两种bias的方法, 中图是减去一个固定的bias, 但是蓝色的点依然处在阴影中, 右图的bias跟slope坡度相关, 这种效果更好.

过多的bias可能会导致一个问题, 如下图中人物的脚: 本来应该在阴影中, 但是却因为bias造成了light leaks, 或者叫peter panning.  
<img src="./_images/real_time_rendering/light_leak.png" width=30%>

#### _7.4.1 Resolution Enhancement

shadow map也有分辨率, 如果分辨率较低, 就会造成下面的效果:
<img src="./_images/real_time_rendering/shadow_map_resolution.png" width=70%>

如左图所示, 地面上的阴影是格子装, 红色的格子是示意shadow map的一个texel, 图中的一个texel可能对应viewpoint看到的地面上的9个pixels, 那么这9个pixels都会rendering成阴影, 从而形成这样的效果. (右图是同样的分辨率, 经过优化算法之后得到的效果.)

直接的解决办法是提高shadow map的resolution, 但是这效率就降低了. 业界对优化方法做了大量的研究.  

其中一种方法是对shadow map进行矩阵变换. shadow map和viewpoint图形一样, 都有成像平面, 也就是说将对象投影到哪个平面上. 这个平面可以最旋转:

<img src="./_images/real_time_rendering/shadow_map_transform.png" width=70%>

左图, 离眼睛近的区域, 一个像素对应的floor实际区域更小, 假如shadow map的成像平面和floor平行, 那么shadow map的分辨率在floor的远端是满足要求的, 但是在近处不能满足. 右图中, 我们把shadow map的成像平面做一下旋转, 这样shadow map的分辨率在近处和远处都能满足要求了. 这个原理衍生出很多种算法, 每种算法都有其优缺点. 另外这种算法在某些场景下是不适用的, 因而应用不是很广.

另外一种方法是提高分辨率, 但是不是单纯的提高, 而是根据视线锥体将光线锥体将分成几个区域, 不同区域生成不用分辨率的shadow map, 从而实现离viewpoint近的区域shadow map的分辨率高, 远的区域分辨率低, 详情这里不再叙述了, 看得一知半解.

### _7.5 Percentage-Closer Filtering

对shadow map进行扩展可以实现soft shadow的效果. 基本的方法是对一个点的周围区域进行采样, 获得shadow map的depth, 然后做blending. 这种方法叫做percentage-closer filtering(PCF).

<img src="./_images/real_time_rendering/pcf.png" width=70%>

对于面光源, 会产生penumbra, 软阴影的实现方法是对面光源进行采样, 获得能看到的光源面积比例. 对于点光源, 要实现soft shadow的效果, 方法是相反的, 是对采样点周围的区域进行采样, 看被光源照亮的比例.

PCF的效果受很多因素的影响, 比如: 采样面积、采样数量, 采样方法, 采样权重等等.  
<img src="./_images/real_time_rendering/pcf_sampling.png" width=70%>  
图1是采样周围4X4的区域的效果, 图2是按照右图的采样模型去采样的效果, 图3是随机采样的效果.

上面说到了为了解决self-shadowing的问题, 引入了bias, PCF有可能会加重self-shadowing的问题, 因为采样多了, bias也累加了. 所以bias需要做优化. 有一些优化的方法, 比如按照一个锥形去做bias, 或者沿着法线做bias, 等等.

PCF的一个问题是, 在所有的阴影区域都会生成相同程度的soft shadow. 有些情况下是不合适的, 比如之前的图里面, 越靠近人的脚, 硬阴影更多.

### _7.6 Percentage-Closer Soft Shadows








