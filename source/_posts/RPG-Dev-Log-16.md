---
title: RPG Game开发日志(十六)
date: 2020-11-10 19:39:49
categories: Logs
tags: UE_4.22
---

涉及内容：创建Npc对话系统（开篇），编辑剧本。

<!--more-->

# 涉及内容

## 准备工作

1. 创建枚举E_TalkType，代表当前对话选项对应的查看内容；<img src='https://img-blog.csdnimg.cn/20201110195932647.png'>
2. 创建枚举E_SelectSubGoalType，代表对话时分支选项的类型；<img src='https://img-blog.csdnimg.cn/2020111020024350.png'>
3. 创建结构体S_SelectSubGoalInfo，包含对话分支选项文本内容以及类型；<img src='https://img-blog.csdnimg.cn/2020111020054178.png'>
4. 创建结构体S_SpeechContent，包含Player与NPC对话的名字、内容、延迟(默认为4s)、是否有分支选项、分支选项内容；<img src='https://img-blog.csdnimg.cn/20201110200855293.png'>
5. 创建结构体S_Conversation，包含对话内容、对话信息、任务类、对话类型、指定任务是否已完成、推荐等级信息；<img src='https://img-blog.csdnimg.cn/20201110201213956.png'>


## 创建BP_FindGrain

<img src='https://img-blog.csdnimg.cn/20201110220053550.png'>

## 创建UI

### 创建对话选项条W_TalkInfo

1. 层级面板和预览效果;<img src='https://img-blog.csdnimg.cn/20201110210022230.png'>
2. 打开事件图表，创建函数Update，用于根据TalkType更新TalkInfoImage、HotKeyText（选项数字）、SelectTalkInfo(按钮文本)，并在事件构造时调用;<img src='https://img-blog.csdnimg.cn/20201110210336891.png'>
3. 为SelectButton添加点击事件，根据SubGoalType判断相应的是OnClickedTalkTypeInfo事件（直接对话选项条）还是ContinueSelect事件（分支对话选项条）；<img src='https://img-blog.csdnimg.cn/2020111021555746.png'>

### 创建对话信息盒W_TalKBorder

1. 层级面板和预览效果(**注意：将WrapperPolicy由默认模式更改为AllowPerCharacterWrapping，否则无法启用按长度换行功能**)；<img src='https://img-blog.csdnimg.cn/20201110210623158.png'>
2. 打开事件图表，创建自定义事件ShowTalkInfo(Name,TalkInfo)和Hide()，用于显示和隐藏对话信息（实现一问一答的效果）；<img src='https://img-blog.csdnimg.cn/20201110211146795.png'>

### 在W_PC_Main中添加对话选项盒和对话信息盒

1. 层级面板和预览效果；<img src='https://img-blog.csdnimg.cn/20201110211446980.png'>

### 在W_Interaction中添加TalkInfoText

1. 层级面板和预览效果；<img src='https://img-blog.csdnimg.cn/20201110211642270.png'>
2. 打开事件图表，创建自定义事件ShowTalkInfo(Text)，用于Player靠近NPC时，显示和隐藏NPC头上的信息；<img src='https://img-blog.csdnimg.cn/20201110211814747.png'>

## BP_RPG_NPC相关

### 将NPC互动更改为蓝图接口模式

1. 删除绑定事件OnInteract;
2. 在ClassSettings中添加蓝图接口I_Interaction;
3. 打开BP_RPG_NPC，在事件开始时将ConversationData(DataTable)添加至TalkInfo(S_Conversation);<img src='https://img-blog.csdnimg.cn/20201110204835322.png'>
4. 将Overlap事件更改为调用蓝图接口函数EventBeginOverlapTargetObject和EventEndOverlapTargetObject;<img src='https://img-blog.csdnimg.cn/20201110205210283.png'>
5. 删除OnInteract事件，更改为蓝图接口函数EventOnInteractWith，主要功能为：NPC头顶信息的显示与隐藏，对话选项盒的UI构建;<img src='https://img-blog.csdnimg.cn/20201110212312661.png'><img src='https://img-blog.csdnimg.cn/20201110212357282.png'><img src='https://img-blog.csdnimg.cn/20201110212443312.png'>

### 其他事件

1. 修改UpdateQuest，改为仅设置PlayLevel;<img src='https://img-blog.csdnimg.cn/20201110205805908.png'>

2. 创建事件OnClickedTalkTypeInfo，根据直接对话选项条类型，点击时响应不同事件；<img src='https://img-blog.csdnimg.cn/20201110213025561.png'>

3. 创建事件ContinueSelect，根据分支对话选项条类型，点击时响应不同事件；<img src='https://img-blog.csdnimg.cn/20201110215727110.png'>

4. [E_TalkType]枚举事件：

   - 创建事件OnClickedQuestInfo，当点击类型为QuestInfo的对话选项条时，播放对话剧本，并展示对话分支选项，并根据分支选项判断是否接取Quest；

   - 创建事件OnClickedEndTalk，结束对话并清空UI;
   - 创建事件OnClickedElseInfo(尚未实装)
   - 创建事件OnClickedGoodsStore(尚未实装)
   - 创建事件OnClickedWeaponStore(尚未实装)
   - 创建事件OnClickedEquipStore(尚未实装)
   - 创建事件OnClickedDrugStore(尚未实装)
   - 创建事件OnClickedFoodStore(尚未实装)

   <img src='https://img-blog.csdnimg.cn/20201110213742981.png'><img src='https://img-blog.csdnimg.cn/20201110213844323.png'>

5. [E_SelectSubGoalType]枚举事件：

   - 创建事件OnClickedContinue
   - 创建事件OnClickedBack
   - 创建事件OnClickedAddQuest
   - 创建事件OnClickedQuit

### 实例设置(NPC皮特)

<img src='https://img-blog.csdnimg.cn/20201110220234754.png'>


## 剧本编辑

### 创建Data Table

1. 创建D_NPC_Conversation；<img src='https://img-blog.csdnimg.cn/20201110202006809.png'>

### 编辑对话选项内容（含分支选项）

1. [QuestInfo]任务接取向——接取FindGrain任务：

   <img src='https://img-blog.csdnimg.cn/20201110203722362.png'>

2. [ElseInfo]剧情介绍向——与NPC闲聊；<img src='https://img-blog.csdnimg.cn/20201110202040377.png'>

3. [GoodsStore]市场交易向——杂货店：<img src='https://img-blog.csdnimg.cn/20201110202215835.png'>

4. [WeaponStore]市场交易向——武器商店：<img src='https://img-blog.csdnimg.cn/20201110202303511.png'>

5. [DrugStore]市场交易向——药品商店：<img src='https://img-blog.csdnimg.cn/20201110202358802.png'>

6. [FoodStore]市场交易向——食物商店：<img src='https://img-blog.csdnimg.cn/20201110202638264.png'>

7. [EquipStore]市场交易向——防具商店：<img src='https://img-blog.csdnimg.cn/20201110202659579.png'>

8. [End]结束对话：<img src='https://img-blog.csdnimg.cn/2020111020280739.png'>

### 编辑对话剧本

|   TalkerName    |                          SpeechInfo                          |
| :-------------: | :----------------------------------------------------------: |
|     Shinbi      |             嗨，大叔，你知道如何才能进入峡谷吗？             |
|      皮特       |                     大叔？小哥年方十八！                     |
|      Shibi      |                             ...                              |
|      皮特       | 咳咳。。。峡谷常年迷雾，又有猛兽出没，凶险万分，姑娘切莫深入。 |
|     Shinbi      | 大叔放心，小女子十八般武艺样样精通，拳打南山猛虎，脚踢北海蛟龙，正好顺道为名除害！ |
|      皮特       | 。。。大叔。。的确，看姑娘的着装并非普通女子，非富即贵，只是。。。 |
|     Shinbi      |     哈哈...就是，小女子才貌双全，大叔有何难处尽管直言！      |
|      皮特       | 呃。。这个，最近天降大雪，家中缺粮，我这不出来寻粮了吗，如果姑娘能带来一袋小麦，我将带你进入峡谷的入口。。。 |
| Shinbi(分支1-1) |    哈哈，小事情，交给我吧！不过这荒天雪地的能不能折现？？    |
| Shinbi(分支1-2) |               这荒天雪地的上哪去找一袋小麦。。               |
|      皮特       |            这个、、、多谢女侠，折合成市价30硬币。            |
| Shinbi(分支2-1) |                         给与30硬币。                         |
| Shinbi(分支2-2) |                        给与一袋小麦。                        |
| Shinbi(分支2-3) |                         我再想想。。                         |

<img src='https://img-blog.csdnimg.cn/20201110203809964.png'>

<img src='https://img-blog.csdnimg.cn/20201110203838787.png'>

# 最终效果展示

<img src='https://img-blog.csdnimg.cn/20201110194837692.png'>

<img src='https://img-blog.csdnimg.cn/20201110194928442.png'>

<img src='https://img-blog.csdnimg.cn/20201110195002450.png'>

<img src='https://img-blog.csdnimg.cn/2020111019504116.png'>

<img src='https://img-blog.csdnimg.cn/20201110195118759.png'>

<img src='https://img-blog.csdnimg.cn/20201110195145659.png'>