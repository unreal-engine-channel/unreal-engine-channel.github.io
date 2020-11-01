---
title: RPG Game开发日志(十二)
date: 2020-11-01 16:01:47
categories: Logs
tags: UE_4.22
---

涉及内容：制作简单的粒子特效，获得经验金币，重生Enemy。

<!--more-->

# 涉及内容

## 敌人死后消亡特效

1. 创建MaterialFunction[MF_DissolveEffect]，添加Input[Visibility]，Type=Function Input Scalar；<img src='https://img-blog.csdnimg.cn/2020110116235018.png'>
2. 打开Enemy材质球，调用MF_DissolveEffect，添加OnHit和Visibility参数，设置EmissiveColor和OpacityMask；<img src='https://img-blog.csdnimg.cn/20201101162845368.png'>
3. 创建Enemy材质球实例，选择PreviewMesh，设置EffectColor，调整OnHit和Visibility的值可预览Enemy被攻击时、消亡时的材质效果；<img src='https://img-blog.csdnimg.cn/20201101164744129.png'>
4. 创建AnimationNotify[ANS_DissolveEffect]，添加重载函数Received_NotifyTick，根据动画曲线[Dissolve_Spawn]渐变Visibility的值；<img src='https://img-blog.csdnimg.cn/20201101165355339.png'>
5. 为Enemy的七种随机死亡动画添加Notify[ANS_DissolveEffect]和Curve[Dissolve_Spawn];<img src='https://img-blog.csdnimg.cn/20201101165816602.png'>

## 击杀敌人获得战利品

### 获取经验值

#### 创建粒子特效

1. 创建材质球[P_Base]，添加Input[ParticleColor]，设置EmissiveColor和Opacity;<img src='https://img-blog.csdnimg.cn/20201101170341930.png'>

2. 创建材质球实例[P_Base_Inst];

3. 打开初学者内容包里的P_Teleport_Tell_Source粒子特效，重命名为P_SoulXP，并打开;

4. 对于第一个ParticleEmitter，将Emitter Duration设置为0，添加DirectLocation;

5. 对于第二个ParticleEmitter，直接关闭;

6. 对于第三个ParticleEmitter，将Emitter Duration设置为0，略微调大InitialSize，调整Sphere StartLocation的Z轴；

7. 新建一个ParticleEmitter，将Material设置为P_Base_Inst，Spawn Rate设置为0，Burst Count设置为1，删除InitVelocity，调整ColorOverLife，添加Direct Location；

8. 新建一个ParticleEmitter,TypeData设置为GPU Sprites,设置Spawn Distribution为Distribution Float Constant,添加Sphere并设置颜色;<img src='https://img-blog.csdnimg.cn/20201101172623580.png'>

#### 创建BP_XP

1. 创建Actor[BP_XP]，添加组件Sphere Collision，设置半径，应用物理属性，设置重量为3kg，更改碰撞预设为Custom;<img src='https://img-blog.csdnimg.cn/20201101173214860.png'>

2. 添加组件ParticleSystem，将Particle Template设置为P_SoulXP;

3. 打开BP_Player事件图表，创建自定义事件AddXP(XP);<img src='https://img-blog.csdnimg.cn/20201101174006324.png'>

4. 打开事件图表，设置物理模拟，添加TimeLine[MoveToPlayer],实现玩家击杀敌人后吸收经验的效果。创建自定义事件FinishCollect，调用自定义事件AddXP；<img src='https://img-blog.csdnimg.cn/20201101174735841.png'><img src='https://img-blog.csdnimg.cn/20201101174658471.png'>

#### 设置EnemyXP

1. 打开BP_Enemy,打开函数SetNameAndLevel，创建变量XP_Points，扩展函数功能，可通过敌人和玩家的等级差设置XP_Points(默认为10)，具体为：

   XP_Points = 10 + 2*(EnemyLevel-1) + MAX(0,EnemyLevel-PlayerLevel);<img src='https://img-blog.csdnimg.cn/20201101175450907.png'>

2. 接上，函数UpdateAttribute中的修改；<img src='https://img-blog.csdnimg.cn/20201101175712965.png'>

3. 创建函数[Void]SpawnXPAndMoney(FirstIndex),用于经验球和金币的生成,并在自定义事件OnDeath中调用；<img src='https://img-blog.csdnimg.cn/20201101180333637.png'><img src='https://img-blog.csdnimg.cn/2020110118043023.png'>

   

### 获取金币

#### 创建BP_SpawnMoney

1. 复制BP_XP,重命名为BP_SpawnMoney，删除ParticleSystem，添加StaticMesh和RotatingMovement；
2. 设置StaticMesh为SM_Coin_Small，编辑材质球，调整EmissiveColor,提高亮度;
3. 修改自定义事件FinishCollect，调用玩家蓝图中的自定义事件GetMoney;<img src='https://img-blog.csdnimg.cn/20201101181026191.png'>

#### 设置EnemyMoney

1. 函数UpdateAttribute中的修改；<img src='https://img-blog.csdnimg.cn/20201101181416558.png'>

2. 函数SpawnXPAndMoney中关于金币生成的部分，在OnDeath中调用；<img src='https://img-blog.csdnimg.cn/20201101181603620.png'>

   

## 重生Enemy

1. 创建结构体S_RespawnEnemy;<img src='https://img-blog.csdnimg.cn/20201101182318659.png'>
2. 创建Actor[BP_SpawnEnemy]，添加组件SpehreCollison，碰撞预设为NoCollison;
3. 打开事件图表，创建变量EnemyList(S_RespawnEnemy),SpawnTimer(TimerHandle),IndicesToSpawn(Integer[]);
4. 创建自定义事件RespawnEnemy(S_RespawnEnemy)，将死亡的敌人记录到EnemtList中，并用SpawnTimer控制自定义事件RespawnTick的调用，IndicesToSpawn数组用于记录敌人重生的索引顺序；<img src='https://img-blog.csdnimg.cn/20201101184746163.png'><img src='https://img-blog.csdnimg.cn/20201101184831482.png'><img src='https://img-blog.csdnimg.cn/20201101184908545.png'>
5. 打开BP_Enemy，在OnDeath中调用RespawnEnemy;<img src='https://img-blog.csdnimg.cn/2020110118502269.png'>
6. 打开BehaviorComponent，修改函数SetBehavior，对传入的AI行为进行判断,若与当前的AI行为不同，则将传入的AI行为设置为当前行为；<img src='https://img-blog.csdnimg.cn/2020110118562015.png'>
7. 打开BTS_UpdateBehaviorTree，添加重载事件Recieve Tick AI；<img src='https://img-blog.csdnimg.cn/20201101185829997.png'>
8. 设置BP_Enemy,Auto Prossess AI = Placed in World or Spawned；
9. 设置Enemy_Grunting，Auto Prossess AI = Placed in World or Spawned，并设置默认属性值，敌人等级的随机生成；<img src='https://img-blog.csdnimg.cn/20201101190150116.png'>
10. 将BP_SpawnSphere添加到场景，并指定场景中的Enemy的RespawnActor为BP_SpawnSphere;




# 最终效果展示

<img src='https://img-blog.csdnimg.cn/2020110116105649.gif'>