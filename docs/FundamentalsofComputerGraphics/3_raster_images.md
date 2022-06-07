<!-- TOC -->

- [_3.1 Raster Devices](#_31-raster-devices)
  - [_3.1.1 Displays](#_311-displays)
  - [_3.1.2 Hardcopy Devices](#_312-hardcopy-devices)
  - [_3.1.3 Input Devices](#_313-input-devices)
- [_3.2 Images, Pixels, and Geometry](#_32-images-pixels-and-geometry)
  - [_3.2.1 Pixel Values](#_321-pixel-values)
  - [_3.2.2 Monitor Intensities and Gamma](#_322-monitor-intensities-and-gamma)
- [_3.3 RGB Color](#_33-rgb-color)
- [_3.4 Alpha Compositing阿尔法合成](#_34-alpha-compositing阿尔法合成)
  - [_3.4.1 Image Storage](#_341-image-storage)

<!-- /TOC -->

**Raster Images光栅图像**

大多数计算机图形都显示在某种raster display光栅显示器上, raster display是由矩形pixel阵列构成, 每个pixel有不同的颜色, 颜色由红绿蓝三种颜色组成.

大多数输入设备也是光栅设备, 比如打印机, 也是对矩形阵列的点加上不同的墨水而实现.  
数字摄像机也是光栅设备, 感应器记录其每个pixel的颜色和光线强度. 扫描仪也类似, 将扫描对象的信息记录在光栅阵列中.

因为大多数设备都是光栅设备, 所以raster image是记录和处理图像的最普遍的格式. 通过raster image存储的像素颜色来控制显示器的像素颜色从而来显示图像.  
但是我们应该将raster image和raster display分开理解, 他们是独立的. raster image是对图像的描述, 和raster display没有关系, raster display要通过taster image尽可能完美地显示图像.

除了raster image之外, 还有一种image的格式是vector image矢量图, 它记录图像的形状, 和pixel没有关系, 所以它可以显示成任何分辨率. 换句话说, 它记录了显示图像的方法. 这也是其缺点, 显示前需要将其光栅化. 它适用于机械制图等精度要求比较高但真实感和复杂着色要求比较低的领域.

<a id="markdown-_31-raster-devices" name="_31-raster-devices"></a>
### _3.1 Raster Devices

- Output
  - Display:
    - Transmissive透射: liquid crystal display LCD液晶显示器
    - Emmissive: light-emitting diode LED显示器
  - Hardcopy:
    - Binary: ink-jet printer喷墨打印机
    - Continuous tone: dye sublimation printer热升华打印机
- Input
  - 2D array sensor: digital camera
  - 1D array sensor: flatbed scanner平板扫描仪

#### _3.1.1 Displays

当前世界上的显示器都是基于像素阵列, 包括电脑显示器、电影放映机、电视等等.  
他们又可以分为两类: transmissive和emissive.

emissive是指自己发光, 比如LED(发光二极管)显示器.  
transmissive是指改变通过其自身的背后光源发出的光, 比如LCD液晶显示器.

LED显示器的一个pixel由一个或多个LED构成, 在不同的电流下发出不同的光, 这是因为一个pixel有多个subpixel, 这些subpixel可以发出不同的红绿蓝色, 人眼只能看到它们混合之后的颜色, 分辨不出单个subpixel的颜色.

LCD有液晶和其前后两个polarizer偏光片构成:  
<img src="_images/fundamentals_of_computer_graphics/LCD_display.png">  
两个偏光片是垂直的, 中间的液晶由可以旋转光的polarization偏光的分子构成, 如果不给液晶加上电压, 那么液晶就不会旋转光的偏振, 那么光就会被全部挡住. 电压的变化可以控制多少光被透过. 和LED显示器一样, LCD的一个pixel也有多个subpixel构成, 改变每个subpixel的透光率, 就可以显示不同的颜色.

pixel的数量就是显示器的reresolution分辨率, 比如我们说$1920\times 1200$分辨率, 指的是230400个pixel, 1920列, 1200行.

#### _3.1.2 Hardcopy Devices

很多打印机只能打印binary image: 要么墨水落在纸上, 要么不落, 不会存在中间量. (注: 这里应该指的是针式打印机)    
例如喷墨打印机. 打印头上有墨水, 一行一行打印. 这种打印机的分辨率取决于墨滴的大小, 和纸张移动的距离.

dye sublimation printer是continuous tone打印, 它通过加热来调整染料打印到印纸上的量. 不像喷墨打印机只有有和无. 加热元件的数量决定了打印机的横向的分辨率, 纵向的分辨率取决于纸张的加热和冷却速度.

不同于显示器, 打印机的分辨率用pixel density描述.  
dye sublimation printer的打印头上有300个加热元件每英寸, 那么我们说分辨率是300ppi(pixels per inch).  
对于喷墨打印机, 一般说dpi(dots per inch), 表示一英寸有多少个网格点.

#### _3.1.3 Input Devices

一般是摄像机和扫描仪, 它们都基于传感器阵列, 对每个pixel做光学测量.

camera的传感器最普遍的有两种: CCD(charge-coupled device电荷耦合器件)和CMOS(complimentary mental-oxide-semiconductor互补金属氧化物半导体).  
一般的摄像机通过color filter或mosaic只获取RGB的一种颜色, 然后软件在成像时通过demosaicking补齐其他颜色.  
还有camera会用三个阵列来单独获取RGB三种颜色, 获取的信息比mosaiccamera要多, 这样成像就不需要额外的处理.  
camera的resolution就是阵列的列和行, 例如$3000 \times 2000$, 就是600万像素, 简称为6megapixel(MP) camera.

flatbed scanner测量RGB三种颜色, 分别用三个独立的阵列.  
分辨率和打印机一样, 以扫描阵列的密度为基准, 以ppi(pixels per inch)为单位.  
扫描阵列有$n_x$个pixels, 那么彩色扫描仪就有$3 \times n_x$个像素.

### _3.2 Images, Pixels, and Geometry

image是像素阵列, 每个像素存储了现实世界的信息(例RGB颜色值). 我们需要找到通用的算法来获取或处理这些信息.

在输出设备里, 每个pixel的值和显示器的位置关联, 在输出设备里, pixel的值和传感器的位置关联. 所以我们可以定义这样一个函数:
$$I(x, y): R \to V$$
$R \in \mathbb{R}^2$, 它是一个矩形区域. $V$是像素的值.  
对于一个只有亮度没有颜色的图像, $V = \mathbb{R}^+$  
对于RGB图像, $V = (\mathbb{R}^+)^3$

$x, y, R$怎么定义? 上面的式子的现实意义是指一个像素对应的值, 像素我们可以用第几列第几行来表示(我们说分辨率都说的是列的数量和行的数量). 如下图:  
<img src="_images/fundamentals_of_computer_graphics/pixel_coordinate.png">  
对于分辨率是$n_x \times n_y$的图像, 左下角是(0, 0)像素, 右上角是(n_x - 1, n_y - 1)像素.  
对于矩形区域的任意点, 它的坐标, 也就是$R$的范围是:
$$R = [-0.5, n_x - 0.5] \times [-0.5, n_y - 0.5]$$

#### _3.2.1 Pixel Values

对于灰度图, pixel的值只需要一个32bit的浮点数表示, RGB图则需要3个32bit的浮点数表示.(浮点数的内存大小是4byte=32bit)  
那么对于一个10MP(megapixel)的图, 就需要占用$3 \times 32 \times 10000000 \div 8 \div 1024 \div 1024 \approx 115MB$的内存空间

需要做优化.  
首先, 我们把值压缩到[0, 1]的范围, 尽管实际上光照强度无限大.  
第二, 在不同场景用不同精度的数字.  
比如8bit的图像, pixel的取值就是0, 1/255, 2/255, ..., 254/255, 255/255.  
精度高的图像称为high dynamic range(HDR)图像.  
精度低的图像称为low dynamic range(LDR)图像.  

1-bit grayscale用于文本和灰度图  
8-bit RGB用于web和电子邮件等场景  
8-10bit RGB用于电脑显示器  
12-14bit RGB用于摄像机的原图  
16-bit grayscale用于医学影像  
16-bit RGB用于实时渲染  
32-bit RGB用于处理HDR图像

这样的处理会导致一些artifacts.  
第一, 亮度很高的光会被剪切掉, 例如太阳的反射光
第二, 精度原因的四舍五入会导致quantization artifacts量化伪影和banding条带, 因为强度和颜色的明显跳跃. 动画和视频可能不太明显, 图像就会让人觉得很讨厌, 尤其是来回移动时.

#### _3.2.2 Monitor Intensities and Gamma

**显示器**

显示器会把pixel的值转换为intensity level亮度. 但是pixel的值和显示器的intensity level不是线性的, 例如pixel的值是0.5, intensity level不是0.5, 可能是0.25. 可以用下面的公式表示:
$$display\ intensity = (maximum\ intensity)a^{\gamma}$$
$a$是0-1的pixel value. 如果$\gamma$是2, pixel value是0.5的话, 那么intensity level就是显示器最大intensity的0.25.

问题是: 显示器为什么要做这样的转换? 以前的CRT显示器做不到电压和亮度的线性转换, 现代显示器如果能做到线性, 是不是就不需要做转换了?

**image pixel value**

pixel值的物理意义是什么?  
上面介绍的input device包括camera和scanner, 它们获取的都是光照强度.  
那么, 如果pixel记录的就是光照强度, 显示器经过intensity level的转换后, 显示的画面不就变暗了吗?  

事实上, pixel value是光照强度经过gamma encode之后的值!  
那么光照强度是0.5, 我们将其校正为:
$a^{\prime} = a^{\frac{1}{\gamma}}$
这样的话, 显示器的intensity level就等于0.5倍maximum intensity了. 因为:
$$display\ intensity = (maximum\ intensity)\left({a^{\frac{1}{\gamma}}}\right)^{\gamma} = a(maximum intensity)$$
这称之为gamma decode, 经过这个过程, 显示器显示的光照强度还是0.5.  
问题来了: 为什么要进行这样的encode, decode? 来回折腾干嘛? 测得多少光照强度就显示多少光照强度不好吗?

**gamma encode**

我们为什么要进行gamma encode? 也就是对光照强度做指数变换? 有两个原因:  

人眼对光线的感知和光线强度不是线性关系:  
<img src="_images/fundamentals_of_computer_graphics/eye_luminance.png">  
横轴是物理光线强度, 纵轴是探测到的强度.  
对于camera来说, 实际的光线强度是多少, 测出来就是多少, 对应紫色的直线.  
对于人眼来说, 我们看红色的箭头, 物理光线强度不到0.25, 我们感受到的亮度就达到了0.5.  
换句话说, 坐标轴0点是黑, 1点是白, camera测到的光线强度是0.25, 我们感受到的亮度是中灰.
而且, 这个特性还让我们知道, 人眼对黑暗的感知能力更强, 光线强度从0.1变到0.2, 人眼能感受到亮度增加了一倍, 但是在更亮的范围, 从0.4到0.8, 人眼才能感受到亮度增加了一倍, 我们更能分辨暗色.  
gamma encode可以将光线强度转换为人眼的颜色感知. 两者显示的效果如下:  
<img src="_images/fundamentals_of_computer_graphics/image_gamma.png">  
我们可以说: 这是因为显示器做了gamma decode, 如果显示器没做, 那么第一张图应该是正常的, 第二张图会过亮.  
是这样的, 接下来说gamma encode的第二个作用.

在上面3.2.1节中, 我们说为了空间压缩, 会用bit位整数来存储pixel value, 如果采用5-bit来存储pixel value, 也就是0-32之间.  
因为光线强度大约0.25, 对于数字$32 \times 0.25 = 8$, 人眼就能感受到中灰色, 所以如果我们把光线强度存储为pixel value, 那么0-8对应的是黑色到中灰的区域, 8-32对应中灰到白色的区域, 相当于我们在光线强度的测量数据里, 暗的区域采集了8个点, 亮的区域采集了24个点, 上面我们讲到人眼更能分辨暗色, 这样做的话, 就丢失了很多对人眼很重要的暗色信息.  
如果我们做gamma encode, 相当于在光线强度0-8之间采样16个点, 8-32之间采样16个点, 然后将其转换成0-32的整数. 这样就保留了更多暗色细节:  
<img src="_images/fundamentals_of_computer_graphics/gamma_encode.png">  
上图是指人眼的颜色感知. 图的中间对应中灰, 光线强度是0.25, 我们可以直观的感受到左侧更能分辨, 右侧好像都是白光一样, 变化很小.   
中图是把线性的光线强度直接作为pixel value对应的人眼的感知颜色, 可以看见暗灰色区域很稀疏, 亮灰色区域很密.  
下图是gamma encode的采样效果, 对应的人眼感知的颜色就是均匀的.

这里是以5-bit作为例子, 电脑显示一般是8-bit, pixel value的取值范围是:
$$a = \left\{\frac{0}{255}, \frac{1}{255}, \frac{2}{255}, ..., \frac{254}{255}, \frac{255}{255}\right\}$$
道理是一样的.

这样, 我们可以理解为gamma encode是对物理光线强度做了不均匀的采样, 然后做了指数变换变成连续的整数值.  
这里gamma encode的gamma值小于1, 光线强度放大了.

**gamma decode**

上面做了gamma encode之后, pixel value对应的人眼感知的颜色是均匀的, 但是请注意, pixel value放大了.  
让我们再回顾上面的1和2.  
现在我们知道了如果如果我们gamma encode之后的pixel value直接显示, 就会过亮.  
所以我们要进行gamma decode, 将其还原成本来的光线强度之后再显示, 这就是显示器的gamma correct.

<img src="_images/fundamentals_of_computer_graphics/gamma_correct.png">  
经过gamma encode和decode之后, 原始的光线强度和输出的光线强度就是线性的了, 暗色的光线数据更多, 人眼感知的颜色是均匀的, 暗色细节没有丢失. 

**其他**

这里有个巧合, 以前的CRT显示器做不到电压和亮度的线性关系, 是指数关系, 最后确定的gamma标准值是2.2, 与光照和人眼感知颜色的关系和数值正好相反, 所以encode是gamma用1/2.2, decode用2.2. 现代显示器虽然能做到电压和亮度的线性关系, 但是还是会加上gamma校正.

如何求得gamma的值, 可以通过下面的方法测量:  
<img src="_images/fundamentals_of_computer_graphics/gamma_measurement.png">  
左边是黑白交替的检查板, 代表0.5亮度, 右侧显示器(色板)调整输入值a(更换色板), 使两边看起来一样, 这样:
$$0.5 = a^{\gamma}$$
$a$的值我们知道, 这样就可以求得$\gamma$的值.  
1996年制定标准时测定的值是2.2.

其实光线强度和人眼感知的关系比较复杂, 不同的环境下感知不一样, 比如星光下光线强度和人眼感知是线性的. 2.2做到了日常环境的近似和精度需求.

另外, 我们说gamma encode是因为pixel value的取值范围较小, 会丢失暗色细节. 那么假如pixel value是浮点数呢?   
那么我们说, 这的确可以不要再做gamma encode和decode, 获取的光线强度1:1显示在显示器, 所见即所得就好.  
因为pixel value的精度很高, 不再是5-bit图像只能要8个整数记录暗色(如果不做gamma encode的话), 它不会丢失暗色细节了. 代价当然是占有内存很大了.

另外, camera和scanner获取自然界的光线强度, 所以需要做gamma操作, 我们在图形学里操作的图形, 也是对光线强度做线性操作, 所以也需要做gamma操作.

**参考资料**

<https://www.cambridgeincolour.com/tutorials/gamma-correction.htm>

### _3.3 RGB Color

RGB三种颜色可以累加成任何其他颜色, 基本的累加效果是:

<img src="_images/fundamentals_of_computer_graphics/RGB.png">

red + green = yellow  
green + blue = cyan青色  
blue + red = magenta紫色  
red + green + blue = white

这样组合的颜色都可以在RGB显示器上显示出来.  

这三种颜色的组合可以组成一个cube:  
<img src="_images/fundamentals_of_computer_graphics/RGB_cube.png">

RGB level和上一章节讲的显示器接受的值一样, 都是整数格式.  
整数占1byte, 也就是8bit. 这样取值范围是0-255.  
RGB就占用24bit, 每种颜色有256个可能的值.

### _3.4 Alpha Compositing阿尔法合成

设想一种场景, 一张image被另一张image叠加  
这样, 一个pixel如果被foreground完全覆盖, 那么这个像素的value就替换成了foreground的value.  
如果被部分覆盖, 那么这个像素的value就是两个image的value的组合. 这种情况称为pixel coverage. (如果foreground是透明或者半透明, 需要格外小心)  
如果foreground的fraction是$\alpha$, 那么这个像素的value就是:
$$c = \alpha c_f + (1 - \alpha)c_b$$
$\alpha$可以存储在一张单独的grayscale image上, 称为alpha mask或者transparency mask.  
或者在RGB之上在增加一个通道, 称为alpha channel. 这种图像称为RGBA image. 如果是8-bit image, 那么一个像素占32bit.

#### _3.4.1 Image Storage

一般RGB image的一种颜色占8-bit, 一张一百万像素的image占差不多3MB.  
为了减小占用, 有很多压缩方法, 有的是有损的, 有的是无损的.  
jpeg是有损的, 但是压缩的是人类视觉系统之外的信息, 适用于自然图片.  
tiff是无损的.  
ppm和png是有损的.  

因为压缩, image的输入和输出的过程会有改变, 为了简单和效率, 一个选择是ppm格式.