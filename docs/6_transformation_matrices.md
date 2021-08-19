<!-- TOC -->

- [_6.1 Linear Transformations线性变换](#_61-linear-transformations线性变换)
  - [_6.1.1 Scaling缩放](#_611-scaling缩放)
  - [_6.1.2 Shearing](#_612-shearing)
  - [_6.1.3 Rotation旋转](#_613-rotation旋转)
  - [_6.1.4 Reflection反射](#_614-reflection反射)
  - [_6.1.5 Composition and Decomposition of Transformations变换的合成和分解](#_615-composition-and-decomposition-of-transformations变换的合成和分解)
- [_6.2 3D Linear Transformations3D线性变换](#_62-3d-linear-transformations3d线性变换)
- [_6.3 Translation and Affine Transformations平移和仿射变换](#_63-translation-and-affine-transformations平移和仿射变换)
- [_6.4 Inverse of Transformation Matrices矩阵变换的逆](#_64-inverse-of-transformation-matrices矩阵变换的逆)
- [_6.5 Coordinate Transformations坐标系变换](#_65-coordinate-transformations坐标系变换)

<!-- /TOC -->

**Transformation Matrices**

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

<a id="markdown-_611-scaling缩放" name="_611-scaling缩放"></a>
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
所以我们要缩放的话, 就把二维向量乘以一个diagonal matrix:
$$
\left[
\begin{matrix}
s_x & 0 \\
0 & x_y
\end{matrix}
\right]
$$
$s_x$就是$x$的缩放比例, $x_y$就是y的缩放比例, 如果等比例缩放, 那么$s_x = s_y$, 否则则不等于

<a id="markdown-_612-shearing" name="_612-shearing"></a>
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

<a id="markdown-_613-rotation旋转" name="_613-rotation旋转"></a>
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

值得注意的是, 这个矩阵是一个orthogonal matrix  

<a id="markdown-_614-reflection反射" name="_614-reflection反射"></a>
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

<a id="markdown-_615-composition-and-decomposition-of-transformations变换的合成和分解" name="_615-composition-and-decomposition-of-transformations变换的合成和分解"></a>
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

<a id="markdown-_62-3d-linear-transformations3d线性变换" name="_62-3d-linear-transformations3d线性变换"></a>
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

<a id="markdown-_63-translation-and-affine-transformations平移和仿射变换" name="_63-translation-and-affine-transformations平移和仿射变换"></a>
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

<a id="markdown-_64-inverse-of-transformation-matrices矩阵变换的逆" name="_64-inverse-of-transformation-matrices矩阵变换的逆"></a>
### _6.4 Inverse of Transformation Matrices矩阵变换的逆

矩阵变换的逆, 在几何上有什么意义呢? 

首先, 如果我们对一个向量做scale缩放操作, 我们要乘以一个diagonal matrix, 假设斜线是$(s_x, s_y, s_z)$  
如果我们要反向缩放回原来的大小, 我们也需要乘以一个diagonal matrix, 斜线是${1/s_x, 1/s_y, 1/s_z}$

如果是旋转的话, 要对一个向量乘以一个orthogonal matrx, 如果要旋转回来, 就要乘以这个matrix的inverse  
根据orthogonal matrix的特性, 它的inverse也是它的transpose  
inverse计算很复杂, transpose就很简单了, 所以要旋转回来, 我们乘以其transpose就可以了  

实例: 假如我们对一个向量旋转$\rm R_1$, 再缩放${s_x, s_y, s_z}$, 再旋转$\rm R_2$, ($\rm R$代表orthogonal matrix), 那么我们要对向量乘以的矩阵是:
$$\rm M = \rm R_2(s_x, s_y, s_z)\rm R_1$$  
做逆向操作的话, 矩阵就是
$$\rm M^{-1} = \rm R_1^T(1/s_x, 1/s_y, 1/s_z)\rm R_2^T$$  

<a id="markdown-_65-coordinate-transformations坐标系变换" name="_65-coordinate-transformations坐标系变换"></a>
### _6.5 Coordinate Transformations坐标系变换

举一例子:
汽车在路上行走在马路上, 路边的大楼再往后退  
那么我们是要将大楼往后移动呢? 还是要将以汽车为原点的坐标系在城市坐标系里移动呢?  
这就涉及到坐标系变换的问题.

术语:  
coordinate system, 或者coordinate frame, 包括原点origin, 和basis(几个向量)  
比如orthonormal bases就是一个包含一个原点, 和三个互相垂直的向量的coordinate system  
假如一个三维坐标系origin是$p$, basis是${u, v, w}$, 那么这个坐标系下的点可以表示为:
$$p + u\bm u + v\bm v + w\bm w$$
二维坐标系, origin是$o$, basis是${x, y}$, 其中的点可以表示为:
$$\bm p = (x_p, y_p) \equiv \bm o + x_p\bm x + y_p \bm y$$

假设$o, x, y$是carnonical coordinate system主坐标系, 怎么把子坐标系$e, u ,v$上的点p转换到主坐标系下呢? 
$$\bm p = (u_p, v_p) \equiv \bm e + u_p\bm u + v_p \bm v$$

我们可以用仿射变换, 将p先旋转再平移:
$$
\left[
\begin{matrix}
x_p \\
y_p \\
1
\end{matrix}
\right] = 
\left[
\begin{matrix}
1 & 0 & x_e \\
0 & 1 & y_e \\
0 & 0 & 1
\end{matrix}
\right]
\left[
\begin{matrix}
x_u & x_v & 0 \\
y_u & y_v & 0 \\
0 & 0 & 1
\end{matrix}
\right]
\left[
\begin{matrix}
u_p \\
v_p \\
1
\end{matrix}
\right] = 
\left[
\begin{matrix}
x_u & x_v & x_e \\
y_u & y_v & y_e \\
0 & 0 & 1
\end{matrix}
\right]
\left[
\begin{matrix}
u_p \\
v_p \\
1
\end{matrix}
\right]
$$
其中$x_u, y_u$和$x_v, y_v$指的是子坐标系的basis vector在carnicol system下的向量表示
可以简化表示为:
$$
\rm P_{xy} =
\left[
\begin{matrix}
\bm u & \bm v & \bm e \\
0 & 0 & 1
\end{matrix}
\right]
\rm P_{uv}
$$
如果要反向转换, 那就是先平移再旋转:
$$
\rm P_{uv} =
\left[
\begin{matrix}
\bm u & \bm v & \bm e \\
0 & 0 & 1
\end{matrix}
\right]^{-1}
\rm P_{xy}
$$
三维坐标系是同样的道理