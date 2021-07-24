## Viewing

在上一章节, 讲到了二维和三维对象在各自空间里的变换, 以及矩阵在其中起到的重要作用  
在本章, 会讲到如何将3D对象转换到2D视图, 称之为*viewing transformation*视图变换  

在第四章节里讲到了ray tracing光线追踪, 以及正交orthographic和透视perspective, 本章内容差不多是光线追踪的逆, 建议先看第四章节  

现实世界的对象转换到2D图片里只提供了wireframe renderings的能力, 也就是说将一个物体的外廓全部转换, 不会考虑近处阻挡远处的情况.  
光线追踪需要找到光线照过去最近的物体表面, 最后转换成solid surface. 在本章的后半部分会讲到.