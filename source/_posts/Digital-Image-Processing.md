---
title: Digital-Image-Processing
date: 2021-06-28 22:11:36
categories: Theories
tags:
- OpenCV
---

数字图像处理包括空间域图像增强、频率域图像增强、形态学图像处理、图像分割、特征提取等。

<!--more-->

## Image enhancement in the spatial domain (空间域图像增强)

### Intensity transformation (灰度变换)

#### 基本灰度变换函数

##### Image negatives (图像反转)

s = L - 1 - r

##### log transformation (对数变换)

s = clog(1+r)

##### power-law transformation (gamma correction) (幂次变换（gamma校正）)

s = cr^r

#### 分段线性变换函数

##### Contrast stretching (对比拉伸)

<img src="https://img-blog.csdnimg.cn/20210629155650684.png">

##### gray-level slicing (灰度切割)

##### bit-plane slicing (位图切割)

#### Histogram Processing (直方图处理)

##### Histogram equalization (直方图均衡化)

(1) 统计各灰度级rk的像素数目n k (k=0,1,2,…L-1)
(2) 用p(rk)=n k /n 计算图像直方图
(3) 用变换函数<img src="https://img-blog.csdnimg.cn/20210629160359854.png">
计算映射后输出的灰度级sk，舍入处理得sk’
(4)将灰度级为rk的像素灰度级变为sk’
(5) 统计映射后新的灰度级sk’的像素数目nk ’
(6) 计算输出图像的直方图 

##### Histogram matching(specification) (直方图规定化（直方图匹配）)

 (1)求出已知图像的直方图

(2)利用 <img src="https://img-blog.csdnimg.cn/20210629161712194.png">对每一灰度级rk预计算映射灰度级sk.
 (3)利用<img src="https://img-blog.csdnimg.cn/20210629161734137.png">从给定的Pz(z)得到变换函数G.
 (4)利用<img src="https://img-blog.csdnimg.cn/20210629161755669.png">对每一个sk值求zk.
 (5)将灰度级为rk的像素灰度级变为zk.
 对于原始图像的每个像素,若像素值为rk,将该值映射到其对应的灰度级sk; 然后映射灰度级sk到最终灰度级zk.

### Spatial filtering (空域滤波)

#### Smoothing spatial filters (平滑空域滤波器)

##### Averaging filter (均值滤波器)

The output (response) of a smoothing, linear spatial filter is simply the average of the pixels contained in the neighborhood of the filter mask. These filters sometimes are called averaging filter. 

<img src="https://img-blog.csdnimg.cn/2021062916243138.png">

##### Order-Statistic Filters (顺序排序滤波器)

Order-statistics filters are nonlinear spatial filters whose response is based on ordering the pixels contained in the image area encompassed by the filter, and then replacing the value of the center pixel with the value determined by the ranking result.

##### median filter (中值滤波器)

Replace the value of a pixel by the median of the gray levels in the neighborhood of that pixel.

#### sharpening spatial filters (锐化空域滤波器)

##### 一阶微分

- the gradient operator (梯度算子)

  <img src="https://img-blog.csdnimg.cn/2021062916362410.png">

  <img src="https://img-blog.csdnimg.cn/20210629163645554.png">

- the Roberts cross-gradient operators (Roberts交叉算子)

  <img src="https://img-blog.csdnimg.cn/20210629163945506.png">

  <img src="https://img-blog.csdnimg.cn/2021062916401035.png">

- Sobel operators (Sobel算子)

  <img src="https://img-blog.csdnimg.cn/20210629164038662.png">

  <img src="https://img-blog.csdnimg.cn/20210629164103664.png">

- Prewitt operators (Prewitt算子)

  <img src="https://img-blog.csdnimg.cn/2021062916413020.png">

<img src="https://img-blog.csdnimg.cn/20210629164149919.png">

##### 二阶微分

- The laplacian operator (拉普拉斯算子)

  <img src="https://img-blog.csdnimg.cn/20210629163003867.png">

  <img src="https://img-blog.csdnimg.cn/20210629163033680.png">

- Laplacian-based Enhancement operator (拉普拉斯增强算子)

  <img src="https://img-blog.csdnimg.cn/20210629163255390.png">

  <img src="https://img-blog.csdnimg.cn/20210629163322935.png">

- unsharp masking (非锐化掩蔽)

- highboost filtering (高提升滤波)

## Image enhancement in the frequency domain (频域图像增强)

### Fourier transform (傅里叶变换)

#### Discrete Fourier Transform (离散傅里叶变换DFT)

<img src="https://img-blog.csdnimg.cn/20210629134320105.png">

<img src="https://img-blog.csdnimg.cn/20210629134354229.png">

<img src="https://img-blog.csdnimg.cn/20210629134811667.png">

- Translational property (平移性)

  <img src="https://img-blog.csdnimg.cn/20210629135229681.png">

- Periodic property (周期性)

  <img src="https://img-blog.csdnimg.cn/20210629135706652.png">

- Conjugate symmetric property (共轭对称性)

  <img src="https://img-blog.csdnimg.cn/20210629135737567.png">

- Average property (平均值定理)

  令u=0,v=0,则F(0,0)=~f(x,y):

  <img src="https://img-blog.csdnimg.cn/20210629140040481.png">

- Convolution theorem (卷积定理)

  

  <img src="https://img-blog.csdnimg.cn/20210629141847116.png">

  <img src="https://img-blog.csdnimg.cn/20210629141918104.png">

  令s=-m,t=-n,g(x,y)=f(x,y)*h(x,y):

  <img src="https://img-blog.csdnimg.cn/20210629141702901.png">

### 频域滤波流程

<img src="https://img-blog.csdnimg.cn/20210629143232377.png">

### Low-pass filtering (低通滤波器)

#### Ideal lowpass filters (理想低通滤波器)

<img src="https://img-blog.csdnimg.cn/20210629143940232.png">

<img src="https://img-blog.csdnimg.cn/20210629144013339.png">

#### Butterworth lowpass filters (巴特沃斯低通滤波器)

<img src="https://img-blog.csdnimg.cn/20210629144115517.png">

<img src="https://img-blog.csdnimg.cn/20210629144141403.png">

#### Gaussian lowpass filters (高斯低通滤波器)

<img src="https://img-blog.csdnimg.cn/20210629144231223.png">

<img src="https://img-blog.csdnimg.cn/20210629144304278.png">

### High-pass filtering (高通滤波器)

#### Ideal highpass filters (理想高通滤波器)

<img src="https://img-blog.csdnimg.cn/20210629144444982.png">

<img src="https://img-blog.csdnimg.cn/20210629144511267.png">

#### Butterworth highpass filters (巴特沃斯高通滤波器)

<img src="https://img-blog.csdnimg.cn/20210629144558939.png">

<img src="https://img-blog.csdnimg.cn/20210629144624469.png">

#### Gaussian highpass filters (高斯高通滤波器)

<img src="https://img-blog.csdnimg.cn/20210629144659924.png">

<img src="https://img-blog.csdnimg.cn/20210629144854316.png">

### Homomorphic Filtering (同态滤波)

#### 同态滤波流程

An image f(x, y) can be expressed as the product of 

illumination i(x, y) 

and 

reflectance components r(x, y):

<img src="https://img-blog.csdnimg.cn/2021062914560259.png">

<img src="https://img-blog.csdnimg.cn/20210629145343962.png">

#### Gray-level range compression (灰度范围压缩)

#### Contrast enhancement (对比度增强)

## Morphological image processing（形态学图像处理）

### Dilation (膨胀)

在B相对于自身原点的反射对这些元素移位操作的结果与A至少重叠一个元素，B的原点位置的集合。

<img src="https://img-blog.csdnimg.cn/20210629131220812.png">

### Erosion (腐蚀)

在B完全包括在A中时，B的原点位置的集合。

<img src="https://img-blog.csdnimg.cn/20210629131029843.png">

### Opening (开)

复合运算：先腐蚀后膨胀。

<img src="https://img-blog.csdnimg.cn/20210629131601431.png">

### Closing  (闭)

复合运算：先膨胀后腐蚀。

<img src="https://img-blog.csdnimg.cn/20210629131635639.png">

### Hit-or-miss (击中与否)

<img src="https://img-blog.csdnimg.cn/20210629132222221.png">

## Image segmentation（图像分割）

### Discontinuity（间断性）

#### Edge detection（边缘检测）

##### Laplacian of Gaussian（LOG算子）

<img src="https://img-blog.csdnimg.cn/20210629115447698.png">

<img src="https://img-blog.csdnimg.cn/20210629115512268.png">

##### The Canny edge detector (Canny算子)

算法的基本步骤：
  1）用Gaussian滤波器对图像进行卷积
  2）计算图像梯度的幅值和方向
  3）对梯度幅值应用非极大值抑制（置零）
 4）用双阈值处理和连接分析来检测并连接边缘

##### Edge linking (边缘连接)

##### Hough transform (霍夫变换)

通过Hough变换可以把在图像空间中的直线检测问题转换到在参数空间的点检测问题。其算法步骤如下：
（1）在ρ、θ 的取值范围内对其分别进行m，n等分，设一个二维数组的下标与ρi 、 θj的取值对应；
（2）对图像上的边缘点作Hough变换，求每个点在θj (j＝0,1,…,n)对应的ρi (i=0,1,…,m)，判断(ρi ,θj )与哪个数组元素对应．则让该数组元素值加1；
（3）比较数组元素值的大小，最大值所对应的（ ρi 、 θj ）就是这些共线点对应的直线方程的参数。

检测到的直线方程为：

<img src="https://img-blog.csdnimg.cn/2021062912062728.png">

### Similarity（相似性）

#### Thresholding（阈值法）

##### Basic global thresholding（基本全局阈值法）

（1）选取一个初始估计值T；
（2）用T分割图像。这样便会生成两组像素集合：G1由所有灰度值大于T的像素组成，而G2由所有灰度值小于或等于T的像素组成。
（3）对G1和G2中所有像素计算平均灰度值u1和u2。
（4）计算新的阈值：T=(u1 + u2)/2。
（5）重复步骤（2）到（4），直到得到的T值之差小于一个事先定义的参数T0。

##### Otsu’s method（Otsu法）

（1）先计算图像的归一化直方图
（2） i表示分类的阈值，也即一个灰度级，从0开始迭代
（3通过归一化的直方图，统计0~i 灰度级的像素(背景像素) 所占整幅图像的比例w0，并统计背景像素的平均灰度u0；统计i~255灰度级的像素(前景像素) 所占整幅图像的比例w1，并统计前景像素的平均灰度u1；
（4）计算前景像素和背景像素的方差 g = w0*w1*(u0-u1) (u0-u1)
（5） i++，直到i为255时循环结束
（6）将最大g相应的i值作为图像的全局阈值

##### Basic adaptive thresholding（基本自适应阈值法）

#### Region-based segmentation（基于区域的分割）

##### Region growing（区域生长）

1）根据图像的不同应用选择一组或多组开始点，把靠近这些点族中心的像素作为种子点（通常采用腐蚀算法）
2）选择一个相似性准则。
（灰度级、 色彩、 纹理、 梯度等特性相似）
3）从该种子开始向外扩张，不断将与集合中各个像素连通、且满足相似性准则的像素加入集合。
4）上一过程进行到不再有满足条件的新结点加入集合为止。（终止准则）

##### Region splitting and mergin（区域分裂与合并）

## Image feature description（图像特征描述）

### Shape feature（形状特征）

#### Euler number（欧拉数）

#### Convexity and concavity（凹凸性）

#### Region measurement（区域测量）

- Area S

- Perimeter L

- Length and Width

- Minimum Enclosing Rectangle（最小外接矩形）

   Rectangle Degree (矩形度) R = S / S(MER)

- Roundness (圆形度)
  
   R0 = 4πS / L²
  
- Shape Complexity (形状复杂度)

   e =  L² / S

#### skeletonization (骨架化)

- thinning algorithm (细化算法)

  假设区域内点的值为1，背景点的值为0。由两个基本操作组成。

  1.对于满足以下四个条件的边界点打标记准备删除：
  (a)2≤N(p1)≤6  (N(p1)=p2+p3+…+p9，是点p1邻域中1的个数)
  (b)S(p1)=1    (S(p1)是按p2,p3,…,p9顺序，0-1转换的个数)
  (c)p2 * p4 * p6 = 0    （p2 、p4 、p6 至少有一个0）
  (d)p4 * p6 * p8 = 0    （p4 、p6 、p8 至少有一个0）

  2.对于满足以下四个条件的边界点打标记准备删除：

  条件(a)、(b)与1相同，条件(c)、(d)改为：
  (c’) p2* p4* p8= 0 （p2 、p4 、p8 至少有一个0）
  (d’) p2* p6* p8= 0（p2 、p6 、p8 至少有一个0）

#### Moment method (矩量法)

- Invariant moment (不变矩)

#### Projection (投影)

<img src="https://img-blog.csdnimg.cn/20210629002244776.png">

<img src="https://img-blog.csdnimg.cn/20210629002312284.png">

#### Shape descriptor (形状描述子)

- Chain code (链码)

  <img src="https://img-blog.csdnimg.cn/20210628230738596.png" alt="schematic diagram">

  - 链码旋转归一化得到差分码
  - 差分码起点归一化得到最小差分码（形状数）
- shape number (形状数)
- Fourier descriptor (傅里叶描述子)

### Color feature（颜色特征）

#### Color Histogram (颜色直方图)

<img src="https://img-blog.csdnimg.cn/20210628234440533.png">

<img src="https://img-blog.csdnimg.cn/20210628234511119.png">

- L1norm distance 

<img src='https://img-blog.csdnimg.cn/20210628234357804.png'>

- L2norm distance

<img src='https://img-blog.csdnimg.cn/2021062823471372.png'>

- histogram intersection (直方图交)

<img src='https://img-blog.csdnimg.cn/20210628234810144.png'>

#### Color Moment (颜色矩)

<img src="https://img-blog.csdnimg.cn/20210629001941115.png">

#### Color Set (颜色集)

<img src="https://img-blog.csdnimg.cn/20210628235137337.png">

### Texture feature（纹理特征）

#### Statistical texture (统计纹理)

- Space gray level dependence matrix/entropy（空间灰度共生矩阵/熵）

<img src="https://img-blog.csdnimg.cn/20210629000539776.png">

<img src="https://img-blog.csdnimg.cn/2021062900061649.png" alt="schematic diagram">

#### Structural texture (结构纹理)