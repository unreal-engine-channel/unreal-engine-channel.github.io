---
title: RPG Game开发日志(十五)
date: 2020-11-08 00:13:02
categories: Logs
tags: UE_4.22
---

涉及内容：大任务系统续，任务分支完成互动。

<!--more-->

# 涉及内容


## W_PC_Main相关

1. 创建动画HideAll，用于隐藏主界面的所有用户自定义控件；<img src='https://img-blog.csdnimg.cn/20201108003424947.png'>

2. 创建函数[Float] PlayHideAll(Boolean)，用于控制HideAll动画的播放；<img src='https://img-blog.csdnimg.cn/20201108004704213.png'>

   
## W_Mission相关

1. 界面优化：打开W_Mission，为任务列表栏和任务详情栏添加边框(Image/Border);<img src='https://img-blog.csdnimg.cn/20201108002312944.png'>
2. 控件添加：添加返回按钮，取消勾选Interaction>IsFocusable，并创建OnClick事件：点击按钮时隐藏任务系统界面并显示主界面；<img src='https://img-blog.csdnimg.cn/20201108004137729.png'><img src='https://img-blog.csdnimg.cn/20201108012623353.png'>
3. 动画添加：为SelectedBorder和CancelBorder添加ButtonPlay和ButtonCancel动画（两个动画除绑定对象不同外，其余都相同），并分别创建OnClick事件：点击SelectButton时播放ButtonPlay动画，调用SelectNewQuest事件，并隐藏SelectButton所在的HorizantalBox（使其无法再次被选择）。点击ButtonCancel时播放ButtonCancel动画。最后修改函数UpdateDetailWindow，比较SelectedQuest是否为CurrentQuest，若为真则隐藏SelectedHorizantalBox，否则显示SelectedHorizantalBox；<img src='https://img-blog.csdnimg.cn/20201108012156563.png'><img src='https://img-blog.csdnimg.cn/20201108014256728.png'><img src='https://img-blog.csdnimg.cn/2020110801435634.png'>
4. 函数修改：修改函数GenertateSubGoals，将参数HuntIndex传入W_MissionSubGoal；<img src='https://img-blog.csdnimg.cn/20201108231031870.png'>





## W_MissionSubGoal相关

1. 修改函数Update，更新Hunt类型分支任务的信息；<img src='https://img-blog.csdnimg.cn/20201108024335447.png'>


## W_Quest相关

1. 修改函数[void]SelectSubGoal(W_SubGoal)，当SelectedSubGoal与传入的SubGoal不同时，更新SelectedSubGoal;<img src='https://img-blog.csdnimg.cn/20201108021956595.png'>


## W_SubGoal相关

1. 控件添加：添加FailedImage，对应SuccessImage;<img src='https://img-blog.csdnimg.cn/20201108020420640.png'>
2. 函数功能扩展：修改函数[void]DisableButton()为[void]DisableButton(Success?)，当任务完成时显示SuccessImage，任务失败时显示FailedImage;<img src='https://img-blog.csdnimg.cn/2020110916482665.png'>
3. 函数功能扩展：修改函数Update，添加CurrentSubGoal索引查询，并设置为HuntedIndex。添加SubGoalState，并对其作一个Select，以更新SubGoal信息（注意，如果此处不作select，SubGoal当前击杀/寻找数会循环0,1,2）;<img src='https://img-blog.csdnimg.cn/20201108023744297.png'><img src='https://img-blog.csdnimg.cn/20201109164737918.png'>



## BP_MasterQuest相关

1. 创建函数[Boolean]SelectInMission()，用于判断SelectedQuest是否为CurrentQuest；<img src='https://img-blog.csdnimg.cn/20201108014943318.png'>
2. 修改函数SetupStartingSubGoals，根据StartSubGoalIndices的数组长度对CurrentHuntedAmount数组进行Resize;<img src='https://img-blog.csdnimg.cn/20201108015344389.png'>
3. 修改函数[void]CompleteSubGoal(SubGoalIndex，Success?)，将传入参数Success提升为本地变量，并传入函数DisableButton(Success?)；<img src='https://img-blog.csdnimg.cn/20201109165028882.png'><img src='https://img-blog.csdnimg.cn/20201109165053562.png'><img src='https://img-blog.csdnimg.cn/20201109165133515.png'><img src='https://img-blog.csdnimg.cn/20201109165224466.png'>
4. 修改自定义事件UpdateCompletedSubGoal，初始化CurrentHuntedAmout数组为0，修改SubGoal的延迟更新条件;<img src='https://img-blog.csdnimg.cn/2020110802323032.png'><img src='https://img-blog.csdnimg.cn/20201108023250768.png'>



## BP_Enemy相关

1. 打开事件图表，修改自定义事件OnDeath，调用QuestManager自定义事件OnEnemyKilled；<img src='https://img-blog.csdnimg.cn/20201108031744839.png'>



## BP_QuestTest相关

1. 打开测试专用Quest，创建Hunt型分支任务，猎杀目标为Enemy_Grunting;<img src='https://img-blog.csdnimg.cn/20201108031950863.png'>



## BP_TargetObject及其相关子类

1. BP_TargetObject组件面板修改如下（StaticMesh已被删除）；

   <img src='https://img-blog.csdnimg.cn/20201108033013777.png'>

2. 为Object_Treasure单独添加StaticMesh组件；

3. 创建新的子类Object_Grain，并添加StaticMesh组件，将谷物袋子模型导入，设置Name和Interact文本初始值。添加接口EventOnInteractWith（Character)，调用OnObjectFound事件；<img src='https://img-blog.csdnimg.cn/20201108033817418.png'>

## QuestManager相关

1. 函数模块化：打开函数[void] AddQuest(QuestClass,DirectlyStart)，将Branch之后有关Quest和SubGoal动画播放的蓝图迁移至自定义事件SelectNewQuest(NewQuest)，当CurrentQuest为NULL时更新Quest并播放动画，不为NULL时检测SubGoal首项是否有效（若无效则更新Quest，否则延迟更新直至SubGoal首项无效）。在AddQuest中被调用;<img src='https://img-blog.csdnimg.cn/20201108011601891.png'>
2. 创建自定义事件OnEnemyKilled(Class)，锁定类型为Hunt的分支任务的Index，并根据击杀的敌人类型判断是否为有效击杀，若为有效击杀则将CurrentHuntedAmount[Index]+1。若SelectedQuest为CurrentQuest，则调用GenerateSubGoals生成分支任务。若击杀数大于需求数，则调用CompleteSubGoal。<img src='https://img-blog.csdnimg.cn/20201108030332689.png'><img src='https://img-blog.csdnimg.cn/20201108031407189.png'>
3. 创建自定义事件OnObjectFound(Class)，锁定类型为Search的分支任务的Index，并根据找到的物品类型判断是否为有效寻找，若为有效寻找则将CurrentHuntedAmount[Index]+1。若SelectedQuest为CurrentQuest，则调用GenerateSubGoals生成分支任务。若找到的数量大于需求数，则调用CompleteSubGoal。（相关蓝图与OnEnemyKilled(Class)类似）


## RPG_PlayerController相关

1. 打开事件图表，修改Tab事件，打开任务系统界面时暂停游戏；<img src='https://img-blog.csdnimg.cn/2020110800393081.png'>

# 最终效果展示

## 【子任务】狩猎小鬼完成效果

<img src='https://img-blog.csdnimg.cn/20201109160208908.png'><img src='https://img-blog.csdnimg.cn/202011091641213.png'>

## 【子任务】寻找小麦完成效果

<img src='https://img-blog.csdnimg.cn/20201108230246514.png'>

<img src='https://img-blog.csdnimg.cn/20201108230310157.png'>

## 【子任务】完成前后对比

<img src='https://img-blog.csdnimg.cn/20201108230013575.png'>

<img src='https://img-blog.csdnimg.cn/20201108230045246.png'>

<img src='https://img-blog.csdnimg.cn/2020110916421358.png'>

<img src='https://img-blog.csdnimg.cn/20201109164242272.png'>