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

# Fragment processing(片元处理)

# Frame buffer(帧缓存)