---
title: RPG Game开发日志(二) 
date: 2020-10-12 17:33:57
categories: Logs
tags: UE_4.22 
---

涉及内容：UI血条创建；绑定角色属性等。
<!--more-->

# 涉及内容

## UI血条创建

UI素材来源：虚幻商城[User Interface Kit]

### 材质球制作

1. 创建Material [M_GlassBall]，将gb（T5,include frame,gloss,over) 拖拽进材质球蓝图;
2. 创建Material Function [MF_Vertical]，用于处理血球垂直方向的颜色线性渐变，此处Clamp限制Alpha(0~1)，减少性能消耗；<img src='https://img-blog.csdnimg.cn/20201012182956977.png'>
3. 打开[M_GlassBall]，完成第一部分蓝图，涉及UV平衡器（Panner）、图像混合模式（Blend Overlay)、插值计算等,完成Fill (V3)参数传递<img src='https://img-blog.csdnimg.cn/20201012183538901.png'>
4. 完成第二部分蓝图，MF_Vertical—>M_GlassBall，这部分主要设计血球的框架和形状，Ceil用于向上取整<img src='https://img-blog.csdnimg.cn/20201012184100410.png'>
5. 创建材质实例Inst_GlassBall,设置Color、Intensity（强度）、Percentage（血量百分比）等参数

### 控件蓝图设计

1. 创建Widget Blueprint [W_ActionBar]，删除默认的canvas panel，设置为custom模式，添加Scale Box,组件结构如下<img src='https://img-blog.csdnimg.cn/20201012181046997.png'>Textures涉及:actionbar frame(T3,include L ,R,Center);
2. 创建Widget Blueprint [W_GlassBall]，同上，从User Created分类下，在Scale Box内添加预设的Inst_GlassBall，添加Event Construct事件，用于设置血球颜色以便于灵力球的添加和血量动态颜色；<img src='https://img-blog.csdnimg.cn/20201012185331871.png'>
3. 创建Widget Blueprint [W_PC_Main]，添加预设的Action Bar以及Glass Ball，并调整anchors,Size,Alignment,etc;
4. 打开Char_Shibi人物蓝图，在Event Graph添加W_PC_Main控件即可。

## 绑定角色属性

### 设计思路

1. 准备工作：创建枚举[E_Attribute]=[None,Health,Mana,Stamain],创建结构体[S_Attribute]=[CurrentValue(Float),MinValue(Float),MaxValue(Float),ClassBallWidget(W Glass Ball)];
2. 核心数据结构：E_Attribute 到 S_Attribute Map 映射关系；
3. 函数[S_Attribute] GetAttribute(E_Attribute),用于根据键值对获取某一属性的相关值；
4. 函数[void] SetAttribute(E_Attribute,S_Attribute),用于添加新的属性键值对；
5. 函数[void] SetupAttributeBars()，实装Health&Mana等属性绑定；
6. 函数[S_Attribute] UpdateAttribute(E_Attribute),用于计算和更新血球百分比，动态颜色；
7. 函数[void] ModifyAttribute(E_Attribute,Float),用于伤害显示，同时调用4、6。

### 受伤/回血实装

1. GetAttribute<img src='https://img-blog.csdnimg.cn/20201012195017253.png'>
2. SetAttribute<img src='https://img-blog.csdnimg.cn/20201012195137455.png'>
3. SetupAttributeBars<img src='https://img-blog.csdnimg.cn/20201012195223773.png'>
4. UpdateAttribute<img src='https://img-blog.csdnimg.cn/20201012195331529.png'>
5. ModiftAttribute<img src='https://img-blog.csdnimg.cn/20201012195518275.png'>

### 受伤/回血数字显示

1. 创建Widget [W_DamageText]，添加FadeOut动画，用于伤害数字浮动显示并消失；<img src='https://img-blog.csdnimg.cn/2020101219394440.png'>
2. 打开W_DamageText 事件图表，事件开始构造时播放1s的Fadeout动画,添加自定义事件UpdateValue，用于受伤/回血时不同的文字内容&颜色显示；
3. 创建Widget Component [WC_OnDamaged],创建自定义事件SetDamageText,内部调用UpdateValue事件；
4. 打开函数ModifyAttribute，在Health值被修改前将WC_OnDamaged控件调出,并调用自定义事件SetDamageText，将会自动播放FadeOut动画。

# 最终效果展示

<img src='https://img-blog.csdnimg.cn/20201012175955500.png'>

<img src='https://img-blog.csdnimg.cn/20201012180028750.png'>