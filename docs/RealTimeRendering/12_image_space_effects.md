**Image-Space Effects**

### _12.1 Image Processing

#### _12.1.1 Bilateral Filtering

### _12.2 Reprojection Techniques

### _12.3 Lens Flare and Bloom

*lens flare镜头眩光是光通过间接反射或者其他非预期的途径进入到镜头或者眼睛引起的现象.  
lens flare可以分为halo光晕和ciliary corona.  
halo表现为圆环, 外圈是红色, 里面是紫色.  
ciliary corona表现为射线, 从halo里射出.

bloom是由lens或者眼睛的其他部位的散射引起的, 表现为光源周围产生glow辉光, 好像光溢出到了周围一样, 然后scene的其他部分的对比度降低.  
camera拍摄照片是通过其charge-coupled device(CCD)将photon光子转换为charge电荷. 当charge site饱和然后溢出到相邻site时, bloom就产生了.  
halo, corona, bloom统称为glare effects眩光效果.

随着camera技术的发展, 这些glare effect能被减少和消除. 但是, 我们会人为的将这些效果加到图像里. 因为display能显示的亮度有限, 这些效果能给人很亮的错觉.

书中介绍了很多实现lens flare效果的方法, 详情需要深入到文献当中去.

bloom效果是一个特别亮的区域, 溢出到了临近的像素.  
实现这个效果的注意思想是创建一个bloom image, 包含过度曝光的对象, 将其blur模糊, 然后再将其放回到图像中去.  
blur通常采用Gaussian. 制作这种图像通常采样的方法是bright-pass filter: 保留明亮的像素, 昏暗的像素会变黑, 在过渡点会进行混合和缩放. 

bloom image的分辨率可以低一点, 这样节省时间, 也可以增强filtering的效果.
因为bloom image过度曝光, 加到原始图像中时需要对颜色进行混合和缩放.