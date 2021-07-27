<!-- TOC -->

- [Linear Algebra线性代数](#linear-algebra线性代数)
  - [_5.1 Determinants行列式](#_51-determinants行列式)
  - [_5.2 Matrics矩阵](#_52-matrics矩阵)
    - [_5.2.1 Matrix Arithmetic矩阵计算](#_521-matrix-arithmetic矩阵计算)
    - [_5.2.2 Operations on Matrics对矩阵的操作](#_522-operations-on-matrics对矩阵的操作)
    - [_5.2.3 Vector Operations in Matrix Form通过matrix对vector进行操作](#_523-vector-operations-in-matrix-form通过matrix对vector进行操作)
    - [_5.2.4 Special Types of Matrics矩阵的特殊类型](#_524-special-types-of-matrics矩阵的特殊类型)
  - [_5.3 Computing with Matrices and Determinants](#_53-computing-with-matrices-and-determinants)
    - [_5.3.1 Computing Inverse](#_531-computing-inverse)

<!-- /TOC -->

<a id="markdown-linear-algebra线性代数" name="linear-algebra线性代数"></a>
## Linear Algebra线性代数

图形程序里最通用的工具恐怕是通过矩阵来改变和变换点和向量.  
这就需要线性代数的相关知识.  

<a id="markdown-determinants" name="determinants"></a>
### _5.1 Determinants行列式

determinants直译过来是决定因素的意思, 但是实际上是:

两个向量组成的平行四边形的区域面积area, 三个向量组成的立方体的区域体积volume  
二维和三维都遵守右手定则, 所以是不满足交换律的, 所以:
$$|\bm{a}\bm{b}| = -|\bm{a}\bm{b}|$$

determinants有下面的特性:
$$|(k\bm{a})\bm{b}| = |\bm{a}(k\bm{b})| = k|\bm{a}\bm{b}|$$
就是把一个平行四边形放大k倍, 放大哪条边都是一样的
$$|(\bm{a}+k\bm{b})\bm{b}| = |\bm{a}(\bm{b}+k\bm{a})| = |\bm{a}\bm{b}|$$
这个不是很好理解, 但是画图后就清晰了, 就是把一个四边形剪去(shearing)一个角, 然后拼到另外一边.  
或者可以这么理解, 一个矩形, 保持底边和高度不变, 将上边的那条边左右移动, 这样这个矩形就变成了平行四边形, 但是面积是不变的  
在shear matrix里有很形象的解释

另外一个特性是, 两个平行四边形想加, 不是直接把他们的面积相加, 而是指首尾两边构成一个新的四边形:
$$|\bm{a}(\bm{b}+\bm{c})| = |\bm{a}\bm{b}| + |\bm{a}\bm{c}|$$  
二维的determinants用笛卡尔表示的话就是:

$$
\begin{aligned}
|ab| &= |(x_ax + y_ay)(x_bx+y_by)| \\
&= x_ax_b|xx| + x_ay_b|xy| + y_ax_b|yx| + y_ay_b|yy| \\
&= x_ax_b(0) + x_ay_b(1) + y_ax_b(-1) + y_ay_b(0) \\
& = x_ay_b - y_ax_b
\end{aligned}
$$

同理, 对于三维:
$$
\begin{aligned}
|abc| &= |(x_ax + y_ay + z_az)(x_bx + y_by + z_bz)(x_cx + y_cy + z_cz)| \\
&= x_ay_bz_c - x_az_by_c - y_ax_bz_c + y_az_bx_c + z_ax_by_c - z_ay_bx_c
\end{aligned}
$$

书中还介绍了克莱姆定理(cramer's rule)

### _5.2 Matrics矩阵

#### _5.2.1 Matrix Arithmetic矩阵计算

矩阵乘以一个数字, 就是矩阵的每个元素都乘以这个数字:
$$
2
\left[
\begin{matrix}
1 & -4 \\
3 & 2
\end{matrix}
\right] =
\left[
\begin{matrix}
2 & -8 \\
6 & 4
\end{matrix}
\right]
$$

两个行列一样的矩阵相加, 那就是对应的元素相加:
$$
\left[
\begin{matrix}
1 & -4 \\
3 & 2
\end{matrix}
\right] + 
\left[
\begin{matrix}
2 & 2 \\
2 & 2
\end{matrix}
\right] =
\left[
\begin{matrix}
3 & -2 \\
5 & 4
\end{matrix}
\right]
$$

上面的两个运算比较简单, 如果是两个矩阵相乘呢?  
首先, 并不是任意两个矩阵都可以相乘, 一个矩阵的行数等于另一个矩阵的列数, 才能相乘.  
一个行列为ab的矩阵, 乘以一个行列为bc的矩阵, 得到行列为ac的矩阵  
新的矩阵的元素$i_{ij}$等于ab矩阵的第i行, 和bc矩阵的第j列, 两两相乘, 然后相加得到的总和  
这里不列公式, 举一个具体的例子就很清晰了:
$$
\left[
\begin{matrix}
0 & 1 \\
2 & 3 \\
4 & 5
\end{matrix}
\right]
\left[
\begin{matrix}
6 & 7 & 8 & 9\\
0 & 1 & 2 & 3
\end{matrix}
\right] = 
\left[
\begin{matrix}
0 & 1 & 2 & 3\\
12 & 17 & 22 & 25 \\
24 & 33 & 42 & 51
\end{matrix}
\right]
$$

矩阵相乘不满足交换律, 也就是说:
$$AB \neq BA$$
但是满足分配律和结合律:
$$
\begin{aligned}
(AB)C &= A(BC) \\
A(B + C) &= AB + AC \\
(A + B)C &= AC + BC
\end{aligned}
$$

#### _5.2.2 Operations on Matrics对矩阵的操作

一个real number x的inverse是$1/x$, x和它的inverse相乘等于1  
那么对于一个矩阵, 我们也想找到类似的规则, 找到matrix的$\mathbf I$  

对于matrix, 我们把这样的$\mathbf I$称之为*identity matrx*, 它是一个square matrix, 对角线都是1, 其他都是0, 例如, four identity matrix是:
$$
\mathbf I = 
\left[
\begin{matrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{matrix}
\right]
$$

我们把矩阵$\mathbf A$的inverse matrix表示为$\mathbf A^{-1}$, $\mathbf A \mathbf A^{-1} = \mathbf I$, 例如:
$$
\left[
\begin{matrix}
1 & 2 \\
3 & 4
\end{matrix}
\right] ^ {-1} = 
\left[
\begin{matrix}
-2.0 & 1.0 \\
1.5 &  -0.5
\end{matrix}
\right]
$$
因为
$$
\left[
\begin{matrix}
1 & 2 \\
3 & 4
\end{matrix}
\right]
\left[
\begin{matrix}
-2.0 & 1.0 \\
1.5 &  -0.5
\end{matrix}
\right] = 
\left[
\begin{matrix}
1 & 0 \\
0 & 1
\end{matrix}
\right]
$$

对于inverse matrx, 有下面的特性:
$$
\mathbf A \mathbf A^{-1} = \mathbf A^{-1} \mathbf A = \mathbf I \\
(\mathbf A \mathbf B)^{-1} = \mathbf A^{-1} \mathbf B^{-1}
$$

说完了inverse, 再说transpose  
matrix $\mathbf A$的transpose表示为$\mathbf A^{\rm T}$, transpose就是把行和列交换  
如果我们把$\mathbf A$的元素(entries)表示为$a_{ij}$, $\mathbf A^{\rm T}$的entries表示为$a_{ji}^{'}$, 那么:
$$a_{ij} = a_{ji}^{'}$$
transpose也满足这样的特性:
$$(\mathbf A \mathbf B)^{\rm T} = \mathbf B^{\rm T}\mathbf A^{\rm T}$$

接下来讲到了matrix的determinant, 不太明白, 第一节说的是向量的determinant, 怎么一下跳到了matrix的determinant, 可能是之前有向量的matrix表示我还没看到, 暂且搁置

#### _5.2.3 Vector Operations in Matrix Form通过matrix对vector进行操作

一个二维向量$a = (x_a, y_a)$, 旋转90度, 得到$a^{'} = (-y_a, x_a)$, 我们可以对向量乘以一个matrix得到这样的效果:
$$
\left[
\begin{matrix}
0 & -1 \\
1 & 0
\end{matrix}
\right]
\left[
\begin{matrix}
x_a \\
y_a
\end{matrix}
\right] = 
\left[
\begin{matrix}
-y_a \\
x_a
\end{matrix}
\right]
$$

如果是横向量, 那就是:
$$
\left[
\begin{matrix}
x_a & y_a
\end{matrix}
\right]
\left[
\begin{matrix}
0 & 1 \\
-1 & 0
\end{matrix}
\right] = 
\left[
\begin{matrix}
-y_a & x_a
\end{matrix}
\right]
$$

两个向量的dot product, 也可以用matrix表示:
$$a\cdot b = a^{\rm T}b$$
对a转置变成横向量然后变成一个横向量乘以一个竖向量:
$$
\left[
\begin{matrix}
x_a & y_a & z_a
\end{matrix}
\right]
\left[
\begin{matrix}
x_b \\
y_b \\
z_b
\end{matrix}
\right] = 
\left[
\begin{matrix}
x_ax_b + y_ay_b + z_az_b
\end{matrix}
\right]
$$

那么$ab^{\rm T}$得到什么呢? 如果是一个三维向量的话, 就会得到一个3X3的matrix  
我们可以把$3\times 3$的matrix理解为3行横向量, 或者3列列向量  
那么这个这个$3\times 3$的matrix乘以一个列向量, 得到一个列向量, 这个列向量的entries是$3\times 3$的matrix的3个横向量和列向量的dot product  
如果是两个$3 times 3$的matrix相乘, 可以理解为3个横向量的组合和3个列向量的组合两两点乘, 得到3个横向量(列向量)

#### _5.2.4 Special Types of Matrics矩阵的特殊类型

*diagonal matrix*: 除了左上角和右下角的对角线上的entries, 其他都是0  
*symmetric*: 对称, 如果一个矩阵的transpose还是它自己, 那么我们称这个matrix symmetric  
*orthogonal matrix*: 我们把一个矩阵的每一行(或者每一列)都看作是一个向量, 如果每个向量的长度都是1, 且于其他向量都垂直, 那么这个matrix就是orthogonal matrix正交矩阵. 表象就是每一行(每一列)有且只有一个entries等于1  

identity matrix满足上面三个特性  

orthogonal matrix还满足这样的特性:
$$\rm R^{\rm T}\rm R = I = \rm R\rm R^{\rm T}$$

$$
\left[
\begin{matrix}
8 & 0 & 0 \\
0 & 2 & 0 \\
0 & 0 & 9
\end{matrix}
\right]
$$
这个矩阵是diagonal, symmetric但是不是orthogonal  

$$
\left[
\begin{matrix}
1 & 1 & 2 \\
1 & 9 & 7 \\
2 & 7 & 1
\end{matrix}
\right]
$$
这个矩阵symmetric, 但是不diagonal, 也不orthogonal

$$
\left[
\begin{matrix}
0 & 1 & 0 \\
0 & 0 & 1 \\
1 & 0 & 0
\end{matrix}
\right]
$$
这个矩阵orthogonal, 但是不diagonal, 也不symmetric

### _5.3 Computing with Matrices and Determinants

我们可以把一个2X2的矩阵看作是两个横向量, 或者两个列向量, 那么我们就可计算一个2X2的矩阵的determinant  
回忆一下, determinant就是向量围起来的面积(体积)

这样:
$$
\left|
\begin{matrix}
\left[
\begin{matrix}
a \\
b
\end{matrix}
\right]
\left[
\begin{matrix}
A \\
B
\end{matrix}
\right]
\end{matrix}
\right| \equiv
\left|
\begin{matrix}
a & A \\
b & B
\end{matrix}
\right| =
aB - Ab
$$
看做两个横向量也是一样的  
(至于为什么这个平行四边形的面积是$aB - Ab$, 应该是中学知识)

我们在看一个$3 \times 3$的矩阵:
$$
\left|
\begin{matrix}
x - x_0 & x - x_1 & x - x_2 \\
y - y_0 & y - y_1 & y - y_2 \\
z- z_0 & z - z_1 & z - z_2
\end{matrix}
\right|
$$
这个矩阵可以看作是4个点, 3个列向量, 分别是$(x, y ,x)$和其他三个点的连线, 如果这个matrix的determinant是0, 那么代表$(x, y, z)$和其他三个点在一个平面上

那么我们怎么计算一个matrix的determinant呢? 需要用到*Laplace's expansion*  
这里不再重复了, 稍微有点复杂. 这个原理为什么成立, 我也不知道. 反正拿来用就好

#### _5.3.1 Computing Inverse

计算你矩阵, 和上面计算determinant相关, 也是直接拿过来用, 这里不重复说了

需要说明的是, 这种计算方法比较耗时, 比较适合小的矩阵. 但是在图形学里, 小矩阵居多, 所以很有用.