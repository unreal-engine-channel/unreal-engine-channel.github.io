---
title: RPG Game开发日志(二十)
date: 2020-11-18 15:07:40
categories: Logs
tags: UE_4.22
---

涉及内容：ToolTip，拖拽功能，RPC。

<!--more-->

# 涉及内容

## 修复W_Window关闭按钮

### 准备工作

1. 创建蓝图接口BPI_HUDInterface;
2. 创建接口函数[Void]UICloseInventoryWindow();

### W_Window相关

1. 修改CloseButton点击事件，更改为调用RPG_PlayerController中的接口事件UICloseInventoryWindow;<img src='https://img-blog.csdnimg.cn/20201118172800898.png'>

### RPG_PlayerController相关

1. 实现接口函数事件EventUICloseInventoryWindow，调用服务端事件Client_CloseInventory；<img src='https://img-blog.csdnimg.cn/20201118172537733.png'>

## 使用ToolTip添加物品描述

### 准备工作

1. 创建结构体E_UseType [Health,Mana,HealthAndMana]；
2. 打开结构体S_InventoryItem，新增Health、Mana、Duration、UseType物品属性；<img src='https://img-blog.csdnimg.cn/20201118153607658.png'>
3. 打开蓝图接口BPI_HUDInterface创建接口函数，[InventoryInformation(S_InventoryItem)]GetToolTip(ID(Name));
4. 打开ItemList，新增物品Health_Red和Mana_Blue，用于测试ToolTip；<img src='https://img-blog.csdnimg.cn/20201118155809237.png'><img src='https://img-blog.csdnimg.cn/20201118155834342.png'>

### W_ToolTip相关

1. 创建控件W_ToolTip，添加物品名称、类别、使用信息、描述信息、价格；<img src='https://img-blog.csdnimg.cn/2020111815241557.png'>
2. 对NameText控件添加文本内容和颜色的绑定函数；<img src='https://img-blog.csdnimg.cn/20201118152520842.png'><img src='https://img-blog.csdnimg.cn/20201118153029489.png'>
3. 对ItemType控件添加文本内容的绑定函数；<img src='https://img-blog.csdnimg.cn/20201118153142211.png'>
4. 对UseInfo控件添加文本内容的绑定函数；<img src='https://img-blog.csdnimg.cn/20201118153928751.png'>
5. 对Description控件添加文本内容的绑定函数；<img src='https://img-blog.csdnimg.cn/2020111815414278.png'>
6. 对ValueText控件添加文本内容的绑定函数；<img src='https://img-blog.csdnimg.cn/20201118154310493.png'>

### W_InventorySlot相关

1. 为SizeBox的ToolTipWidget添加绑定函数；<img src='https://img-blog.csdnimg.cn/20201118155639966.png'>

### RPG_PlayerController相关

1. 点击ClassSettings，将蓝图接口BPI_HUDInterface添加进来；
2. 实现接口函数GetToolTip，根据物品ID从ItemList中获取物品信息；<img src='https://img-blog.csdnimg.cn/20201118155108526.png'>
3. 修改变量InventoryID，将新增的物品Health_Red和Mana_Blue添加进来；<img src='https://img-blog.csdnimg.cn/20201118160116361.png'>

## 拖拽功能

### 准备工作

1. 打开蓝图接口BPI_HUDInterface，创建接口函数[Void]UIMoveInventoryItem(FromInventoryIndex(Integer),ToInventoryIndex(Integer));

### W_DragItem相关

1. 创建控件W_DragItem，用于拖拽物品时生成的图标；<img src='https://img-blog.csdnimg.cn/20201118160913939.png'>
2. 对Icon控件的BurshImage添加绑定函数；<img src='https://img-blog.csdnimg.cn/20201118161148857.png'>
3. 对Text控件的文本内容添加绑定函数；<img src='https://img-blog.csdnimg.cn/20201118161259306.png'>

### W_InventorySlot相关

1. 为UseButton内的Border添加OnMouseButtonDown绑定事件，监控鼠标左键；<img src='https://img-blog.csdnimg.cn/20201118160454867.png'>
2. 创建蓝图BP_DragDrop，选取父类为DragDropOperation，并创建变量InventoryItem(S_InventoryItem)和InventoryIndex(Integer)；
3. 创建重载函数OnDragDetected，当鼠标左键拖拽物品时生成相应的拖拽图标(W_DragItem)实例，并创建拖拽Operation实例(BP_DragDrop)，将物品信息和物品索引保存在BP_DragDrop中；<img src='https://img-blog.csdnimg.cn/20201118162311682.png'>
4. 创建重载函数OnDrop，当拖拽后鼠标左键释放后，从BP_DragDrop中获取InventoryIndex(FromInventoryIndex)和InventorySlotIndex(ToInventoryIndex)，并调用接口函数UIMoveInventoryItem；<img src='https://img-blog.csdnimg.cn/2020111816420391.png'>

### C_Inventory相关

1. 创建函数[Output(S_InventoryItem)]GetInventoryItem(InventoryIndex(Integer))，根据物品索引获取物品信息；<img src='https://img-blog.csdnimg.cn/2020111816541335.png'>

### C_InventoryManager相关

1. 创建函数[Void]ClearInventoryIndexItem(ToInventoryIndex(Integer))，用于根据索引删除对应插槽处的物品；<img src='https://img-blog.csdnimg.cn/20201118170944877.png'>
2. 创建函数[Void]AddItem(Inventory(C_Inventory),ToInventoryIndex(Integer),InventoryItem(S_InventoryItem))，调用客户端事件Client_SetInventorySlotItem；<img src='https://img-blog.csdnimg.cn/20201118170440602.png'>
3. 创建函数[Void]RemoveItem(Inventory(C_Inventory),FromInventoryIndex(Integer))，调用客户端事件Client_ClearInventoryIndexItem；<img src='https://img-blog.csdnimg.cn/20201118171358351.png'>
4. 创建函数[Void]MoveItem(FromInventoryIndex(Integer),ToInventoryIndex(Integer))，调用C_Inventory的GetInventoryItem获取From和To插槽处的物品信息（可能为空）。若为空，则调用AddItem和RemoveItem函数，实现物品拖拽后背包UI更新；<img src='https://img-blog.csdnimg.cn/20201118165913588.png'><img src='https://img-blog.csdnimg.cn/20201118165939535.png'>

### RPG_PlayerController相关

1. 实现接口函数事件EventUIMoveInventoryItem，调用服务端事件Server_MoveInventoryItem；<img src='https://img-blog.csdnimg.cn/20201118172144413.png'>

## RPC(Remote Procedure Call)

### Run on Server(Reliable)

#### Server_MoveInventoryItem

在C_InventoryManager事件图表中创建，调用函数MoveItem；<img src='https://img-blog.csdnimg.cn/20201118164516806.png'>

### Run on owning Client(Reliable)

#### Client_ClearInventoryIndexItem

在C_InventoryManager事件图表中创建，调用函数ClearInventoryIndexItem；<img src='https://img-blog.csdnimg.cn/20201118170841858.png'>

# 最终效果展示

## 物品描述

<img src='https://img-blog.csdnimg.cn/20201118151239939.png'>

## 拖拽效果

<img src='https://img-blog.csdnimg.cn/20201118151448629.png'>