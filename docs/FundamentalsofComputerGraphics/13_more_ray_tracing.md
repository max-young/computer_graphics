<!-- TOC -->

- [_13.1 Transparency and Refraction透明度和折射](#_131-transparency-and-refraction透明度和折射)

<!-- /TOC -->

**More Ray Tracing光线追踪的更多内容**

<a id="markdown-_131-transparency-and-refraction透明度和折射" name="_131-transparency-and-refraction透明度和折射"></a>
### _13.1 Transparency and Refraction透明度和折射

在第四章, 讲到了镜面反射, 但是对于导体物质dielectric, 例如钻石、玻璃、空气等, 它们除了反射还会发生折射.  
不同的物质refrective index折光率n不一样, 例如空气接近1, 水是1.33-1.34, 钻石是2.42  
假设从折光率n的介质以入射角$\theta$照射到折光率为$n_t$的介质, 折射后的角度是$\phi$, 那么:
$$n\sin\theta = n_t\sin\phi$$
这个等式经过一些列推导, 可以得到:
$$\cos^2\phi = 1 - \frac{n^2(1-\cos^2\theta)}{n_t^2}$$
为什么要转换成余弦? 因为dot product, 后面就知道了

<img src="./_images/refraction.png" width=60%>

我们接着计算折射后的光线向量t

<img src="./_images/refraction_cal.png" width=30%>

我们现在看2D情况下的计算  
以n和b为坐标轴, 我们可以这样表示t:
$$t = \sin\phi b - \cos\phi n$$
入射光d也可以这样表示:
$$d = \sin\theta b - \cos\theta n $$
从而得到:
$$b = \frac{d+n\cos\theta}{\sin\theta}$$
再和折光率等式结合, 我们就能计算出折射后的光线向量:
$$
\begin{aligned}
  t &= \frac{n(d+n\cos\theta)}{n_t} - n\cos\phi \\
  &=\frac{n(d-n(d\cdot n))}{n_t} - n\sqrt{1- \frac{n^2(1-(d\cdot n)^2)}{n_t^2}}
\end{aligned}
$$
注意这个等式里有一个开根号, 假如里面是负数呢? 这就代表折射不成立, 全部反射了, 这叫total internal reflection, 比如镜子, 就是这个特性.

计算出了折射角度后, 接下来的问题是, 有多少光折射, 有多少光反射呢? 这就涉及到反射率reflectivity.  
根据实际的生活经验, 反射率是跟入射角度相关的, 入射角度越小, 也就是越垂直于照射表面, 折射的光(透过的光)越多, 反射的光越少. 如果入射角度是90度, 也就是平行于表面, 那么反射率就是1, 没有折射.  
有Fresnel equations菲涅尔方程来表示入射角$\Theta$方向的折射率, Schlik approximation简化了这个方程:
$$R(\theta) = R_0 + (1-R_0)(1-\cos\theta)^5$$
当垂直照射时, $\theta=0, \cos\theta=1$, 折射率$R(\Theta)$就等于$R_0$, $R_0$可以通过折光率得到:

$$R_0 = \left(\frac{n_t - 1}{n_t + 1}\right)^2$$

这里表示的是从空气照射到介质, 我们可以把1换成别的介质的折光率

还有别的算法, 参照[菲涅尔方程](https://zh.wikipedia.org/wiki/%E8%8F%B2%E6%B6%85%E8%80%B3%E6%96%B9%E7%A8%8B)的光强方程

<img src='./_images/fresnel_equation.png' width=60%>  

我们可以看到, 垂直照射玻璃时, 大部分光都透(折射)过去了  
以比较小的角度照射时, 看到反射光很亮, 玻璃背面折射过去的光很少, 所以很暗

这里还提到了[beer's law比尔定律](https://zh.wikipedia.org/wiki/%E6%AF%94%E5%B0%94-%E6%9C%97%E4%BC%AF%E5%AE%9A%E5%BE%8B), 光透过介质时会衰减(这里假设光线从空气出发), 衰减的程度和介质的厚度和密度成正比, 不同光谱的衰减是不相关的, 也就是说我们可以单独计算RGB三通道的衰减  
