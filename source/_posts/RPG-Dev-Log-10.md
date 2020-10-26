---
title: RPG Game开发日志(十)
date: 2020-10-25 19:58:21
categories: Logs
tags: UE_4.22 
---

涉及内容：创建Enemy；创建战斗系统（开篇）。

<!--more-->

# 涉及内容

素材来源：

虚幻商城[InfinityBlade:Adversaries]

虚幻商城[Action RPG]

虚幻商城[User Interface Kit]

## 创建Enemy

### 创建敌人动画蓝图

1. 以BP_RPG_Character为父类，创建BP_Enemy;

2. 将InfinityBlade:Adversaries资源[Enemy_Grunting]导入，创建动画蓝图ABP_Grunting和一维混合空间BS_Grunting_Skeleton_BlendSpace1D;

3. 打开项目Action RPG,将骨骼动画Idle_Grunting导出为fbx文件，随后添加至ContentBrowser；

4. 打开BS_Grunting_Skeleton_BlendSpace1D，设置水平坐标为Speed，最大水平值为400。将骨骼动画Idle、Walk_Slow、Walk、Run添加至适当坐标，并勾选Walk_Slow、Walk、Run的ForceFontXAxis选项；

5. 打开ABP_Grunting，添加蓝图获取PawnOwner的Speed;

6. 进入动画图表，新建状态机，添加Start<=>Locomotion状态及过渡规则（此处省略状态动画的相关蓝图设置）；<img src='https://img-blog.csdnimg.cn/20201025201901720.png'><img src='https://img-blog.csdnimg.cn/20201025202013508.png'>

### 创建敌人蓝图

1. 以BP_Enemy为父类，创建蓝图Enemy_Grunting，添加骨骼和骨骼动画，调整碰撞胶囊体大小；

2. 打开BP_Enemy，修改PaperSprite样式，添加组件Widget,命名为NameAndHealthBar，并添加隐藏PaperSprite的相关蓝图；<img src='https://img-blog.csdnimg.cn/20201025203024776.png'>

### 创建武器蓝图

1. 创建BP_Weapon，添加组件Sphere、StaticMesh、SkeletalMesh、Capsule，设置相关参数；

2. 以BP_Weapon为父类，创建Weapon_Grunting,添加Grunting武器模型，调整胶囊体和武器朝向；
3. 打开Enemy_Grunting骨骼，添加Sockect[Weapon]至b_MF_Weapon_R,添加preview assets，设置预览动画，调整Socket位置;
4. 打开事件图表，添加蓝图，将Weapon添加至敌人身上；<img src='https://img-blog.csdnimg.cn/20201025C204213703.png'>

### 创建敌人名称血量UI

1. 创建Widget[W_Enemy]，层级面板和预览效果；<img src='https://img-blog.csdnimg.cn/20201025204659690.png'>
2. 打开BP_Enemy，将W_Enemy添加至NameAndHealthBar，并调整其位置;
3. 创建结构体S_EnemyAttribute，添加敌人血量上限、物理攻击/法术攻击、物理防御/法术防御等属性；<img src='https://img-blog.csdnimg.cn/20201025205105844.png'>
4. 打开BP_Enemy，创建函数[void] SetHealthPercent()，用于控制敌人血量百分比显示,在EventBegin添加调用；<img src='https://img-blog.csdnimg.cn/2020102520543750.png'>

### 创建敌人AI行为——Random

1. 打开E_Behaviors，添加AI行为枚举[Random]；

2. 创建Componet[Behavior创建Componet],创建事件调度OnBehaviorChanged(E_Behaviors);

3. 创建函数[void] SetBehavior(E_Behaviors),用于Call事件调度；<img src='https://img-blog.csdnimg.cn/20201025210057184.png'>

4. 将BehaviorComponet添加至BP_RPG_Character；

5. 打开行为树BT_NPC，打开服务BTS_UpdateBehaviorTree，删除重载函数ReceiveTickAI，添加重载函数ReveiveSearchStartAI,创建自定义事件OnBehaviorChanged(E_Behaviors),调用宏SetBehavior;<img src='https://img-blog.csdnimg.cn/20201025211113194.png'>

6. 打开之前创建的BP_RPG_NPC_Warriors巡逻NPC，通过Behavior组件调用宏SetBehavior，实现AI行为改变；

7. 打开BB_Base，添加Key[Originl]、[Random]、[Target];

8. 创建Task[BTTask_RandomLocation],重载ReceiveExecuteAI事件，控制AI随机半径巡逻;<img src='https://img-blog.csdnimg.cn/20201025211959614.png'>

9. 打开行为树BT_NPC，创建Sequence并添加Blackboard Decorator[RandomLocation],序列执行顺序为:Wait（5)—>BTTask_RandomLocation(SelfActor,Random,2000)—>MoveTo(Random)（then Loop);<img src='https://img-blog.csdnimg.cn/20201026163507753.png'>

10. 打开BP_Enemy，初始化Blackboard的Original值为ActorLocation,设置AI行为为Random;<img src='https://img-blog.csdnimg.cn/20201025214715454.png'>

11. 设置AI Controller Class为AIC_Controller。

## 创建战斗系统（开篇）

### 创建敌人AI行为——HasSeenPlayer

1. 打开BP_Enemy，添加组件PawnSensing，设置视野半径和周边视野角度；<img src='https://img-blog.csdnimg.cn/20201026162705473.png'>

2. 打开BB_Base，添加key[Target]，类型为Actor;添加IsInAttackRange，类型为Boolean。打开E_Behavior，添加枚举HasSeenPlayer；

3. 打开BP_Enemy事件图表，添加OnSeePawn(PawnSensing)事件，当Enemy感知Player时，加速靠近Player并执行HasSeenPlayer行为；<img src='https://img-blog.csdnimg.cn/20201026163212433.png'>

4. 创建任务BTTask_Check，重载事件ReceiveExcuteAI，用于检测Player是否在AI视野范围内；<img src='https://img-blog.csdnimg.cn/2020102616394518.png'>

5. 创建任务BTTask_Attack，重载事件ReceiveExecuteAI,调用BP_Enemy的Attack事件。创建自定义事件FinalAttack，成功执行返回success；<img src='https://img-blog.csdnimg.cn/20201026164404943.png'>

### 创建敌人攻击动画

1. 将InfinityBlade:Adversaries资源[Enemy_Anim_Sword_Combat_Swing_Montage]导入，创建动画Montage;
2. 打开ABP_Grunting，状态Locomotion添加DefaultSlot，使动画Montage生效；
3. 打开BP_Enemy事件图表，创建自定义事件Attack,执行Attack蒙太奇动画，并调用BTTask_Attack中的自定义事件FinalAttack;<img src='https://img-blog.csdnimg.cn/20201026165559988.png'>
4. 打开行为树BT_NPC,创建Selector并添加Blackboard Decorator[HasSeenPlayer],设置BlackboardKey=Behavior,KeyValue=HasSeenPlayer；创建Sequence，序列执行顺序为BTTask_Check(Target,IsInAttackRange,AttackRange)——>MoveTo(Target)[CheckAttackRange(Target)]——>BTTask_Attack(then loop);<img src='https://img-blog.csdnimg.cn/20201026165758704.png'>
5. 创建AnimNotifyState[ANS_Weapon]，创建重载函数Reveived_NotifyBegin，Received_NotifyEnd，当敌人攻击角色时，临时关闭胶囊体碰撞；<img src='https://img-blog.csdnimg.cn/20201026174431299.png'>

### 创建敌人攻击伤害显示

1. 创建蓝图接口I_Damageable,创建接口函数[void] OnRecieveDamage(Float:PhysicalDamage,Float:MagicDamage,Float:CriticalChance,Float:CriticalDamage);

2. 打开BP_Enemy，添加蓝图接口I_Damageable；

3. 打开BP_RPG_Character，创建函数[Boolean:IsCritical?，Float:Damage] CalculateDamage(Float:PhysicalDamage,Float:MagicDamage,Float:CriticalChance,Float:CriticalDamage,Float:PhysicalDefence，Float:MagicDefence，Boolean:AbsoluteDefence);

4. 函数CalculateDamage用于综合敌人攻击属性和玩家防御属性，计算最终伤害，计算公式为：

   Damage=

   [

   PhysicalDamage×random(0.9,1.1)-PhysicalDefence

   +

   MagicDamage×random(0.7,1.3)-MagicDefence

   ]

   ×

   (IsCritical ?  1 :  CriticalDamage)；<img src='https://img-blog.csdnimg.cn/20201026171644771.png'>
   
5. 打开Enemy_Grunting，整合BeginPlay事件至父类BP_Enemy中的自定义事件CreateWeapon，并修改为在BP_Enemy中的BeginPlay事件中调用；<img src='https://img-blog.csdnimg.cn/20201026173117268.png'>

6. 打开BP_Shibi，添加OnRecieveDamage函数实现，调用CalculateDamage和ModifyAttribute(新的参数Critical,用于暴击判定，最终传入SetDamageText)，实现敌人伤害计算并应用于玩家的血量变更；<img src='https://img-blog.csdnimg.cn/20201026173731535.png'>

7. 打开BP_Weapon，添加事件ActorBeginOverlap，当敌人追上玩家时，传入敌人属性，调用OnRecieveDamage事件；<img src='https://img-blog.csdnimg.cn/20201026174639179.png'>

8. 打开W_DamageText，扩展UpdateValue函数功能，添加新的传入参数[Critical]，用于暴击伤害时伤害数字的特殊显示；<img src='https://img-blog.csdnimg.cn/20201026175016824.png'>



# 最终效果展示



<img src='https://img-blog.csdnimg.cn/20201026011936218.gif'>