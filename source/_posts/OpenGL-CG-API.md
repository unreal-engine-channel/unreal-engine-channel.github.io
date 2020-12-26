---
title: OpenGL-CG-API
date: 2020-12-16 23:52:36
categories: References
tags: 
- Computer Graphics
- OpenGL
---

基于OpenGL绘制一个简单的三维场景，涉及场景建模、透视投影、光照和材质、纹理映射、*阴影处理、简单动画、鼠标键盘交互等技术内容。
<!--more-->
# 基于OpenGL绘制三维场景

## Models(建模)

### glu库中常用的模型生成函数

```c++
void gluSphere(GLUquadricObj* pobj,GLdouble radius,GLint silces,GLint stacks); // 球
void gluCylinder(GLUquadricObj* pobj,GLdouble baseRadius,GLdouble topRadius,GLint slices,GLint stacks); // 圆柱
void gluDisk(GLUquadricObj* pobj,GLdouble innerRadius,GLdouble outerRadius,GLint slices,GLint stacks); // 圆盘
void gluPartialDisk (GLUquadric *qobj, GLdouble innerRadius, GLdouble outerRadius, GLint slices, GLint loops, GLdouble startAngle, GLdouble sweepAngle); // 不完整圆盘
```

### glut库中常用的模型生成函数

#### 实心模型

```c++
void glutSolidCube(GLdouble size); // 实心立方体
void glutSolidSphere(GLdouble radius, GLint slices, GLint stacks); // 实心球
void glutSolidCone(GLdouble radius, GLdouble height, GLint slices, GLint stacks); // 实心圆锥体
void glutSolidTorus(GLdouble innerRadius, GLdouble outerRadius, GLint nsides, GLint rings); // 实心圆环
void glutSolidTeapot(GLdouble size); // 实心茶壶
void glutSolidTetrahedron(void); // 实心4面体
void glutSolidOctahedron(void); // 实心8面体
void glutSolidDodecahedron(GLdouble radius); // 实心12面体
void glutSolidIcosahedron(void); // 实心20面体
```

#### 线框模型

以glutWire前缀开头，同实心模型。

### 透视投影

#### 注册自适应窗口函数

```c++
glutReshapeFunc(void(*func)(int w,int h));
```

#### 改变窗口形状函数原型

```c++
/*
Parameters:
- func   处理改变窗口形状事件的函数名
- w,h	 窗口的宽高
*/
void (*func)(int w,int h);
```

#####  处理改变窗口形状事件的函数模板

```c++
void reshape(int w, int h)
{
    glViewport(0, 0, (GLsizei)w, (GLsizei)h);
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();	//在进行变换之前需要将当前矩阵转换为单位矩阵才能进行操作
    gluPerspective(45, (GLfloat)w / (GLfloat)h, 1, 1000);
    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();
}
```

## Lights(光照)

### 光源设置函数原型

#### [glLightf](https://docs.microsoft.com/en-us/windows/win32/opengl/gllightf)

```c++
/*
Parameters:
- light  灯的标识符。可能的指示灯数量取决于实现方式，但至少支持八个指示灯。
		 它们由GL_LIGHTi形式的符号名称标识，其中i是值：0到GL_MAX_LIGHTS-1。
- pname  光源参数
		 可选值：GL_SPOT_EXPONENT、GL_SPOT_CUTOFF、
		 		GL_CONSTANT_ATTENUATION、GL_LINEAR_ATTENUATION、GL_QUADRATIC_ATTENUATION
		 
- params  根据pname的值指定确切的参数值
*/
void glLightf(GLenum light,GLenum pname,GLfloat params);
```

##### 实例

###### 设置光源最大发散角

```c++
glLightf(GL_LIGHT0, GL_SPOT_CUTOFF, 30.0);
```

###### 设置光源的强度分布

```c++
glLightf(GL_LIGHT0, GL_SPOT_EXPONENT, 4.0);
```

###### 设置光源的衰减因子

```c++
/*
- [GL_CONSTANT_ATTENUATION] 	the constant factor
- [GL_LINEAR_ATTENUATION]		the linear factor multiplied by the distance between the 								 light and the vertex being lighted
- [GL_QUADRATIC_ATTENUATION]	the quadratic factor multiplied by the square of the same 								  distance
*/
glLightf(GL_LIGHT0, GL_LINEAR_ATTENUATION, 1.0);
```



#### [glLightfv](https://docs.microsoft.com/en-us/windows/win32/opengl/gllightfv)

```c++
/*
Parameters:
- light  灯的标识符。可能的指示灯数量取决于实现方式，但至少支持八个指示灯。
		 它们由GL_LIGHTi形式的符号名称标识，其中i是值：0到GL_MAX_LIGHTS-1。
- pname  光源参数
		 可选值：GL_AMBIENT、GL_DIFFUSE、GL_SPECULAR、GL_POSITION、
		 		GL_SPOT_DIRECTION、GL_SPOT_EXPONENT、GL_SPOT_CUTOFF、
		 		GL_CONSTANT_ATTENUATION、GL_LINEAR_ATTENUATION、GL_QUADRATIC_ATTENUATION
		 
- params  根据pname的值指定确切的参数值
*/
void glLightfv(GLenum light,GLenum pname,const GLfloat *params);
```

##### 实例

###### 环境光

```c++
GLfloat light0_ambient[] = { 0.2, 0.8, 0.8, 1.0 };
glLightfv(GL_LIGHT0, GL_AMBIENT, light0_ambient);
```

###### 漫反射光

```c++
GLfloat light0_diffuse[] = { 0.5, 0.5, 0.5, 1.0 };
glLightfv(GL_LIGHT0, GL_DIFFUSE, light0_diffuse);
```

###### 镜面反射光

```c++
GLfloat light0_specular[] = { 0.8, 0.8, 0.2, 1.0 };
glLightfv(GL_LIGHT0, GL_SPECULAR, light0_specular);
```

###### 平行光

```c++
GLfloat light0_position[] = { -3.0, -3.0, 3.0, 1.0 };
glLightfv(GL_LIGHT0, GL_POSITION, light0_position);
```

###### 聚光灯

```c++
GLfloat spot_direction[] = { 3.0,3.0,-3.0 };
glLightfv(GL_LIGHT0, GL_SPOT_DIRECTION, spot_direction);
```

#### [glLightModeli](https://docs.microsoft.com/en-us/windows/win32/opengl/gllightmodeli)

```c++
/*
Parameters:
- pname  光源参数
		 可选值：GL_LIGHT_MODEL_LOCAL_VIEWER
				GL_LIGHT_MODEL_TWO_SIDE
		 
- params  根据pname的值指定确切的参数值
*/
```
###### 设置视点为局部视点

```c++
glLightModeli(GL_LIGHT_MODEL_LOCAL_VIEWER, GL_TRUE); 
```

###### 设置多边形着色方式为双面着色

```c++
glLightModeli(GL_LIGHT_MODEL_TWO_SIDE, GL_TRUE);
```

#### [glLightModelfv](https://docs.microsoft.com/en-us/windows/win32/opengl/gllightmodelfv)

```c++
/*
Parameters:
- pname  光源参数
		 可选值：GL_LIGHT_MODEL_AMBIENT
		 		GL_LIGHT_MODEL_LOCAL_VIEWER
				GL_LIGHT_MODEL_TWO_SIDE
		 
- params  根据pname的值指定确切的参数值
*/
void glLightModelfv(GLenum pname,const GLfloat *params);
```

###### 开启全局环境光

```c++
GLfloat amb[] = { 0.2,0.3,0.1,1.0 };
glLightModelfv(GL_LIGHT_MODEL_AMBIENT, amb);
```

####  启用光照

```c++
/* 
If enabled, normal vectors specified with glNormal are scaled to unit length after transformation. 
*/
glEnable(GL_NORMALIZE);
/*
If enabled, use the current lighting parameters to compute the vertex color or index. If disabled, associate the current color or index with each vertex.
*/
glEnable(GL_LIGHTING);
/*
If enabled, include light i in the evaluation of the lighting equation.
*/
glEnable(GL_LIGHT0);
/*
If enabled, do depth comparisons and update the depth buffer. 
*/
glEnable(GL_DEPTH_TEST);
```



## Materials(材质)

### 材质设置函数原型

#### [glMaterialfv](https://docs.microsoft.com/en-us/windows/win32/opengl/glmaterialfv)

```c++
/*
Parameters:
- face  
		 可选值：GL_FRONT、GL_BACK、GL_FRONT_AND_BACK
		
- pname		材质参数
	 		可选值：GL_AMBIENT、GL_DIFFUSE、GL_SPECULAR
	 				GL_EMISSION、GL_SHININESS、
	 				GL_AMBIENT_AND_DIFFUSE、GL_COLOR_INDEXES	
- params  	根据pname的值指定确切的参数值
*/
void glMaterialfv(GLenum face,GLenum pname,const GLfloat *params);
```

##### 说明

> （1）GL_AMBIENT、GL_DIFFUSE、GL_SPECULAR属性。这三个属性与光源的三个对应属性类似，每一属性都由四个值组成。GL_AMBIENT表示各种光线照射到该材质上，经过很多次反射后最终遗留在环境中的光线强度（颜色）。GL_DIFFUSE表示光线照射到该材质上，经过漫反射后形成的光线强度（颜色）。GL_SPECULAR表示光线照射到该材质上，经过镜面反射后形成的光线强度（颜色）。通常，GL_AMBIENT和GL_DIFFUSE都取相同的值，可以达到比较真实的效果。使用GL_AMBIENT_AND_DIFFUSE可以同时设置GL_AMBIENT和GL_DIFFUSE属性。
> （2）GL_SHININESS属性。该属性只有一个值，称为“镜面指数”，取值范围是0到128。该值越小，表示材质越粗糙，点光源发射的光线照射到上面，也可以产生较大的亮点。该值越大，表示材质越类似于镜面，光源照射到上面后，产生较小的亮点。
> （3）GL_EMISSION属性。该属性由四个值组成，表示一种颜色。OpenGL认为该材质本身就微微的向外发射光线，以至于眼睛感觉到它有这样的颜色，但这光线又比较微弱，以至于不会影响到其它物体的颜色。
> （4）GL_COLOR_INDEXES属性。该属性仅在颜色索引模式下使用，由于颜色索引模式下的光照比RGBA模式要复杂，并且使用范围较小，这里不做讨论。

##### 实例

###### 遗留环境光

```c++
GLfloat mat_ambient[] = { 0.8, 0.0, 0.0, 1.0 }; 
glMaterialfv(GL_FRONT, GL_AMBIENT, mat_ambient);
```

###### 遗留漫反射光

```c++
GLfloat mat_diffuse[] = { 0.0, 0.8, 0.0, 1.0 };
glMaterialfv(GL_FRONT, GL_DIFFUSE, mat_diffuse);
```

###### 遗留镜面光

```c++
GLfloat mat_shininess[] = { 50.0 }; 
glMaterialfv(GL_FRONT, GL_SPECULAR, mat_specular);
```

###### 设置镜面指数

```c++
GLfloat mat_specular[] = { 0.0, 0.0, 0.8, 1.0 }; 
glMaterialfv(GL_FRONT, GL_SHININESS, mat_shininess);
```

## Textures(纹理)

### 二维纹理

#### 指定二维纹理

##### 函数原型

```c++
/*
Parameters:
- target 目标纹理
		 唯一可选值：GL_TEXTURE_2D
- level  表示纹理多分辨率层数，通常取值为0，表示只有一种分辨率
		 可选值：0~n
- internalformat 表示纹理元素中存储的哪些分量（RGBA颜色、深度等）在纹理映射中被使用，
				 1表示使用R颜色分量，2表示使用R和A颜色分量，3表示使用RGB颜色分量，4表示使用RGBA颜色分量
				 可选值：1~4以及多种符号常量(如GL_RGBA)				  
- width,height	纹理图像的宽高
- border 		纹理边框的宽度
				可选值：0,1
- format		指定图像的数据格式
				可选值：GL_COLOR_INDEX、GL_RED、GL_GREEN、GL_BLUE、GL_ALPHA、GL_RGB、GL_RGBA、
				      GL_BGR_EXT、GL_BGRA_EXT、GL_LUMINANCE、GL_LUMINANCE_ALPHA
- type			指定图像的数据类型
				可选值：GL_UNSIGNED_BYTE、GL_BYTE、GL_BITMAP、GL_UNSIGNED_SHORT、
					   GL_SHORT、GL_UNSIGNED_INT、GL_INT、and GL_FLOAT
- pixels	    指向内存中指定的纹理图像数据
*/
void glTexImage2D(GLenum target,GLint level,GLint internalformat,GLsizei width,
GLsizei height,GLint border,GLint format,GLenum type,const GLvoid *pixels);
```
启用二维纹理：
```c++
/* If enabled, two-dimensional texturing is performed.  */
glEnable(GL_TEXTURE_2D);
```

 在启用纹理之后，需要建立物体表面上点与纹理空间的对应关系，即在绘制基本图元时，在glVertex函数调用之前调用glTexCoord函数，明确指定当前顶点所对应的纹理坐标：

```c++
glBegin(GL_QUADS);  
   glTexCoord2f(0.0,0.0);glVertex3f(1.0,2.5,1.5);
   glTexCoord2f(0.0,0.6);glVertex3f(1.0,3.7,1.5);
   glTexCoord2f(0.8,0.6);glVertex3f(2.0,3.7,1.5);
   glTexCoord2f(0.8,0.0);glVertex3f(2.0,2.5,1.5);
glEnd();
```

其图元内部点的纹理坐标利用顶点处的纹理坐标采用线性插值的方法计算出来。

#### 设置纹理环绕和纹理滤波

##### 函数原型

```c++
/*
Parameters:
- target 目标纹理
		 可选值：GL_TEXTURE_1D、GL_TEXTURE_2D
- pname	 纹理参数
		 可选值：GL_TEXTURE_MIN_FILTER、GL_TEXTURE_MAG_FILTER、
		 		GL_TEXTURE_WRAP_S、GL_TEXTURE_WRAP_T
- param  根据pname的值指定确切的参数值
*/
void glTexParameterf(GLenum target,GLenum pname,GLfloat param);
void glTexParameteri(GLenum target,GLenum pname,GLfloat param);
```


在OpenGL中，纹理坐标的范围被指定在[0,1]之间，而在使用映射函数进行纹理坐标计算时，有可能得到不在[0,1]之间的坐标。此时OpenGL有两种处理方式，一种是截断，另一种是重复，它们被称为环绕模式:

- GL_CLAMP	将大于1.0的纹理坐标设置为1.0，将小于0.0的纹理坐标设置为0.0
- GL_REPEAT   如果纹理坐标不在[0,1]之间，则将纹理坐标值的整数部分舍弃，只使用小数部分，这样使纹理图像在物体表面重复出现

```c++
glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP);
glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
```

在变换和纹理映射后，屏幕上的一个像素可能对应纹理元素的一小部分（放大），也可能对应大量的处理元素（缩小）。在OpenGL中，允许指定多种方式来决定如何完成像素与纹理元素对应的计算方法（滤波）:

- GL_NEAREST	取比较接近的那个像素
- GL_LINEAR       以周围四个像素的加权平均值做为纹理

```c++
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
```

#### 设置纹理调和

##### 函数原型

```c++
/*
Parameters:
- target 目标纹理
		 唯一可选值：GL_TEXTURE_ENV
- pname	 纹理参数
		 唯一可选值：GL_TEXTURE_ENV_MODE
- param  根据pname的值指定确切的参数值
		 可选值：GL_MODULATE、GL_DECAL、GL_BLEND、GL_REPLACE
*/
void glTexEnvf(GLenum target,GLenum pname,GLfloat param);
```

- Modulate：把表面的颜色与纹理颜色相乘
- Decal：若纹理中包含Alpha值，则用它与表面的颜色进行混合
- Replace： 把表面的颜色替换为纹理的颜色

#### 使用纹理坐标

##### 函数原型

```c++
/*
Parameters:
- coord 纹理坐标
		 可选值：GL_S、GL_T、GL_R、GL_Q
- pname	 纹理坐标生成函数标识符
		 唯一可选值：GL_TEXTURE_GEN_MODE
- param  纹理生成参数
		 可选值：GL_OBJECT_LINEAR、GL_EYE_LINEAR、GL_SPHERE_MAP
*/
void glTexGeni(GLenum coord,GLenum pname,GLint param);

/*
Parameters:
- coord 纹理坐标
		 可选值：GL_S、GL_T、GL_R、GL_Q
- pname	 纹理坐标生成函数标识符
		 可选值：GL_OBJECT_LINEAR、GL_EYE_LINEAR、GL_SPHERE_MAP
- param  包含相应纹理生成函数的浮点数组
*/
void WINAPI glTexGenfv(GLenum coord,GLenum pname,const GLfloat *params);
```

##### 实例

```c++
GLfloat planes[] = { 0.5,0.0,0.0,0.5 };
GLfloat planet[] = { 0.0,0.5,0.0,0.5 };
glTexGeni(GL_S, GL_TEXTURE_GEN_MODE, GL_OBJECT_LINEAR);
glTexGeni(GL_T, GL_TEXTURE_GEN_MODE, GL_OBJECT_LINEAR);
glTexGenfv(GL_S, GL_OBJECT_LINEAR, planes);
glTexGenfv(GL_T, GL_OBJECT_LINEAR, planet);
glEnable(GL_TEXTURE_GEN_S);
glEnable(GL_TEXTURE_GEN_T);
```

#### 纹理导入

##### 读取bmp位图

```c++
static AUX_RGBImageRec* LoadBMP(CHAR* Filename)
{
	FILE* File = NULL;
	if (!Filename)
	{
		return NULL;
	}
	File = fopen(Filename, "r");
	if (File)
	{
		fclose(File);
		return auxDIBImageLoadA(Filename);
	}
	return NULL;
}
```

##### 从bmp位图加载纹理

```c++
GLuint texture[6];
GLsizei textureSize = 6;
static int LoadGLTextures(CHAR* Filename, GLuint ID)
{
	int Status = FALSE;
	AUX_RGBImageRec* TextureImage[1];
	memset(TextureImage, 0, sizeof(void*) * 1);
	if (TextureImage[0] = LoadBMP(Filename))
	{
		Status = TRUE;
		glBindTexture(GL_TEXTURE_2D, texture[ID]);
		/*	glBindTexture(GL_TEXTURE_2D, ID);*/
		glTexImage2D(GL_TEXTURE_2D, 0, 3, TextureImage[0]->sizeX, TextureImage[0]->sizeY, 0, GL_RGB, GL_UNSIGNED_BYTE, TextureImage[0]->data);
		glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
		glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
	}

	if (TextureImage[0])
	{
		if (TextureImage[0]->data)
		{
			free(TextureImage[0]->data);
		}
		free(TextureImage[0]);
	}
	return Status;
}
```



### 三维纹理

## Animations(动画)

#### 注册闲置响应函数

```c++
void glutIdleFunc(void(*func)(void));
```

#### 注册指定时间响应函数

```c++
void  glutTimerFunc(unsigned int msecs, void (*func)(int value), int value); 
```

#### 闲置响应回调函数(以旋转为例)

```c++
void (*func)(void) {
        spin = spin + 2.0;
    if (spin > 360.0)
        spin = spin - 360.0;
    glutPostRedisplay();
}
```

## Shaders(着色器)

### 数据类型

- 标量：float、int和bool

- 矢量：vec2、vec3、vec4

  ​			ivec2、ivec3、ivec4

  ​			bvec2、bvec3、bvec4

- 矩阵：mat2、mat3、mat4

### 修饰符

- const

  变量值

- attribute

  只能出现在vertex shader中，只能修饰float类型的变量

- uniform

  从Host Language将我们需要的变量传递到GLSL中，主要出现在顶点（vertex shader）和片元（fragment shader）着色器

- varying

  用来传递顶点着色器（vertex shader）中的数据到片元（fragment shader）着色器中

### 顶点程序

<img src='https://img-blog.csdnimg.cn/20201216173103310.png'>

顶点程序都必须执行的一项任务就是把输入顶点程序的顶点位置从对象坐标系变换到裁剪坐标系。

#### 直通顶点程序

```glsl
void main(void)
{
 	gl_Position = gl_ProjectionMatrix * (gl_ModelViewMatrix * gl_Vertex);
}
```



### 片元程序

<img src='https://img-blog.csdnimg.cn/20201216173128560.png'>

片元程序在光栅器之后执行，它们处理的对象是每个片元而不是每个顶点。

#### 直通片元程序

```glsl
void main(void)
{
	gl_FragColor = gl_Color;
}
```



### 着色器与OpenGL的连接

#### 读取GLSL源文件

```c++
/* create a null-terminated string by reading the provided file */
static char* readShaderSource(const char* shaderFile)
{
	FILE* fp = fopen(shaderFile, "rb");
	char* buf;
	long size;

	if (fp == NULL) return NULL;
	fseek(fp, 0L, SEEK_END);
	size = ftell(fp);
	fseek(fp, 0L, SEEK_SET);

	buf = (char*)malloc((size + 1) * sizeof(char));
	fread(buf, 1, size, fp);
	buf[size] = '\0';
	fclose(fp);
	return buf;
}
```

#### 初始化GLSL

```c++
GLuint program;
GLuint vxParam, vyParam, timeParam;
/* GLSL initialization */
static void initShader(const GLchar* vShaderFile, const GLchar* fShaderFile)
{
	GLint status;
	GLchar* vSource, * fSource;
	GLuint vShader, fShader;

	/* read shader files */
	vSource = readShaderSource(vShaderFile);
	if (vSource == NULL)
	{
		printf("Failed to read vertex shaderi\n");
		exit(EXIT_FAILURE);
	}

	fSource = readShaderSource(fShaderFile);
	if (fSource == NULL)
	{
		printf("Failed to read fragment shader");
		exit(EXIT_FAILURE);
	}

	/* create program and shader objects */
	glewInit();
	vShader = glCreateShader(GL_VERTEX_SHADER);
	fShader = glCreateShader(GL_FRAGMENT_SHADER);
	program = glCreateProgram();

	/* attach shaders to the program object */
	glAttachShader(program, vShader);
	glAttachShader(program, fShader);

	/* read shaders */
	glShaderSource(vShader, 1, (const GLchar**)&vSource, NULL);
	glShaderSource(fShader, 1, (const GLchar**)&fSource, NULL);

	/* compile shaders */
	glCompileShader(vShader);
	glCompileShader(fShader);

	/* error check */
	glGetShaderiv(vShader, GL_COMPILE_STATUS, &status);
	checkError(status, "Failed to compile the vertex shader.");

	glGetShaderiv(fShader, GL_COMPILE_STATUS, &status);
	checkError(status, "Failed to compile the fragment shader.");

	/* link */
	glLinkProgram(program);
	glGetProgramiv(program, GL_LINK_STATUS, &status);
	checkError(status, "Failed to link the shader program object.");

	/* use program object */
	glUseProgram(program);

	/* set up uniform parameter */
	timeParam = glGetUniformLocation(program, "time");
	vxParam = glGetAttribLocation(program, "vx");
	vyParam = glGetAttribLocation(program, "vy");
}
```

#### 打印错误信息

```c++
/* error printing function */
static void checkError(GLint status, const char* msg)
{
	if (status == GL_FALSE)
	{
		printf("%s\n", msg);
		exit(EXIT_FAILURE);
	}
}
```

## Interaction(交互)

### 鼠标交互

#### 注册鼠标交互函数
```c++
// 注册鼠标点击事件
void glutMouseFunc(void(*func)(int button,int state,int x,int y));
// 注册鼠标移动事件
void glutMotionFunc(void(*func)(int x,int y)); // 鼠标移动并且点击
void glutPassiveMotionFunc(void (*func)(int x,int y));// 鼠标移动 
// 注册鼠标窗口事件
void glutEntryFunc(void(*func)(int state));   
```

#### 鼠标交互函数原型

```c++
/*
Parameters:
- func   处理鼠标click事件的函数名
- button 表明哪个鼠标键被按下或松开
		 可选值: GLUT_LEFT_BUTTON
                GLUT_MIDDLE_BUTTON
                GLUT_RIGHT_BUTTON
- state  表明鼠标键是按下还是松开
		 可选值: GLUT_DOWN
				GLUT_UP
- x,y	 表明鼠标当前的窗口坐标（原点为左上角）
*/
void (*func)(int button,int state,int x,int y);

/*
Parameters:
- func 	 处理各自类型motion的函数名
- x,y	 表明鼠标当前的窗口坐标（原点为左上角）
*/
void (*func)(int x,int y);

/*
Parameters:
- func   处理这些事件的函数名
- state  表明是离开还是进入窗口
		 可选值：GLUT_LEFT
                GLUT_ENTERED
*/
void(*func)(int state);   
```
##### 处理鼠标点击事件的函数模板
```c++
void (*func)(int button, int state, int x, int y)
{
    switch (button) {
    case GLUT_LEFT_BUTTON:
        if (state == GLUT_DOWN)
            void glutIdleFunc(void(*func)(void));
        break;
    case GLUT_MIDDLE_BUTTON:
    	if (state == GLUT_DOWN)
            void glutIdleFunc(void(*func)(void));
    case GLUT_RIGHT_BUTTON:
        if (state == GLUT_DOWN)
            void glutIdleFunc(void(*func)(void));
        break;
    default:
        break;
    }
}
```

### 键盘交互

#### 注册键盘交互函数

```c++
// 注册键盘事件
void glutKeyboardFunc(void(*func)(unsigned char key,int x,int y));
```

#### 键盘交互函数原型

```c++
/*
Parameters:
- func   处理键盘事件的函数名
- key    表明按下的键的ASCII码
		 可选值: 
                GLUT_KEY_F1               F1 function key
                GLUT_KEY_F2               F2 function key
                GLUT_KEY_F3               F3 function key
                GLUT_KEY_F4               F4 function key
                GLUT_KEY_F5               F5 function key
                GLUT_KEY_F6               F6 function key
                GLUT_KEY_F7               F7 function key
                GLUT_KEY_F8               F8 function key
                GLUT_KEY_F9               F9 function key
                GLUT_KEY_F10              F10 function key
                GLUT_KEY_F11              F11 function key
                GLUT_KEY_F12              F12 function key
                GLUT_KEY_LEFT             Left function key
                GLUT_KEY_RIGHT            Up function key
                GLUT_KEY_UP               Right function key
                GLUT_KEY_DOWN             Down function key
                GLUT_KEY_PAGE_UP          Page Up function key
                GLUT_KEY_PAGE_DOWN        Page Down function key
                GLUT_KEY_HOME             Home function key
                GLUT_KEY_END              End function key
                GLUT_KEY_INSERT           Insert function key
               	此外，也可直接输入数字或字符（如27、‘w’)
- x,y	 表明键盘按下时鼠标的窗口坐标（原点为左上角）
*/
void(*func)(unsigned char key,int x,int y);
```

##### 处理键盘事件的函数模板

```c++
void(*func)(unsigned char key,int x,int y){
    switch (key) {
       	case 97:
 		case 'a':
        case GLUT_KEY_LEFT:
			// do something
            break;
        case 100:
 		case 'd':    
        case GLUT_KEY_RIGHT:
            // do something
            break;
        case 119:
 		case 'w':
        case GLUT_KEY_UP:
			// do something
            break;
        case 115:
        case 's':
        case GLUT_KEY_DOWN:
            // do something
            break;
        case 27:
            exit(0);
            break;
    }
}
```

