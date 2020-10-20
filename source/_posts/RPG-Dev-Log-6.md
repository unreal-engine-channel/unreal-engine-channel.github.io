---
title: RPG Game开发日志(六)
date: 2020-10-20 13:33:47
categories: Logs
tags: UE_4.22 
---

涉及内容：创建NPC，骨骼重定向，任务互动发放等；

<!--more-->

# 涉及内容

素材来源：

虚幻商城[Generic NPC Anim Pack]

虚幻商城[Infinity Blade:Warriors]

虚幻商城[Advanced Locomotion System]

内置Package[Mannequin]添加组件S

Adobe官网[Mixamo](https://www.mixamo.com/)

[阿里巴巴矢量图表库](https://www.iconfont.cn/) [关键词：圆、问号]

## 创建NPC

1. 打开BP_RPG_Character，删除组件MapArm及其附属组件ChildActor;
2. 以BP_RPG_Character为父类，创建蓝图BP_RPG_NPC;
3. 下载地图指示图标，右键Apply Paper2D Texture Settings,并Create Sprite。将圆形图标添加到NPC的PaperSprite；
4. 为BP_RPG_NPC添加组件SpringArm及其附属组件ChildActor,ChildActor Class选取MiniCapture;
5. 打开PaperSprite，取消勾选Owner No See。并打开事件图表，添加隐藏PaperSprite的相关蓝图；<img src='https://img-blog.csdnimg.cn/20201020140456848.png'>
6. 以BP_RPG_NPC为父类，创建蓝图BP_NPC_Warriors,将SK_CharM_Pit导入进来；

## NPC骨骼重定向

### 基础篇

1. 打开SK_CharM_Pit的Retarget Manager,在SetupRig项目选择引擎默认的Humanoid人形骨骼；
2. 点击AutoMap，自动匹配和建立骨骼映射，ShowBase/ShowAdvanced可用于查看基本骨骼和精细骨骼映射关系；
3. 打开UE4_Mannequin_Skeleton，同样选取rig为Humanoid人形骨骼，自动建立骨骼映射，在ManageRetargetSource项目Add重定向源SK_Mannequin;
4. 右键ThirdPerson_AnimBP,点击Retarget Anim Blueprints，Source为UE4_Mannequin_Skeleton，Target为SK_Mannequin_Skeleton，并确保base pose一致（Here is Apos)；<img src='https://img-blog.csdnimg.cn/20201020143635944.png'>
5. 将重定向后生成的动画添加至NPC蓝图即可；

### 进阶篇（Mixamo)

1. 进入虚幻商城[Advanced Locomotion System]，可下载Tpos模型[ALS_Mannequin_T_Pose]；
2. 打开ALS_Mannequin_T_Pose，选取CurrentPose创建PoseAssets[UE4_PoseAsset];
3. 打开Apos的小白人骨骼模型，在Manage Retarget Base Pose项目，点击Modify,选择UE4_PoseAsset为当前pose,并import进来；
4. 打开Mixamo官网下载的模型，手动建立骨骼映射；
5. 右键ThirdPerson_AnimBP,点击Retarget Anim Blueprints，Source为UE4_Mannequin_Skeleton，Target为maria_j_j_ong_Skeleton，并确保base pose一致（Here is Tpos)；<img src='https://img-blog.csdnimg.cn/20201020150458513.png'>

## 任务互动发放

1. 创建Widget[W_NameForNPC]，

   层级面板<img src='https://img-blog.csdnimg.cn/2020102015102250.png'>,

   预览效果<img src='https://img-blog.csdnimg.cn/20201020151116574.png'>

2. 打开父类BP_RPG_Character，添加组件Widget，并在子类BP_RPG_NPC中添加Widget Class为W_NameForNPC，设置Space=Screen;

3. 添加Sphere碰撞组件，设置碰撞预设;<img src='https://img-blog.csdnimg.cn/20201020151809375.png'>

4. 添加NPC互动效果蓝图 <img src='https://img-blog.csdnimg.cn/20201020152018538.png'>
5. 打开Char_Shibi,创建事件调度OnInteract，将与E键盘事件关联；
6. 打开BP_RPG_NPC,创建自定义事件OnInteract，用于判断角色与NPC交互时，NPC身上是否有未分发的任务<img src='https://img-blog.csdnimg.cn/20201020152724167.png'>
7. 打开Char_Shibi，创建事件调度UpdateLevelForQuest(Integer),在UpLevel之后调用；
8. 打开BP_RPG_NPC,创建变量QuestInLevel(Map)，将Level映射到Quest类。创建自定义事件UpdateQuest(Integer),用于在角色升级后，判断NPC身上是否产生新的任务；<img src='https://img-blog.csdnimg.cn/20201020153139435.png'>
9. 复制BP_MasterQuest两份，命名为BP_FirstEventQuest和BP_FirstSideQuest，并修改任务信息；

## UI动画添加
1. 打开W_Quest，为Border创建HideAll动画；<img src='https://img-blog.csdnimg.cn/20201020154300252.png'>
2. 为Text创建Text动画；<img src='https://img-blog.csdnimg.cn/20201020154331675.png'>
3. 打开W_SubGoal，为Border创建Show动画；<img src='https://img-blog.csdnimg.cn/20201020154514839.png'>
4. 打开W_Quest事件蓝图，添加宏[Exec] PlayHideAll(Exec,Boolean)，组织和关联HideAll、Text、Show动画执行时机<img src='https://img-blog.csdnimg.cn/20201020154615593.png'>
5. 创建自定义事件PlayQuest调用PlayHideAll，并在UpdateQuest末尾添加调用;

# 最终效果展示

<img src='https://img-blog.csdnimg.cn/20201020134241578.gif'>