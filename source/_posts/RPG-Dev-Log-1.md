---
title: RPG Game开发日志(一) 
date: 2020-10-11 11:53:42
categories: Logs
tags: UE_4.22 
---

涉及内容：简单场景创建；鼠标跟随贴花制作；动画蓝图制作；移动操控绑定等。
<!--more-->

# 涉及内容

## 简单场景创建

1. 清理默认物体，如Floor、Capsule等;
2. 添加Box Brush，Transform>Location=(0,0,0)，Brush Settings > X=5000,Y=5000,Z=800;
3. 快捷键ALT复制一个，Transform>Location(0,0,50)，Brush Settings > X=4800,Y=4800,Z=700, Brush Type=Subtractive;
4. 利用Geometry Editing创建几个测试平台、楼梯等；
5. 添加StarterContent初学者内容包，为场景物体添加材质。
6. 保存场景为Training至RPG_Game>Maps。

<img src="https://img-blog.csdnimg.cn/20201011122013690.png">

## 鼠标跟随贴花制作

1. 图标来源：[阿里巴巴矢量图标库](https://www.iconfont.cn/)  
2. 添加Goal图标至RPG_Game>Textures；
3. 创建材质球M_GoalDecal，设置Material Domain=Deferred Decal，Blend Mode=Translucent，Decal Blend Mode = Emissive；
4. 设置Material蓝图参数；<img src='https://img-blog.csdnimg.cn/20201011123828721.png'>
5. 创建材质实例Inst_GoalDecal，Sets the preview mesh to a plane primitive，修改公开变量Color和EmissiveValue(高光度)；
6. 以上M_GoalDecal、Inst_GoalDecal添加至RPG_Game>Materials；
7. 创建Actor BP_GoalDecal至RPG_Game>Actors，添加Decal组件和Box collision，在Event Graph 内添加Begin Overlap(Box)事件。<img src='https://img-blog.csdnimg.cn/20201011124835193.png'>

## 动画蓝图制作

1. 骨骼网格体、动画来源：虚幻商城[虚幻争霸：心菲]
2. 以下将会用到Skeletal Mesh (Shinbi)，Animation Sequence (Shibi_Fwd、Shibi_Idle);
3. 创建一维混合空间Shibi_BlendSpace1D，Horizontal Axis 命名为 Speed，区间设置为[0,400],0处添加Idle,400处添加Fwd;
4. 创建动画蓝图ABP_Shibi至RPG_Game>Characters>Animations，新建状态机:Entry->Idle_Move,新建变量:Speed,设置动画蓝图如下；<img src='https://img-blog.csdnimg.cn/20201011130431251.png'>
5. 进入Event Graph,设置动画蓝图事件如下。<img src='https://img-blog.csdnimg.cn/20201011130521861.png'>

## 移动操控绑定

1. 创建Character BP_RPG_Character至RPG_Game>Blueprints，并创建子类Char_Shibi;
2. 打开Char_Shibi,添加SpringArm和Camera(依附于SpringArm)，设置SpringArm Rotation=(0,-45,0),Target Arm Length=650;
3. 创建RPG_PlayerController至RPG_Game>Blueprints；
4. 创建RPG_Game_Mode至RPG_Game>Blueprints,设置Player Controller Class为RPG_PlayerController,Default Pawn Class为Char_Shibi;
5. 打开项目设置，设置Default GameMode 为RPG_GameMode,Default Maps为Training,添加轴映射MoveForward(W/S)、MoveRight(A/D)、LookUp(Mouse Y)、Turn(Mouse X);
6. 打开RPG_PlayerController,创建函数CancelMovementCommand,用于处理角色移动状态时销毁鼠标贴花；<img src='https://img-blog.csdnimg.cn/20201011132117897.png'>
7. Shibi Object获取；<img src='https://img-blog.csdnimg.cn/20201011132352696.png'>
8. 鼠标贴花生成；<img src='https://img-blog.csdnimg.cn/20201011132417698.png'>
9. 移动转向控制。<img src='https://img-blog.csdnimg.cn/20201011133109943.png'><img src='https://img-blog.csdnimg.cn/20201011133157497.png' ><img src='https://img-blog.csdnimg.cn/20201011133223590.png'><img src='https://img-blog.csdnimg.cn/2020101113325827.png'><img src='https://img-blog.csdnimg.cn/20201011133327798.png'>

# 最终效果展示

<img src='https://img-blog.csdnimg.cn/20201012174104608.png'>

<img src='https://img-blog.csdnimg.cn/20201012174324500.png'>