<!-- TOC -->

- [_4.1 The Basic ray-tracing Algorithm](#_41-the-basic-ray-tracing-algorithm)
- [_4.2 Perspective透视](#_42-perspective透视)

<!-- /TOC -->

**Ray Tracing光线追踪**

计算机图形学最重要的任务之一是rendering渲染3D对象  
通俗的说是将在某一个点观测到的4D对象显示在2D图像上.  
渲染过程的输入是3D对象, 输出是像素pixel.  
渲染有两种方法: object-order rendering, image-order rendering
object-order rendering是对每个对象做loop循环处理  
image-order rendering是对每个pixel做loop循环处理  
image-order rendering更加简单和灵活, 通常会耗时, 但是会生成更好的图像  

<a id="markdown-_41-the-basic-ray-tracing-algorithm" name="_41-the-basic-ray-tracing-algorithm"></a>
### _4.1 The Basic ray-tracing Algorithm

object通过pixel被看到. 这需要viewing ray视线和物体之间的intersecton  
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

最简单的投影是parallel projection平行投影, 就是将3D物体沿着统一的一条线路(视线)移到2D平面上  
如果这条线路是垂直于2D平面的, 我们称之为orthographic正交. 不垂直的话称之为oblique倾斜  
这种投影的决定因素是视线view direction和平面plane(角度) 

平行投影也是有弊端的, 比如他和我们的观感是不一样的, 我们人眼观测是近大远小, 两条铁轨实际是平行不相交的, 但是我们人眼看在远处它们相交了.  
这是因为人眼观测不是只有单条统一的视线viewing direction, 而是通过一个视点viewpoint  
这种投影方式叫perspective projection, 这种投影方式的决定因素是视点viewpoint和平面plane的角度和其离视线的距离

