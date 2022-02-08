<!-- TOC -->

- [_8.1 Light Quantities光的度量](#_81-light-quantities光的度量)
  - [_8.1.1 Radiometry](#_811-radiometry)

<!-- /TOC -->

**Light and Color**

<a id="markdown-_81-light-quantities光的度量" name="_81-light-quantities光的度量"></a>
### _8.1 Light Quantities光的度量

从物理角度来度量light我们用radiometry  
从人的感知角度来度量light我们用photometry  
色彩的感知我们用colorimetry

<a id="markdown-_811-radiometry" name="_811-radiometry"></a>
#### _8.1.1 Radiometry

radiometry通过电磁辐射的方式来度量light  
电磁辐射有wavelength, 可见光只是一小部分波长范围的电磁辐射  
radiometry通过下面几种单位来度量light的强度、功率等等:
- radiant flux: 功率  
  radimetry的基本单位, 电磁辐射能量除以时间, 用$\Phi$表示, 单位是watt($W$)
- irridiance:  
  功率再除以面积, 一般用在物体表面, 用$\textit{E}$表示, 单位是$W/m^2$
- radiant intensity:  
  功率除以solid angle, 单位是steradians(sr), 用$\textit{I}$表示, 单位是$W/sr$  
  注: 角度angle是指二维平面上的角度, 单位用radians弧度表示, 而solid angle是指三维空间中的角度. radians是指单位角度在半径是1的圆的弧长, 一个单位圆的radians是2π.  
  steridians就是指单位solid angle在半径是1的球体上的面积, 一个球体的steridians是4π.
- radiance:  
  radiance这个概念在rendering非常重要, 因为camera和eye感知的就是radiance, 它描述了单条射线的强度  
  radiance是功率除以面积再除以solid angle, 用$\textit{L}$表示, 单位是$W/(m^2sr)$  
  因为要除以面积, 就跟入射角度有关系, 面积需要乘以余弦  
  radiance的特性是跟距离、环境无关, 例如我们观察一个物体表面反射的光, 这个反射光的radiance和我们观察的距离和环境无关.  

很多light并不是单一波长构成的, 是有多种波长组成, 我们称之为spectral power distribution(SPD), 但是我们并不用波长来表示light, 因为人眼感受到的是颜色, 并不能很精确的感受到波长, 甚至有些不同的波长组合, 人感受到的颜色是一样的, 所以我们用RGB来表示颜色.