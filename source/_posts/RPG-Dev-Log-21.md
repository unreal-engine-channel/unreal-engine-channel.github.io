---
title: RPG Game开发日志(二十一)
date: 2020-11-21 16:41:36
categories: Logs
tags: UE_4.22
---

涉及内容：创建NPC交易窗口，RPC。

<!--more-->

# 涉及内容

## 创建NPC交易窗口

### 准备工作

1. 打开E_ItemType，新增枚举[恢复物品]；
2. 打开ItemList，将Mana_Blue和Health_Red的ItemType更改为恢复物品；
3. 创建结构体S_ActorContainerInfo，包含商店名字和商店格子数信息；<img src='https://img-blog.csdnimg.cn/20201121180731205.png'>

### BPI_InventoryInterface相关

1. 创建接口函数[Name(Text),InventorySize(Integer)]GetContainerProperties()；
2. 创建接口函数[InventoryComponent(C_Inventory)]GetContainerInventory()：

### BPI_HUDInterface相关

1. 新增接口函数[void]UIOpenDrugStore(NPCActor(Actor))；

### W_TransactionWindow相关

1. 创建控件W_TransActionWindow，作为玩家与NPC交易的窗口；<img src='https://img-blog.csdnimg.cn/20201121190422457.png'>

### W_Window相关

1. 创建函数[void]InitializeWindow()，用于初始化窗口大小，并根据是否为玩家背包设置与背包有关的UI控件的可视性。此函数在EventPreConstruct中调用；<img src='https://img-blog.csdnimg.cn/20201121182921756.png'>

### W_PC_Main相关

1. 复制W_InventoryWindow并重命名为W_StoreWindow，作为商店窗口，并修改默认属性；

### BP_RPG_NPC相关

1. 添加组件C_Inventory，初始化构造函数中调用服务端事件ServerInitializeInventory，初始化商店格数InventorySize为40；<img src='https://img-blog.csdnimg.cn/20201121165624322.png'>
2. 创建函数[LocalInventoryItems(S_InventoryItem[])]GetDrugStoreList()，用于返回ItemList中类型为恢复物品的物品；<img src='https://img-blog.csdnimg.cn/20201121170545847.png'>
3. 创建函数[Success(Boolean)]LoadingInventoryItems(InventorySize(Integer),InventoryItems(S_InventoryItem[]))，调用C_Inventory中的LoadInventoryItem，将InventoryItems中的物品信息加载到C_Inventory；<img src='https://img-blog.csdnimg.cn/20201121171623894.png'>
4. 创建函数[Success(Boolean)]InitializeInventory()，调用函数GetDrugStoreList和LoadingInventoryItems完成药品商店初始化。此函数在事件图表EventBeginPlay中调用；<img src='https://img-blog.csdnimg.cn/202011211719312.png'>
5. 打开ClassSettings，将蓝图接口BPI_InventoryInterface添加进来，并实现接口函数GetContainerProperties和GetContainerInventory；<img src='https://img-blog.csdnimg.cn/20201121173006831.png'><img src='https://img-blog.csdnimg.cn/20201121173047376.png'>
6. 实现自定义事件OnClickedDrugStore，调用接口函数UIOpenDrugStore，并调用OnClickedEndTalk，打开商店窗口时结束对话；<img src='https://img-blog.csdnimg.cn/2020112118591311.png'>

### C_InventoryManager相关

1. 创建函数[void]ClearActorContainerSize()，用于初始化W_Window中的变量AcotrContainerSlots并清空InventorySlotWrapBox中的子控件；<img src='https://img-blog.csdnimg.cn/20201121183836481.png'>
2. 创建函数[void]CreateContainerSize(ContainerSize(Interger))，调用函数ClearActorContainerSize，并根据AcotrContainerSlots创建W_InventorySlot的控件实例，并添加至InventorySlotWrapBox中；<img src='https://img-blog.csdnimg.cn/20201121184312962.png'>
3. 创建函数[void]SetStoreInventorySlot(StoreContainerIndex(Integer),ItemInformation(S_InventoryItem))，根据商店物品索引从ActorContainerSlots中获取ItemInformation；<img src='https://img-blog.csdnimg.cn/20201121184626593.png'>
4. 创建函数[void]OpenStoreContainerWindow()和[void]CloseStoreContainerWindow()，控制商店的显示和隐藏；<img src='https://img-blog.csdnimg.cn/20201121185413105.png'><img src='https://img-blog.csdnimg.cn/20201121185439139.png'>
5. 创建函数[void]LocalContainerSlots(ItemInformation(S_InventoryItem),ActorContainerProperties(S_ActorContainerInfo))，调用函数CreateContainerSize和SetStoreInventorySlot，循环添加商店物品，循环结束调用函数OpenStoreContainerWindow，打开商店窗口；<img src='https://img-blog.csdnimg.cn/20201121181311172.png'>
6. 创建函数[void]OpenContainer(ActorContainer(Actor))，调用接口函数GetContainerProperties和GetContainerInventory，并调用客户端事件Client_OpenContainer;<img src='https://img-blog.csdnimg.cn/2020112118024116.png'>
7. 创建函数[void]UseContainer(ActorContainer(Actor))，调用函数OpenContainer；<img src='https://img-blog.csdnimg.cn/20201121175646264.png'>

### RPG_PlayerController相关

1. 调用蓝图接口事件EventUIOpenDrugStore，调用服务端事件ServerUseContainer和ClientOpenInventory；<img src='https://img-blog.csdnimg.cn/20201121174630228.png'>

## RPC(Remote Procedure Call)

### Run on Server(Reliable)

#### Server_UseContainer

在C_InventoryManager事件图表中创建，调用函数UseContainer；<img src='https://img-blog.csdnimg.cn/20201121175304205.png'>

### Run on owning Client(Reliable)

#### Client_OpenContainer

在C_InventoryManager事件图表中创建，调用函数LocalContainerSlots；<img src='https://img-blog.csdnimg.cn/20201122145956711.png'>

# 最终效果展示

(Bug尚未解决，后续补上)