<!-- TOC -->

- [Transformation Matrices](#transformation-matrices)
  - [_6.1 Linear Transformations线性变换](#_61-linear-transformations线性变换)
    - [_6.1.1 Scaling缩放](#_611-scaling缩放)
    - [_6.1.2 Shearing](#_612-shearing)
    - [_6.1.3 Rotation旋转](#_613-rotation旋转)
    - [_6.1.4 Reflection反射](#_614-reflection反射)
    - [_6.1.5 Composition and Decomposition of Transformations变换的合成和分解](#_615-composition-and-decomposition-of-transformations变换的合成和分解)
  - [_6.2 3D Linear Transformations3D线性变换](#_62-3d-linear-transformations3d线性变换)
  - [_6.3 Translation and Affine Transformations平移和仿射变换](#_63-translation-and-affine-transformations平移和仿射变换)
    - [_6.4 Inverse of Transformation Matrices矩阵变换的逆](#_64-inverse-of-transformation-matrices矩阵变换的逆)

<!-- /TOC -->

<a id="markdown-transformation-matrices" name="transformation-matrices"></a>
## Transformation Matrices

线性代数linear algebra在图形学中非常有用.  
比如矩阵的乘法在几何变换(geomatric transformations)中起到关键性的作用

<a id="markdown-_61-linear-transformations线性变换" name="_61-linear-transformations线性变换"></a>
### _6.1 Linear Transformations线性变换

我们对一个二维向量乘以一个2*2的矩阵, 会得到一个新的二维向量:
$$
\left[
\begin{matrix}
a_{11} & a_{12} \\
a_{21} & a_{22}
\end{matrix}
\right]
\left[
\begin{matrix}
x \\
y
\end{matrix}
\right] = 
\left[
\begin{matrix}
a_{11}x + a_{12}y \\
a_{21}x + a_{22}y
\end{matrix}
\right]
$$

通过这样一个特性, 我们可以实现很多变换

#### _6.1.1 Scaling缩放

我们要对一个图形进行缩放, 图形其实是有很多点组成的, 这些点也可以用二维向量来表示.  
我们对图形进行缩放, 其实就是对其构成的所有点(向量)进行缩放  
例如, 我们等比例缩小一半, 那就是把所有的点的x和y都变成0.5x和0.5y, 怎么实现呢? 我们将其乘以一个2*2的矩阵:
$$
\left[
\begin{matrix}
0.5 & 0 \\
0 & 0.5
\end{matrix}
\right]
\left[
\begin{matrix}
x \\
y
\end{matrix}
\right] = 
\left[
\begin{matrix}
0.5x \\
0.5y
\end{matrix}
\right]
$$
所以我们要缩放的话, 就把二维向量乘以这样的一个矩阵:
$$
\left[
\begin{matrix}
s_x & 0 \\
0 & x_y
\end{matrix}
\right]
$$
$s_x$就是$x$的缩放比例, $x_y$就是y的缩放比例, 如果等比例缩放, 那么$s_x = s_y$, 否则则不等于

#### _6.1.2 Shearing

对照书上的图更容易理解, 讲一个正方形的上边往右挪, 挪45度  
这样y是不变的, x变成了x+y:
$$
\left[
\begin{matrix}
1 & 1 \\
0 & 1
\end{matrix}
\right]
\left[
\begin{matrix}
x \\
y
\end{matrix}
\right] = 
\left[
\begin{matrix}
x + y \\
y
\end{matrix}
\right]
$$
所以, 如果我们从x轴进行shearing的话, 可以将二维向量乘以:
$$
\left[
\begin{matrix}
1 & s \\
0 & 1
\end{matrix}
\right]
$$
如果是从y轴进行shearing, 那就是乘以:
$$
\left[
\begin{matrix}
1 & 0 \\
s & 1
\end{matrix}
\right]
$$
我们也可以把$s$理解为移动角度的$\tan\phi$

#### _6.1.3 Rotation旋转

旋转默认是以坐标原点、逆时针旋转  
旋转也是将二维向量乘以一个2*2的矩阵, 假设旋转$\phi$度(逆时针), 那么这个矩阵是:
$$
rotate(\phi) = 
\left[
\begin{matrix}
\cos\phi & -\sin\phi \\
\sin\phi & \cos\phi
\end{matrix}
\right]
$$
证明过程这里就不说了, 也不复杂, 看书去吧

#### _6.1.4 Reflection反射

反射就是将图像以x轴或者y轴为轴, 旋转180度  
如果以y轴为轴, 那么y不变, x变成-x
如果以x轴为轴, 那么x不变, y变成-y
那么很简单, 对应需要相乘的矩阵是:
$$
reflect(y) = 
\left[
\begin{matrix}
-1 & 0 \\
0 & 1
\end{matrix}
\right], 
reflect(x) = 
\left[
\begin{matrix}
1 & 0 \\
0 & -1
\end{matrix}
\right]
$$

#### _6.1.5 Composition and Decomposition of Transformations变换的合成和分解

如果我们对一个向量先做reflection-x, 再做rotation, 应该是这样做:
$$
rotate(\phi)reflect(x)vector
$$
注意上面的顺序!!
矩阵相乘是不满足交换律的, 也就是说$rotate(\phi)reflect(x)vector$和$reflect(x)rotate(\phi)vector$是不一样的  
也就是说我们对一个向量先反射再旋转, 和先旋转再反射是不一样的.  
但是矩阵相乘是满足结合律的, 也就是说, 我们不用先乘以$reflect(x)$, 再乘以$rotate(\phi)$  
我们可以把这两个矩阵相乘, 再去乘以向量  
而且不管多少个2X2的向量相乘, 最后都会得到一个2X2的向量

这是变换的合成. 变换的分解就是一个逆向的过程. 有很多复杂的变换都可以分解成上面的那些基本的变换.  
书中的内容较多, 这里我只表达简单的概念, 更多的内容留到实际项目里再去详细理解.

### _6.2 3D Linear Transformations3D线性变换

3D的变化就是对2D的扩展, 比如缩放scale, 2D是对xy缩放, 那么3D就是对xyz缩放:
$$
scale(s_x, s_y, s_z) =
\left[
\begin{matrix}
s_x & 0 & 0 \\
0 & s_y & 0 \\
0 & 0 & s_z
\end{matrix}
\right]
$$
其他变化类似

### _6.3 Translation and Affine Transformations平移和仿射变换

在6.1章节的各种transformations里, 不管是缩放还是旋转、反射, 都是将一个点(向量)乘以一个2X2的矩阵, 变换后的坐标格式都是这样的:
$$
x' = ax + by \\
y' = cx + dy
$$
但是我们将一个点translate平移呢, 也就说是:
$$
x' = x + x_t \\
y' = y + y_t
$$
这就无法好像通过乘以一个2X2的矩阵无法实现, 只能是再加上一个2X1的矩阵来得到  
但是这样是很麻烦的, 我们能不能将平移和上面的变换统一起来, 都用矩阵的乘法来实现呢? 答案是可以的!!

我们将二维向量表示成一个三维向量, 只是增加的那个向量是单位向量: $[x, y , 1]^{\rm T}$  
如果我们要将这个向量进行缩放旋转反射等变化, 再进行平移, 那么只需将其乘以这样一个矩阵:
$$
\left[
\begin{matrix}
x' \\
y' \\
1
\end{matrix}
\right] = 
\left[
\begin{matrix}
a & b & x_t \\
c & d & y_t \\
0 & 0 & 1
\end{matrix}
\right]
\left[
\begin{matrix}
x \\
y \\
1
\end{matrix}
\right] = 
\left[
\begin{matrix}
ax + by + x_t \\
cx + dy + y_t \\
1
\end{matrix}
\right]
$$
我们把这种变换叫做affine transformation仿射变换  
这种增加一个维度的实现方式叫做homogeneous coordinates齐次坐标  
非常神奇

我们在上面说将这个向量进行缩放旋转反射等变化, 再进行平移, 将其乘以这样一个矩阵  
我们可以做一个分解, 就会发现:
$$
\left[
\begin{matrix}
a & b & x_t \\
c & d & y_t \\
0 & 0 & 1
\end{matrix}
\right] = 
\left[
\begin{matrix}
1 & 0 & x_t \\
0 & 1 & y_t \\
0 & 0 & 1
\end{matrix}
\right]
\left[
\begin{matrix}
a & b & 0 \\
c & d & 0 \\
0 & 0 & 1
\end{matrix}
\right]
$$

#### _6.4 Inverse of Transformation Matrices矩阵变换的逆

