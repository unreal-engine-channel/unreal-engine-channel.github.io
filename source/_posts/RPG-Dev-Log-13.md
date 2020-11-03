---
title: RPG Game开发日志(十三)
date: 2020-11-03 11:31:15
categories: Logs
tags: UE_4.22
---

涉及内容：大任务系统创建（进阶），UI设计应用细讲。

<!--more-->

# 涉及内容

## 任务系统主界面W_Mission

### 任务类型侧边栏

涉及组件：

- Panel: [Size Box]、[Horizontal Box]、[Vertical Box]、[Overlay]
- Common: [Border]、[Button]、[Text]

#### 层级面板

<img src='https://img-blog.csdnimg.cn/20201103114016404.png'>

#### 实装效果

<img src='https://img-blog.csdnimg.cn/20201103114544272.png'>

#### 组件动画

Widget可视性：

> **Visible(可视)**： 可见、可点击
> **Collapsed(已折叠)**： 不可见、不占用布局空间（性能优于 Hidden）
> **Hidden(隐藏)**： 不可见、占用布局空间
> **HitTestInvisible(非可命中测试（自身和所有子项）)**： 可见、当前 Widget 不可点击、所有 Child Widget 不可点击
> **SelfHitTestInvisible(非可命中测试（仅自身）)**： 可见、当前 Widget 不可点击、不影响 Child Widget

1. 打开W_Mission,添加组件动画OpenMission;
<img src='https://img-blog.csdnimg.cn/20201103120349278.png'>
2. 创建宏ShowMission(Visible);
<img src='https://img-blog.csdnimg.cn/20201103121441665.png'>
3. 创建自定义事件PlayMission(Visible)，调用宏ShowMission(Visible)；

### 任务列表

设计组件：

- Panel: [Size Box]、[Vertical Box]、[Overlay]、[Scroll Box]
- Common: [Border]、[Text]

#### 层级面板

<img src='https://img-blog.csdnimg.cn/20201103114907317.png'>

#### 实装效果

<img src='https://img-blog.csdnimg.cn/20201103114943124.png'>

### 任务详情

涉及组件：

- Panel: [Size Box]、[Horizontal Box]、[Vertical Box]、[Overlay]、[Scroll Box]
- Common: [Border]、[Button]、[Text]、[Image]

#### 层级面板

<img src='https://img-blog.csdnimg.cn/20201103115323410.png'>

#### 实装效果

<img src='https://img-blog.csdnimg.cn/2020110311543341.png'>

## 任务提示界面W_QuestBorder

### 层级面板

<img src='https://img-blog.csdnimg.cn/20201103120105457.png'>

### 实装效果

<img src='https://img-blog.csdnimg.cn/20201103120130439.png'>

## 热键测试

打开RPG_PlayerController，添加热键Tab响应事件，切入和切出任务系统主界面；
<img src='https://img-blog.csdnimg.cn/20201103121909957.png'>

# 最终效果展示

<img src='https://img-blog.csdnimg.cn/2020110311322945.png'><img src='https://img-blog.csdnimg.cn/20201103113402801.png'>