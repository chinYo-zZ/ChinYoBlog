---
title: UE5的一些报错 
date: 2024-03-05 22:18:28
tags: UE5
categories: UE5
sitemap: true
---


## 动态多播委托不触发

    AddDynamic绑定的方法得被UFUNCTION标记，否则绑定无效

动态代理对象类型可以使用UPROPERTY标记，并设置为BlueprintAssignable，从而暴露给蓝图使用，其他代理均无法使用（不加编译可过，调用出错）

## Unable to find 'class', 'delegate', 'enum', or 'struct' with name 'XXX'	
<div align=center><img  alt="示例" src="image.png"/></div>

由Unreal Header Tool (UHT)发出

    原因是函数中有数据结构没有被宏标记

删除或使用其他委托