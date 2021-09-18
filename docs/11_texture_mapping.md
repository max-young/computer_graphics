<!-- TOC -->

- [_11.1 Looking Up Texture Values](#_111-looking-up-texture-values)

<!-- /TOC -->

**Texture Mapping纹理映射**

我们观察世界时, 会发现太多的细节, 木头有纹理, 皮肤有皱纹, 衣服有编织结构, 就算是塑料和金属, 也有凹凸不平, 它们并不是光滑规整的模型.  

在图形学里我们引入texture的概念, 我们通过texture来呈现这些细节而不改变对象的形状.  
具体实现就是texture mapping纹理映射: 用texture map、 texture image、 texture来存储对象表面细节, 然后映射到对象表面.

纹理可以呈现细节、阴影、 反射. 难点是纹理映射过程中可能会造成变形、重叠等问题. 因为texture是二维的, 映射到三维物体表面, 涉及到投影等等问题.

<a id="markdown-_111-looking-up-texture-values" name="_111-looking-up-texture-values"></a>
### _11.1 Looking Up Texture Values