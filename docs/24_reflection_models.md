<!-- TOC -->

- [_24.1 Real-World Materials](#_241-real-world-materials)
  - [_24.1.1 Smooth Dielectrics and Metals光滑的电介质和金属](#_2411-smooth-dielectrics-and-metals光滑的电介质和金属)
  - [_24.1.2 Rough Surfaces粗糙的表面](#_2412-rough-surfaces粗糙的表面)
  - [_24.1.3 Diffuse Materials漫反射材料](#_2413-diffuse-materials漫反射材料)
  - [_24.1.4 Translucent Materials半透明材料](#_2414-translucent-materials半透明材料)
  - [_24.1.5 layered Materials分层材料](#_2415-layered-materials分层材料)
- [_24.2 Implementing Reflection Models实现反射模型](#_242-implementing-reflection-models实现反射模型)

<!-- /TOC -->

**Reflection Models反射模型**

在第18章节里我们讲到用BRDF来表示对象的反射特性. 本章讲材质属性对视觉的影响和BRDF在一些模型上的表现.

<a id="markdown-_241-real-world-materials" name="_241-real-world-materials"></a>
### _24.1 Real-World Materials

一些物体在微观状态下, 或者足够近的情况下, 能观察到很多细节, 比如地毯的绒毛, 纸的纤维  
但是我们在足够远的距离观察, 这些细节不再重要, 它们形成了统一的BRDF特性.

<a id="markdown-_2411-smooth-dielectrics-and-metals光滑的电介质和金属" name="_2411-smooth-dielectrics-and-metals光滑的电介质和金属"></a>
#### _24.1.1 Smooth Dielectrics and Metals光滑的电介质和金属

我们需要关注两点:
- 有多少光被反射
- 有多少光被折射
在第13.1章节里我们讲到了如何计算发射率等知识, 我们可以得到下图:

<img src="./_images/smooth_dielectics_and_metals.png" width=30%>

#### _24.1.2 Rough Surfaces粗糙的表面

microfacets微表面  

我们对金属进行打磨, 使之一定程度的粗糙, 在很近的距离, 依然可以镜面反射, 但是离远一点, 光好像被四处反射, 一个例子是拉丝钢

#### _24.1.3 Diffuse Materials漫反射材料

例如石头、纸、木头等, 这种情况, 可以近似的用constant BRDF来处理.

#### _24.1.4 Translucent Materials半透明材料

一些比较薄的物体, 例如树叶、纸张等

#### _24.1.5 layered Materials分层材料

<img src="./_images/layered_materials.png" width=20%>

### _24.2 Implementing Reflection Models实现反射模型

在第18章节里, 我们讲到某个入射方向$k_i$的反射率:
$$R(k_i) = \int_{all\ k_o}\rho(k_i, k_o)\cos\theta_0d\sigma_0$$

另外, BRDF是对称的, 也就是说入射和反射方向交换, BRDF是相等的:
$$\rho(k_i, k_o) = \rho(k_o, k_i)$$

根据monte carlo采样定律:
$$\int f(x)d_\mu \approx \frac{1}{N}\sum_{j=1}^N \frac{f(x_j)}{p(x_j)}$$

根据18.2的rendering equation:
$$L_s(k_o) = \int_{all\ k_i}\rho(k_i, k_o)L_f(k_i)\cos\theta_id\sigma_i$$
出射方向$k_o$的irridiance是所有入射方向$k_i$的BRDF乘以其irridiance再乘以入射角的余弦的和,
那么, 根据monte carlo, 我们可以这样计算, 对入射光线(入射角$\theta$)进行采样:
$$L_s(k_o) \approx \frac{1}{N}\sum_{j=1}{N}\frac{\rho(k_j, k_o)L_f(k_j)\cos\theta_j}{p(k_j)}$$

这个等式只有$p(k_j)$光线采样的概率是未知的, 书中的讲法没有理解.  
GAMES101 ray tracing第四讲里讲的比较简单  
对于一个入射点, 光线的照射角度范围是一个半球, 因为下半球照射不到, 我们在这个半球范围内均匀采样, 那么光线采样的概率就是半球面积分之一, 半球面积是$2\pi$, 所以:
$$p(k_j) = \frac{1}{2\pi}$$ 
