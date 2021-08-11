<!-- TOC -->

- [The Graphics Pipeline](#the-graphics-pipeline)
  - [_8.1 Rasterization光栅化](#_81-rasterization光栅化)
    - [_8.1.1 Line Drawing](#_811-line-drawing)
    - [_8.1.2 Triangle Rasterization三角形光栅化](#_812-triangle-rasterization三角形光栅化)

<!-- /TOC -->

<a id="markdown-the-graphics-pipeline" name="the-graphics-pipeline"></a>
## The Graphics Pipeline

上一章节讲到了各种转换, 将现实坐标系里的对象经过积层转换到屏幕显示空间坐标系  
这一章就要讲到如何显示到屏幕上, 这就是object-order rendering(渲染)  

例如, 屏幕是有一个个像素组成的, 可以理解为一个个小方格, 现实中的一个三角形怎么现实在屏幕上呢?  
就需要将这个三角形所占的像素找出来, 然后给这些像素赋值, 就把三角形显示出来了.  

将屏幕或者image上的像素用几何图形occupy圈占出来的过程就是rasterization(光栅化)  
rasterization依赖praphics pipeline(图形管道)就是过程, 这个过程包括一系列操作, 起于object对象, 终于像素pixel更新

番外:  
pixel就是picture element的缩写  
rasterization是德语图像化的意思

rendering有很多方法, 本章介绍基本的一种, 原理都是相通的  
rendering的过程包括:  
<img src="./_images/graphics_pipeline.png" width=50%>  
application指的是存储vertex的文件, 经过vertex processing得到primitive(暂且译作多边形), 经过rasterization stage得到fragment, 然后经过fragment processing和blending(融合)就可以显示了

<a id="markdown-_81-rasterization光栅化" name="_81-rasterization光栅化"></a>
### _8.1 Rasterization光栅化

rasterization是rendering的核心, 也是graphics pipeline的核心  
rasterization有两项工作:  
1. 找出primitive覆盖的pixel
2. 给pixel赋值  

rasterization的输出是一系列fragment, fragment就是primitive覆盖的一组pixel

#### _8.1.1 Line Drawing

我们应该怎么在屏幕上画一条线呢?  
这就需要找到这条线所经过的像素

画线依赖这条线的equation方程式, 方程式有两种形式: implicit和parameter  
我们采用implicit方程式

**Drawing Using Implicit Line Equation**

在2.5.2章节里, 我们根据两个端点$(x_0, y_0)$和$(x_1, y_1)$就能得到这条线的implicit equation:
$$f(x, y) \equiv (y_0 - y_1)x + (x_1 - x_0)y + x_0y_1 - x_1y_0 = 0$$

我们假设$x_0 < x_1$, 我们得到这条线的slope:
$$m = \frac{y_1 - y_0}{x_1 - x_0}$$
我们假设$m \in (0, 1]$, 也就是说这条线往右上角的角度小于等于45度  
这样假设是为了接下来解释方便, 其他情况这是这种情况的变种

接下来的要做的工作是要通过这个方程式来找到这条线经过的pixel  
我们知道pixel是一个个小方格, 如果我们将line经过的小方格都找出来, 就会发现这些小方格组成的形状是不规则的, 而且比较粗, 因为存在一列下有两个pixel的情况.  

这里我们采用midpoint algorithm的算法, 找到一条不间断且最thin瘦的线, 也就是说需要连续(对角的两个pixel也是连续的), 且每列只选择一个像素.  
我们从一个像素开始画下一个像素的时候, 有两个选择, 要么画右侧的一个像素, 要么画右上角的一个像素, preseudo代码就是:
```
y = y0
for x = x0 to x1 do
  draw (x, y)
  if (some condition) then
    y = y + 1
```
这个some condition怎么确定呢?  
上面说到画下一个像素时有两个选择, 用坐标表示就是$(x + 1, y)$或者$(x + 1, y + 1)$, 他们的中点就是$(x + 1, y + 0.5)$  
如果$(x, y)$和这个中点$(x + 1, y + 0.5)$的连线, 在要画的线下面, 那么我们就选择右上角的像素$(x + 1, y + 1)$, 如果在线上面, 那么就选择右侧的像素$(x + 1, y)$  
那么怎么判断这条中线是否在要画的线的上面还是下面呢? euqtion就起到作用了.  
我们看equation的一项$(x_1 - x_0)y$, 因为我们假设$x_1 > x_0$, 所以如果x不变, y变大, 那么$f(x, y)$应该大于0 , 所以如果某个点(m, n)满足$f(m, n) > 0$, 那么这个点在这条线的上方  
所以上面的condition就可以变成:
```
if f(x + 1, y + 0.5) < 0 then:
  y = y + 1
```
这个midpoint algorithm就完成了  
我们还可以做优化, 因为$f(x, y)$equation满足:
$$
\begin{aligned}
f(x+1, y) &= f(x, y) + (y_0 - y_1) \\
f(x+1, y+1) &= f(x, y) + (y_0 - y_1) + (x_1 - x_0) \\
\end{aligned}
$$
所以midpoint algorithm算法可以优化成
```
y = y0
d = f(x0 + 1, y0 + 0.5)
for x = x0 to x1 do
  draw (x, y)
  if d < 0 then
    y = y + 1
    d = d + (x1 - x0) + (y0 - y1)
  else
    d = d + (y0 - y1)
```

#### _8.1.2 Triangle Rasterization三角形光栅化