<!-- TOC -->

- [_2.4 Rasterization](#_24-rasterization)
- [_2.5 Pixel Processing](#_25-pixel-processing)
  - [_2.5.1 Pixel Shading](#_251-pixel-shading)
  - [_2.5.2 Merging](#_252-merging)

<!-- /TOC -->

本章介绍实时图形的核心: 图形渲染管线.  
管道的主要功能是在给定虚拟相机、三维对象、光源等资源的情况下, 生成或渲染二维图像.  
二维图像中对象的位置和形状由它们的几何形状、环境(例如相互遮挡关系等)以及相机位置决定.  
对象的外观受其材质、光源、纹理和着色方程的影响.

### _2.1 Architecture

pipeline可以理解为有规则的过程, 例如工厂的装配线、快餐厨房的食品制作流程.  
图形渲染也有其pipeine.

pipeline的各个阶段可以并行, 例如三明治的制作, 一个人准备面包，另一个人加肉，另一个人加配料, 每个人将其成果传递给下个人之后就可以立即再次制作。如果每个人需要20秒来完成他们的任务，那么理论上每20秒就可以制作一个三明治，每分钟3个. 但假设其中的一个阶段变慢, 例如加肉需要30秒, 那么制作一个三明治最快也是30秒, 每分钟两个三明治. 那么这个阶段就是性能瓶颈，因为它决定了整个生产的速度。因为后面的加配料的阶段需要等待加肉完成, is starved.

渲染的pipeline可以分为4个阶段, 如下图所示:  
<img src="_images/real_time_rendering/pipeline.png">

渲染速度可以用每秒帧数(frames per second, FPS)表示，即每秒渲染的图像数量, 这个指标和渲染计算的效率和显卡的性能相关.    
还有一个指标是赫兹(Hz), 是指每秒刷新率, 和显示器相关, 如果fps比刷新率低, 那么会出现"卡顿", 如果fps比刷新率高, 是一种浪费, 最好是两者一致.

顾名思义，application阶段由应用驱动，因此通常是通过在CPU上运行的软件实现.  
CPU通常高效的利用多线程去运行applicationn阶段负责的各种任务, 例如碰撞检测、全局加速算法、动画、物理模拟等，具体取决于应用程序的类型.  
下一个阶段是geometry processing，它处理变换、投影和其他几何处理. 这个阶段计算要绘制什么，应该如何绘制，以及应该在哪里绘制. 几何处理阶段通常在包含许多可编程内核以及固定操作硬件的图形处理单元GPU上执行.
rasterization光栅化阶段通常将三个顶点作为输入，形成一个三角形，并找到该三角形内考虑的所有像素，然后将它们转发到下一个阶段.
最后，pixel processing像素处理阶段对每个像素执行一个程序以确定其颜色，并可能执行深度测试以查看它是否可见。它还可以执行逐像素操作，例如将新计算的颜色与先前的颜色混合, 光栅化和像素处理阶段也完全在GPU上处理.  
所有这些阶段及其内部管道将在接下来的四节中讨论。有关GPU如何处理这些阶段的更多详细信息，请参见第3章

### _2.2 The Application Stage

开发人员可以完全控制application阶段发生的事情，因为它通常在CPU上执行.  
因此，开发人员可在这个阶段做一些事情以提高性能, 性能的提升会影响后续阶段的表现.  
例如这个阶段的算法或者配置可以减少渲染的数量, 从而提高性能.

一些工作也会在GPU里进行, GPU可以高度并行.  

application最后会将几何元素送到下一个阶段geometry processing, 这是application阶段最重要的工作.  
这一阶段不会像geometry processing、rasterization和pixel processing那样细分成几个子阶段.  
它为了提高性能, 会在多个CPU上并行执行.

这个阶段通常的一个任务碰撞检测, 还会处理来自其键盘、鼠标或头戴式显示器的输入, 加速算法，例如特定的剔除算法(第19章)也在这里实现，以及其他阶段无法处理的工作.

<a id="markdown-_24-rasterization" name="_24-rasterization"></a>
### _2.4 Rasterization

<img src="_images/real_time_rendering/rasterization.png">

<a id="markdown-_25-pixel-processing" name="_25-pixel-processing"></a>
### _2.5 Pixel Processing

rasterization找到三角形和primitive里面的pixels之后, 就进入到pixel processing阶段.  
pixel processing分为pixel shading和merging两个部分, 如上图所示.  
pixel processing会对每个像素或每个sample单元做处理

<a id="markdown-_251-pixel-shading" name="_251-pixel-shading"></a>
#### _2.5.1 Pixel Shading

pixel shading的结果是颜色.  
rasterization的计算是有cpu完成, 而pixel shading是gpu完成的.  
pixel shader(OpenGL的fragment shader)可以实现这些计算.  
pixel shading比较重要的技术是texing, 从texture获取颜色、法线等等属性. chapter 6会详细讲解.

<a id="markdown-_252-merging" name="_252-merging"></a>
#### _2.5.2 Merging

fragment shader生成的颜色之后会merge到color buffer, color buffer存储每个像素的颜色值.  

这一阶段要解决可见性的问题. 基本的方法就是利用z-buffer做深度测试, 来判断是否需要渲染. 但是如果是透明的场景, 就复杂一些, 比z-buffer更大的对象也需要渲染, 最后做blend处理, section 5.5会做说明.

透明的场景就涉及到alpha channel, 可以用alpha test来discard掉不需要渲染的像素. section 6.6会详细讲解.

还有一个操作是stencil buffer, 也可以通过stencil test来discard掉不需要渲染的像素. stencil buffer可以实现特殊的效果. 例如简单的一个效果:  
<img src="_images/real_time_rendering/stencil_buffer.png">  
还可以实现选中物体(点击显示边框)的效果, 用模板函数将要显示边框的对象的模版值设置为1, 其它为0, 然后将物体放大一点, 再重新绘制这个对象, 但是是有另外一个单色的shader, 只绘制模版值为0的部分, 这样就只绘制外框了. 详情见: <https://learnopengl-cn.github.io/04%20Advanced%20OpenGL/02%20Stencil%20testing/>  
但是如何鼠标选中是一个问题.

这些操作执行完之后, 就把color buffer的数据显示到屏幕上.