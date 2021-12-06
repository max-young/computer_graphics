<!-- TOC -->

- [_12.1 Triangle Meshes](#_121-triangle-meshes)
  - [_12.1.1 Mesh Topology网格拓扑](#_1211-mesh-topology网格拓扑)
  - [_12.1.2 Indexed Mesh Storage索引网格存储](#_1212-indexed-mesh-storage索引网格存储)
  - [_12.1.3 Triangle Strips and Fans带状三角形和扇形三角形](#_1213-triangle-strips-and-fans带状三角形和扇形三角形)
  - [_12.1.4 Data Structures for Mesh Connectivity网状连接的数据结构](#_1214-data-structures-for-mesh-connectivity网状连接的数据结构)
- [_12.2 Scene Graphs场景图](#_122-scene-graphs场景图)
- [_12.3 Spatial Data Structures空间数据结构](#_123-spatial-data-structures空间数据结构)
  - [_12.3.1 Bounding Boxes](#_1231-bounding-boxes)
  - [_12.3.2 Hierarchical Bounding Boxes分层bounding boxes](#_1232-hierarchical-bounding-boxes分层bounding-boxes)
  - [_12.3.3 Uniform Spatial Subdivision](#_1233-uniform-spatial-subdivision)
  - [_12.3.4 Axis-Aligned Binary Space Partitioning轴对齐二分空间划分BSP](#_1234-axis-aligned-binary-space-partitioning轴对齐二分空间划分bsp)

<!-- /TOC -->

**Data Structures for Graphics**

下面几种数据结构在图形学中得到广泛应用
- meshes. 网格, 我们把重点放在三角形网格
- scene-gragh data structure场景图数据结构
- spatial data structures空间数据结构. 应用在3D等场景下.
- tiled multidimensional array平铺多维数组. 涉及到存储.

<a id="markdown-121-triangle-meshes" name="121-triangle-meshes"></a>
### _12.1 Triangle Meshes

大部分现实世界模型都是由复杂的共用顶点的三角网构成的.  
称之为**triangular mesh**, **triangle mesh**, **triangular irregular networks(TIN)**

访问三角形的效率异常重要, 带宽、内存空间非常包裹, 通过网络传输或者从CPU到图形系统, 都会占用带宽.

三角网除了存储三角形的位置信息, 还需要存储三角形顶点或者边、面对应的纹理坐标、辐射参数等等信息, 用来做渲染等等操作

<a id="markdown-1211-mesh-topology网格拓扑" name="1211-mesh-topology网格拓扑"></a>
#### _12.1.1 Mesh Topology网格拓扑

三角形网格的必须满足**manifold**的拓扑关系. 什么意思呢?

manifold是什么意思呢? (注意, 这里特指2维manifold)  
是指一个表面上的任意一个点以及它临近的小块区域都可以平铺成一个平坦的平面.  
如果还不太好理解, 我们看看反例:

<img src="./_images/triangle_mesh_manifold.png" width=30%> 

第一张图的左侧, 边上的点可以形成两个平面. 或者可以这么理解, 边上的点的平面和三角形内部的一个点的平面可以不是同一个平面. 
第二张图的左侧, 两个平面得压在一起才能得到一个平面. 

我们可以通过下面两个条件来检测三角形网是否是manifold:
- 每一条边都被两个三角形共用
- 每个顶点都被一个完整的loop三角形序列环绕
通过这两个条件我们在回看上面这张图  
第一张图的左侧的边被三条边共用. 第二张图的左侧的顶点, 被两个完整的loop三角形序列环绕

但是, 很多情况下, 我们希望三角网是有边界的, 那么, 边界的三角形是不满足上面的两个条件的.  
我们需要修改一下条件:
- 每条边必须被一个或者两个三角形使用
- 每个顶点和一组边连接的三角形连接
如下图所示:

<img src="./_images/triangle_mesh_manifold1.png" width=30%>  

最右侧的图的顶点和两组三角形连接

另外一个问题, 我们需要判定三角形的外侧和里侧, 判定规则是:  
三角形的顶点使用排列顺序, 如果从一面看, 顶点按逆时针排列, 这一面就是外侧.  
这样三角形网的所有三角形的顶点都按同一个方向排列.  
这样我们发现一个有趣的特点, 两个三角形的公共边的两个顶点, 在两个三角形的排列顺序是相反的.  
番外: 一个特殊的三角形网Mobius band, 不满足这个特性

#### _12.1.2 Indexed Mesh Storage索引网格存储

给定一个简单的三角网, 如下图:

<img src="./_images/indexed_mesh_storage.png" width=50%>

三个三角形, 4个顶点. 我们可以很直接的如左侧这样存储, 每个三角形的三个顶点的位置都存储下来.  
但是这里有很多重复的地方, 那么我们把三角形和顶点分来来存储, 4个顶点单独存储, 三角形至存储对应顶点的引用、或者指针.  

左侧的存储方法可以写为:
```C++
Triangle {
  vector3 vectorPosition[3]
}
```
右侧的存储方法可以写为
```C++
// 顶点, vector3向量
Vertex {
  vector3 position
}

// 三角形, 包含三个Vertex的数组
Triangle {
  Vertex v[3]
}
```
我们把这种方法总结一下, 把一个三角网写在一起:
```C++
// 这个三角网包含nv个顶点, 顶点以vector3向量表示, 构成数组verts
// 以及包含nv个大小为3的整数的数组的数组, 也就是nv个三角形, 以iInt表示
// 这个三角形以3个整数表示, 整数代表其在verts里的索引
IndexedMesh
{
  int iInd[nt][3]
  vector3 verts[nv]
}
```
下图是一个实例:

<img src="./_images/indexed_mesh_storage_example.png" width=50%>

#### _12.1.3 Triangle Strips and Fans带状三角形和扇形三角形

利用indexed mesh的存储形式, 我们可以节省很多空间.  
我们还可以做进一步优化: 带状三角形和扇形三角形. 它们在图形程序中的效率更高.

triangles fan如下图:

<img src="./_images/triangle_fan.png" width=30%>

我们可以把这4个三角形用一个包含6个顶点索引的数组表示: [0, 1, 2, 3, 4, 5]  
第一个数字0表示扇形的中点, 然后012构成一个三角形, 023构成一个三角形, 依此类推

triangle strip如下图:

<img src="./_images/triangle_stripe.png" width=30%>

同样的, 我们把所有顶点的索引放到一个数组里表示这一个带状三角形: [0, 1, 2, 3, 4, 5, 6, 7]  
012构成一个三角形, 123构成一个三角形, 依此类推  
我们注意到, 三角形的方向应该一致, 都应该是顺时针或者逆时针, 我们改一下:  
012构成一个三角形, 213构成一个三角形, 234构成一个三角形, 依此类推  

书上有推到效率对比, 这里不详述了

#### _12.1.4 Data Structures for Mesh Connectivity网状连接的数据结构

上面的数据结构存在一个问题, 不能解决快速查询一个顶点的相连三角形、相连的边, 或者一条边的共用三角形等问题.  
我们继续做优化:
```C++
Triangle {
  Vertex v[3];
  Edge e[3];
}
Edge {
  Vertex v[2];
  Triangle t[2];
}
Vertex {
  Triangle t[];
  Edge e;
}
```
这种结构直接回答了上面的三个问题, 但是存储了过多的冗余数据. 下面介绍三种数据结构

**The Triangle-Neighbor Structure**

```C++
Triangle {
  Triangle nbr[3]; // 相邻的三个三角形, 第k个三角形和此三角形共用v[k]和v[k+1]两个顶点
  Vertex v[3];
}
Vertex {
  // ...向量数据
  Triangle t; // 任意一个相连三角形
}
```
例子如下:

<img src="./_images/triangle_neighbor_structure1.png" width=50%>

数据可以这样存储:
```C++
Mesh {
  // ...顶点向量数据
  int tInt[nt][3]; // 三角形的三个顶点id
  int tNbr[nt][3]; // 相邻的三个三角形的id
  int vTri[nv]; // 顶点相邻的某个三角形id
}
```
查询一个顶点相邻的所有三角形的算法是:
```C++
Triangle[] findTrianglesByVertex(Vertex v) {
  Triangle allTriangles[];
  t = v.Triangle;
  allTriangle.push_back(t);
  do {
    vIndex = findIndex(t, v); // 找到这个顶点在这个三角形的顶点数组里的index
    t = t.nbr[vIndex]; // 这个索引对应的相邻三角形和此三角形共用此顶点
    allTriangle.push_back(t);
  } while t != v.Triangle
  return allTriangles
}
```
这个算法存在一个问题: 里面有一个findIndex这个函数, 遍历查找. 有没有更好的数据结构?
```C++
Triangle {
  Edge nbr[3]; // 三条边
  Vertex v[3]; // 三个顶点
}
Edge {
  // 这里注意, 一条边会存两条数据, 分别对应两个相邻的三角形
  // 在Triangle里的edge指的是相邻三角形表示的边
  Triangle t; 
  int i; // in [0, 1, 2]
}
Vertex {
  // ... 向量
  Edge e; // 任意连接的边
}
```
查询一个顶点相邻的所有三角形的算法是:
```C++
Triangle[] findTrianglesByVertex(Vertex v) {
  auto e = v.Edge();
  t, i = e.Edge, e.i
  allTriangle.push_back(t);
  do {
    i = mod((i + 1), 3); // 逆时针找到下一条边的index
    int newE = t.nbr(i);
    t = newE.Triangle; // 下一个三角形
    allTriangle.push_back(t);
  } while t != v.e.t
  return allTriangles
}
```
特别要注意, 三角形存的边, 是由相邻的三角形和索引来表示的, 也就是说, 存储的是这个三角形的这条边是相邻的三角形的第几条边  
cornell大学的这篇文章讲的比较详细[http://www.cs.cornell.edu/courses/cs4620/2017sp/slides/03trimesh2.pdf](http://www.cs.cornell.edu/courses/cs4620/2017sp/slides/03trimesh2.pdf)

**The Winged-Edge Structure**

看图说话:

<img src="./_images/winged_edge_structure.png" width=30%>

edge是有方向的, 来区分左右, 从而逆时针得到pred left, next left以及pred right, next right

**The Half-Edge Structure**

<img src="./_images/half_edge_structure.png" width=30%>

这张图比较好理解.  
把中间那条边分成左右两侧两个部分来分别存储.  
对于这条边的左侧, 我们存储它的对应的右侧半边(pair), 逆时针方向的下一条半边(next), 已经逆指针方向的下一个顶点(head), 以及其方向上的面(三角形face)  
```C++
HEdge {
  HEdge pair, next;
  Vertex v;
  Face f;
}
Face {
  // 其他数据
  Hedge h; // 这个面的任意一条HEdge
}
Vertex {
  // 其他数据
  Hedge h; // 这个顶点相连的任意一条Hedge
}
```
通过这样的数据结构, 我们很容易能得到一个顶点相连的所有edge  
以及一个面的所有edge

### _12.2 Scene Graphs场景图

三角网解决了表示单个物体的问题, 但是在复杂场景中有很多物体, 这些物体还可能是相关联的.   
在第六章我们讲到了矩阵转换, 例如旋转、平移等等. 我们对一个物体进行平移旋转时, 其关联物体也会跟着平移旋转, 其自身也会有独立的平移旋转.  
这就比较复杂了, 我们为了解决这个问题, 一般会采用分层管理, 用scene graph来实现.

<img src="./_images/scene_gragh.png" width=50%>

图里是一个铰链, 上面的杆会带动下面的杆运动.  
对于上面的杆, 旋转平移可以这样转换:
$$
M_1 = rotate(\theta) \\
M_2 = translate(p) \\
M_3 = M_2M_1
$$
我们对上面的杆上的所有点都乘以$M_3$进行转换  
对于下面的杆, 除了自身的转换, 还要加上上面的杆的转换:
$$
M_a = rotate(\phi)  \\
M_b = translate(b) \\
M_c = M_bM_a \\
M_d = M_3M_c
$$
对下面的杆的所有点都乘以$M_d$进行转换

更形象一点的例子:

<img src="./_images/scene_graph_car.png" width=30%>

一条船上一辆汽车, 汽车的前后轮. 我们分层进行管理.  
对于轮船, 转换矩阵是$M_0$  
对于汽车, 转换矩阵是$M_0M_1$
对于前轮, 转换矩阵是$M_0M_1M_2$
对于后轮, 转换矩阵是$M_0M_1M_3$

我们对这些矩阵分层管理, 计算时可以灵活的进行push和pop

### _12.3 Spatial Data Structures空间数据结构

在很多图形程序中, 快速定位几何对象非常重要.  
例如光线追踪ray tracing需要定位对象来反射.  
交互式导航程序中需要显示可见对象.  
游戏和物理模拟中需要定位对象来计算何时何地发生碰撞.  
这就需要用到Spatial Data Structures空间数据结构

Spatial data structures大体可分为三类: 根据对象划分, 根据空间划分, 空间划分又可分为规则空间划分和不规则空间划分.

这里我们以光线追踪为例来说明空间数据结构. 对象查找和碰撞探测和其原理类似.

本章会讨论三项技术:
- bounding volume hierarchies(BVH)
- uniform spatial subdivision规则空间划分
- binary space partition
下图说明了前两种技术:

<img src="./_images/spatial_data_structure1.png" width=30%>

左边是用uniform spatial subdivision, 对空间进行均匀划分  
右边是bounding box hierarchies, 根据几何对象的边界进行划分

#### _12.3.1 Bounding Boxes

对于一个复杂的有很多几何对象的场景, 一条射线穿过, 它会和哪些几何对象相交呢?  
最直接的方法就是拿这条射线和每个几何对象进行计算, 来判断是够相交.  
但是这样太慢了, 几何对象可能非常多. 我们需要加速算法.  
其中bounding box就是一种有效的方法, 我们把若干个几何对象的边界组成一个box, 我们来判断射线和这个box是否相交. 至于如何划分, 之后再做讨论.  

接下来的问题是如何进行计算和判断? 我们先简化为二维的场景, 一条射线和一个矩形是否相交. 如下图所示:

<img src="./_images/bounding_box.png" width=30%>

这个矩形有四个边界: $x_{min}, x_{max}, y_{min}, y_{max}$  
$$(x, y) \in [x_{min}, x_{max}] \times [y_{min}, y_{max}]$$
假设射线的起点是: $(x_e, y_e)$, xy方向的导数是$d_x, d_y$  
这里可以理解为, 起点分别在x和y方向上的运行速度, 或者我们可以理解为起点是光源, 这两个参数是光束.  
这样我们就能计算出射线分别和这个矩形的四条边界相交的时间:
$$
t_{xmin} = (x_{min} - x_e)/d_x \\
t_{xmax} = (x_{max} - x_e)/d_x \\
t_{ymin} = (y_{min} - y_e)/d_y \\
t_{ymin} = (y_{max} - y_e)/d_y \\
$$
和x轴的两个边界相交的时间段, 和y轴的两个边界相交的时间段, 这两个时间段如果有交集的话, 这个射线就和这个box相交. 如果没有交集, 那就是不相交.  
物理上可以这么理解, 如果射线通过x轴的两个边界时, 也通过y轴的两个边界, 那这个时间段就是在矩形内.  
还不好理解的话, 我们反过来理解, 如果涉嫌通过x轴的两个边界时, 这个时间段没有通过y轴的两个边界, 那么这条射线肯定完全从矩形的左侧或者右侧通过, 避开了矩形的左上角和右下角  
pseudocode如下:
```
if (t_xmin > t_ymax) or (t_ymin > t_xmax) then
  return false
else
  return true
```

书中还讨论了$d_x$$d_y$为负值或者为0的情况, 这里不再详述  
负值就是方向相反, 把min,max调换一下即可  
0的情况, 用正负无穷来处理

#### _12.3.2 Hierarchical Bounding Boxes分层bounding boxes

这一章节讲的是如何对一个bounding box进行分割. 分割成一个二叉树, 直到这个bounding box只包含一个几何对象.  
分割是按照几何对象来进行划分,可以沿着一个轴分割, 可以按几何对象的数量, 可以按照体积.  

<img src="./_images/hierrarchical_bounding_boxes.png" width=30%>

在GAMES101里讲的比较详细, 作业6也和这个相关

#### _12.3.3 Uniform Spatial Subdivision

hierarchical bounding boxes是按照几何对象来划分, uniform spatial subdivision是按照空间来划分  
hierarchical bounding boxes的划分情况下, 一个点可能属于两个bounding box, 但是一个几何对象只属于一个bounding box  
uniform spatial subdivision则相反, 一个点只属于一个bounding box, 但是一个对象可能被切分, 属于两个bounding box

Uniform Spatial Subdivision是按照轴均匀的划分, 称为**axis aligned bounding box(AABB)**  
当光线和一个几何对象相交时, 则传播停止.

<img src="./_images/uniform_spatial_subdivision.png" width=50%>

#### _12.3.4 Axis-Aligned Binary Space Partitioning轴对齐二分空间划分BSP

这种划分方式是上面两种方式的结合, 只是如果一个几何对象被划分线切割, 那么这个几何对象同时属于两个bounding box

