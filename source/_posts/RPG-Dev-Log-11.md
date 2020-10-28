---
title: RPG Game开发日志(十一)
date: 2020-10-27 18:21:00
categories: Logs
tags: UE_4.22 
---

涉及内容：战斗系统（续篇），AI设置，蒙太奇动画。

<!--more-->

# 涉及内容

## 蒙太奇动画

### 玩家被击蒙太奇

1. 添加ShinBi被攻击时的骨骼动画，创建Montage，将HitRect_Back、HitRect_Font、HitReact_Left、HitRect_Right分成4个section添加进来;<img src='https://img-blog.csdnimg.cn/20201027183226400.png'>
2. 打开BP_Shibi事件图表，在OnRecieveDamage事件添加蒙太奇动画，每次角色被攻击时，随机播放4个section中的一个；<img src='https://img-blog.csdnimg.cn/20201027183642919.png'>

### 敌人被击蒙太奇

1. 创建[AM_React_LightHit]蒙太奇，添加轻击时的骨骼动画，共2种,分成2个Section;<img src='https://img-blog.csdnimg.cn/20201027190551416.png'>
2. 创建[AM_React_HeavyHit]蒙太奇，添加两种重击时的骨骼动画，共2种Section,分成2个Section；<img src='https://img-blog.csdnimg.cn/20201027190633867.png'>
3. 打开BP_Enemy事件图表，在OnRecieveDamage事件添加蒙太奇动画，每次敌人被攻击时，随机播放2个section中的一个，根据伤害是否暴击播放不同的蒙太奇动画；<img src='https://img-blog.csdnimg.cn/20201027191532830.png'>


### 敌人死亡蒙太奇

1. 创建[AM_Death]蒙太奇，添加敌人死亡时的骨骼动画，共7种，分成7个Section；<img src='https://img-blog.csdnimg.cn/20201027191035160.png'>
2. 打开BP_Enemy事件图表，在OnDeath事件添加敌人死亡的随机蒙太奇动画播放，随机个数为7；<img src='https://img-blog.csdnimg.cn/20201027194633761.png'>

## 战斗系统（续篇）

### 玩家相关

1. 打开BP_Shibi，添加自定义事件LineAttack，调用OnRecieveDamage事件，用于判定玩家是否成功命中敌人；<img src='https://img-blog.csdnimg.cn/20201027184947351.png'>
2. 打开三连攻击和空中攻击Montage动画，在PlaySound后添加AttackHit动画通知；在动画图表中调用BP_Shibi中的自定义事件LineAttack;<img src='https://img-blog.csdnimg.cn/20201027184701875.png'>
3. 修改函数[void]ModifyAttribute(Attribute,ModifyValue,Critical);
4. 创建函数[void]UpdateLevel()，用于玩家升级后自身属性和技能点的更新；<img src='https://img-blog.csdnimg.cn/20201027213303392.png'>
5. 打开事件图表，修改自定义事件UpLevel，调用UpdateLevel()完成所有属性的更新；<img src='https://img-blog.csdnimg.cn/2020102721360097.png'>
6. 修改MeleeAttack（普通三连攻击）和Space Bar（二段跳)事件，实装耐力值消耗；
7. 创建自定义事件CoolDownFighting和CountTime，用于玩家当前是否脱战的判定;<img src='https://img-blog.csdnimg.cn/20201027215037170.png'>
8. 精简Regeneration分类下的所有Tick事件的功能；

### 敌人相关

1. 打开E_Behavior，添加新的AI行为Death；
2. 打开BP_Enemy，添加自定义事件ModifyValue(Boolean:Critical,Float:Damage),用于敌人攻击玩家时的伤害显示，并根据自身当前血量，设置Death或HasSeenPlayer行为；<img src='https://img-blog.csdnimg.cn/20201027193429331.png'>
3. 添加OnRecieveDamage事件,接收来自玩家的伤害数据，调用事件ModifyValue,计算伤害并应用伤害;<img src='https://img-blog.csdnimg.cn/20201027193207452.png'>
4. 添加自定义事件OnDeath,用于敌人死亡时的动画播放和自身销毁（如果有武器，武器也销毁）；<img src='https://img-blog.csdnimg.cn/20201027223511642.png'>
5. 创建函数[Boolean] IsPlayingMontage()，用于判断当前是否已添加蒙太奇；<img src='https://img-blog.csdnimg.cn/2020102719201244.png'>
6. 创建函数[void]UpdateAttribute,用于不同等级的敌人的属性更新（在敌人生成时调用）；<img src='https://img-blog.csdnimg.cn/20201027195155263.png'>
7. 创建函数[void]SetNameAndLevel(Integer),此函数除了设置NameAndHealthBar的文本外，还可根据玩家和敌人的等级差，设置不同的文本颜色渐变(在敌人生成时调用）；<img src='https://img-blog.csdnimg.cn/2020102720012115.png'><img src='https://img-blog.csdnimg.cn/20201027200251916.png'>
9. 添加胶囊碰撞体ShowEnemyWidget，并添加BeginOverlap和EndOverlap事件，当玩家与敌人的胶囊体发生碰撞时，显示NameAndHealthBar，离开胶囊体区域时隐藏；<img src='https://img-blog.csdnimg.cn/20201027200721727.png'>

## AI设置

### 敌人攻击流畅度优化

1. 为BTTask_Attack添加Decorator[CoolDown]，冷却时间设置为1s；
2. 打开BTTask_Attack，新增蒙太奇动画检测，若敌人正在攻击则延迟播放；<img src='https://img-blog.csdnimg.cn/20201027192842439.png'>

### 敌人死亡Sequence

1. 创建BTTask_Death，调用BP_Enemy中的OnDeath事件；<img src='https://img-blog.csdnimg.cn/20201027194039645.png'>
2. 创建Sequence并添加Blackboard Decorator[EventDeath],设置BlackboardKey=Behavior,KeyValue=Death，序列执行顺序为:BTTask_Death—>Wait(5)（then Loop);<img src='https://img-blog.csdnimg.cn/20201027194357652.png'>

### 离开敌人仇恨范围后脱战

1. 打开E_Behavior，添加新的AI行为RunBack；
2. 创建服务BTS_CheckOriginalDistance，用于判断玩家是否已经离开了敌人AI的仇恨范围，并执行RunBack行为；<img src='https://img-blog.csdnimg.cn/20201027220346975.png'>
3. 打开BB_Base，添加新的键值IsRunningBack?(Boolean)和RandomRange(Float);
4. 将服务BTS_CheckOriginalDistance添加至Selector(HasSeenPlayer)，并添加Blackboard Decorator[IsNotRunningBack]至子节点sequence，设置BlackboardKey=IsRunningBack?;
5. 打开BTTask_RandomLocation，将浮点型变量RandomRadius修改为BlackBoardKey类型，并在BP_Enemy BeginPlay中添加传值；
6. 创建任务BTTask_Reset，用于判断敌人AI是否已经脱仇，并执行来自BP_Enemy的Reset事件；<img src='https://img-blog.csdnimg.cn/2020102722221863.png'><img src='https://img-blog.csdnimg.cn/20201027222301945.png'>
7. 创建Sequence并添加Blackboard Decorator[RunBack],设置BlackboardKey=Behavior,KeyValue=RunBack，序列执行顺序为:MoveTo(Original)—>BTTask_Reset（then Loop);<img src='https://img-blog.csdnimg.cn/20201027222635974.png'>

# 最终效果展示

<img src='https://img-blog.csdnimg.cn/2020102813041126.gif'>

<img src='https://img-blog.csdnimg.cn/20201028130602858.gif'>