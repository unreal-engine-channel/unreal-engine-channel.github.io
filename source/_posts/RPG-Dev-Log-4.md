---
title: RPG Game开发日志(四) 
date: 2020-10-14 17:04:18
categories: Logs
tags: UE_4.22 
---

涉及内容：自动回复系统，迷你小地图制作，IceLands场景导入等。
<!--more-->

# 涉及内容

UI素材来源：

虚幻商城[Infinity Blade IceLands]

[阿里巴巴矢量图表库](https://www.iconfont.cn/) [关键词：目标]

## 自动回复系统

1. 打开W_Level，创建Event Dispatchers(事件调度器) UpdateLevel(Integer)，并在CharacterLevel更新后调用;
2. 打开人物蓝图Char_Shibi，创建自定义事件[void] UpLevel(Integer),用于升级后更新血量/蓝条/耐力值上限,并将W_Level中有关升级粒子动画特效的蓝图迁移整合；<img src='https://img-blog.csdnimg.cn/20201014174148886.png'>
3. 修改[void] ModifyAttribute(E_Attribute,Float)为[void] ModifyAttribute(E_Attribute,Float,Float),扩展函数功能，第三个参数MaxValue用于更新升级后的血量/蓝条/耐力值上限，思路与ModifyValue类似；
4. 在Event BeginPlay事件末尾绑定事件调度器UpdateLevel至自定义事件UpLevel;<img src='https://img-blog.csdnimg.cn/20201014174040200.png'>
5. 打开W_Level事件图表，将GetXP事件后的结点折叠至图表LoopLevel;
6. 打开W_PC_Main事件图表，删除Event Construct后有关获取W_Level等自定义组件的节点；
7. 打开S_Attribute，添加RegenInterval[Float]（回复间隔）、MaxRegenTime[Float]（最大回复时间）、RegenFunctionName[String]（回复回调函数名）、RegenTimerHandle[TimerHandle]（回复TimerHandle);
8. 打开Char_Shibi，设置Attributes，单次回复量=MaxValue*(RegenInterval/MaxRegenTime)，按照公式，设置数值，使得单次回复量耐力值>血量>灵力；
9. 创建回调函数[void]HealthRegenTick()、[void]ManaRegenTick()、[void]StaminaRegenTick()回调函数，用于脱战状态和战斗状态回复数值计算，三个函数仅参数不同；<img src='https://img-blog.csdnimg.cn/20201027161819931.png'>
10. 创建函数[void] HandleRegeneration(E_Attribute)，当状态回满时，使用TimerHandle中断回调函数;<img src='https://img-blog.csdnimg.cn/20201027214351835.png'>
11. 创建自定义事件Regenerations,在Event BeginPlay调用，用于循环更新Health/Mana/Stamina相关值，事件末尾调用HandleRegeneration，启用和中止相关tick函数；<img src='https://img-blog.csdnimg.cn/20201014180647554.png'>
12. 函数ModifyAttribute末尾也添加HandleRegeneration的调用，每次修改后都进行一次TimerHandle核对；

## 迷你小地图制作

1. 创建Render Target [MiniMap],右键创建Material [M_MiniMap_Mat]，设置Material Domain=User Interface,Blend Mode=Translucent，编辑蓝图如下;<img src='https://img-blog.csdnimg.cn/2020101423132354.png'>
2. 创建材质实例[Inst_MiniMap_Mat];
3. 选取父类为SceneCapture2D创建蓝图MiniCapture,将TextureTarget设置为[MiniMap],并取消勾选天气、光线、特效的ShowFlags；
4. 打开人物蓝图BP_RPG_Character，添加SpringArm、ChildActor,调整SpringArm的Rotation和Arm Length；
5. 导入direction图标，修改Compression Settings=UserInterface2D(RGBA)，Texture Group=UI，右键创建sprite；
6. 返回BP_RPG_Character，添加PaperSprite，将刚创建的direction_sprite添加进来，调整rotation，修改颜色，并将碰撞预设设为NoCollision,并勾选Render>Owner No See;
7. 创建Widget [W_MiniMap]，将Inst_MiniMap_Mat以及相关minimap frame添加进来，并调整zOder顺序，根据[W_MiniMap]中的效果修改Inst_MiniMap_Mat中Density和Radius的值；
8. 在W_MiniMap中添加Plus和Minus按钮，用于缩放地图，设计思路为：获取Actor[MiniCapture]的拷贝，并通过它获取Capture Component 2D,修改FOV Angle;<img src='https://img-blog.csdnimg.cn/20201014232845269.png'>

## IceLands场景导入

1. 将Map [FrozenCove]导入并构建，打开World Settings，设置Game Mode 为RPG_GameMode;
2. 完成光源的简单布置，光源大小、颜色、数量等，将部分PointLight由Stationary改为Static；
3. 完成场景地面的延伸扩展，Nav Mesh Bounds的延伸扩展（需勾选Can ever Affect Navigation),为扩展的地面添加Default碰撞预设；
4. 修改玩家出生点至平台。

# 最终效果展示



<img src='https://img-blog.csdnimg.cn/20201014171750989.gif'>

<img src='https://img-blog.csdnimg.cn/2020101417191758.png'>