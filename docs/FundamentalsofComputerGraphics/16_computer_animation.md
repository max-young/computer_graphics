<!-- TOC -->

- [_16.1 Principles of Animation动画原理](#_161-principles-of-animation动画原理)
  - [_16.1.1 Timing](#_1611-timing)
  - [_16.1.2 Action Layout动作布局](#_1612-action-layout动作布局)
  - [_16.1.3 Animation Techniques动画技术](#_1613-animation-techniques动画技术)
  - [_16.1.4 Animator Control vs. Automatic Methods动画师的掌控和自动化方法](#_1614-animator-control-vs-automatic-methods动画师的掌控和自动化方法)
- [_16.2 Keyframing关键帧](#_162-keyframing关键帧)

<!-- /TOC -->

**Compouter Animation计算机动画**

Animation出自于拉丁语anima, 意思是赋予生命、兴趣、精神、运动的过程、行为和结果  
在计算机动画领域里我们主要取赋予运动的这一部分.  

在本章主要介绍计算机动画的四种技术方法:
- KeyKeyframing关键帧
  动画效果其实也是由一张张图片构成, 一秒24张图片, 人眼就能感觉到这是连续的动画. 这24帧里, 我们重点关注关键帧, 其他帧可以通过内插等方式来得到.
- Procedural animation过程动画
  某些特定的运动可以通过数学函数和算法来实现
- Physics-based techniques基于物理的技术
  其可以求解运动微分方程
- Motion Capture动作捕捉
  指的是捕捉真实世界的动作, 然后将其转换为计算机模型的动作. 例如电影阿凡达, 将演员的动作表情捕捉, 应用到动画人物上.

<a id="markdown-_161-principles-of-animation动画原理" name="_161-principles-of-animation动画原理"></a>
### _16.1 Principles of Animation动画原理

1987年, john lasseter在SIGGRAPH杂志上发表了一篇论文, 讲述了从1930年代开始, 迪士尼在动画领域的相关技术和12条原则, 在当时还不成熟的计算机动画领域引起了很大反响.  
这么多年过去了, 这12条原则依然非常重要.

#### _16.1.1 Timing

时间, 或者说是动作速度, 是动画的心脏.  
声音可以作为速度的参照, 事实上很多需要录制声音的动画作品, 是录制声音之后, 再把动画和声音来匹配.  
另外时间能传达很多信息, 比如重物移动的速度更慢, 可以通过时间来体现物体的重量

#### _16.1.2 Action Layout动作布局

人的注意力更多的被相对变化吸引. 例如静止环境中的突然运动和繁忙环境中的静止.  
同一个画面的不同角度, 表示的变化要素不同, 变化要素越多, 效果越好.  
例如书中举例人的动作

动作可以分为三个部分: anticipation(预期), the action itself(动作本身), follow-through(动作完结)  
动作本身可能是很短的, 动作的anticipation和follow-through可能能传达很多信息.  
例如踢球, 真正触球的动作非常短, 但是球员的前期准备和踢出去之后的动作和表情, 非常重要, 是传达氛围的关键.

#### _16.1.3 Animation Techniques动画技术

*squash and strech*, 一个球打到地上弹开的过程, 球的形状是会变化的, 运行过程中会被拉伸, 速度越快, 拉伸越大  
打到地上又会被压扁. 形状的变化就能体现动感. 如果球的形状不变化, 我们会觉得这个球很硬.  

传统画师会绘制一段动画的关键性的图(帧), 其他图再由助手补充.  
对于计算机动画, 也需要了解图形学和这些传统动画的知识, 才能得到吸引人的效果.

#### _16.1.4 Animator Control vs. Automatic Methods动画师的掌控和自动化方法

在传统动画中, 动画师掌控一切.  
计算机动画时代, 动画师依然需要控制核心工作, 比如关键帧, 来保证动画的效果、艺术性、等等  
但是需要利用计算机将一些工作自动化, 来减轻动画师的负担.  
这就需要将两者的工作做好平衡, 以达到用最高的效率完成最好的动画.

### _16.2 Keyframing关键帧

