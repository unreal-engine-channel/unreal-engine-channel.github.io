---
title: RPG Game开发日志(二十二)
date: 2020-11-22 15:14:25
categories: Logs
tags: UE_4.22
---

涉及内容：联网测试交易物品，RPC。

<!--more-->

# 涉及内容

说明：

为了描述方便，定义**SWAP_SLOT_INFO** =(FromInventoryIndex(Integer),ToInventoryIndex(Integer),FromInventorySlotType(E_InventorySlotType),ToInventoryType(E_InventorySlotType)）

## 联网测试交易物品

### 准备工作

#### 相关枚举及结构体

1. 创建枚举E_InventorySlotType[PlayerInventory,DrugStore]；
2. 打开S_ActorContainerInfo，新增成员InventorySlotType(E_InventorySlotType)；

#### 相关蓝图接口

##### BPI_InventoryInterface相关

1. 接口函数功能扩展：将GetContainerProperties返回值[Name,InventorySize]扩展至[Name,InventorySize,InventorySlotType]；
2. 创建接口函数[PlayersViewing(PlayerState[])GetPlayersViewing();

##### BPI_HUDInterface相关

1. 创建接口函数[void]UIOpenTransactionWindow(**SWAP_SLOT_INFO**,ItemInformation(S_InventoryItem))；
2. 创建接口函数[Level(Integer),Moneny(Integer)]GetPlayerStatus()；
3. 创建接口函数[void]AddMoney(Money)；
4. 创建接口函数[void]UISplitInventoryItem(**SWAP_SLOT_INFO**,Amount(Integer));

##### BPI_UsableActorContainer相关

1. 创建接口函数[Success(Boolean)]OnActorUsed(Controller(PlayerController))；

### BP_DragDrop相关

1. 新增变量InventorySlotType(E_InventorySlotType)；

### W_InventorySlot相关

1. 函数功能扩展：对拽取物品的来源和去向InventorySlotType类型进行判断。若相同=>若为玩家背包，则可直接拖拽；否则不执行任何动作。若不相同=>调用蓝图接口事件UIOpenTransactionWindow，完成物品交易；<img src='https://img-blog.csdnimg.cn/2020112414060153.png'>

### W_TransactionWindow相关

1. 创建函数[void]Update(**SWAP_SLOT_INFO**,ItemInformation(S_InventoryItem))，完成交易物品信息的更新；<img src='https://img-blog.csdnimg.cn/20201124142235157.png'>
2. 创建函数[Value(Integer),Success(Boolean)]CheckMoneyforShopping()，调用蓝图接口函数GetPlayerStatus获取玩家Money信息，用于检查玩家是否有足够的金额完成物品交易；<img src='https://img-blog.csdnimg.cn/20201124144439936.png'>
3. 为标题Text文本内容绑定函数；<img src='https://img-blog.csdnimg.cn/20201124142437710.png'>
4. 为交易物品Text文本内容及文本颜色绑定函数；<img src='https://img-blog.csdnimg.cn/20201124142522849.png'><img src='https://img-blog.csdnimg.cn/20201124142703490.png'>
5. 为交易数量EditableTextBox文本输入内容绑定函数，并添加OnTextCommitted事件；<img src='https://img-blog.csdnimg.cn/20201124142703490.png'><img src='https://img-blog.csdnimg.cn/20201124143142740.png'>
6. 为交易金额Text文本内容绑定函数。买入是全额，卖出将收取40%税收；<img src='https://img-blog.csdnimg.cn/20201124143451360.png'>
7. 为TextToolTip添加文本内容和文本颜色绑定函数；<img src='https://img-blog.csdnimg.cn/2020112419420578.png'><img src='https://img-blog.csdnimg.cn/20201124194330426.png'>
8. 为Minus和Plus按钮添加绑定事件；<img src='https://img-blog.csdnimg.cn/20201124194820238.png'>
9. 为Min和Max按钮添加绑定事件；<img src='https://img-blog.csdnimg.cn/20201124194928322.png'>
10. 为CancelButton和ConfirmButton添加绑定事件；<img src='https://img-blog.csdnimg.cn/20201124194629484.png'>


### C_Inventory相关

1. 创建函数[Index(Integer),Success(Boolean)]GetEmptyInventorySpace()，用于背包空闲slot的寻找；<img src='https://img-blog.csdnimg.cn/20201124182952573.png'>

### C_InventoryManager相关

1. 创建函数[void]OpenTransactionWindow(**SWAP_SLOT_INFO**,ItemInformation(S_InventoryItem))，调用函数[void]Update(**SWAP_SLOT_INFO**,ItemInformation(S_InventoryItem))；<img src='https://img-blog.csdnimg.cn/20201124144935982.png'>
2. 创建函数[void]CountMoney()，将BP_Player中的CountMoney函数迁移过来；<img src='https://img-blog.csdnimg.cn/20201124145309889.png'>
3. 创建函数[StructOut(S_InventoryItem)]AddToItemAmount(ItemInformation(S_InventoryItem),AmountToAdd(Integer))，将修改后的ItemInformation导出；<img src='https://img-blog.csdnimg.cn/20201124163958914.png'>
4. 创建函数[void]SetActorContainerSlotItem(ActorContainerIndex)；<img src='https://img-blog.csdnimg.cn/20201124181438471.png'>
5. 创建函数[void]SetViewersActorContainerSlot(ActorContainerSlot(Integer),ItemInformation(S_InventoryItem))，，调用客户端事件ClientSetActorContainerSlotItem，用于同步客户端交易信息；<img src='https://img-blog.csdnimg.cn/20201124180507375.png'>
6. 函数功能扩展：为函数AddItem添加Inventory判断，若为PlayerInventory则正常移动，否则调用函数SetViewersActorContainerSlot;<img src='https://img-blog.csdnimg.cn/20201124164936346.png'>
7. 创建函数[RemainingAmount(Integer)]AddItemToStack(InventoryComponent(C_Inventory),InventoryIndex(Integer),Amount(Integer))，根据叠加上限MaxStackSize减去叠加数目Amount计算出剩余生成数量RemainingAmount，调用函数AddToItemAmount完成InventoryItem信息修改，并返回RemainingAmount；<img src='https://img-blog.csdnimg.cn/20201124164315887.png'><img src='https://img-blog.csdnimg.cn/2020112416440786.png'><img src='https://img-blog.csdnimg.cn/20201124164435782.png'>
8. 创建函数[Struct(S_InventoryItem)]SetItemAmount(InventoryItem(S_InventoryItem),Amount(Integer))，用于物品数量的修改；<img src='https://img-blog.csdnimg.cn/20201124182550508.png'>
9. 创建函数[Output_Get(Integer),WasFullAmountRemove(Boolean),StructOut(S_InventoryItem)]RemoveFromItemAmount(InventoryItem(S_InventoryItem),AmountToRemove(Integer))；<img src='https://img-blog.csdnimg.cn/20201124191915761.png'>
10. 函数功能扩展：为函数MoveItem添加两个新的传入参数FromInventoryComponent和ToComponent
   - 判断是否为有效移动，若为有效移动，将传入参数创建为本地变量<img src='https://img-blog.csdnimg.cn/20201124184819754.png'>

   - 判断移动到另一物品上还是空白位置<img src='https://img-blog.csdnimg.cn/20201124184927617.png'>

     - 移动到空白位置<img src='https://img-blog.csdnimg.cn/20201124185055331.png'>

     - 移动到另一物品上，判断是否为相同物品<img src='https://img-blog.csdnimg.cn/2020112418501065.png'>
       - 为相同物品，判断是否超出堆叠上限<img src='https://img-blog.csdnimg.cn/20201124185330475.png'>
         - 未超出，直接进行叠加<img src='https://img-blog.csdnimg.cn/20201124185413385.png'>
         - 超出，需要寻找空白位置并额外创建物品<img src='https://img-blog.csdnimg.cn/20201124185603858.png'>
       - 不是相同物品，判断是否为玩家背包<img src='https://img-blog.csdnimg.cn/20201124185747664.png'>
         - 若为玩家背包，直接进行物品对调<img src='https://img-blog.csdnimg.cn/20201124185836885.png'>
         - 若为商店背包<img src='https://img-blog.csdnimg.cn/20201124195641468.png'>
11. 创建函数[void]SplitItem(FromInventory(C_Inventory),ToInventory(C_Inventory),FromInventoryIndex(Integer),ToInventoryIndex(Integer),Amount(Integer))，具体思路同MoveItem函数；<img src='https://img-blog.csdnimg.cn/20201124193737972.png'>
12. 创建函数[void]CloseActorContainer()，调用接口函数GetPlayersViewing和客户端事件Client_CloseActorContainer；<img src='https://img-blog.csdnimg.cn/20201124172748143.png'>
13. 创建函数[void]ClearContainerSlotItem(ContainerSlot(Integer))；<img src='https://img-blog.csdnimg.cn/20201124200200108.png'>
14. 创建函数[void]ClearViewersActorContainer(ActorContainerSlot(Integer))，调用客户端事件Client_ClearActorContainerSlotItems；<img src='https://img-blog.csdnimg.cn/2020112420035559.png'>

### BP_RPG_NPC相关

1. 实现接口函数GetPlayersViewing，将PlayerState信息导出；<img src='https://img-blog.csdnimg.cn/20201124165756372.png'>
2. 将蓝图接口BPI_UsableActorContainer添加进来，并实现接口函数OnActorUsed，调用服务端事件ServerUseContainer；<img src='https://img-blog.csdnimg.cn/20201124170731704.png'>

### RPG_PlayerController相关

1. 实现蓝图接口事件EventUIOpenTransactionWindow，调用客户端事件Client_OpenTransactionWindow；<img src='https://img-blog.csdnimg.cn/20201124141309624.png'>

2. 实现蓝图接口事件EventAddMoney，调用客户端事件Client_AddMoney；<img src='https://img-blog.csdnimg.cn/20201124152053403.png'>

3. 实现蓝图接口事件EventUISplitInventoryItem，调用客户端事件ServerSplitInventoryItem；<img src='https://img-blog.csdnimg.cn/2020112415291059.png'>

4. 实现接口函数[Level(Integer),Moneny(Integer)]GetPlayerStatus()，将服务端Replicated到客户端的变量Level和Money信息导出；<img src='https://img-blog.csdnimg.cn/20201124150829781.png'>

5. 创建函数[HitActor(Actor)]GetUsableActor()，HitActor在BP_Player中完成设置；

   <img src='https://img-blog.csdnimg.cn/20201124171800115.png'><img src='https://img-blog.csdnimg.cn/202011241715382.png'>

6. 创建函数[void]OnActorUsed()，调用函数GetUsableActor和接口函数OnActorUsed；<img src='https://img-blog.csdnimg.cn/20201124172200596.png'>

## RPC(Remote Procedure Call)

### Run on Server(Reliable)

#### Server_SplitInventoryItem

在C_InventoryManager事件图表中创建，调用函数SplitItem；<img src='https://img-blog.csdnimg.cn/20201124153530678.png'>

#### Server_OnActorUsed

在RPG_PlayerController事件图表中创建，调用函数OnActorUsed和客户端事件Client_OpenInventory；<img src='https://img-blog.csdnimg.cn/20201124171333609.png'>

#### Server_CloseActorContainer

在C_InventoryManager事件图表中创建，调用函数CloseActorContainer；<img src='https://img-blog.csdnimg.cn/20201124195158920.png'>

#### Server_TransactionItem

在W_TransactionWindow事件图表中创建；<img src='https://img-blog.csdnimg.cn/20201124195108141.png'>

### Run on owning Client(Reliable)

#### Client_OpenTransactionWindow

在C_InventoryManager事件图表中创建，调用函数OpenTransactionWindow；<img src='https://img-blog.csdnimg.cn/20201124140959167.png'>

#### Client_AddMoney

在C_InventoryManager事件图表中创建，将BP_Player中的自定义事件GetMoney迁移过来；<img src='https://img-blog.csdnimg.cn/20201124145658353.png'>

#### Client_CloseActorContainer

在C_InventoryManager事件图表中创建，调用函数CloseActorContainerWindow;<img src='https://img-blog.csdnimg.cn/20201124173309109.png'>

#### Client_SetActorContainerSlotItem

在C_InventoryManager事件图表中创建，调用函数SetActorContainerSlotItem;<img src='https://img-blog.csdnimg.cn/20201124181051489.png'>

#### Client_ClearActorContainerSlotItems

在C_InventoryManager事件图表中创建，调用函数ClearContainerSlotItem;<img src='https://img-blog.csdnimg.cn/20201124195950691.png'>

# 最终效果展示

(Bug尚未解决，后续补上)