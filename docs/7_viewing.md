<!-- TOC -->

- [_7.1 Viewing Transformations](#_71-viewing-transformations)
  - [_7.1.1 The Viewport Transformation](#_711-the-viewport-transformation)
  - [_7.1.2 The Orthographic Projection Transformation正交投影变换](#_712-the-orthographic-projection-transformation正交投影变换)
  - [_7.1.3 The Camera Transformation](#_713-the-camera-transformation)
- [_7.2 Projective Transformations投影变换](#_72-projective-transformations投影变换)
- [_7.3 Perspective projection透视投影](#_73-perspective-projection透视投影)
- [_7.5 Field of View视野](#_75-field-of-view视野)

<!-- /TOC -->

**Viewing**

在上一章节, 讲到了二维和三维对象在各自空间里的变换, 以及矩阵在其中起到的重要作用  
在本章, 会讲到如何将3D对象转换到2D视图, 称之为*viewing transformation*视图变换  

在第四章节里讲到了ray tracing光线追踪, 以及正交orthographic和透视perspective, 本章内容差不多是光线追踪的逆, 建议先看第四章节  

现实世界的对象转换到2D图片里只提供了wireframe renderings的能力, 也就是说将一个物体的外廓全部转换, 不会考虑近处阻挡远处的情况.  
光线追踪需要找到光线照过去最近的物体表面, 最后转换成solid surface. 在本章的后半部分会讲到.

<a id="markdown-_71-viewing-transformations" name="_71-viewing-transformations"></a>
### _7.1 Viewing Transformations

大多数图形系统在3D to 2D的过程中会经过三个转换:
- camera transformation: 摄像机转换, 将现实空间(world space)转换到摄像机空间(camera space)
- projection transformation: 投影转换, 将点从camera space转换到canonical view colume
- viewport transformation: 视窗转换, 将canonical view colume转换到screen space(显示空间)

个人粗浅理解: 第一步是摄取现实对象(和摄像机的位置、角度相关), 第二步是将第一步取的景投射到相机成像元件上, 第三步是洗照片  
我们接着往下看

<a id="markdown-_711-the-viewport-transformation" name="_711-the-viewport-transformation"></a>
#### _7.1.1 The Viewport Transformation

*canonical view volume*是相机坐标系  
用笛卡尔坐标系表示的话是一个2X2X2的空间坐标系, $(x, y, z) \in [-1, 1]^3$  
观测方向是$-z$, $x = -1$是screen的左侧, $y = -1$是screen的底部  
很奇怪, 为什么观测方向是$-z$呢? 好像有点反直觉, 但是这样设定的话, 正好是一个右手坐标系.

我们要将其中的点“打印”到$n_x * n_y$个像素的图片上, 应该怎么转换呢?  
也就是要将相机坐标系的坐标转换到图片坐标系下  
在第3章里, 我们提到$n_x * n_y$个像素坐标系范围应该是$[-0.5, n_x - 0.5] \times [-0.5, y_x - 0.5]$, 因为这样坐标原点就正好落在第一个像素的中心位置(我们假设单个像素的单位是1)  
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
canonical coordinate的左下角坐标$(x_{canonical} = -1, y_{canonical} = -1)$  
计算得到screen下的坐标就是$(x_{screen} = -0.5, y_{screen} = -0.5)$  
canonical coordinate的右上角坐标$(x_{canonical} = 1, y_{canonical} = 1)$  
计算得到screen下的坐标就是$(x_{screen} = n_x - 0.5, y_{screen} = n_y-0.5)$

我们得到齐次坐标转换矩阵:
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

这里我们暂不考虑z方向坐标, 但是并不是说没用, 在第8.2.3章节里z-buffer需要用到z坐标

<a id="markdown-_712-the-orthographic-projection-transformation正交投影变换" name="_712-the-orthographic-projection-transformation正交投影变换"></a>
#### _7.1.2 The Orthographic Projection Transformation正交投影变换

上面提到的canonical view volume是一个单位三维坐标系, 我们怎么把一个任意大小的空间转换到这个单位三维坐标系呢?  

我们任意大小的坐标系和canonical view volume一样, 观测方向是$-z$, $x = -1$是screen的左侧, $y = -1$是screen的底部  
但是大小不一样, $[l, r]\times[b, t]\times[f, n]$, 分别对应left, right, bottom, top, far, near  
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

<a id="markdown-_713-the-camera-transformation" name="_713-the-camera-transformation"></a>
#### _7.1.3 The Camera Transformation

现在讲第一步: 相机取景的那一步

这一步由相机的位置、观测角度、相机自身的角度相关, 这个坐标系也和canonical view volume一样
这一步的关键是如何将现实世界中的坐标转换到这个坐标系下  
其实就是将现实的坐标旋转平移做放射变换, 和相机坐标相匹配   
这就是6.5章节的坐标系变换

假设这一步的三个决定因素这样表示:
- 观测位置e
- 视线方向g
- 相机的向上方向的向量
这三个因素构成了一个$(u, v, w)$的orthonormal坐标系:
$$
\begin{aligned}
w &= - \frac{g}{\parallel g \parallel} \\
u &= \frac{t \times w}{\parallel t \times w \parallel} \\
v &= w \times u
\end{aligned}
$$

现在的问题就变成了, 将现实坐标系$(x, y, z)$的点, 转换到相机坐标系$(u, v, w)$下, 根据6.5章节:
$$
\rm M_{cam} =
\left[
\begin{matrix}
u & v & w & e \\
0 & 0 & 0 & 1
\end{matrix}
\right]^{-1} =
\left[
\begin{matrix}
x_u & y_u & z_u & 0 \\
x_v & y_v & z_v & 0 \\
x_w & y_w & z_w & 0 \\
0 & 0 & 0 & 1
\end{matrix}
\right]
\left[
\begin{matrix}
1 & 0 & 0 & -x_e \\
0 & 1 & 0 & -y_e \\
0 & 0 & 1 & -z_e \\
0 & 0 & 0 & 1
\end{matrix}
\right]
$$

这样, 拍照的完整的转换过程就是对现实坐标系下的一个向量乘以这样的一个矩阵:
$$
\rm M = M_{vp}M_{orth}M_{cam}
$$
这里我们假设投影变换只需要进行orthographic projection  
通俗的理解就是一个正方体直接压缩到一个平面上  
但实际的情况不是这样的, 我们接下来就会说到透视变换

<a id="markdown-_72-projective-transformations投影变换" name="_72-projective-transformations投影变换"></a>
### _7.2 Projective Transformations投影变换

上一章节里说到了其中一种简单的投影变换
投影变换有多种方式, 比如perspective projection透视投影, 相对比较复杂, 所以我们单独拿出来讲.

本一章节主要讲homogeneous coordinate齐次坐标的特殊表示.  
一个3D point$(x, y, z)$用齐次坐标坐标表示的话, 就是加一个维度, 数字是1: ${x, y, z, 1}$  
我们对它进行变换时, 是乘以一个$4 \times 4$的矩阵, 矩阵的最后一行是$(0, 0, 0, 1)$  

我们在这里定义${x, y, z, w}$, $w \neq 1$并且$w \neq 0$, 也表示一个3D的point, 它和$(x/w, y/w, z/w, 1)$是等价的  
这样我们对其进行变换的时候, 最后一行就不必是$(0, 0, 0, 1)$了:
$$
\left[
\begin{matrix}
\widetilde{x} \\
\widetilde{y} \\
\widetilde{z} \\
\widetilde{w}
\end{matrix}
\right] =
\left[
\begin{matrix}
a_1 & b_1 & c_1 & d_1 \\
a_2 & b_2 & c_2 & d_2 \\
a_3 & b_3 & c_3 & d_3 \\
e & f & g & h
\end{matrix}
\right]
\left[
\begin{matrix}
x \\
y \\
z \\
1
\end{matrix}
\right]
$$
$(x, y, z)$经过转换后得到$(x^{\prime}, y^{\prime}, z^{\prime}) = (\widetilde{x}/\widetilde{w}, \widetilde{y}/\widetilde{w}, \widetilde{z}/\widetilde{w})$

举一个实际的例子:
$$
\left[
\begin{matrix}
2 & 0 & -1 \\
0 & 3 & 0 \\
0 & \frac{2}{3} & \frac{1}{3}
\end{matrix}
\right]
\left[
\begin{matrix}
1 \\
0 \\
1
\end{matrix}
\right] =
\left[
\begin{matrix}
1 \\
0 \\
\frac{1}{3}
\end{matrix}
\right]
$$
一个二维的点$(1, 0)$经过转换后得到$(1, 0, 1/3)$, 也就是$(3, 0, 1)$, $(3, 0)$

而且还有一个特性, 如果我们对上面的矩阵乘以一个数字, 再进行变换, 得到的点是一样的.  
另外如果我们对一个正方形执行上面的变换, 会发现得到了一个变形的四边形, 好像有点像近大远小的效果  
对, 接下来我们就要讲到perspective projection透视变换

<a id="markdown-_73-perspective-projection透视投影" name="_73-perspective-projection透视投影"></a>
### _7.3 Perspective projection透视投影

看这张图:  
<img src="./_images/perspective_projection.png">  

上面说到得了正交投影, 是假设看到的视界是一个长方体, 视界的近处是n, 远处是f, 是一束平行的光看过去, 这种情况下, 同样大的物体, 在近处和在远处, 在这个orthographic coordinate下是同样大的.  
但是实际的情况是这样, 从相机的中心位置, 或者从人眼位置去观测, 将实景投射到视窗上, 是按比例缩放的:  
假设观测位置是e, 离视窗(显示的坐标系)的距离是d, 离实景的距离是z, 那么y坐标投射到视窗上就变成了:  
视窗就是正交投影的orthographic coordinate坐标系, 长方体盒子的near平面  
$$y_s = \frac{d}{z}y$$
x也是一样  
$$x_s = \frac{d}{z}x$$
d其实就是near  
那么对于z轴呢? 咋一看z轴好像不变, 也需要做一下变换:  
$$
\rm P =
\left[
\begin{matrix}
n & 0 & 0 & 0 \\
0 & n & 0 & 0 \\
0 & 0 & n+f & -fn \\
0 & 0 & 1 & 0
\end{matrix}
\right]
$$
投影计算后得到:
$$
\rm P
\left[
\begin{matrix}
x \\
y \\
z \\
1
\end{matrix}
\right] = 
\left[
\begin{matrix}
nx \\
ny \\
(n+f)z - fn \\
z
\end{matrix}
\right] =
\left[
\begin{matrix}
\frac{nx}{z} \\
\frac{ny}{z} \\
n + f - \frac{fn}{z} \\
1
\end{matrix}
\right]
$$
这里的n和f指的是near和far  
near是指视窗的z坐标, far指的是视窗的z坐标, 都是负数, 因为0点是视点, 他们都在-z方向

这样, 实际的拍照过程, 就要加上这一个投影过程:  
1. camera transformation将实景坐标系转换到相机坐标系下
2. perspective projection透视投影, 将一个梯形体转换到到orthographic coordinate长方体
3. 将camera coordinate转换成carnonical coordinate(1X1X1)的标准坐标系, 将长方体转换到1X1X1的标准坐标系
4. 将carnonical coordinate成像, 扩展成任意大小
所以:
$$\rm M = M_{vp}M_{orth}PM_{cam}$$  
我们将$\rm M_{orth}P$合并成:
$$
\rm M_{per} = M_{orth}P =
\left[
\begin{matrix}
\frac{2n}{r-l} & 0 & \frac{l+r}{l-r} & 0 \\
0 & \frac{2n}{t-b} & \frac{b+t}{b-t} & 0 \\
0 & 0 & \frac{f+n}{n-f} & \frac{2fn}{f-n} \\
0 & 0 & 1 & 0
\end{matrix}
\right]
$$

### _7.5 Field of View视野

我们可以用(l, r, b, t)以及n来定义视窗, 为了方便起见, 我们规定:  
$$
l = -r \\
b = -t 
$$
注: r和t为正  
这样我们就能用3个参数来定义视窗了, 这些参数可以通过其他参数来获得.  
比如长宽比, 可以通过图像的像素比来获得:  
$$
\frac{n_x}{n_y} = \frac{r}{t}
$$
$n_x, n_y$分别是横向和纵向的像素数量  
我们还可以通过*field of view*来获得n和t的比例, field of view是指从视点能观看到的纵向角度, 也就是从视点看bottom和top的夹角, 这样:
$$
\tan{\frac{\theta}{2} = \frac{t}{|n|}}
$$
注: n是负的, 所以加上绝对值