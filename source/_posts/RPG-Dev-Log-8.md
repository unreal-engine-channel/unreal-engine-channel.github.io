---
title: RPG Game开发日志(八)
date: 2020-10-21 16:47:22
categories: Logs
tags: UE_4.22 
---

涉及内容：任务坐标，Money互动等;

<!--more-->

# 涉及内容

素材来源：

虚幻商城[Content Examples]

[阿里巴巴矢量图表库](https://www.iconfont.cn/) [关键词：脚印、钱包]

## 创建搜寻目标

### 准备工作

1. 创建Interface [I_Interaction],添加函数BeginOverlapTargetObject、EndOverlapTargetObject、OnInteractWith(Char_Shibi);

2. 创建Actor [BP_TargetObject]，添加组件StaticMesh、Widget、PaperSprite，并在事件开始时隐藏PaperSprite;

3. 将I_Interaction蓝图接口添加至BP_TargetObject;

4. 打开S_SubGoalInfo，添加属性TargetClass(Actor)、TargetClassID(Integer);

5. 打开Char_Shibi,添加Box碰撞体，并添加碰撞事件，调用I_Interaction接口函数；<img src='https://img-blog.csdnimg.cn/20201021171021322.png'>

6. 为E键盘事件扩展功能，使其借助蓝图接口实现互动；<img src='https://img-blog.csdnimg.cn/20201021171350706.png'>

7. 重命名W_NameForNPC为W_Interaction，并添加到BP_TargetObject的Widget上，在事件开始运行时设置W_Interaction的Name和Interact的文本；<img src='https://img-blog.csdnimg.cn/20201021171832273.png'>

8. 打开BP_TargetObject事件蓝图，实现蓝图接口中三个接口函数;<img src='https://img-blog.csdnimg.cn/20201021172214999.png'>

### 创建目标实例(藏宝箱)

1. 以BP_TargetObject为父类，创建子类Object_Treasure；
2. 将[Content Examples]资源包中的StaticMesh [SM_Chest_Bottom、SM_Chest_Lid、SM_Coin_Small]添加进来，调整Transform;
3. 实现蓝图接口I_Interaction中的接口函数OnInteractWith，使用TimeLine实现宝箱打开、金币弹出并旋转的交互动画;<img src='https://img-blog.csdnimg.cn/20201021173246323.png'><img src='https://img-blog.csdnimg.cn/20201021173324992.png'>

### 创建目标小地图指示

1. 打开W_MiniMap，扩展Widget显示内容，注意**TargetDirection**属性**Pivot**的设置，需要根据Angle作调整,使其无论如何旋转，都能正确显示位置，层级面板作如下修改<img src='https://img-blog.csdnimg.cn/20201021173858989.png'>预览效果<img src='https://img-blog.csdnimg.cn/20201021174021728.png'>

2. 打开QuestManager，创建纯函数[void] DistanceToTarget(Integer),用于计算角色到目标的空间距离；<img src='https://img-blog.csdnimg.cn/20201021174336258.png'>

3. 创建函数[void] UpdateTargetDirection()，用于更新目标相对于角色的所在方位；<img src='https://img-blog.csdnimg.cn/2020102117461731.png'>

4. 打开函数AddQuest,在末尾设置CurrentQuest=LocalQuest;

5. 创建自定义事件OnSwitchSubGoalInfo，用于点击子任务列表时，根据当前选择的子任务（是否为Search任务），显示和隐藏目标距离和方位；<img src='https://img-blog.csdnimg.cn/20201021175129176.png'><img src='https://img-blog.csdnimg.cn/20201021175154953.png'>

6. 创建自定义事件OnPlayerMove，用于角色移动时更新搜寻目标距离和方位信息；<img src='https://img-blog.csdnimg.cn/202010211755540.png'>

7. 打开RPG_PlayerController，在移动事件末尾添加OnPlayerMove事件的调用；

## Money互动

1. 打开W_CharacterBorder，添加一个Horizatal Box，添加Money图标和文本；
2. 打开Char_Shibi，创建函数[void]CountMoney()，用于更新Money文本；<img src='https://img-blog.csdnimg.cn/20201021213916456.png'>
3. 创建自定义事件[void] GetMoney(Integer)，使用TimerHandle控制CountMoney函数的循环调用；<img src='https://img-blog.csdnimg.cn/20201021214258222.png'>
4. 打开Object_Treasure事件图表，在Event OnInteractWith末尾添加GetMoney事件的调用，可实现宝箱打开金币旋转动画结束后，自动增加Money的值；


# 最终效果展示

<img src='https://img-blog.csdnimg.cn/20201021165022723.gif'>