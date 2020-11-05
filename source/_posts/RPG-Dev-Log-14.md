---
title: RPG Game开发日志(十四)
date: 2020-11-04 23:01:43
categories: Logs
tags: UE_4.22
---

涉及内容：大任务系统创建(续篇),蓝图UI任务数据绑定。

<!--more-->

# 涉及内容

## 创建W_MissionSubGoal

1. 创建控件W_MissionSubGoal，作为W_Mission中任务分支信息SubGoalInfoVerticalBox的子控件；
<img src='https://img-blog.csdnimg.cn/20201105072242910.png'><img src='https://img-blog.csdnimg.cn/20201105072311908.png'>
3. 创建函数Update，用于更新SubGoalText的文本内容(这部分蓝图可从W_SubGoal中的Update函数迁移过来)，并根据任务状态设置CurrentImage、SuccessImage、FailedImage的可视性，并在事件开始构造时调用；<img src='https://img-blog.csdnimg.cn/20201105072948473.png'>

## 任务列表项W_QuestBorder

1. 打开S_QuestInfo，添加属性SuggestedLevel(Integer)；
2. 打开W_QuestBorder，创建函数[void] Update()，用于绑定W_QuestBorder相关任务数据，包括QuestName、Frame、RegionText、LevelText、QuestTypeIcon等控件的文本、颜色、图标等，并且对QuestName控件的文本进行长度检测（任务名过长时缩略显示）；<img src='https://img-blog.csdnimg.cn/20201104232620180.png'>
3. 创建自定义事件UpdateSuggestedLevelColor(PlayerLevel)并绑定至事件调度UpdateLevelQuest。每当角色升级时，调用此事件，更新任务推荐等级的文本颜色，为游戏玩家提供更好的视觉效果（具体方法与Enemy的头衔颜色更新类似，通过PlayerLevel/Suggested的值设置Alpha的值，从而设置颜色过渡）。最后调用Update函数，完成任务列表所有数据项的显示更新;<img src='https://img-blog.csdnimg.cn/20201104233806488.png'>

## 任务系统主界面W_Mission

1. 打开W_QuestBorder，为AnimBorder创建划入(AddQuestBorder)和划出(RemoveQuestBorder)动画；<img src='https://img-blog.csdnimg.cn/20201104234440454.png'><img src='https://img-blog.csdnimg.cn/20201104234537409.png'>
2. 创建宏ShowQuestBorder(Visible)，用于根据当前控件的可视性播放划入或划出动画；<img src='https://img-blog.csdnimg.cn/20201104234832171.png'>
3. 为CurrentQuestButton、CompletedQuestButton、FailedQuestButton、HasQuestButton添加绑定事件：以CurrentQuestButton为例，当点击按钮时，隐藏CompletedScrollBox、FailedScrollBox、HasQuestScrollBox三个滚动框，并显示CurrentScrollBox（其余三个按钮同理）；<img src='https://img-blog.csdnimg.cn/2020110423525363.png'>
4. 创建自定义事件UpdateSuggestedLevel(PlayerLevel)，用于设置任务详情中推荐等级的文本颜色（设置方法同上）,并将此自定义事件绑定至事件调度UpdateLevelForQuest；<img src='https://img-blog.csdnimg.cn/2020110500005662.png'><img src='https://img-blog.csdnimg.cn/20201105000436852.png'>
5. 创建函数UpdateDescription，用于更新任务的具体描述，相关控件为DescriptionText，这里需要注意文本换行符的转义方式;<img src='https://img-blog.csdnimg.cn/2020110500113572.png'>
6. 创建函数UpdateDetailWindow，用于绑定W_Mission任务详情的相关任务数据，包括QuestName、TypeText、QuestTypeImage、RegionText、SuggestedLevel、MoneyText、ExpText、PrestigeText等控件的文本、颜色和图标等，数据源为SelectedQuest(BP_MasterQuest，所有Quest的父类)中的QuestInfo，并调用函数UpdateSuggestedLevel(PlayerLevel)，UpdateDescription，GenerateSubGoals；<img src='https://img-blog.csdnimg.cn/20201105064907715.png'><img src='https://img-blog.csdnimg.cn/20201105065000665.png'><img src='https://img-blog.csdnimg.cn/20201105065053167.png'><img src='https://img-blog.csdnimg.cn/2020110506512638.png'>
7. 创建函数GenerateSubGoals，用于更新SubGoalVerticalBox控件，绑定任务分支信息的相关数据，数据源为SelectedQuest(BP_MasterQuest)中的CompleteSubGoalInfo和CurrentSubGoalInfo;<img src='https://img-blog.csdnimg.cn/20201105071813592.png'>
8. 创建枚举E_QuestState；<img src='https://img-blog.csdnimg.cn/20201105073933672.png'>
9. 创建函数AddQuestBorder(Quest)，用于任务列表项W_QuestBorder(CurrentScrollBox等滚动框的子控件)的添加；<img src='https://img-blog.csdnimg.cn/20201105073719617.png'>
10. 创建自定义事件OnQuestBorderClicked(QuestBorder)，绑定至W_QuestBorder中的按钮QuestBorderButton。当任务列表项被点击时，禁用该列表项按钮，并调用函数UpdateDetailWindow，完成任务详情的更新；<img src='https://img-blog.csdnimg.cn/20201105074414237.png'><img src='https://img-blog.csdnimg.cn/20201105074646874.png'>

## 任务组件QuestManager

1. 创建结构体S_CompletedSubGoalInfo；<img src='https://img-blog.csdnimg.cn/20201105065535758.png'>
2. 创建枚举E_SubGoalState；<img src='https://img-blog.csdnimg.cn/20201105065701141.png'>



## 任务父类BP_MasterQuest

1. 打开BP_MasterQuest，重新设置CompletedSubGoalsInfo变量类型为S_CompletedSubGoalInfo（原来为S_SubGoalInfo，即扩展SubGoalIndex、Succesful两个引脚);
2. 函数功能扩展：函数CompleteSubGoal(SubGoalIndex)变更为CompleteSubGoal(SubGoalIndex,Success?)，传入任务是否成功完成的参数，以便于CompletedSubGoalsInfo变量的更新;
3. 函数功能扩展：函数SetupStartingSubGoals新增Description的数据更新，数据源为QuestInfo;
4. 函数功能扩展：函数AddQuest(QuestClass)变更为AddQuest(QuestClass,DirectlyStart)，根据当前任务数量和DirectlyStart的布尔值判断是否自动接取任务。若自动接取任务，则播放动画PlayQuest和PlaySubGoal(来着W_Quest)；
5. 关联事件修改：打开事件图表，修改自定义事件UpdateCompletedSubGoal；

## 任务实例设置

### 主线任务

<img src='https://img-blog.csdnimg.cn/20201105080724734.png'>

### 支线任务

<img src='https://img-blog.csdnimg.cn/20201105080758220.png'>

### 副本任务

<img src='https://img-blog.csdnimg.cn/20201105080823342.png'>

# 最终效果展示

<img src='https://img-blog.csdnimg.cn/20201105082833827.png'><img src='https://img-blog.csdnimg.cn/20201105082916789.png'><img src='https://img-blog.csdnimg.cn/20201105083011449.png'><img src='https://img-blog.csdnimg.cn/20201105082938769.png'>