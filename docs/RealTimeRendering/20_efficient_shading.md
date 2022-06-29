<!-- TOC -->

- [_20.1 Deferred Shading](#_201-deferred-shading)
- [_20.2 Decal Rendering](#_202-decal-rendering)

<!-- /TOC -->

**Efficient Shading**

对于复杂场景的效率提升. 上一章节聚焦于算法, cut triangle or mesh.  
这一章节聚集于材料和灯光.  
效率提升的方法都会有一定的cost, 希望这种tradeoff是划算的.

一个surface有多个光源照射, 有两种策略来实现渲染.  
一是用一个支持多个光源的shader, 这样只需要一个pass.  
而是用多个只支持一个光源的shader, 称为multi-pass shading, 计算每一个光源的效果, 然后叠加到framebuffer上.  
第二种方法效率更高, 因为每个shader更简单. one-pass shader太复杂了.

在18.4.5章节里讲到了通过减少和消除overdraw来避免不必要的shading. 如果我们能判断一个surface对最后的成像没有贡献, 那么我们就没有必要对其shading.  
其中一个技术是执行z-prepass. 对作为遮挡物的几何体渲染并记录其z-depth, 这个几何体遮挡住的部分(z-depth更小的部分)就不再渲染了. 这项技术将寻找几何体从对几何体shading的过程中分离开来. 这个idea在本章也非常重要, 它被很多替代的渲染方案所使用.

例如, z-prepass的问题是一个几何体要render两次(一次写入z-buffer, 一次正常render), 额外的开销可能比节省的时间还要多. 如果网格是通过tesselation、skinning等过程生成的, 额外的开销会更多. 再者, 对于有alpha值的对象, 每次render都要获取此值, 或者忽略这个对象, 只在第二次render时纳入, 但是有遗漏的风险. 基于这些原因, 第一次render时只写入大的occluder. 不够完整的第一次prepass在一些screen-space effects(例如ambient occlusion和ambient reflection)的实现中还是必要的. 还有一些加速技术也需要精确的z-prepass.

不考虑overdraw, 一个scene有大量的光源的话也会有非常大的开销. 例如一个scene里有50个光源, 对于multi-pass的方案, 则对每个object都有进行一次shader pass, 那就是50次. 一种技术是将一个object的local light限时在一定范围内, 比如一个有限半径的球体, 一个一定高度的锥体, 等等. 这里的假设是光源在超过一定范围时候其贡献可以忽略不计. 在本章我们定义光源的范围是一个球体, 实际上光源的范围也可以是其他形状. 通常光源的强度可以由半径来决定. 对于glossy specular materials, 光源对其影响的距离更远, 其对光照更加敏感. 对于极其光滑的surface, 这个影响的距离会无穷大, 这个时候就需要使用environment map等技术了.  

既然光源的影响范围有限, 那么一个简单的预处理是为每个mesh创建一个影响它的光源列表. 我们可以理解为这是在mesh和光源之间做碰撞实验, 找到overlap. 对mesh做shading时, 就可以只考虑这些光源, 而不用对全部光源都执行一遍. 这个方案的问题是, object或者光源移动了, 那么光源列表也就失效了. 另外, 出于性能考虑, 同样的材质的mesh会组成一个更大的mesh, 但是其中的单个mesh的光源列表可能是不一样的, 那么我们可以根据材质来组合, 根据空间做拆分.

另外一种方案是将静态光源固定在世界坐标系中. 例如，在游戏Just Cause 2正当防卫2的光照系统中，世界空间自上而下的网格存储场景的光照信息。一个grid cell是4m * 4m的区域大小. 每个cell都存储为RGBα纹理中的一个纹理像素，因此可以保存一个最多包含四个灯光的列表. 当渲染一个像素时, 这个区域的相关光源就会被找到. 缺点是可存储的光源数量有限, 有些场景下不适用.

我们的目标是高效的处理动态网格和光源. view或者scene的微小变化不能导致过大的渲染成本. 游戏DOOM2016毁灭战士有的场景有300个光源, Ashes of the Singularity有10000个. 一些场景里海量的particles会被当作单个光源. 还有的光源是探照灯模式, 照射范围很小.

<a id="markdown-_201-deferred-shading" name="_201-deferred-shading"></a>
### _20.1 Deferred Shading

到目前为止，在本书中，我们已经描述了forward shading，每个三角形都被发送到管道中，最后屏幕上的图像会更新其着色值. <font color=red>deferred shading则是在光照计算之前先执行visibility testing和surface属性的evaluation</font>. 该概念于1988年首次在硬件架构中引入，后来作为实验性PixelFlow系统的一部分包含在内，并用作离线软件解决方案，以帮助通过图像处理生成non-photoreal style. Calver 2003年的文章描述了在GPU中使用deferred shading的basic idea. 之后得到了推广和普遍的使用.

在forward shading中，我们使用shader和表示对象的mesh执行一个pass以计算最终图像. 这个pass获取材质属性, 然后将light应用在其上. 用于forward shading的z-prepass方法可以看作是geometry rendering和shading的一种decouple，因为第一个geometry pass旨在仅确定可见性，而所有shading工作，包括材质参数获取，都推迟到第二个执行几何过程, 以shading所有可见pixels.  
对于交互式渲染，deferred shading是指与可见对象相关的所有材质属性都由初始geometry pass生成和存储，后面的过程再将light应用于这些存储的表面材质属性. first pass存储的数据包括position、normal、texture coordinate等等. 这个pass获取了所有几何和材质信息, 这样objects就不再被需要了. model的几何彻底和光照计算decouple了. 请注意, 这个pass会发生overdraw, 因为我们获取了所有objects的信息, 包括不可见的, 但是这种额外的开销比光照计算的overdraw要小得多. shading pass的开销也比forward shading要小. 

<font color=red>存储surface属性的buffer通常被称为G-buffers, 意思是geometric buffers</font>. 一个G-buffer可以存储后续照明计算所需的任何内容, 如下图所示. 每个G-buffer都是一个单独的渲染目标(图中的各种image)。通常使用三到五个渲染目标作为G-buffers, 最多高达八个, 拥有更多目标会使用更多带宽，这会增加此buffer成为瓶颈的概率.   
<img src="_images/real_time_rendering/g_buffer.png">

再创建G-buffers之后, 进入到第二个pass, 也就是计算光照. 一种方法是多个光照一次进行, 根据G-buffers的数据来进行计算. 对于每个光照, 我们绘制一个屏幕尺寸的四边形, G-buffers作为texture来访问得到数据. 每个像素我们找到最近的surface, 并判断其是否在光照影响范围内. 如果在, 则计算光照效果, 把结果添加到output buffer. 依次计算每个光照贡献, 把效果叠加上去. 最后得到所有的光照贡献.

这个过程非常低效, 因为每个像素都要被所有光照计算一遍. 而且由于G-buffers的读写成本, 效率可能比forward shading更低. 效率提升的第一步, 我们可以根据light volume的屏幕边界, 来决定只绘制屏幕的一部分面积. 这样pixel的处理可能就显著的减少了. 我们还可以利用z-depth进一步的减少pixel的处理, 如果光源的sphere volume在z-buffer之后, 也就是把最近的surface遮挡了, 那么也就没有光照效果了. 概括地说，如果light的sphre volume在pixel的最小和最大depth不与最近的surface overlap，则light不会影响该像素. 这种overlap的判定有多种算法, 哪种算法最有效取决于实际情况.

传统的forward shading, shader获取光照和材质, 依次计算其效果. 可以用一个大而复杂的shader处理所有的情况, 也可以对不同的情况建立各自的小的shader. 第二种方式往往更有效率, 但是也存在管理成本. 而且shader之间的切换也导致效率低下.

deferred shading分离了光照计算和材质定义. 每个shader专注于材质参数提取或者光照计算, 不会同时专注于两者. 更小的shader的效率更高, 因为其更小, 也因为其更容易优化. 一个shader的register决定了其占有率, 这是决定shaders并行数量的关键因素. lighting和material的decouple也简化了系统管理. 如果scene里新增了一个light或者material, 只需增加一个小的shader.

single pass forward shading因为是一次计算所有光照和材质, 所以shadow maps需要在计算时都是可用的. 对于一个light一个pass的deferred shading, 一次计算只需要一个shadow map可用, 因为可以单独计算. 但是对于多个光源的组合效果则不适用.

basic deferred shading的单个material shader支持固定字段的材质, 这就意味着一个shader只对应一个材质, 有没有方法可以让一个shader支持多个材质? 我们可以让一个字段存储一个材质的blender factor, 存储另一个材质的tangent vector, 这样就实现了这个目的. 但是shader相应的也会变复杂, 可能会影响性能.

basic deferred shading还有其他缺点. 例如G-buffer对<font color=red>内存和带宽</font>有一定要求, 可以有低精度和压缩来解决. Pesce讨论了world-space和scene-space的normal数据的压缩和转换.

deferred shading的两个重要技术局限涉及transparency和<font color=red>antialiasing</font>. basic deferred shading不支持transparency, 因为一个pixel只存储一个surface的数据. 一个解决方案是用deferred shading渲染opaque objects之后再用forward shading渲染transparency objects. 现在可以为pixel存储transparency属性, 但第一方案是标准.

forward shading的优点是其可以很容易的支持MSAA(multi sampling antialiasing)等antialiasing技术. deferred shading要支持的话需要额外付出更多的内存、计算等开销(因为G-buffers的每个target都需要记录multi sampling的结果). 一种解决方法是edge检测, 这样只对edge上的pixel做MSAA.

虽然deferred shading有一些局限, 但它依然是商业领域中很实用的渲染方法. <font color=red>它自然地将geometry与shading, 以及light与material分开</font>，这意味着每一项都可以各自优化。

OpenGL里有具体的实施方法: [OpenGL course](https://learnopengl.com/Advanced-Lighting/Deferred-Shading)

<font color=red>总结一下:</font>

defered shading将

### _20.2 Decal Rendering

