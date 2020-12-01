---
title: Graphic-Drawing-System
date: 2020-11-27 21:16:36
categories: Theories
tags: 
- Computer Graphics
- OpenGL
mathjax: true
---

图形绘制系统的内容包括Modeling(建模)、Geometric processing(几何处理)、Rasterization(光栅化)、Fragment processing(片元处理)、Frame buffer(帧缓存)。其中四个主要任务为：Clip(裁剪)、Rasterization(光栅化)、Hidden surface(隐藏面消除)、Anti-aliasing technology(反走样技术)。

<!--more-->

# Modeling(建模)

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

## Clip(裁剪)

### 直线裁剪算法

#### Cohen-Sutherland算法

##### 建立区域码

<img src='https://img-blog.csdnimg.cn/20201128143628195.png'>

Cohen-Sutherland直线裁剪算法的核心是把所有直线的端点均分配一个表示其相对位置的4位二进制代码。此代码称为区域码。区域码按照端点与窗口边界的相对位置编码，即区域码的4位分别代表端点位于窗口的上、下、右、左。

<img src='https://img-blog.csdnimg.cn/20201128145429326.png'>

Cohen-Sutherland算法通常在两种情况下，可以快速的检测。即平凡接受（两个码字都是0000）和平凡拒绝（两个码字在某一位元素上都是1(相与不为0)）。

<img src='https://img-blog.csdnimg.cn/2020112815010542.png'>

<img src='https://img-blog.csdnimg.cn/20201128150125553.png'>

对于第三种情况，取窗口外的一个端点与窗口边界比较以确定可排除直线的哪一部分，然后，把直线剩下的部分与其他边界比较，这样一直到直线全部被排除或确定直线的哪一部分在窗口之内为止。可按“左、右、下、上”的次序建立检查直线端点与窗口边界关系的算法。

##### 代码实现(C++)

```c++
/*
Implementation of Cohen-Sutherland Algorithm, written by Jocoboy.
*/
struct Point{
    int x;
    int y;
    string code;
    friend bool ordinaryAccept(const Point& p1,cont Point& p2){
        return p1.code == "0000" && p2.code == "0000";
    }
    friend bool ordinaryRefused(const Point& p1,cont Point& p2){
        for(int i = 0 ; i < 4 ; i++){
            if(p1.code[i]== p2.code[i] && p,.code[i] == '1'){
            	return true;
        	}
        }
     	return false;
    }
}

struct RealRect{
    int left;
    int top;
    int right;
    int bottom;
    void leftClip(Point& p1,Point& p2){
        int e = this.left - p1.x;				
        int delx = p2.x - p1.x;
        int dely = p1.y - p2.y; // 此处p1和p2顺序无所谓
        if(dely!=0){
   			int d = e*dely/delx;
            p1.y += -d;	
        }
        p1.x = this.left;	
    }
    void rightClip(Point& p1,Point& p2){
        int e = p1.x - this.right;				
        int delx = p1.x - p2.x;
        int dely = p1.y - p2.y; // 此处p1和p2顺序无所谓
        if(dely!=0){
             int d = e*dely/delx;
             p1.y += -d;	
        }
        p1.x = this.right;		 
    }
    void bottomClip(Point& p1, Point& p2){
        int d = this.bottom - p1.y;				
        int delx = p2.x - p1.x; // 此处p1和p2顺序无所谓
        int dely = p2.y - p1.y;
        if(delx!=0){
              int e = d*delx/dely;
           	  p1.x += -e;	
         }
         p1.y = this.bottom;		 
    }
    void topClip(Point& p1，Point& p2){
        int d = p1.y - this.top;				
        int delx = p2.x - p1.x; // 此处p1和p2顺序无所谓
        int dely = p1.y - p2.y;
        if(delx!=0){
              int e = d*delx/dely;
              p1.x += -e;	
        }
        p1.y = this.top;	
    }
}

int clipSegment(Point& p1, Point& p2, RealRect w)
{
   do {
   	if(ordinaryAccept(p1,p2)) return 1;// 平凡接受(完全可见)
   	if(ordinaryRefused(p1,p2)) return 0;// 平凡拒绝(完全不可见)
   	if(p2.x>=w.left&&p2.x<=w.right&&p2.y>=w.bottom&&p2.y<=w.top){// p1在窗口外面
        if(p1.x<=w.left){// p1在窗口左边,用左边界截断，更新p1； 	    
	  		w.leftClip(p1,p2);  
        } 			 
    	else if(p1.x>=w.right){// p1在窗口右边,用右边界截断，更新p1；
            w.rightClip(p1,p2);		  	   
        }
    	else if(p1.y<=w.bottom){// p1 在窗口下面,用下边界截断，更新p1；
            w.bottomClip(P1,P2); 				
        }
    	else if(p1.y>=w.top){// p1 在窗口上面,用上边界截断，更新p1；
            w.topClip(p1,p2);			
        } 
    }
	else{  // p2 在窗口外面
        if(p2.x<=w.left){// p2在窗口左边,用左边界截断，更新p2；    	    
           w.leftClip(p2,p1);   					   
        } 			 
    	else if(p2.x>=w.right){// p2在窗口右边,用右边界截断，更新p2；
           w.rightClip(p2,p1);	 				  	  
        }
    	else if(p2.y<=w.bottom){// p2 在窗口下面,用下边界截断，更新p2；
           w.bottomClip(P2,P1);   				
        }
    	else if(p2.y>=w.top){// p2 在窗口上面,用上边界截断，更新p2；
           w.topClip(p2,p1);	  				
        } 
   }while(1);
}
```



#### Liang-Barsky算法

一个复杂的画面中可能包含有几千条直线，为了提高算法效率，加快裁剪速度，应当采用计算量较小的算法求直线与窗口边界的交点。

##### 基本思想

1. Liang-Barsky算法又称为参数方程法，以直线的参数方程为基础，对不同情况下的裁剪求得相应的参数值。

2. 首先写出端点及之间连线的参数方程如下：
   $$
   x = x_1 + (x_2-x_1)u = x_1 + t\Delta x
   \\
   y = y_1 + (y_2-y_1)u = y_1 + t\Delta y
   \\
   \Delta x = x_2 - x_1,\Delta y = y_2 - y_1
   $$
     参数u可取0到1之间的值，坐标表示此范围内的u值定义的直线上的一个点。
   
3. 如果直线上的某点处于(x,y)即(xmin,ymin)以及(xmax,ymax)所定义的窗口之内，则满足以下条件：

$$
x_{wmin} \leqslant x_1 + t\Delta x  \leqslant x_{wmax}
\\
y_{wmin} \leqslant y_1 + t\Delta y  \leqslant y_{wmax}
$$

##### 算法步骤

1. 输入直线段的两端点坐标以及窗口的四条边界坐标。

2. 若Δx=0，则有
   $$
   x_{wmin} \leqslant x_1 \leqslant x_{wmax}
   $$
   

   进一步判断是否满足
   $$
   x_1 < x_{wmin}或x_1 > x_{wmax}
   $$
   若满足，则该直线段不在窗口内，转(7)。

   否则，满足
   $$
   x_{wmin} \leqslant x_1   \leqslant x_{wmax}
   $$
   则进一步计算ts和te。转(5)。

3. 若Δy=0，则有
   $$
   y_{wmin} \leqslant y_1 \leqslant y_{wmax}
   $$
   进一步判断是否满足
   $$
   y_1 < y_{wmin}或y_1 > y_{wmax}
   $$
   若满足，则该直线段不在窗口内，转(7)。

   否则，满足
   $$
   y_{wmin} \leqslant y_1   \leqslant y_{wmax}
   $$
   则进一步计算ts和te。转(5)。

4. 若上述两条均不满足,则有Δx≠0且Δy≠0。此时计算ts和te。

5. 求得ts和te后进行判断：

   若ts>te,则直线段在窗口外,转(7)。

   若ts<te,利用直线的参数方程求得直线段在窗口内的两端点坐标。

6. 利用直线的扫描转换算法绘制在窗口内的直线段。

7. 算法结束。

### 多边形裁剪算法

#### Sutherland-Hodgeman算法

##### 基本思想(分冶)

1. 将多边形的边界作为一个整体，每次用窗口的一条边界对要裁剪的多边形进行裁剪。

2. 窗口的一条边以及延长线构成的裁剪线把平面分为两个区域：包含窗口区域的域称为可见侧，不包含窗口区域的域为不可见侧。

3. 将每条线段的端点S, P与裁剪线比较之后，可以输出0～2个点:

   (1) S在不可见一侧，P在可见一侧，输出SP与裁剪线的交点I和顶点P。

   (2) S, P都在可见一侧，输出顶点P。

   (3) S在可见一侧，P在不可见一侧，输出SP与裁剪线的交点I。

   (4) S, P都在不可见一侧，输出0个顶点。

   

<img src='https://img-blog.csdnimg.cn/20201128233030391.png'>

##### 举例

<img src='https://img-blog.csdnimg.cn/20201128232040829.png'>

(a)用左边界裁剪

​	输入顶点：ABCDEFGH

​	输出顶点：12DEFGHA

<img src='https://img-blog.csdnimg.cn/20201128232219190.png'>



(b)用下边界裁剪

​	输入顶点：12DEFGHA

​	输出顶点：34D56FFGHA1

<img src='https://img-blog.csdnimg.cn/2020112823231526.png'>

(c)用右边界裁剪

​	输入顶点：34D56FFGHA1

​	输出顶点：78GHA134D56

<img src='https://img-blog.csdnimg.cn/20201128232408110.png'>

(d)用上边界裁剪

​	输入顶点：78GHA134D56

​	输出顶点：9IHJK34D5678

# Rasterization(光栅化)

## 线段光栅化

### DDA算法

#### 基本思想

1. DDA算法是计算机图形学中最简单的绘制直线算法。其主要思想是由直线公式y = kx + b推导出来的。 
   我们已知直线段两个端点P0(x0,y0)和P1(x1,y1)，就能求出 k 和 b 。

2. 在k，b均求出的条件下，只要知道一个x值，我们就能计算出一个y值。如果x的步进为1（x每次加1，即x = x +1），那么y的步进就为k+b；同样知道一个y值也能计算出x值，此时y的步进为1，x的步进为(1-b)/k。根据计算出的x值和y值，向下取整，得到坐标(x’,y’)，并在(x’,y’)处绘制直线段上的一点。

3. 为进一步简化计算，通常可令b取0，将起点看作(0,0)。设当前点为(xi, yi)则用DDA算法求解(xi+1，yi+1)的计算公式可以概括为：

   $$
   x_{i+1} = x_i + x_{step}
   \\
   y_{i+1} = y_i + y_{step}
   $$

4. 我们一般通过计算 Δx 和 Δy 来确定xStep和yStep：

   如果 |Δx| > |Δy| ，即k=|Δy/Δx| <1，说明x轴方向为步进的主方向，xStep = 1，yStep = k；
   如果 |Δy|> |Δx|，即k=|Δy/Δx| >1，说明y轴方向为步进的主方向，yStep = 1，xStep = 1 / k。

5. 根据这个公式，就能通过(xi，yi)迭代计算出(xi+1,yi+1)，然后在坐标系中绘制计算出的(x,y)坐标点。

#### 代码实现(C++)

```c++
/*
Implementation of DDA Algorithm, written by Jocoboy.
*/
struct Point{
    int x;
    int y;
    friend void drawPixel(int x,int y);
}
void drawDDALine(Point p1,Point p2){
    float dx = p2.x - p1.x;
    float dy = p2.y - p1.y;
    float k = dy / dx;
    if(fabs(k) == 1){
        for(int x = p1.x,y = p1.y ; x <= p2.x ; x++,y++){
            drawPixel(x,y);
        }
    }
    else if(fabs(k) < 1){
        for(int x = p1.x ; x <= p2.x ; x++){
            int y = (int)((x+1)*k + 0.5);
            drawPixel(x,y);
        }
    }
    else{
          for(int y = p1.y ; y <= p2.y ; y++){
            int x = (int)((y+1)*k + 0.5);
            drawPixel(x,y);
        }
    }
}
```



#### 例题

> 画出直线段P0(0,0)-P1(5,2)

> 由k = Δy/Δx  = 2 / 5 < 1， 可知x轴方向为步进的主方向，xStep = 1，yStep = k
>
> |  x   |  y   | yStep |
> | :--: | :--: | :---: |
> |  0   |  0   |   0   |
> |  1   |  0   |  0.4  |
> |  2   |  1   |  0.8  |
> |  3   |  1   |  1.2  |
> |  4   |  2   |  1.6  |
> |  5   |  2   |  2.0  |
>
> <img src='https://img-blog.csdnimg.cn/20201130191413305.png'>

### Bresenham算法

#### 基本思想

1. Bresenham算法也是一种计算机图形学中常见的绘制直线的算法，其本质思想也是步进的思想，但由于避免了浮点运算，相当于DDA算法的一种改进算法。

2. 设直线的斜率为k，当|k| <=1时，x方向为主步进方向；当|k| >1时，y方向为主步进方向。现以|k| <1时为例，推导Bresenham算法的原理。

   <img src='https://img-blog.csdnimg.cn/20201130194214901.png'>

3. 图中绘制了一条直线，红色点表示该直线上的点，蓝色和绿色点表示光栅下绘制的点。 当x方向是主要步进方向时，以每一小格的中点为界，如果当前的yi在中点(图中黑色短线)下方，则y取yi; 如果当前的yi在中点上方，则y取yi+1。

4. 现考虑这种方法的误差，因为直线的起始点在像素中心，所以误差项d的初值d0＝0。x下标每增加1，d的值相应递增。直线的斜率值k，即d＝d＋k。一旦d≥1，就把它减去1，这样保证d在[0,1]之间。当d<0.5时，更接近于右方像素(x+1,y）;当d≥0.5时，更接近于当前像素的右上方像素（x+1,y+1）。为方便计算，令e＝d-0.5，e的初值为-0.5，增量为k。当e<0时，取当前像素（xi，yi）右方像素（x+1,y）；当e≥0时，取当前像素（xi，yi）的右上方像素（x+1,y+1），从而可以改用整数以避免除法。

5. 由于算法中只用到误差项的符号，因此可作如下替换：

$$
\left\{
\begin{aligned}
e = k - 0.5 \\Δy=kΔx\end
{aligned}
\right.
\\
令e' = 2e*Δx = 2(k-0.5)Δx = 2Δy - Δx
$$



#### 代码实现(C++)

```c++
/*
Implementation of Bresenham Algorithm, written by Jocoboy.
*/
struct Point{
    int x;
    int y;
    friend void drawPixel(int x,int y);
}
void drawDDALine(Point p1,Point p2){
    float dx = p2.x - p1.x;
    float dy = p2.y - p1.y;
    float k = dy / dx;
	if(fabs(k) == 1){
        for(int x = p1.x,y = p1.y ; x <= p2.x ; x++,y++){
            drawPixel(x,y);
        }
    }
    else if(fabs(k) < 1){
        int e = -dx;
        for(int x = p1.x ; x <= p2.x ; x++){
            e += 2*dy;
            if(e > 0){
                y++;
                e -= 2*dx;
            }
            drawPixel(x,y);
        }
    }
    else{
          int e = -dy;
          for(int y = p1.y ; y <= p2.y ; y++){
            e += 2*dx;
            if(e > 0){
                x++;
                e -= 2*dy;
            }
            drawPixel(x,y);
        }
    }
}
```



## 多边形光栅化

#### 内外测试法

##### 奇偶规则

<img src='https://img-blog.csdnimg.cn/20201201120558130.png'>

1. 从任意位置p作一条射线（注意不要经过多边形的顶点）；
2. 若与该射线相交的多边形边的数目为奇数，则p是多边形内部点，否则是外部点。

##### 非零环绕数规则（Nonzero Winding Number Rule）

<img src='https://img-blog.csdnimg.cn/20201201120641458.png'><img src='https://img-blog.csdnimg.cn/20201201120704886.png'>

1. 从任意位置p作一条射线（注意不要经过多边形的顶点）；
2. 将多边形的边矢量化，当从p点沿射线方向移动时，对在每个方向上穿过射线的边计数；
3. 每当多边形的边从右到左穿过射线时，环绕数加1，从左到右时，环绕数减1；
4. 若环绕数为非零，则p为内部点，否则，p是外部点。

#### 扫描线算法

##### 基本思想

用水平扫描线从上到下（或从下到上）扫描由多条首尾相连的线段构成的多边形，每根扫描线与多边形的某些边产生一系列交点。将这些交点按照x坐标排序，将排序后的点两两成对，作为线段的两个端点，以所填的颜色画水平直线。多边形被扫描完毕后，颜色填充也就完成了。

##### 算法步骤

1. 求交，计算扫描线与多边形的交点；

2. 交点排序，对第2步得到的交点按照x值从小到大进行排序；

3. 颜色填充，对排序后的交点两两组成一个水平线段，以画线段的方式进行颜色填充；

4. 是否完成多边形扫描？如果是就结束算法，如果不是就改变扫描线，然后转第1步继续处理。

##### 算法分析

 整个算法的关键是第1步，需要用尽量少的计算量求出交点，还要考虑交点是线段端点的特殊情况，最后，交点的步进计算最好是整数，便于光栅设备输出显示。对于每一条扫描线，如果每次都按照正常的线段求交算法进行计算，则计算量大，而且效率低下。观察多边形与扫描线的交点情况，可以得到以下两个特点：

1. 每次只有相关的几条边可能与扫描线有交点，不必对所有的边进行求交计算；
2. 相邻的扫描线与同一直线段的交点存在步进关系，这个关系与直线段所在直线的斜率有关；

<img src='https://img-blog.csdnimg.cn/20201201123507744.png'>

第一个特点是显而易见的，为了减少计算量，扫描线算法需要维护一张由活动边组成的表，称为活动边表（Active Edge Table，又称有效边表，以下简称AET）。例如扫描线4的AET由P1P2和P3P4两条边组成，而扫描线7的AET由P1P2、P6P1、P5P6和P4P5四条边组成。

  第二个特点可以进一步证明，假设当前扫描线与多边形的某一条边的交点已经通过直线段求交算法计算出来，得到交点的坐标为（x, y），则下一条扫描线与这条边的交点不需要再求交计算，通过步进关系可以直接得到新交点坐标为（x + △x, y + 1）。前面提到过，步进关系△x是个常量，与直线的斜率有关，下面就来推导这个△x。

  假设多边形某条边所在的直线方程是：ax + by + c = 0，扫描线yi和下一条扫描线yi+1与该边的两个交点分别是（xi，yi）和（xi+1，yi+1），则可得到以下两个等式：
$$
\left\{
\begin{aligned}
ax_i+by_i+c = 0 \\ax_{i+1}+by_{i+1}+c = 0\end
{aligned}
\right.
$$
求解xi，xi+1可得：
$$
\left\{
\begin{aligned}
x_i = -(by_i+c)/a \\x_{i+1} = -(by_{i+1}+c)/a \end
{aligned}
\right.
$$
两式相减，可得：
$$
x_{i+1}-x_i=-b(y_{i+1}-y_i)/a
$$
由于扫描线存在yi+1 = yi + 1的关系，代入上式可得：
$$
x_{i+1}-x_i = -b/a
$$
即△x = -b / a，是个常量（直线斜率的倒数）。

AET是扫描线填充算法的核心，整个算法都是围绕者这张表进行处理的。要完整的定义AET，需要先定义边的数据结构。每条边都和扫描线有个交点，扫描线填充算法只关注交点的x坐标。每当处理下一条扫描线时，根据△x直接计算出新扫描线与边的交点x坐标，可以避免复杂的求交计算。一条边不会一直待在AET中，当扫描线与之没有交点时，要将其从AET中删除，判断是否有交点的依据就是看扫描线y是否大于这条边两个端点的y坐标值，为此，需要记录边的y坐标的最大值。根据以上分析，边的数据结构可以定义如下：

```cpp
struct ETNode{
    double xi;
    double dx;
    int ymax;
    ETNode* next;
}
```

根据ETNode的定义，扫描线4和扫描线7的AET如下所示：

<img src='https://img-blog.csdnimg.cn/20201201130026187.png'>

<img src='https://img-blog.csdnimg.cn/20201201130053238.png'>

 前面提到过，扫描线算法的核心就是围绕AET展开的，为了方便活性边表的建立与更新，我们为每一条扫描线建立一个边表（Edge Table，以下简称ET），存放该扫描线第一次出现的边。当算法处理到某条扫描线时，就将这条扫描线的ET中的所有边逐一插入到AET中。ET通常在算法开始时建立，建立ET的规则就是：如果某条边的较低端点（y坐标较小的那个点）的y坐标与扫描线y相等，则该边就是扫描线y的新边，应该加入扫描线y的ET。上例中各扫描线的ET如下图所示：

<img src='https://img-blog.csdnimg.cn/20201201130813624.png'>

##### AET边表算法步骤

1. 初始化：构造边表，AET表置空；
2. 将第一个不空的ET表中的边与AET表合并；
3. 由AET表中取出交点对进行填充。填充之后删除ymax=y的边；
4. yi+1=yi+1,根据xi+1=xi+1/k计算并修改AET表，同时合并ET表中y=yi+1桶中的边，按次序插入到AET表中，形成新的AET表；
5. AET表不为空则转(3)，否则结束。

##### 例题

> 如下图所示多边形，若采用ET边表算法进行填充，试写出该多边形的ET表和当扫描线Y=3时的AET表。
>
> <img src='https://img-blog.csdnimg.cn/20201201161023841.png'>

> 
>
> 边表节点ETNode形式如下：
>
> |  X   | Ymax | 1/k  | next |
> | :--: | :--: | :--: | :--: |
> |      |      |      |      |
>
> ET表：
>
> <img src='https://img-blog.csdnimg.cn/20201201162353749.png'>
>
> Y = 3时的AET表：
>
> <img src='https://img-blog.csdnimg.cn/20201201162419678.png'>
>
> 

#### 种子填充算法

种子填充是指从区域内的某一个象素点（种子点）开始，由内向外将填充色扩展到整个区域内的过程。

算法的输入：种子点坐标(x,y)，填充色以及边界颜色。

利用堆栈实现简单的种子填充算法:

​	算法从种子点开始检测相邻位置是否是边界颜色，若不是就用填充色着色，并检测该像素点的相邻位置，直到检测完区域边界颜色范围内的所有像素为止。

##### 区域连通方式

- 4-邻接点和8-邻接点

  <img src='https://img-blog.csdnimg.cn/20201201173807602.png'>

- 4连通种子填充算法的填充结果

  <img src='https://img-blog.csdnimg.cn/20201201171635347.png'>

- 8连通种子填充算法的填充结果

  <img src='https://img-blog.csdnimg.cn/20201201171659172.png'>


对于一个区域的内部点而言，如果能够通过4-邻接点完全遍历，也就能够通过8-邻接点完全遍历。

##### 算法步骤

栈结构实现4-连通种子填充算法的算法步骤为：

1. 种子象素入栈；
2. 当栈非空时重复执行如下三步操作：
   - 栈顶象素出栈；
   - 将出栈象素置成填充色；
   - 检查出栈象素的4-邻接点，若其中某个象素点不是边界色且未置成多边形色，则把该象素入栈。
3. 检查栈是否为空，若栈非空重复执行2，若栈为空则结束。

#### 扫描线种子填充算法

扫描线种子填充算法：

​	扫描线通过在任意不间断扫描线区间中只取一个种子像素的方法使堆栈的尺寸极小化。不间断区间是指在一条扫描线上的一组相邻像素。

##### 算法步骤

算法步骤为：

1. 种子象素入栈；
2. 当栈非空时重复执行如下三步操作：
   - 栈顶象素出栈；
   - 填充出栈像素所在扫描行的连续像素段，从出栈的像素开始沿扫描线向左和向右填充，直到遇到边界像素为止，并且记录下此时扫描线区间的x坐标范围；
   - 分别检查上下扫描线上位于上述x区间内的未被填充的连续水平像素段，将其最右像素取作种子像素压入堆栈。
3. 检查栈是否为空，若栈非空重复执行2，若栈为空则结束。

##### 举例

<img src='https://img-blog.csdnimg.cn/20201201182357394.png'>


# Fragment processing(片元处理)

# Frame buffer(帧缓存)

# Hidden surface(隐藏面消除)

## 深度缓存方法

1. 对于平面上每个像素P\[i][j]，深度缓存一个b比特的量d\[i][j]；
2. P\[i][j]保存屏幕上该点目前最近面片的颜色；
3. d\[i][j]保存屏幕上该点目前最近面片的伪深度;

### 伪代码

```
for（场景中的每一个多边形）
{
      扫描转换该多边形；
      for（多边形所覆盖的每一个像素点(x,y)）
     {
           计算多边形在该像素点的深度值z(x,y)；
           if（z(x,y) < Z-buf中对应此像素点(x,y)的z值）
           {
                 把多边形在(x,y)处的深度值z(x,y)存入Z-buf中的(x,y)处；
                 把多边形在(x,y)处的亮度值存入f-buf中的(x,y)处；
            }
      }
}

```

当所有的多边形都处理完后，帧缓冲器中的内容即为消除隐藏面后的图像。

### OpenGL实现隐藏面消除的相关API

- 初始化显示模式时指明使用深度缓存：glutInitDisplayMode(GLUT_DEPTH|GLUT_RGB);
- 启用深度测试[glEnable(GL_DEPTH_TEST)](https://docs.microsoft.com/en-us/windows/win32/opengl/glenable);
- 渲染每一幅图像之前要初始化深度缓存：[glClear(GL_COLOR_BUFFER_BIT|GL_DEPTH_BUFFER_BIT)](https://docs.microsoft.com/en-us/windows/win32/opengl/glclear);
- 关闭深度测试[glDisable(GL_DEPTH_TEST)](https://docs.microsoft.com/en-us/windows/win32/opengl/gldisable); 
- 深度测试范围：[glDepthRange (zNear, zFar)](https://docs.microsoft.com/en-us/windows/win32/opengl/gldepthrange);
- 选择比较测试的方法：[glDepthFunc(func)](https://docs.microsoft.com/en-us/windows/win32/opengl/gldepthfunc);

# Anti-aliasing technology(反走样技术)

用离散量表示连续量引起的失真，就叫做走样（Aliasing）。用于减少或消除这种现象效果的技术，称为反走样（Antialiasing）

<img src='https://img-blog.csdnimg.cn/20201201193529109.png'>

产生原因：
    数学意义上的图形是由无线多个连续的、面积为零的点构成；但在光栅显示器上，用有限多个离散的，具有一定面积的象素来近似地表示他们。

## Super sampling(过取样)

高分辨率计算，低分辨率显示。在高于显示分辨率的较高分辨率下用点取样方法计算，然后对几个象素的属性进行平均得到较低分辨率下的象素属性。

### 简单过取样

在x，y方向把分辨率都提高一倍，使每个象素对应4个子象素，然后扫描转换求得各子象素的颜色亮度，再对4个象素的颜色亮度进行平均，得到较低分辨率下的象素颜色亮度。

简单的过取样方式：

<img src='https://img-blog.csdnimg.cn/20201201194654625.png'>

### 基于加权模板的过取样

简单过取样在确定像素的亮度时，仅仅是对所有子像素的亮度进行简单的平均。更常见的做法是给接近像素中心的子像素赋予较大的权值，即对所有子像素的亮度进行加权平均。

常用的加权模板：

<img src='https://img-blog.csdnimg.cn/20201201194609603.png'>

## Area sampling(区域取样)

在整个像素区域内进行采样，这种技术称为区域取样。像素的亮度是作为一个整体被确定的，不需要划分子像素。

根据相交的面积值决定像素显示的亮度级别：

<img src='https://img-blog.csdnimg.cn/20201201194753455.png'>

## OpenGL反走样技术的相关API

- 启用反走样：[glEnable(primitiveType)](https://docs.microsoft.com/en-us/windows/win32/opengl/glenable);

- 启用OpenGL颜色混和并指定颜色混合函数： 

  [glEnable(GL_BLEND)](https://docs.microsoft.com/en-us/windows/win32/opengl/glenable);
  [glBlendFunc(GL_SCR_ALPHA, GL_ONE_MINUS_SRC_ALPHA)](https://docs.microsoft.com/en-us/windows/win32/opengl/glblendfunc);