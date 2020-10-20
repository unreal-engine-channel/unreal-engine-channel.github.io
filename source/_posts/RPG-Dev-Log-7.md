---
title: RPG Game开发日志(七)
date: 2020-10-20 21:19:37
categories: Logs
tags: UE_4.22 
---

涉及内容：AI初讲，BehaviorTree--Patrol等;

<!--more-->

# 涉及内容

## 角色边界跳跃

1. 在场景中添加NavLinkProxy，设置PointLink[0].Left和PointLinks[0].Right在场景中的位置；
2. 打开细节面板，按照实际情况将Simple Link和Smart Link Direction都设置为Right to Left，将Area Class都设置为NavArea_Default;<img src='https://img-blog.csdnimg.cn/20201020213951911.png'>



## AI行为树初讲

### PatrolPath

1. 清除场景中的所有Block Volume;
2. 创建Actor [BP_PatrolPath],添加组件Scene、Billboard、Spine;
3. 进入事件图表，创建变量CloseLoop（Boolean),在构造函数中设置ClosedLoop;<img src='https://img-blog.csdnimg.cn/20201020215735408.png'>
4. CloseLoop可用于设置AI巡逻路线是否闭环；<img src='https://img-blog.csdnimg.cn/20201020220053700.png'>
5. 在场景中添加BP_PatrolPath，并自定义路线；<img src='https://img-blog.csdnimg.cn/20201020220243597.png'>

### PatrolComponent

1. 创建Component [PatrolComponent],添加变量PatrolPath(BP_PatrolPath)、PatrolIndex(Integer)、RevereDirection(Boolean);
2. 创建纯函数[Vector] GetSpinePointLocation(Integer)，用于获取巡逻路线各个节点在世界坐标系中的位置；<img src='https://img-blog.csdnimg.cn/20201020220912376.png'>
3. 创建函数[void] UpdatePatrolIndex()，用于实时更新PatrolIndex；当ColoseLoop=true时，实现闭环寻路；当CloseLoop=false时，实现来回寻路；<img src='https://img-blog.csdnimg.cn/20201020221021193.png'>
4. 为BP_RPG_Character添加组件PatrolComponent;
5. 点击场景中的巡逻NPC，将场景中的PatrolPath指定给NPC;



### BehaviorTree--Patrol

1. 创建Enum [Behaviors] = {Idle,Patrol,Hit,Ability};

2. 创建BehaviorTree [BT_NPC]；

3. 创建BlackBoard [BB_Base],新建Keys [Behaviors],类型为Enum Behaviors;

4. 新建Service [BTS_UpdateBehaviorTree],添加变量BehaviorKey(Blackboard Key),添加宏[Exec] SetBehavior(Behaviors,Exec),将BehaviorKey设置为枚举值；<img src='https://img-blog.csdnimg.cn/20201020222511510.png'>

5. 在Event Receive Tick AI中调用SetBehavior，选择枚举类型为Patrol;<img src='https://img-blog.csdnimg.cn/20201020222734181.png'>

6. 新建Task [BTTask_Patrol],添加事件Event Receive Execute AI和Event Receive Abort AI,用于AI行为的执行和中断；<img src='https://img-blog.csdnimg.cn/20201020223107766.png'>

7. 打开BT_NPC，建立Root——Selector(Service)——Sequence(Decorator)——Leaf(Task);

   > 行为树节点
   >
   > 一、Composite组合节点：
   >
   > 1、Selector
   >
   > 　　要求比较低：只要有一个子节点成功就可以了。
   >
   > 　　只要子节点有一个返回true，则停止执行其它子节点，并且Selector返回true。如果所有子节点都返回false，则Selector返回false。
   >
   > 2、Sequence
   >
   > 　　要求比较高：期望所有子节点都成功。
   >
   > 　　只要有一个子节点返回false，则停止执行其它子节点，并且Sequence返回false。如果所有子节点都返回true，则Sequence返回true。
   >
   > 二、Task叶子：
   >
   > 　　实际执行操作，不含输出链接。
   >
   > 三、Decorator装饰节点
   >
   > 　　依附于其它节点，条件节点：如果返回true则执行位于“Decorator”下面的节点，否则就不执行。
   >
   > 四、Services服务节点
   >
   > 　　只能附加在Composite组合节点上，只要其附加的节点被执行，Services节点就会被执行，通常用来检查和更新黑板。

   <img src='https://img-blog.csdnimg.cn/20201020223938437.png'>

8. 选取父类AIController，创建蓝图AIC_NPCController,进入蓝图，在EventBeginPlay添加BlackBoard [BB_Base]和BehaviorTree [BT_NPC];<img src='https://img-blog.csdnimg.cn/20201020214613446.png'>

9. 选择场景中的NPC，设置

   Animation Mode=Use Animaion Blueprint;

   Pawn>Auto Possess AI = Placed in World or Spawned,AI Controller Class = AIC_NPCController;

   

# 最终效果展示

<img src='https://img-blog.csdnimg.cn/2020102021303566.gif'>