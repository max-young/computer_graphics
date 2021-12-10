<!-- TOC -->

- [_18.1 Radiometry辐射度量学](#_181-radiometry辐射度量学)
  - [_18.1.1 Photons光子](#_1811-photons光子)
  - [_18.1.2 Spectral Energy光谱能量](#_1812-spectral-energy光谱能量)
  - [_18.1.3 Power功率](#_1813-power功率)
  - [_18.1.4 Irradiance辐射度](#_1814-irradiance辐射度)
  - [_18.1.5 radiance辐射率](#_1815-radiance辐射率)
  - [_18.1.6 BRDF](#_1816-brdf)
- [_18.2 Transport Equation](#_182-transport-equation)

<!-- /TOC -->

**Light**

如何对light进行量化, 用radiometry辐射度量学  
光的量化和感知光的量化不一样, 例如我们会觉得同样强度的绿光比蓝光要亮, 这就涉及到photometry(光度学)

<a id="markdown-_181-radiometry辐射度量学" name="_181-radiometry辐射度量学"></a>
### _18.1 Radiometry辐射度量学

我们用国际单位系统SI(international system of unit)来量化光  
SI用meter(m)米来量化长度, gram(g)克来量化重量  
用joule(J)焦耳来量化energy能量  
光本质上是传播能量的一种形式

<a id="markdown-_1811-photons光子" name="_1811-photons光子"></a>
#### _18.1.1 Photons光子

为了描述radiometry, 我们需要借助photon光子.  
photon是light的一个单位, 它有位置、传播方向和波长wavelength  
波长的单位是nanometer(nm)纳米, 一纳米是$10^{-9}$米, 用$\lambda$表示  
photon的传播速度用c表示, 也就是光速, 这个速度和介质的折射率相关.  
我们得到光子的频率是:
$$f=c/\lambda$$
怎么理解呢?  
频率是指单位时间经过了多少个波长, 是时间t内, 传播的距离是l, 那么波长的数量就是$l/\lambda$, 再除以时间, 那么就是速度除以波长  
我们通过频率计算一个photon所携带的能量:
$$q = hf = \frac{hc}{\lambda}$$
h是Plank's Constant普朗克常数, $h = 6.63\times10^{-34}Js$, 单位是焦耳秒, 这样算出来q的单位就是J焦耳

#### _18.1.2 Spectral Energy光谱能量

我们知道了一个photo携带的能量, 乘以photon的数量就是光的能量.  
有个问题是, 这些能量在不同的波长下是怎么分布的呢? 我们引入spectral energy的概念.  
例如波长在500-600nm的范围下, 能量是10.2J, 那么:
$$Q_\lambda[500, 600] = \frac{10.2}{100} = 0.12J(nm)^{-1}$$
我们把$Q_\lambda$简写为$Q$  

怎么理解呢? 光谱能量是一个intensive强度单位, 能量、长度、面积是extensive广度单位.  
例如一个省的人口是extensive单位, 人口除以省面积得到人口密度, 就是intensive单位.  

实际的例子, 一道光的能量是q, 我们用一个滤光片放在这道光的传播路径下, 只让一定波长的光穿过, 这样穿过滤光片的能量是$\Delta q$, 波长范围是$\Delta \lambda$, spectral energy就是$Q = \Delta q/\Delta \lambda$

#### _18.1.3 Power功率

光源产生能量的效率(功率)的量化也很有用. 我们用power来表示, 单位是watts(W)瓦特, 或者称为焦耳每秒.  

例如100瓦的灯泡, 就是一秒钟能产生100焦耳的能量. 
假设这个灯泡产生的光子平均波长是$\lambda = 500nm$, 我们能算出这个灯泡一秒钟产生多少photon, 一个光子的能量是
$$f = \frac{c}{\lambda} = \frac{3\times10^8ms^{-1}}{500\times10^{-9}m} = 6\times10^{14}s^{-1}$$
这样一个photon携带的能量就是$hf \approx 4 \times 10^{-19}J$, 100瓦除以这个数, 就是一秒钟产生的光子数量, 大概是$10^{20}$的数量级.

spectral energy光谱能量, 可以理解为单位波长的能量, 因为power概念的引入, 我们自然想到将光谱能量再除以时间  
这样就得到了spectral power光谱功率, 用符号$\Phi$表示, 单位是$W(nm)^{-1}$  
例如100W的灯泡, 产生的光子波长范围是400nm至800nm, 我们就计算出spectral power是$100W/400nm=0.25W(nm)^{-1}$  
光谱测量装置里, 有快门装置, 快门的时间是$\Delta t$, 在测量通过的能量和波长范围, 就计算出$\Phi = \Delta q / (\Delta t \Delta \lambda)$

#### _18.1.4 Irradiance辐射度

spectral energy是光谱上的能量分布, spectral power又加上了时间的维度.  
一个光源会往四面八方发射能量, 我们想问, 在光的辐射范围内某个点或者说某个面积内接受到的能量是多少呢?  
例如上面提到的光谱测量装置, 是有一个快门面积的.  
我们再加上面积的维度, 就得到了spectral irridiance光谱辐射度, 用符号$H$表示:
$$H = \Delta q / (\Delta A \Delta t \Delta \lambda)$$
单位是$Jm^{-2}s^{-1}(nm)^{-1}$  
这个量度告诉了我们一个点获得的light

另外: 如果光照射到一个片面后发生反射, 这个反射也有irradiance辐射度, 称之为radiant exitance, 用大$E$表示  
来区分incident和exidant light入射光和出射光

#### _18.1.5 radiance辐射率

irridiance告诉了我们一个点获得的light  
另外一个问题一个点可以获取四面八方的光, 那么从某个角度照射过来的光是多少呢?  
这句话很关键.

<img src="./_images/radiance1.png" width=10%>

有一个观测装置,光从右侧照入, 在面积为$\Delta A$的照射面右侧放一个圆锥体, 这个圆锥的solid angle立体角的范围是$\Delta \sigma$, 在圆锥体的左侧观测这个立体角范围内的光照强度, 这样, 我们在irridiance的基础上再除以这个角度:
$$response = \frac{\Delta H}{\Delta \sigma} = \frac{\Delta q}{\Delta A \Delta \sigma  \Delta t \Delta \lambda}$$  
(之后我们会隐去光谱$\Delta \lambda$)  
我们会想, 这个圆锥体的长度, 或者说光线长度会影响这个值吗? 答案是不会:

<img src="./_images/radiance2.png" width=30%>

上面两种情况, $\Delta A$是一样的, $\Delta q$是一样的吗?  
答案是一样, 因为面积和距离平方成正比, 但是强度和距离平方成反比(因为衰减, 因为一个光源的power是恒定的, 以光源为中心画一个半径为1的球, 和半径为r的球, 两个球观测到的光子数量是一样的, 但是球的面积不一样, 所以强度和$r^2$成反比)  
这样ridiance和距离是没有关系的. 这是一个很好的特性.

但是这个参数和角度有关系  

<img src="./_images/radiance3.png" width=10%>

如上图, 假设圆锥和观测表面不垂直, 其实就是入射角度和表面法线不平行, 那么实际的照射面积是$\Delta A \cos \theta$,  
回忆一下第4章的shading. 从而:
$$response = \frac{\Delta H}{\Delta \sigma \cos \theta} = \frac{\Delta q}{\Delta A \cos \theta \Delta \sigma  \Delta t \Delta \lambda}$$  

在上一章节我们说到入射和出射, 其实都是光照. 都有其radiance, 我们做一下区分:  
入射的radiance称为field radiance  
出射的radiance称为surface radiance
$$
\begin{aligned}
L_s = \frac{\Delta E}{\Delta \sigma \cos \theta} \\
L_f = \frac{\Delta H}{\Delta \sigma \cos \theta}
\end{aligned}
$$

**Radiance and Other Tadiometric Quantities**

因为surface radiance是和入射角度相关的, 假设我们算出所有角度的surface radiance, 那么我们就能得到irridiance:
$$
H = \int_{all\ k}L_f(K) \cos \theta d\sigma
$$
K是光照方向. 可以理解为光照的单位向量, 或者spherical coordinates球坐标的$(\theta, \phi)$:  

<img src="./_images/sphere_coordinate.png" width=20%>

differential solid angle微分立体角用球面坐标的等式是:
$$d_{\sigma} \equiv \sin \theta d \theta d \phi$$
这样
$$H = \int_{\phi=0}^{2\pi}\int_{\theta=0}^{\frac{\pi}{2}}L_{f}\cos\theta\sin\theta d\theta d \phi = \pi L_f$$
为什么是$\pi L_f$, 涉及到微积分的计算  
注意这个半球坐标, $\phi$的范围是$[0, 2\pi]$, $\theta$的范围是$[0, \frac{\pi}{2}]$

进而, 我们可以得到power功率:
$$\Phi = \int_{all\ x}H(x)dA$$
x是表面的点, dA是differential area微分面积

#### _18.1.6 BRDF

我们想描述物体表面的反射.  
需要用反射的某个值除以入射的某个值得到一个比例, 来定义反射.

<img src='./_images/BRDF.png' width=70%>

我们在入射点安装一个irridiance meter, 在$k_0$方位安装一个radiance detector, 我们用反射方向$k_0$的ridiance除以入射方向的irridiance, 这样我们得到:  
$$\rho = \frac{L_s}{H}$$
这个函数称之为BRDF(bidirectional reflectance distribution function双向反射分配函数)  
我们可以理解为从$k_i$方向过来的光线, 在物体表面一个微小面积获得此方向的能量(irridiance), 然后反射到$k_0$方向的一个立体角的能量(ridiance)  
显然这个值是和radiance detector的角度相关的  
借助时机的物理场景来说, 如果是镜面反射, BRDF只会在镜面反射方向有值  
如果是漫反射, 反射是均匀分布的, 不论在哪个观测角度, BRDF都是一样的

**Directional Hemispherical Reflectance定向半球反射率**

我们的问题是, 入射光有多少比例被反射出去了呢?  
我们定义Directional Hemispherical Reflectance函数, 来说明$k_i$方向的光线有多少被反射了:
$$R(k_i)=\frac{所有反射的能量power,方向记作k_0}{k_i方向照射过来的能量power}=\frac{E}{H}$$
反射方向$k_0$的irridiance是入射方向$k_i$的ridiance乘以BRDF
$$L(k_0) = H\rho(k_i, k_0)$$
根据radiance的定义:
$$L(k_0) = \frac{\Delta E}{\Delta \sigma_0\cos\theta_0}$$
从而:
$$
\begin{aligned}
H\rho(k_i, k_0) = \frac{\Delta E}{\Delta \sigma_0\cos\theta_0} \\
\frac{\Delta E}{H} = \rho(k_i, k_0)\Delta\sigma_0\cos\theta_0
\end{aligned}
$$
我们把所有反射方向加起来, 就得到了某个入射方向$k_i$的反射率:
$$R(k_i) = \int_{all\ k_0}\rho(k_i, k_0)\cos\theta_0d\sigma_0$$

**Ideal Diffuse BRDF完美漫反射BRDF**

如果是完美的漫反射, 这样的表面称为lambertian, 所有反射角度的BRDF值$\rho$都是一样的, 我们记作$C$, 这样:
$$
\begin{aligned}
R(k_i) &= \int_{all\ k_0}\rho(k_i, k_0)\cos\theta_0d\sigma_0 \\
&= \int_{\phi_0=0}^{2\pi}\int_{\theta_0=0}^{\frac{\pi}{2}}C\cos\theta_0\sin\theta_0 d\theta_0 d \phi_0 \\
&= \pi C
\end{aligned}
$$
如果是一个完美的lambertian, 没有能量损失, 那么$R=1$, 从而$\rho = 1/\pi$

### _18.2 Transport Equation

根据上面BRDF的定义, BRDF等于出射方向$k_0$的radiance除以入射方向的irridiance, irridiance跟surface面积相关, 和入射角没关系.  
如果我们假设入射只从某一个入射立体角$k_i$过来的光线呢? 我们就能根据入射立体角的ridiance计算得出irridiance, 从而:
$$\rho = \frac{L_0}{L_i\cos\theta_i\Delta\sigma_i}$$
从而我们能得到某个入射方向$k_i$在出射方向$k_0$的radiance:
$$\Delta L_0 = \rho(k_i, k_0)L_i \cos \theta_i \Delta \sigma_i$$
我们把所有入射方向做积分:
$$L_s(k_0) = \int_{all\ k_i}\rho(k_i, k_0)L_f(k_i) \cos \theta_i d\sigma_i$$

在games101 rendering低讲里讲到, 这里做积分, 是选取入射方向的半球范围内做采样, 然后做积分, 这种采样和积分效率比较低, 效果也不太好.  
原因是, 假如光源只占很小的一片区域, 那么, 在半球范围内的采样很大部分是没用的, 因为没有光源照射.  
所以我们能不能把采样范围只限定在光源范围呢? 如下图:

<img src="./_images/rendering_equation.png" width=30%>

右上角是面光源, 入射角度可以这样计算:
$$\Delta \sigma_i = \frac{\Delta A^{\prime} \cos \theta^{\prime}}{\left\|x-x^{\prime}\right\|^2}$$
相当于光源面积做一个旋转, 然后再除以半径的平方  
从而我们就可以从对入射角的积分转换为对光源上的点做积分
$$L_s(x, k_0) = \int_{all\ x^{\prime}\ visible\ to\ x}\frac{\rho(k_i, k_0)L_s(x^{\prime}, x-x^{\prime})\cos\theta_i\cos\theta^{\prime}}{\left\|x-x^{\prime}\right\|^2}dA^{\prime}$$
$x-x^{\prime}$是指$x^{\prime}$到$x$的向量, 也就是光源到入射点的向量

做一下整理:
$$
\begin{aligned}
L_s(x, k_0) &= \int_{all\ x^{\prime}}\frac{\rho(k_i, k_0)L_s(x^{\prime}, x-x^{\prime})v(x, x^{\prime})\cos\theta_i\cos\theta^{\prime}}{\left\|x-x^{\prime}\right\|^2}dA^{\prime} \\
v(x, x^{\prime}) &= \left\{
\begin{aligned}
&1\ if\ x\ and\ x^{\prime}\ are\ mutually\ visible \\
&0\ otherwise
\end{aligned}
\right.
\end{aligned}
$$
