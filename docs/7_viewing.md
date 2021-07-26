<!-- TOC -->

- [Viewing](#viewing)
  - [_7.1 Viewing Transformations](#_71-viewing-transformations)
    - [_7.1.1 The Viewport Transformation](#_711-the-viewport-transformation)
    - [_7.1.2 The Orthographic Projection Transformation正交投影变换](#_712-the-orthographic-projection-transformation正交投影变换)
    - [_7.1.3 The Camera Transformation](#_713-the-camera-transformation)

<!-- /TOC -->

<a id="markdown-viewing" name="viewing"></a>
## Viewing

在上一章节, 讲到了二维和三维对象在各自空间里的变换, 以及矩阵在其中起到的重要作用  
在本章, 会讲到如何将3D对象转换到2D视图, 称之为*viewing transformation*视图变换  

在第四章节里讲到了ray tracing光线追踪, 以及正交orthographic和透视perspective, 本章内容差不多是光线追踪的逆, 建议先看第四章节  

现实世界的对象转换到2D图片里只提供了wireframe renderings的能力, 也就是说将一个物体的外廓全部转换, 不会考虑近处阻挡远处的情况.  
光线追踪需要找到光线照过去最近的物体表面, 最后转换成solid surface. 在本章的后半部分会讲到.

### _7.1 Viewing Transformations

大多数图形系统在3D to 2D的过程中会经过三个转换:
- camera transformation: 摄像机转换, 将现实空间(world space)转换到摄像机空间(camera space)
- projection transformation: 投影转换, 将点从camera space转换到canonical view colume
- viewport transformation: 视窗转换, 将canonical view colume转换到screen space(显示空间)

个人粗浅理解: 第一步是摄取现实对象(和摄像机的位置、角度相关), 第二步是将第一步取的景投射到相机成像元件上(经过不用的投影转换), 第三步是洗照片  
我们接着往下看

#### _7.1.1 The Viewport Transformation

*canonical view volume*是我们想看到的几何空间  
用笛卡尔坐标系表示的话是一个2X2X2的空间坐标系, $(x, y, z) \in [-1, 1]^3$  
观测方向是$-z$, $x = -1$是screen的左侧, $y = -1$是screen的底部  
很奇怪, 为什么观测方向是$-z$呢? 好像有点反直觉, 但是这样设定的话, 正好是一个右手坐标系.

我们要将其中的点“打印”到$n_x * n_y$个像素的图片上, 应该怎么转换呢?  
在第3章里, 我们提到$n_x * n_y$个像素坐标系应该是$[-0.5, n_x - 0.5] \times [-0.5, y_x - 0.5]$, 因为这样坐标原点就正好落在第一个像素的中心位置(我们假设单个像素的单位是1)  
这个转换只需要进过缩放和平移, 所以我们可以得到:
$$
\left[
\begin{matrix}
x_{screen} \\
y_{screen} \\
1
\end{matrix}
\right] =  
\left[
\begin{matrix}
\frac{n_x}{2} & 0 & \frac{n_x - 1}{2} \\
0 & \frac{n_y}{2} & \frac{n_y - 1}{2} \\
0 & 0 & 1
\end{matrix}
\right]
\left[
\begin{matrix}
x_{canonical} \\
y_{canonical} \\
1
\end{matrix}
\right]
$$
我们可以看到:  
canonical coordinate的左下角坐标$(x_{canonical} = -1, y_{canonical} = -1)$计算得到screen下的坐标就是$(x_{screen} = -0.5, y_{screen} = -0.5)$
canonical coordinate的右下角坐标$(x_{canonical} = 1, y_{canonical} = 1)$计算得到screen下的坐标就是$(x_{screen} = n_x - 0.5, y_{screen} = n_y-0.5)$

这种投影和$z$轴没有关系, 我们得到:
$$
M_{vp} =
\left[
\begin{matrix}
\frac{n_x}{2} & 0 & 0 & \frac{n_x - 1}{2} \\
0 & \frac{n_y}{2} & 0 & \frac{n_y - 1}{2} \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{matrix}
\right]
$$

这一步好像很简单

#### _7.1.2 The Orthographic Projection Transformation正交投影变换

上面提到的canonical view volume是一个单位三维坐标系, 我们怎么把一个任意大小的空间转换到这个单位三维坐标系呢?  

我们把这个任意大小的坐标系和canonical view volume一样, 观测方向是$-z$, $x = -1$是screen的左侧, $y = -1$是screen的底部  
但是大小不一样, $[l, r]\times[b, t]times[f, n]$, 分别对应left, right, bottom, top, far, near  
我们称之为*orthographic view volume*  
(这里我们为了讲解整个3D to 2D的过程, 这一步投影变换选取最简单的正交投影做讲解)

orthographic view volume转换为canonical view volume, 同样只需要经过缩放和平移, 平行的两条线经过变换之后还是平行的:
$$
\rm M_{orth} =
\left[
\begin{matrix}
\frac{2}{r-l} & 0 & 0 & -\frac{r+l}{r-l} \\
0 & \frac{2}{t-b} & 0 & -\frac{t+b}{t-b} \\
0 & 0 & \frac{2}{n-f} & -\frac{n+f}{n-f} \\
0 & 0 & 0 & 1
\end{matrix}
\right]
$$
我们可以计算得到, 在orthographic view volume的最左侧最底部最远处的点$(l, b, f)$转换之后就是$(-1, -1, -1)$

这样, 摄像机在某个位置某个角度取景之后获得的点, 要转换到输出2D图像上, 就需要进过这两轮转换:
$$
\left[
\begin{matrix}
x_{pixel} \\
y_{pixel} \\
z_{canonical}  \\
1
\end{matrix}
\right] =
(\rm M_{vp}\rm M_{orth})
\left[
\begin{matrix}
x \\
y \\
z \\
1
\end{matrix}
\right]
$$

#### _7.1.3 The Camera Transformation

现在讲第一步: 相机取景的那一步

这一步由相机的位置、观测角度、相机自身的角度相关, 这个坐标系也和canonical view volume一样
这一步的关键是如何将现实世界中的坐标转换到这个坐标系下  
其实就是将现实的坐标平移然后旋转, 和相机坐标相匹配  

先看6.5章节