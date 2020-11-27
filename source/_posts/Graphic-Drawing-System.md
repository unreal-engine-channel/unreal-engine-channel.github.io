---
title: Graphic-Drawing-System
date: 2020-11-27 21:16:36
categories: Theories
tags: 
- Computer Graphics
- OpenGL
mathjax: true
---

图形绘制系统的主要内容：Modeling(建模)、Geometric processing(几何处理)、Rasterization(光栅化)、Fragment processing(片元处理)、Frame buffer(帧缓存)。

<!--more-->

# Geometric processing(几何处理)

<img src='https://img-blog.csdnimg.cn/20201127213955149.png'>

> - 完整的图形绘制管道
>   1. 通过给P添加分量1将其扩展为齐次4元组；
>   2. 用这个4元组乘上矩阵modelview，得到视点坐标系下的4元组；
>   3. 再将点乘上投影矩阵，得到裁剪坐标下的4元组；
>   4. 对以此点为端点的边实施裁剪操作；
>   5. 执行透视除法，得到一个3元组（规范化的设备坐标）；
>   6. 在视区变换过程中3元组被乘上一个矩阵：得到的结果被用于绘制和深度计算（窗口坐标）。
>
> - 从三维空间到二维平面，就如同用相机拍照一样，通常都要经历以下几个步骤（括号内表示的是相应的图形学概念） 
>
>   1. 将相机置于三角架上，让它对准三维景物（视点变换，Viewing Transformation）； 
>
>   2. 将三维物体放在适当的位置（模型变换，Modeling Transformation ）； 
>
>   3. 选择相机镜头并调焦，使三维物体投影在二维胶片上（投影变换，Projection Transformation ）;
>
>   4. 决定二维像片的大小（视口变换，Viewport Transformation ）。 
>
>   这样，一个三维空间里的物体就可以用相应的二维平面物体表示了，也就能在二维的计算机屏幕上正确显示了。

## Viewing & Modeling  Transformation(视点&模型变换)

### 摄像机的定位和定向

#### 定义

<img src='https://img-blog.csdnimg.cn/20201128001507408.png'>

给定eye、look和up，我们得到：

<img src='https://img-blog.csdnimg.cn/20201128002206685.png'>



#### Modelview matrix(模型视点矩阵)

<img src='https://img-blog.csdnimg.cn/20201128013208174.png'>

<img src='https://img-blog.csdnimg.cn/20201128013227487.png'>

模型视点矩阵的作用是把世界坐标系转换成摄像机坐标系。

#### 例题

> 考虑一个摄像机，其eye=(4,4,4)，look=(0,1,0)，up为(0,1,0)，求出u，v，n。

> n=(4,4,4)-(0,1,0)=(4,3,4)
> $$
> \left[
>  \begin{matrix}
>    i & j & k \\
>    0 & 1 & 0 \\
>    4 & 3 & 4
>   \end{matrix}
>   \right]
> $$
> u=(0,1,0)×(4,3,4)=(4,0,-4)
> $$
> \left[
>  \begin{matrix}
>    i & j & k \\
>    4 & 3 & 4 \\
>    4 & 0 & -4
>   \end{matrix}
>   \right]
> $$
> v=(4,3,4)×(4,0,-4)=(-12,32,-12)

#### OpenGL视点变换API

- [gluLookAt(GLdouble eyex,   GLdouble eyey,   GLdouble eyez,   GLdouble centerx,   GLdouble centery,   GLdouble centerz,   GLdouble upx,   GLdouble upy,   GLdouble upz)](https://docs.microsoft.com/en-us/windows/win32/opengl/glulookat)

## Projection Transformation(投影变换)

### 平行投影(正交投影)

#### 平行投影矩阵

<img src='https://img-blog.csdnimg.cn/20201128020858774.png'>

[推导过程](https://www.cnblogs.com/leixinyue/p/11166135.html)

#### OpenGL平行投影API

- [glOrtho(GLdouble left,   GLdouble right,   GLdouble bottom,   GLdouble top,   GLdouble zNear,   GLdouble zFar)](https://docs.microsoft.com/en-us/windows/win32/opengl/glortho)

- [gluOrtho2D(GLdouble left,   GLdouble right,   GLdouble top,   GLdouble bottom)](https://docs.microsoft.com/en-us/windows/win32/opengl/gluortho2d)

### 透视投影

#### 透视除法

<img src='https://img-blog.csdnimg.cn/20201128020312218.png'>

<img src='https://img-blog.csdnimg.cn/20201128020336728.png'>

在Eye coordinates(观察坐标）被Projection matrix相乘后，得到的Clip coordinates(裁剪坐标)仍然是homogeneous coordinates(齐次坐标)。最终它需要除以Clip coordinates的w分量，才能变成Normalized device coordinates(规范化设备坐标NDC)。因此，我们可以把裁剪坐标的Wc分量设置为-Ze，则投影矩阵第4行变为(0, 0, -1, 0)。

#### 透视投影矩阵

<img src='https://img-blog.csdnimg.cn/20201128020621398.png'>

[推导过程](https://www.cnblogs.com/leixinyue/p/11166135.html)

#### OpenGL透视投影API

- [glFrustum(GLdouble left,   GLdouble right,   GLdouble bottom,   GLdouble top,   GLdouble zNear,   GLdouble zFar)](https://docs.microsoft.com/en-us/windows/win32/opengl/glfrustum)
- [gluPerspective(GLdouble fovy,   GLdouble aspect,   GLdouble zNear,   GLdouble zFar)](https://docs.microsoft.com/en-us/windows/win32/opengl/gluperspective)

## Viewport Transformation(视口变换)

<img src='https://img-blog.csdnimg.cn/20201127214343608.png'>

### 仿射变换

#### 特点

1. 仿射变换可以很容易地对物体实施缩放、旋转和平移操作
2. 一系列仿射变换可以组合成一个单一的仿射变换
3. 仿射变换可以使用一个紧凑的矩阵形式表示

#### 基本变换

- 平移<img src='https://img-blog.csdnimg.cn/2020112723281810.png'>
- 缩放<img src='https://img-blog.csdnimg.cn/20201127232917795.png'>
- 旋转<img src='https://img-blog.csdnimg.cn/20201127234421587.png'><img src='https://img-blog.csdnimg.cn/20201127234441934.png'>

#### 例题

> 假设裁剪矩形的左下角坐标为（wxl=10,wyb=10），右上角坐标为（wxr=50，wyt=50）。
>
> 设备坐标系中视口的左下角坐标为（vxl=10,vyb=30），右上角坐标为（vxr=50,vyt=90）。
>
> 已知在窗口内有一点p(20,30)，要将点p映射到视区内的点p’，请问p’点在设备坐标系中的坐标是多少？

> 1. 将窗口坐标系（window coordinates）左下角（10,10）平移至观察坐标系的坐标原点（0,0），平移矢量为（-10，-10）。
>
> 2. 针对坐标原点进行比例变换，使窗口大小和视区相等，比例因子为 Sx=(50-10)/(50-10)=1， Sy=(90-30)/(50-10)=1.5。
>
> 3. 将窗口内的点映射到设备坐标系（device coordinates）的视区中，再进行反平移，将视区的左下角移回到设备坐标系中的位置（10,30），平移矢量为（10,30）。
>
> 4. 计算得到变换矩阵T。
>    <img src='https://img-blog.csdnimg.cn/20201127225018489.png'>
>    <img src='https://img-blog.csdnimg.cn/20201127225157233.png'>
>    
> 5. 将变换矩阵T左乘p点齐次坐标得到p'齐次坐标。
>    
>    <img src='https://img-blog.csdnimg.cn/20201127234649217.png'>

### OpenGL视口变换API

- [glViewport(GLint x, GLint y , GLsizei width , GLsizei height)](https://docs.microsoft.com/en-us/windows/win32/opengl/glviewport)

- [glDepthRange(GLclampd zNear, GLclampd zFar )](https://docs.microsoft.com/en-us/windows/win32/opengl/gldepthrange)

# Rasterization(光栅化)

# Fragment processing(片元处理)

# Frame buffer(帧缓存)