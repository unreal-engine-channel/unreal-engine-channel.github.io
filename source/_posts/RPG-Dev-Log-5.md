---
title: RPG Game开发日志(五) 
date: 2020-10-18 12:23:00
categories: Logs
tags: UE_4.22 
---

涉及内容：跳跃动作制作，创建任务系统（开篇）

<!--more-->

# 涉及内容

虚幻商城[Advanced Mission And Notification System]

## 跳跃动作制作

1. 打开ABP_Shibi事件图表，将Is Falling返回值保存至变量IsInAir;
2. 进入状态机，添加Idle_Move<=>Jump状态过渡，并将过渡规则与IsInAir关联；
3. 进入Jump状态，新建状态机Jump，点击进入，设置Entry->JumpStart=>Loop=>JumpEnd状态过渡，并将过渡规则设置为自动；
4. 为JumpStart、Loop、JumpEnd添加Jump_Start、Jump_Apex、Jump_Land,并调整播放速率；
5. 打开Char_Shibi事件图表，添加Space Bar键盘事件，使得角色获得二段跳能力；<img src='https://img-blog.csdnimg.cn/20201018130500837.png'>
6. 打开CharacterMovement细节面板，设置Character Movement:Jump/Falling，Jump Z Velocity=620，Air Control=0.5;

## 创建任务系统（开篇）

### 准备工作

1. 创建枚举E_QuestCategories = {主线任务，支线任务，副本任务}；
2. 创建枚举E_Regions = {冰岛，洞穴，香格里拉}；
3. 创建枚举E_SubGoalType = {Custom,Hunt,Search,Talk};
4. 创建结构体S_QuestReward = {Money(Integer),Experience(Integer),Prestige(Integer)};
5. 创建结构体S_TargetLocation = {HasLocation?(Boolean),Location(Vector)};
6. 创建结构体S_SubGoalInfo;<img src='https://img-blog.csdnimg.cn/20201018132339967.png'>
7. 创建结构体S_QuestInfo;<img src='https://img-blog.csdnimg.cn/20201018132440276.png'>

### 创建任务实例的父类

#### 变量创建

1. 创建Actor BP_MasterQuest,作为创建任务实例的父类；
2. 创建变量QuestInfo(S_QuestInfo)，用于记录所有**Quest**信息；
3. 创建变量StartSubGoalIndices(Integer [])，用于记录开始**Quest**所有**SubGoal**索引；
4. 创建变量CurrentSubGoalIndices(Integer [])，用于记录当前**Quest**所有**SubGoal**索引；
5. 创建变量CurrentHuntedAmount(Integer[])，用于记录当前猎杀数量；
6. 创建变量CurrentSubGoalInfo(S_SubGoalInfo []),用于记录当前**Quest**所有**SubGoal**信息；
7. 创建变量SelectedSubGoalIndex(Integer)，用于记录当前选择的**SubGoal**索引；

#### 函数创建

1. 创建函数void UpdateSubGoalInfo(),用于根据CurrentSubGoalIndices，更新CurrentSubGoalInfo；<img src='https://img-blog.csdnimg.cn/20201018135432102.png'>
2. 创建函数void SetopStartingSubGoals()，用于设置CurrentSubGoalIndices,并调用UpdateSubGoalInfo;<img src='https://img-blog.csdnimg.cn/20201018135750624.png'>
3. 创建函数Boolean GoToNextSubGoal()，用于判断当前Quest是否还有SubGoal，若有，则将NextIndex保存至CurrentSubGoalIndices，并返回true;<img src='https://img-blog.csdnimg.cn/20201018140354801.png'>

### 创建UI

#### 创建W_SubGoal

2. 创建Widget [W_SubGoal],

   层级面板如下<img src='https://img-blog.csdnimg.cn/20201018141429316.png'>,

   最终效果<img src='https://img-blog.csdnimg.cn/20201018141521486.png'>;

3. 打开事件图表

   **创建变量**

   W_Quest_SubGoalInfo(S_SubGoalInfo)，

   W_Quest_AssignedQuest(BP_MasterQuest),

   W_Quest(W_Quest),

   SubGoalIndex(Integer),

   SubGoalInfo(Text);

   **创建函数**

   void Update(),用于根据W_Quest_SubGoalInfo从W_Quest_AssignedQuest中获取QuestInfo，并更新Text;<img src='https://img-blog.csdnimg.cn/20201018143047970.png'>

   **函数调用**

   事件开始构造时调用Update()，点击SelectedButton时调用W_Quest中的SelectSubGoal()；

#### 创建W_Quest

1. 创建Widget [W_Quest]，

   层级面板如下,<img src='https://img-blog.csdnimg.cn/20201018144005433.png'>

   最终效果<img src='https://img-blog.csdnimg.cn/20201018144100772.png'>;
   
2. 打开事件图表

   **创建变量**

   SubGoalWidgets(W_SubGoal),

   AssignedQuest(BP_MasterQuest),

   SelectedSubGoal(W_SubGoal);

   **创建函数**

   1. void GenerateSubGoal(),用于清空SubGolaVerticalBox，并从AssignedQuest中获取CurrentSubGoalInfo,循环添加至SubGolaVerticalBox;<img src='https://img-blog.csdnimg.cn/20201018145035536.png'>

   2. void UpdateCurrentSubGoal(),用于更新SubGoalIcon、SubGoalType、SubGoalInfo等控件信息；<img src='https://img-blog.csdnimg.cn/20201018152129831.png'>

   3. void SelectSubGoal(W_SubGoal)，用于激活SelectedButton,并调用UpdateCurrentSubGoal;<img src='https://img-blog.csdnimg.cn/202010181515354.png'>

   4. void UpdateQuest(Quest),用于从AssignedQuest中获取CurrentSubGoalInfo，根据任务Categories设置文本和任务图标样式，并调用GenerateSubGoal和UpdateCurrentSubGoal;<img src='https://img-blog.csdnimg.cn/20201018151930868.png'>

### 创建QuestManager

1. 创建Actor Componet [QuestManager]，并添加至Char_Shibi;

2. 创建变量

   CurrentQuest(BP_MasterQuest),

   AllQuests(BP_MasterQuest[] Object reference),

   AllQuestClasses(BP_MasterQuest[] class reference),

   Shibi(Char_Shibi);

3. 创建函数void AddQuest(BP_MasterQuest),用于从所有Quest class中添加、生成Quest Actor实例,并调用BP_MasterQuest.SetupStartingSubGoals和W_Quest.UpdateQuest;<img src='https://img-blog.csdnimg.cn/20201018153733195.png'>

### 绑定Q键盘事件

<img src='https://img-blog.csdnimg.cn/20201018154325823.png'>

   

# 最终效果展示

<img src='https://img-blog.csdnimg.cn/20201018123029836.png'>

<img src='https://img-blog.csdnimg.cn/20201018123051461.png'>

