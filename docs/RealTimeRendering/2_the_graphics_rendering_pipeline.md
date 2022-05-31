<!-- TOC -->

- [_2.4 Rasterization](#_24-rasterization)
- [_2.5 Pixel Processing](#_25-pixel-processing)
  - [_2.5.1 Pixel Shading](#_251-pixel-shading)
  - [_2.5.2 Merging](#_252-merging)

<!-- /TOC -->

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