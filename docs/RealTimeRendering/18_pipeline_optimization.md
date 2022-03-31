<!-- TOC -->

- [_18.1 Profiling and Debugging Tools](#_181-profiling-and-debugging-tools)
- [_18.2 Locating the Bottleneck](#_182-locating-the-bottleneck)
- [_18.3 Performance Measurements](#_183-performance-measurements)
- [_18.4 Optimization](#_184-optimization)
  - [_18.4.1 Application Stage](#_1841-application-stage)
  - [_18.4.2 API Calls](#_1842-api-calls)
  - [_18.4.3 Geometry Stage](#_1843-geometry-stage)
  - [_18.4.4 Rasterization Stage](#_1844-rasterization-stage)
  - [_18.4.5 Pixel processing Stage](#_1845-pixel-processing-stage)

<!-- /TOC -->

**Pipeline Optimization**

<a id="markdown-_181-profiling-and-debugging-tools" name="_181-profiling-and-debugging-tools"></a>
### _18.1 Profiling and Debugging Tools

<a id="markdown-_182-locating-the-bottleneck" name="_182-locating-the-bottleneck"></a>
### _18.2 Locating the Bottleneck

<a id="markdown-_183-performance-measurements" name="_183-performance-measurements"></a>
### _18.3 Performance Measurements

<a id="markdown-_184-optimization" name="_184-optimization"></a>
### _18.4 Optimization

<a id="markdown-_1841-application-stage" name="_1841-application-stage"></a>
#### _18.4.1 Application Stage

<a id="markdown-_1842-api-calls" name="_1842-api-calls"></a>
#### _18.4.2 API Calls

<a id="markdown-_1843-geometry-stage" name="_1843-geometry-stage"></a>
#### _18.4.3 Geometry Stage

<a id="markdown-_1844-rasterization-stage" name="_1844-rasterization-stage"></a>
#### _18.4.4 Rasterization Stage

<a id="markdown-_1845-pixel-processing-stage" name="_1845-pixel-processing-stage"></a>
#### _18.4.5 Pixel processing Stage

优化pixel processing是有利的, 因为pixel一般比vertex要多. 也有例外, vertex都要处理, 尽管vertex最后没有生成可见的pixel. 渲染引擎中无效的剔除可能会使顶点着色成本超过像素着色. 太小的三角形不止造成顶点着色的开销, 还会创建很多四边形, 这些四边形又会有其他的工作需要做, 例如纹理. 仅覆盖几个像素的纹理网格往往只有较低的线程占有率. 纹理采样会耗费大量的时间, 当从texture获取数据时, GPU会切换到其他的shader program, 然后等到获取到texture数据后再切换回来. 过于复杂的shader也会导致较低的线程占有率, 同一时间只有很少的线程运行, 这称为register pressure. 频繁切换线程也会导致其他问题, 比如缓存未命中. Wronski讨论了很多这一类的问题.

首先，使用原生纹理和像素格式，即使用图形加速器内部使用的格式，以避免额外的格式转换。还可以只加载需要的mipmap, 以及texture compression. 更少更小的texture占有更少的内存空间, 意味着更低的传输和时间的节省. texture compression也能提高缓存效率, 因为同样的缓存空间对应更多的像素.

一个技术细节是, 不同距离采用不同的pixel shader program. 例如有的scene, 近的对象需要bump mapping, 远的对象则不需要, 但是可能有需要其他技术. 

三角形被rasterized时只要fragment可见才应该调用pixel shader. GPU的early-z test将fragment的z-depth和z-buffer做对比检查. 如果不可见, 就可以discard, 不用再调用shader. 

为了理解program的行为, 这里有一张depth complexity(覆盖一个像素的surface的数量)的可视化图:  
<img src="_images/real_time_rendering/depth_complexity.png">  
生成depth complexity image可以调用OpenGL的glBlendFunc(GL_ONE, GL_ONE), 同时关闭z-buffering. 最开始image是黑色的, scene里的每个object都会被渲染成(1/255, 1/255, 1/255), 一个pixel上每渲染一个object, 则增加1.

pixel overdraw的数量和多少surface被渲染有关. 启用z-buffer, 再次渲染, 就能知道pixel shader的调用次数. overdraw是不必要的对不可见surface的着色计算, 因为之后它会被pixel shader隐藏. [deferred rendering](docs/RealTimeRendering/20_efficient_shading?id=_201-deferred-shading)和Ray tracing的优点是在执行所有可见性计算之后执行着色. 从而避免overdraw.

如果两个三角形覆盖一个pixel, 那么depth complexity就是2, 如果远处的三角形先drawn, 近处的triangle再overdraw, 那么overdraw的数量就是1. 如果近处的triangle先draw, 远处的triangle depth test失败, 不写入, 那么就没有overdraw. 对于覆盖一个像素的随机的一组不透明的triangles, 平均draw的数量是harmonix series:
$$H(n) = 1 + \frac{1}{2} + \frac{1}{3} + ... + \frac{1}{n}$$
背后的逻辑是: 第一个triangle肯定会被写入, 第二个triangle有1/2的概率在第一个triangle的前面, 然后被写入, 第三个triangle有1/3的概率在第二个triangle的前面, 然后被写入, 以此类推. 假设n是无穷, 那么:
$$\lim_{n \to \infty} H(n) = ln(n) + \gamma$$
$ln(n) = \log_{e}n$, $\gamma = 0.57721...$, 是Euler-Mascheroni常数. depth complexity在比较小的时候, overdraw快速上升, 但是速度很快降下去. 例如, depth complexity是4时, 平均2.08 draws, 但是都11时, draws才3.02. depth complexity达到12367时, 平均10 draws.

overdraw的问题好像看起来没那么糟糕, 但是我们依然想避免它. 就opaque objects从近到远排序是一个解决办法. 被遮挡住的object不会写入颜色和z-buffer. fragment也会被occlusion culling hardware剔除从而到达不了pixel shader program. 有很多方法可以实现排序, 可以利用距离, 也可以利用数据结构.

有一项技术适用于复杂的pixel shader program. 执行z-prepass将所以几何图形的深度写入z-buffer，然后再正常渲染整个场景. 这消除了所有overdraw, 但代价是所以几何体都要被执行一遍. Pettineo在其论文里提到使用z-prepass的原因就是为了消除overdraw. 但是，粗略的从前到后的顺序绘制可能会更好. 一种折中的方法是识别并首先绘制几个大的、简单的遮挡物。不过不同的application适用不同的方法, 视情况而定.

之前我们建议按着色器和纹理进行分组以最小化状态变化；这里我们又提出按距离顺序渲染对象。这两个方案通常给出不同的绘制顺序，相互冲突。混合方案可能能解决这个问题，例如，按depth对近处的对象进行排序，按材料对其他对象进行排序。一个常见的、灵活的解决方案是为每个对象创建一个排序键, 进行封装:  
<img src="_images/real_time_rendering/shading_sort.png">

我们可以选择按距离排序，因为精度限制, 我们可以按着色器分组, 着色器相关联的给定距离范围内的对象组成一组。如果某些对象具有相同的深度并使用相同的着色器，则使用纹理标识符对对象进行排序，将具有相同纹理的对象组成一组. 我们要注意尽可能的用整数存储, 这样能够高效的排序, 从而减少overdraw和state change.