---
title: RPG Game开发日志(十七)
date: 2020-11-14 15:55:56
categories: Logs
tags: UE_4.22
---

涉及内容：Npc对话系统续，创建背包。

<!--more-->

# 涉及内容

## NPC对话系统（续篇）

1. 打开枚举E_SelectSubGoalType，添加两个新的枚举BackToLastQuest和FinishQuest;
2. 打开结构体S_Conversation，添加两个新的成员RecieveTalkTypeInfo(Text)和RecieveQuestTalkInfo(S_SpeechContent[]);
3. 打开BP_RPG_NPC，完善自定义事件OnClickBack，返回对话上一级时清空对话信息条并隐藏对话信息盒；<img src='https://img-blog.csdnimg.cn/20201114161008148.png'>
4. 修改自定义事件EventOnInteractWith,对TalkInfo做一次Select。交互NPC时将根据玩家是否已经接取任务，显示不同的对话信息条；<img src='https://img-blog.csdnimg.cn/20201114161616981.png'>
5. 修改自定义事件OnClickedQuestInfo，分别对SpeechContent和TalkInfo做一次Select。点击类型为QuestInfo的对话信息条时，将根据玩家是否已经接取任务，显示不同的下级对话信息条和对话信息盒内容；<img src='https://img-blog.csdnimg.cn/20201114161932684.png'>
6. 创建自定义事件SkipTalkInfo，用于跳过对话信息；<img src='https://img-blog.csdnimg.cn/20201114162118327.png'>
7. 打开W_TalkBorder，为HorizatontalBox添加一个SkipButton，并添加绑定事件SkipTalkInfo;<img src='https://img-blog.csdnimg.cn/20201114162351432.png'><img src='https://img-blog.csdnimg.cn/20201114162420284.png'>
8. 创建函数UpdateQuestInfo(HasRecievedQuest?)，用于接取任务时更新对应的对话信息条，在OnClickAddQuest自定义事件中调用；<img src='https://img-blog.csdnimg.cn/20201114164749817.png'>
9. 创建自定义事件OnBackToLastSelect，返回对话上一级时清空对话信息条并隐藏对话信息盒；<img src='https://img-blog.csdnimg.cn/20201114164037125.png'>
10. 创建自定义事件OnFinishQuest，完成任务时根据当前索引移除对应的对话信息条,并返回上一级对话；<img src='https://img-blog.csdnimg.cn/20201114164220856.png'>
11. 更新自定义事件ContinueSelect，添加事件OnBackToLastSelect和OnFinishQuest的调用；<img src='https://img-blog.csdnimg.cn/20201114165003579.png'>
12. 打开D_NPC_Conversation，将TalkInfo[7].SelectSubGoalInfo[1].SelectSubGoalType改为Back，将TalkInfo[8].SelectSubGoalInfo[2].SelectSubGoalType改为BackToLastSelect;<img src='https://img-blog.csdnimg.cn/20201114163211314.png'>
13. 设置新增成员RecieveTalkTypeInfo和RecieveQuestTalkInfo;<img src='https://img-blog.csdnimg.cn/20201114163638476.png'>

## 创建背包

> Spacer：留白占位控件
>
> WrapBox：流布局控件，其子控件可以根据WrapBox的大小自动换行s

1. 创建控件W_Window，用作背包窗口；<img src='https://img-blog.csdnimg.cn/20201114171504794.png'>
2. 将变量Name绑定至NameForWindow文本控件;
3. 为CloseButton添加OnClicked绑定事件，添加节点RemoveFromParent；
> 当前鼠标的本地位置（LocalPosition）由当前鼠标的绝对位置（AbsolutePosition）经过窗体几何变换得到。
>
> Canvas Panel Slot插槽位置 = 拖拽终点鼠标的本地位置 - 拖拽起点鼠标的本地位置
4. 为DraggableBorder控件添加OnMouseButtonDown事件绑定**（DraggableBorder的所有父控件可视性必须设置为Self Hit Test Invisible,否则无法响应事件）**，判断鼠标是否按在DraggableBorder上，并获取鼠标拖拽后的位置，最后返回Event Reply；<img src='https://img-blog.csdnimg.cn/20201114172104549.png'>
5. 创建重载函数OnMouseMove，更新拖拽后OutterVerticalBox的位置；<img src='https://img-blog.csdnimg.cn/20201114172158989.png'>
6. 创建重载函数OnMouseButtonUP，当鼠标抬起时结束DraggingWindow可拖拽判定，并返回Event Reply；<img src='https://img-blog.csdnimg.cn/20201114172238741.png'>
7. 打开RPG_PlayerController，在EventBeginPlay中设置InputGameModeAndUI;<img src='https://img-blog.csdnimg.cn/20201114172545263.png'>
8. 创建Key I键盘输入事件，控制背包的显示和隐藏；<img src='https://img-blog.csdnimg.cn/20201114172647259.png'>
9. 创建控件W_Inventory，用作背包物品的图表和数量显示；<img src='https://img-blog.csdnimg.cn/20201114173149429.png'>
10. 打开事件图表，创建函数GenerateInventorySlot，用于测试背包物品的生成，在事件开始构造时调用;<img src='https://img-blog.csdnimg.cn/20201114173457586.png'>

# 最终效果展示

## 接取NPC任务

<img src='https://img-blog.csdnimg.cn/20201114165442401.png'><img src='https://img-blog.csdnimg.cn/20201114170429745.png'>

## 提交NPC任务

<img src='https://img-blog.csdnimg.cn/20201114165550957.png'><img src='https://img-blog.csdnimg.cn/20201114170513599.png'>

## 完成NPC任务

<img src='https://img-blog.csdnimg.cn/20201114165629575.png'>

## 背包效果

<img src='https://img-blog.csdnimg.cn/20201114155859633.png'>