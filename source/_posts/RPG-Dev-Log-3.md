---
title: RPG Game开发日志(三) 
date: 2020-10-13 21:30:26
categories: Logs
tags: UE_4.22 
---

涉及内容：人物等级属性绑定，经验条动画效果等。
<!--more-->

# 涉及内容

UI素材来源：

虚幻商城[Advanced Mission And Notification System]

[爱给网](http://www.aigei.com/) [SlicedImages]

## 经验条动画效果

1. 创建Widget [W_Level]，涉及素材资源：进度条和经验徽章边框,

   层级面板如下,<img src='https://img-blog.csdnimg.cn/20201013221953333.png'>

   最终效果,<img src='https://img-blog.csdnimg.cn/20201013222300719.png'>

2. 创建HideXPBar动画(0.5s，第一帧X轴缩放为0，最后一帧为1);<img src='https://img-blog.csdnimg.cn/20201013222531735.png'>

3. 打开Event Graph，事件构造时绑定Character Level 至Level Text;

4. 创建宏 [Exec] PlayAnimation(Exec,Boolean)，用于经验条动画循环播放以及播放模式设置；<img src='https://img-blog.csdnimg.cn/20201013223428875.png'>

5. 创建函数 [void] IncreaseXP()，用于更新level经验条百分比，此处利用Handle可以在任意位置暂停和恢复运行经验条动画；<img src='https://img-blog.csdnimg.cn/20201013223800968.png'>

6. 在StarterContent资源包中搜索P_Explosion粒子特效，删除FireBall、Sparks、fire_Light、Smoke效果，更改shockwave参数，达到以下效果；<img src='https://img-blog.csdnimg.cn/20201013225153345.png'>

7. 搜索Shibi角色升级的相关动画，在合适位置添加粒子特效动画和音效，并以此为基础创建AnimMontage;<img src='https://img-blog.csdnimg.cn/2020101322552387.png'>

8. 打开控件蓝图W_PC_Main，关联Shibi和W_Level，以便获取character movement;

9. 创建自定义事件 void GetXP(Float)，调用PlayAnimation和IncreaseXP，完成经验的增加，等级的更新，经验条上限更改，以及升级时添加动画蒙太奇（蓝图较复杂，不再展示）；

10. 打开人物蓝图Char_Shibi，添加Key F键盘事件，调用自定义事件GetXP。



## 任务等级属性绑定

### 血条/蓝条材质球创建

1. 创建Material，Material Domain设置为User Interface，Blend Mode设置为Masked,导入uf_fill_green（填充效果图），相关蓝图如下；<img src='https://img-blog.csdnimg.cn/20201013230852397.png'> <img src='https://img-blog.csdnimg.cn/20201013230948299.png'>

2. 创建材质实例Inst_HealthBar和HealthRemove，修改公开变量Texture 、Scaler(PercentLeft、PercentRight)、Vector(Color)；

3. 用同样的方法复制创建Inst_ManaBar和Inst_ManaRemove;

   

### 血条/蓝条/角色Border控件蓝图创建

1. 创建Widget [W_HealthBar]，

   层级面板如下，<img src='https://img-blog.csdnimg.cn/2020101323190811.png'>

   最终效果；<img src='https://img-blog.csdnimg.cn/20201013232000940.png'>

2. 用同样的方法复制一份，修改参数命名为W_ManaBar;

3. 创建Widget [W_CharacterBorder]，整合frame、HealthBar、ManaBar、StaminaBar、LevelText&Border、Figure&Border、BuffBox，

   层级面板如下,<img src='https://img-blog.csdnimg.cn/20201013232626224.png'>

   最终效果；<img src='https://img-blog.csdnimg.cn/20201013232731899.png'>

### Shibi人物蓝图修改

1. 修改[S_Attribute] UpdateAttribute(E_Attribute)为[void] UpdateAttribute(E_Attribute)，扩展函数功能（由原先的计算和更新血球百分比，动态颜色，扩展至Mana、Stamina，函数核心逻辑不变；<img src='https://img-blog.csdnimg.cn/20201013233125525.png'>
2. 修改[void] SetupAttributeBars()，扩展函数功能（由原先的Health、Mana扩展至Stamina);<img src='https://img-blog.csdnimg.cn/20201013233517102.png'>
3. 修改Num-/+ 键盘事件，测试扩展至Mana、Stamina;<img src='https://img-blog.csdnimg.cn/20201013233813393.png'>

# 最终效果展示

<img src='https://img-blog.csdnimg.cn/20201013220531552.gif'>

<img src='https://img-blog.csdnimg.cn/20201013220800638.gif'>

