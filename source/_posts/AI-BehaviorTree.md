---
title: 游戏人工智能:从FSM到Behavior Tree
date: 2020-11-01 21:34:44
categories: Theories
tags: AI BehaviorTree FSM
---

> A behavior tree is to express the behavior of artificial intelligence. The behavior tree has a characteristic that is easy to change state transitions than FSM(Finite State Machine).

<!--more-->

# 概述

以RPG游戏为例，无论是PVE模式还是PVP模式，都会涉及人工智能这个概念。其中包括引导新手的NPC，玩家对战的NPC等。

## FSM(Finite State Machine)

双FSM是目前最常用的一样方法，它定义了有限多个状态（行为），用图表来表达状态之间的因果关系。虽然理解状态的组成和转移很容易，但有个问题：状态和转移越多，维护就越困难。

<img src='https://img-blog.csdnimg.cn/20201101214601997.png' alt='[Fig.1] FSM'>

### [Fig.1] FSM

图示给出了FSM的一个例子。它由Search、Chase和Attack状态组成，状态转换条件显示在箭头之上。为了改进FSM，实现更加精密的人工智能开发，提出了行为树的概念。它仍然具有类似于FSM的形式，但是即便状态和状态之间的关系再多，管理难度也远小于FSM。它更适合具有复杂行为和功能的智能体。

## Behavior Tree

<img src='https://img-blog.csdnimg.cn/20201101220407953.png' alt='[Fig.2] Behavior Tree'>

### [Fig.2] Behavior Tree

图示给出了一个行为树的例子。这些行为树在PRG/RTS/FPS等类型的游戏中都具备可用性。对于单个对象的行为或控制逻辑，我们认为没有必要使用行为树。

### 原理和方法

行为树是通过将AI的角色行为定义为树的方法，通过连接角色行为的条件和因果关系，优先级等对应的节点来完成。 

### Unreal Engine中的BT

#### 注解

- Task：基础节点。
- Decorator：定义当前分支返回结果是否为真。
- Sequence：顺序执行子节点，直到有一个返回失败。如果所有的子节点都返回成功，则该序列也返回成功。
- Selector：顺序执行子节点，直到有一个返回成功。如果所有的子节点都返回失败，则该选择器也返回失败。
- Service：在固定更新频率的黑板上运行。

<img src='https://img-blog.csdnimg.cn/20201101222235173.png' alt='[Table 2] Unreal Engine'>

#### 示例

<img src='https://img-blog.csdnimg.cn/20201101225753418.png' alt='[Fig.5] Unreal Engine Behavior Tree'>

### Unity Engine中的BT

#### 注解

- Action：基础节点。
- Decorator：定义当前分支返回结果是否为真。
- Sequence：顺序执行子节点，直到有一个返回失败。如果所有的子节点都返回成功，则该序列也返回成功。
- Selector：顺序执行子节点，直到有一个返回成功。如果所有的子节点都返回失败，则该选择器也返回失败。
- Priority selector：决定最高优先级的节点。
- Parallel: 平行子节点将被同时执行。

<img src='https://img-blog.csdnimg.cn/20201101222050462.png' alt='[Table 1] Unity Engine'>

#### 示例

<img src='https://img-blog.csdnimg.cn/20201101230244379.png' alt='[Fig.6] Unity Engine Behavior Tree'>

#### 伪代码

```python
def sequence_node(owner, node_name):
    run_index = get_run_index(node_name)
    while run_index < len(child_list):
        res = child_list[run_index](owner)
        if res == FAILURE:
            set_run_index(node_name, 0)
            return res
        elif res == RUNNING:
            set_run_index(node_name, run_index)
            return res
        else:
            run_index += 1
    set_run_index(node_name, 0)
    return SUCCESS

def selector_node(owner, node_name):
    run_index = get_run_index(node_name)
    while run_index < len(child_list):
        res = child_list[run_index](owner)
        if res == SUCCESS:
            set_run_index(node_name, 0)
        elif res == RUNNING:
            set_run_index(node_name, run_index)
        else:
            run_index += 1
    set_run_index(node_name, 0)
    return FAILURE
    
 def parallel(owner, node_name):
    run_index_list = get_run_index(node_name)
    if len(run_index_list) > 0:
        for index in run_index_list:
            state_dic[index] = child_list[index](owner)
    else:
        run_index = 0
        while run_index < len(child_list):
            state_dic[run_index] = child_list[run_index](owner)
            run_index += 1
    if state_dic.count(SUCCESS) >= MIN_SUCCESS:
        return SUCCESS
    elif state_dic.count(FAILURE) >= MIN_FAILURE:
        return FAILURE
    else:
        return RUNNING
```



### 两大引擎中的BT对比(参照UE4)

​		两个引擎的行为树具有很多共同点，就是可以很容易地了解一个行为实现的条件和下一个行为是什么。

我们定义一组AI行为：

  1. [EnemyNotInRange]以AI当前位置为中心，在一定范围内没有基地的情况下，为了寻找基地，AI将自主移动前进。

  2. [EnemyInRange]若在前进途中遇到敌人，则攻击它，受到伤害降低生命值。

  3. [NotHealth]若生命值为0，则死亡，否则继续执行1。

  4. [EnemyBaseInRange]若发现基地，则攻击基地。

<img src='https://img-blog.csdnimg.cn/20201101225148412.png'  alt='[Fig.3] Unreal Engine Behavior Tree'>

<img src='https://img-blog.csdnimg.cn/20201101230351316.png' alt='[Fig.4] Unity Engine Behavior Tree'>

#### 行为树为事件驱动型

> 行为树可避免在每帧中执行大量工作。行为树并不会经常检查相关变化是否已经发生，它只会被动地等待在树中引起变化的“事件”。
>
> 事件驱动型结构对运行性能和除错大有帮助。如果希望获得这些帮助，必须理解虚幻引擎树的其他不同点，并正确搭建行为树结构。
>
> 代码在整个树的每个标记中无需迭代，因此运行性能将大大提升！从概念上而言，我们不需要不停地问“我们到了吗”，只需要轻松休息，会有人来告诉我们“到啦！”
>
> 在行为树执行日志中反复查看对行为进行纠错时，最佳的方法是让日志显示相关的变更，不显示无关的变更。在事件驱动型应用中，无需过滤出在树上迭代并选择之前相同行为的无关步骤，因为额外的迭代从开始便并未发生！只有树中执行位置或黑板数值发生的变化会产生影响，不同之处显而易见。

#### 条件语句并非叶节点

> 在行为树标准模型中，条件语句为 Task 叶节点，除成功和失败外不执行任何操作。虽然也可以使用传统的条件 tasks，但还是强烈推荐您使用 Decorator 系统作为条件语句。
>
> 使用 decorators 而非 tasks 作为条件语句有多个明显优点。
>
> 首先，条件 decorators 使行为树 UI 显得更加直观易读。条件语句位于其所控制分支树的根部，因此一旦条件语句未被满足，便可直接看到树的哪个部分被“关闭”。此外，所有叶节点均为行动 tasks，因此更容易分辨树在对哪些实际行动下达命令。 在传统模型中，条件语句混于叶节点中，因此需要花更多时间分辨哪些叶节点是条件语句，哪些叶节点是行动。

#### 并发行为的特殊处理

> 标准行为树通常使用 **Parallel composite** 节点来处理并发行为。Parallel 节点同时执行其所有子项。在一个或多个子项树完成时，特殊规则将决定如何执行（取决于所需行为）。
>
> Parallel 节点不是绝对的多线程（完全同步执行任务）。它们只是概念上一次执行多个任务的方式。它们经常在同一线程上运行，并在一些序列中开始。该序列应为不相关，因为它们全部在同一框架中发生。但有时它也很重要。
>
> UE4 行为树抛弃了复杂 Parallel 节点，使用 **Simple Parallel** 节点和称为 **Services** 的特殊节点，以实现同类行为。
>
> #### Simple Parallel 节点
>
> Simple Parallel 节点只允许两个子项的存在：一个必为单独任务节点（含可选 decorators）、另一个为一个完整的分支树。
>
> 可以将 Simple Parallel 节点理解为：“执行 A 的同时也执行 B。”举例而言：“在攻击敌人时向敌人移动。”基本上来说，A 是主要任务，而 B 是等待 A 完成时执行的次要或补充任务。
>
> 有一些选项可以处理同时进行的次要任务（任务 B）。较之于传统 Parallel 节点，该节点在概念上还是相对简单。然而，它支持 Parallel 节点的多数常规用法。
>
> 可利用 Simple Parallel 节点简便地进行一些事件驱动型优化。而 Full Parallel 节点的优化则要复杂许多。
>
> #### Services
>
> **Services** 是与 composite 节点（Selector、Sequence、或 Simple Parallel）相关的特殊节点。它能在每 X 秒注册回呼并执行多个定期发生类型的更新。
>
> 举例而言，service 能用于确定哪名敌人是 AI pawn 的最佳追逐目标；与此同时，pawn 在其行为树中继续正常追踪当前敌人。
>
> 只有执行仍然处于以 composite 节点（与 service 相关）为根的分支树中时，Services 方为有效。
>
> #### Decorator “Observer Aborts” 属性
>
> 标准 Parallel 节点的一个常见用途是不断对条件进行检查，以便在条件无法达成时中止任务。举例而言，有一只猫执行序列，如“摇屁股”或“猛扑”，在老鼠已经逃进洞后，直接放弃任务。如使用 Parallel 节点，有一个子项将检查猫是否可以扑到老鼠，另一子项就是要执行的序列。因为 UE4 行为树是事件驱动型，条件 decorators 将观察数值，并在需要时中止任务。（在该例中，只需要在序列上添加“老鼠是否能被扑到” decorator，然后将 “Observer Aborts” 设为 “Self”。）

# 参阅

[1].Myoun-Jae Lee.A Proposal on Game Engine Behavior Tree[J].Journal of Digital Convergence,2016,14(8),415-421.