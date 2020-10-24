---
title:  RPG Game开发日志(九)
date: 2020-10-23 18:13:57
categories: Logs
tags: UE_4.22 
---

涉及内容：任务完成的互动，混合骨骼动画，普通攻击三连制作等；

<!--more-->

# 涉及内容

素材来源：

虚幻商城[Infinity Blade:Adversaries]

虚幻商城[Infinity Blade:Effects]

## 优化开箱互动

1. 打开Object_Treasure事件图表，创建TimeLine [MoveToPlayer]，创建Float Track，作为金币移动至人物身上的Alpha通道（类指数曲线）;<img src='https://img-blog.csdnimg.cn/20201023182304187.png'>
2. 扩展自定义事件Event On Interact With功能，实现金币被角色吸引——产生马里奥金币音效的效果；<img src='https://img-blog.csdnimg.cn/20201023182357341.png'>

## 创建移动平台

1. 创建Acotr [BP_MovePlane]，添加一个StaticMesh,添加SM_Env_Ice_Cliffs；
2. 打开事件图表，创建TimeLine[Move]，创建Float Track，作为平台移动的Alpha通道（类正弦曲线）,勾选Loop模式；<img src='https://img-blog.csdnimg.cn/20201023183342627.png'>
3. 在Event BeginPlay实现平台相对于自身的偏移移动；<img src='https://img-blog.csdnimg.cn/20201023183559104.png'>

## 任务完成的互动

1. 打开S_SubGoalInfo，添加成员HasFollowingIndex?(Boolean);

2. 打开W_SubGoal，创建函数DisableButton，用于SelectButton按钮的disable,Done图标和SubGoalText文本的高亮显示；<img src='https://img-blog.csdnimg.cn/20201023184922881.png'>

3. 打开W_Quest,创建宏AnimateSubGoalWidget(Exec，Boolean)，将宏PlayHideAll中有关SubGoal的显示动画（Show/Text）整合进来，并扩展功能，使其在Quest完成时播放隐藏动画；<img src='https://img-blog.csdnimg.cn/20201023190840946.png'><img src='https://img-blog.csdnimg.cn/20201023190652645.png'>

4. 打开EvenGraph，添加自定义事件PlaySubGoal(Boolean)，调用宏AnimateSubGoalWidget(Boolean)，修改自定义事件PlayQuest()为PlayQuest(Boolean)，调用宏PlayHideAll(Boolean);<img src='https://img-blog.csdnimg.cn/20201023191105631.png'>

5. 打开函数GenerateSubGoal，当SubGoal循环添加完成时,调用自定义事件PlaySubGoal，播放相关动画；

6. 打开BP_MasterQuest，创建自定义事件UpdateCompleteSubGoal,创建变量CompletedSubGoalsInfo(S_SubGoalInfo)、CompletedSubGoalIndex(Integer),用于记录已完成的SubGoal信息和索引。当所有**已显示**的SubGoal都被标记为完成时，执行PlaySubGoal(false)动画，并判断是否仍有FollowingIndex。若有，则延迟1.5s后调用函数GenrateSubGoal和SelectSubGoal，再生SubGoal；<img src='https://img-blog.csdnimg.cn/20201023193814396.png'>

7. 创建函数CompletedSubGoal(Integer)，用于记录已完成的SubGoal信息和索引更新，并同步UI更新（调用函数DisableButton和自定义事件UpdateCompleteSubGoal)；<img src='https://img-blog.csdnimg.cn/20201023194332860.png'><img src='https://img-blog.csdnimg.cn/20201023194354363.png'>

8. 复制一份BP_FirstQuest，命名为BP_FirstQuestTest，修改index=3的SubGoal,勾选HasFollowingIndex?,添加FollowingSubGoalIndices;<img src='https://img-blog.csdnimg.cn/20201023195017556.png'>

9. 打开BP_Shibi事件图表，创建J键盘事件，测试任务完成时的互动效果；<img src='https://img-blog.csdnimg.cn/20201023194602440.png'>

## 普通攻击三连制作

1. 导入Shibi角色MeleeAttack动画（Air/B/C/D)，创建Montage，命名为Air_Melee、Melee_B、Melee_C、Melee_D；
2. 导入Shibi角色Attack音效（1/2/3/4）,创建Cue，命名为Shinbi_Effect_Attack，编辑蓝图实现4种攻击音效随机播放的效果；<img src='https://img-blog.csdnimg.cn/20201024173918860.png'>
3. 打开Montage[Air_Melee],添加Notify [Play Sound]、Skeleton Notify [Reset]；<img src='https://img-blog.csdnimg.cn/2020102417450352.png'>
4. 打开Montage[Melee_B],添加Notify [Play Sound]、Skeleton Notify [Save] [Reset],添加Curves [Body]；<img src='https://img-blog.csdnimg.cn/20201024174714903.png'>
5. 打开Montage[Melee_C],添加Notify [Play Sound]、Skeleton Notify [Save] [Reset],添加Curves [Body]；<img src='https://img-blog.csdnimg.cn/20201024174753486.png'>
6. 打开Montage[Melee_C],添加Notify [Play Sound]、Skeleton Notify [Save] [Reset],添加Curves [Body]；<img src='https://img-blog.csdnimg.cn/20201024174837728.png'>
7. 打开BP_Shibi，创建自定义事件MeleeAttack、SaveAttack、ResetComb;
8. 打开ABP_Shibi事件图表，AnimNotify_Save调用SaveAttack事件，AnimNotify_Reset调用ResetComb事件；<img src='https://img-blog.csdnimg.cn/20201024175438930.png'>
9. 返回BP_Shibi，完成自定义事件MeleeAttack、SaveAttack、ResetComb，SavaAttack用于记录当前普通攻击归属第几段，当角色停止普通攻击时，ResetComb将初始化Attack信息;<img src='https://img-blog.csdnimg.cn/20201024175625435.png'>
10. 创建Q键盘事件，关联普通攻击，调用MeleeAttack事件，并根据角色是否滞空播放相应的蒙太奇动画；<img src='https://img-blog.csdnimg.cn/20201024175856577.png'>

## 混合骨骼动画

1. 打开ABP_Shibi事件图表，完善动画更新事件，判断角色是否处于加速状态或FullBody状态；<img src='https://img-blog.csdnimg.cn/20201024180437185.png'>

2. 打开ABP_Shibi动画图表，添加按骨骼分层混合，以及按布尔值混合姿势。当角色处于加速状态并且Body Curve返回值为0时，输出按骨骼分层混合姿势，否则输出UpBody姿势；<img src='https://img-blog.csdnimg.cn/20201024181347328.png'>

   <img src='https://img-blog.csdnimg.cn/20201024180639713.png'>



# 最终效果展示

<img src='https://img-blog.csdnimg.cn/2020102417301295.gif'>

<img src='https://img-blog.csdnimg.cn/20201024173043229.gif'>