<!-- TOC -->

- [_23.1 Particle Tracing for Lambertian Scenes](#_231-particle-tracing-for-lambertian-scenes)
- [_23.2 Path Tracing](#_232-path-tracing)
- [_23.3 Accurate Direct Lighting精确直接光照](#_233-accurate-direct-lighting精确直接光照)
  - [_23.3.1 Mathematical Framework](#_2331-mathematical-framework)
  - [_23.3.2 Sampling a Spherical Luminaire](#_2332-sampling-a-spherical-luminaire)
  - [_23.3.3 Nondiffuse Luminaries](#_2333-nondiffuse-luminaries)

<!-- /TOC -->

**global illumination全局光照**

在实际场景中, 有非常多的间接光照, 直接光照照射到物体上再反射到另外一个物体上.  
而且有些物体只受到间接光照, 例如天花板.  
下图的左边的图只有间接光照, 中间的图只有直接光照, 右图是混合之后的效果:

<img src="./_images/FundamentalsOfComputerGraphics/global_illumination.png" width=70%>

试想一下间接光照就很复杂, 无限反射.  
本章介绍两种算法: particle tracing粒子追踪, path tracing路径追踪  
最后介绍如何分离直接光照

<a id="markdown-_231-particle-tracing-for-lambertian-scenes" name="_231-particle-tracing-for-lambertian-scenes"></a>
### _23.1 Particle Tracing for Lambertian Scenes

### _23.2 Path Tracing

 particle tracing在diffuse的场景下比较适合.  
 但是在更通用的场景, 比如有很多对象, BRDF多种多样, 就不太适合了.  
 path tracing可以解决这个问题并得到很好的渲染效果.    
 path tracing是一种概率方法, 一般来说只计算间接光照, 从视点出发, 追踪线路到光源.  
 <img src="./_images/FundamentalsOfComputerGraphics/path_tracing.png" width=70%>

 我们先用一种粗暴的方法来解决, full transport equation是:
 $$L_s(k_o) = L_e(k_o) + \int_{all\ k_i}\rho(k_i, k_o)L_f(k_i)\cos\theta_id\sigma_i$$
 $L_e(k_o)$是照射点自身的发光irridiance

根据monte carlo积分, 我们粗暴的就取一个样本$k_i$, 概率是$p(k_i)$, 那么我们就得到了:
 $$L_s(k_o) \approx L_e(k_o) + \frac{\rho(k_i, k_o)L_f(k_i)\cos\theta_id\sigma_i}{p(k_i)}$$

 pseudocode实现raycor函数就是:
 ```
 RGB raycolor(ray a+tb, int depth)
 if (ray hit some point c) then
  RGB c = Le(-b)
  if (depth < maxdepth) then
    compute random direction d
    return c + R raycolor(c + sd, depth + 1)
 else
  return background color
 ```
 这里说的是lambertian sureface, 每个方向的漫反射参数和概率都是一样的.  
 对去现实场景, 我们需要一个material class, 其需要有一个成员函数来计算随机方向以及其概率

### _23.3 Accurate Direct Lighting精确直接光照

#### _23.3.1 Mathematical Framework

对于直接光照, 我们希望照射点的采样方向都是光源方向, 假设光源是一个面, 那么我们都在这个面上采样. 因为其他区域对光照没有贡献.  
例如我们只采样一次, 这一次不是光源方向, 那么照射点就是黑的, 这不符合实际情况.

在第18.2章节里, 我们知道:
$$L_s(x, k_o) = \int_{all\ x^{\prime}}\frac{\rho(k_i, k_o)L_e(x^{\prime}, -k_i)v(x, x^{\prime})\cos\theta_i\cos\theta^{\prime}}{\left\|x-x^{\prime}\right\|^2}dA^{\prime}$$
假设我们支取一个样本, 那么:
$$L_s(x, k_o) \approx \frac{\rho(k_i, k_o)L_e(x^{\prime}, -k_i)v(x, x^{\prime})\cos\theta_i\cos\theta^{\prime}}{p(x^{\prime})\left\|x-x^{\prime}\right\|^2}dA^{\prime}$$
概率$p = 1/A$, $A$是光源面积, 所以:
$$L_s(x, k_o) \approx \frac{\rho(k_i, k_o)L_e(x^{\prime}, -k_i)v(x, x^{\prime})A\cos\theta_i\cos\theta^{\prime}}{\left\|x-x^{\prime}\right\|^2}dA^{\prime}$$
这样, 对于直接光照, pseudocode就是:
```
color directLight(x, ko, n)
pick random point x' with normal vector n' on light
  d = x' - x
  ki = d/||d||
  if (ray x + td has no hit for t < 1 + ∊) then
    return 𝜌(ki, ko)Le(x', -ki)(n.d)(-n'.d)/||d||4
  else
    return 0
```

#### _23.3.2 Sampling a Spherical Luminaire

#### _23.3.3 Nondiffuse Luminaries

在上面的式子里, 光的irridiance是用$L_e(x')$来表示的, 也就是说强度只和位置相关.  
强度也有可能和光照方向相关, 例如汽车车灯, 所以我们可以把$L_e(x')$改成$L_e(x', -k_i)$  
详情这里不再描述