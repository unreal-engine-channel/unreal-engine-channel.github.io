---
title: RPG Game开发日志(十九)
date: 2020-11-17 15:43:14
categories: Logs
tags: UE_4.22
---

涉及内容：背包系统数据绑定，服务端客户端初讲，RPC。

<!--more-->

# 涉及内容

## 背包系统数据绑定

### 相关枚举

1. 创建枚举E_ItemType [其他物品,武器装备,消耗物品,任务物品]，用于设置物品类别；
2. 创建枚举E_QualityType [Poor,Common,Uncommon,Rare,Epic,Legendary]，用于设置物品品级；

### 相关结构体

1. 创建结构体S_InventoryItem，用于设置物品的ID、图标、名字、描述、品级、类别、数量、价值、是否可堆叠、堆叠上限、是否可拖拽等信息；<img src='https://img-blog.csdnimg.cn/2020111716025619.png'>

2. 创建结构体S_InventoryID，用于设置物品ID等从服务端发送过来的简单字符串；<img src='https://img-blog.csdnimg.cn/20201117171747606.png'>

3. 创建结构体S_QualityColor，用于设置不同品级的物品颜色；<img src='https://img-blog.csdnimg.cn/20201117181151413.png'>

### 创建数据表ItemList

1. 选择S_InventoryItem，创建数据表ItemList;<img src='https://img-blog.csdnimg.cn/20201117161557370.png'><img src='https://img-blog.csdnimg.cn/20201117161720162.png'>
2. 创建任务物品Food_Grain；<img src='https://img-blog.csdnimg.cn/20201117161014212.png'>
3. 创建其他物品Rock；<img src='https://img-blog.csdnimg.cn/20201117161122363.png'>
4. 创建消耗物品Food_Cupcake;<img src='https://img-blog.csdnimg.cn/2020111716124483.png'>
5. 创建消耗物品Bush;<img src='https://img-blog.csdnimg.cn/20201117161320380.png'>

### 创建组件C_Inventory

1. 创建函数[Success(Boolean)] InitializeInventory(InventorySize(Integer))，用于初始化Inventory(S_InventoryItem[])数组;<img src='https://img-blog.csdnimg.cn/20201117162452567.png'>
2. 创建函数[Inventory(S_InventoryItem[])] GetInventoryItem()，用于获取Inventory数组；<img src='https://img-blog.csdnimg.cn/20201117163625345.png'>
3. 创建函数[Success(Boolean)] SetInventoryItem(Index(Integer),Item(S_InventoryItem))，用于设置Inventory[Index]物品属性；<img src='https://img-blog.csdnimg.cn/20201117162831113.png'>
4. 创建函数[Success(Boolean)]LoadInventoryItem (InventorySize(Integer),TargetArray(S_InventoryItem[]))，调用函数InitializeInventory和SetInventoryItem，加载所有物品属性；<img src='https://img-blog.csdnimg.cn/20201117164028913.png'>

### 创建组件C_InventoryManager

1. 创建函数[Void]Initialize_UI_PC_Main()，用于初始化PC_Main(W_PC_Main)；<img src='https://img-blog.csdnimg.cn/20201117172454711.png'>
2. 创建函数[Void]InitializePlayerInventoryComponent，用于初始化PlayerInventory(C_Inventory)；<img src='https://img-blog.csdnimg.cn/20201117172848864.png'>
3. 创建函数[Void]LoadInventory(),用于根据背包物品属性创建W_InventorySlot实例，并添加到W_InventoryWindow内的InventorySlotWrapBox内，创建的实例个数由InventorySize决定。最后调用服务端事件Server_RefreshInventory；<img src='https://img-blog.csdnimg.cn/20201117174722222.png'>
4. 创建函数[Void]ClearInventorySlotItem()，用于初始化W_InventorySlot中的变量ItemInformation；<img src='https://img-blog.csdnimg.cn/20201117180831721.png'>
5. 创建函数[Void]SetInventorySlotItem()，用于设置InventorySlot[SlotIndex]的ItemInformation属性;<img src='https://img-blog.csdnimg.cn/20201117182259271.png'>
6. 创建函数[Void]RefreshInventorySlots()，调用客户端事件Client_SetInventorySlotItem和Client_SetInventorySlotItem，刷新背包数据并更新Slot；<img src='https://img-blog.csdnimg.cn/20201117182927647.png'><img src='https://img-blog.csdnimg.cn/20201117182532491.png'>
7. 创建函数[Void]OpenInventoryWindow()，用于显示背包窗口；<img src='https://img-blog.csdnimg.cn/20201117183626422.png'>
8. 创建函数[Void]CloseInventoryWindow()，用于隐藏背包窗口；<img src='https://img-blog.csdnimg.cn/20201117183710298.png'>

### W_InventorySlot属性绑定

1. 物品边框BrushColor属性绑定；<img src='https://img-blog.csdnimg.cn/20201117181452720.png'>
2. 物品图表BrushImage属性绑定；<img src='https://img-blog.csdnimg.cn/20201117181717696.png'>
3. 物品数量Text属性绑定；<img src='https://img-blog.csdnimg.cn/20201117181817509.png'>



## RPC(Remote Procedure Call)

### RPG_PlayerController

1. 添加组件C_Inventory和C_InventoryManager；

2. 勾选Replication>Replicates，将自身复制到服务端；

3. 创建函数Server_LoadPlayerItem，限制事件执行位置位于服务端，本地创建InventoryID数组模拟解包后的数据（此处省略了服务端向客户端发送数据包的过程）。根据从服务器获取的InventoryID，从本地ItemList加载物品属性，并存入本地变量LocalInventory。添加完成后，调用C_Inventory的函数LoadInventoryItem加载背包物品，传入的参数InventorySize来自C_InventoryManager的变量InventorySlot(Replicated)；

   <img src='https://img-blog.csdnimg.cn/20201117171957372.png'><img src='https://img-blog.csdnimg.cn/20201117171456274.png'>

4. 在EventBeginPlay中调用函数InitializePlayerInventoryComponent初始化C_Inventory，随后调用ServerLoadPlayerItem从服务端加载（简单）数据，然后调用函数Initialize_UI_PC_Main初始化W_PC_Main，最后调用事件Client_LoadInventory，从客户端加载（更为详细的）数据;<img src='https://img-blog.csdnimg.cn/2020111717564280.png'>

5. 删除BP_Player中的热键I事件，更改为在RPG_PlayerController中调用，根据InventoryManager中的IsInventoryOpen布尔值，判断是调用客户端事件Client_OpenInventory，还是调用Client_ClosenInventory；<img src='https://img-blog.csdnimg.cn/2020111718430680.png'>

### Run on Server(Reliable)

#### Server_InitializeInventory

在C_Inventory事件图表中创建，调用函数InitializeInventory；<img src='https://img-blog.csdnimg.cn/20201117164840990.png'>

#### Server_RefreshInventory

在C_InventoryManager事件图表中创建，调用函数RefreshInventorySlots；<img src='https://img-blog.csdnimg.cn/20201117180225333.png'>

### Run on owning Client(Reliable)

#### Client_LoadInventory

在C_InventoryManager事件图表中创建，调用函数LoadInventory;<img src='https://img-blog.csdnimg.cn/20201117173447637.png'>

#### Client_ClearInventorySlotItem

在C_InventoryManager事件图表中创建，调用函数ClearInventorySlotItem;<img src='https://img-blog.csdnimg.cn/2020111718045423.png'>

#### Client_SetInventorySlotItem

在C_InventoryManager事件图表中创建，调用函数SetInventorySlotItem;<img src='https://img-blog.csdnimg.cn/20201117182358410.png'>

#### Client_OpenInventory

在C_InventoryManager事件图表中创建，调用函数OpenInventoryWindow;<img src='https://img-blog.csdnimg.cn/20201117183848642.png'>

#### Client_CloseInventory

在C_InventoryManager事件图表中创建，调用函数CloseInventoryWindow;<img src='https://img-blog.csdnimg.cn/20201117183911582.png'>

# 最终效果展示

<img src='https://img-blog.csdnimg.cn/20201117155229532.png'>