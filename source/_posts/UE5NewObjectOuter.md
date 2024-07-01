---
title: UE5 NewObject ： Outer参数
date: 2024-06-08 22:11:27
tags: UE5
categories: UE5
sitemap: true
---

在创建对象的过程中，使用NewObejct等一系列创建对象的函数的时候，会看到Outer参数，在官方文档中，参数的意思为：
<div align=center><img  alt="官方解释" src="image.png"/></div>

看到这样的描述其实是很难意会的

深入源码,直到看到new
```C++
if (!bSubObject)
{
	FMemory::Memzero((void *)Obj, TotalSize);
	new ((void *)Obj) UObjectBase(
        const_cast<UClass*>(InClass), 
        InFlags|RF_NeedInitialization, 
        InternalSetFlags, 
        InOuter, 
        InName, 
        OldIndex,
        OldSerialNumber);
}
```
在UObjectBase中，outer赋值给OuterPrivate

```C++
/** Object this object resides in. */
UObject*						OuterPrivate;
```
根据注释可知，Outer可以理解为新创建对象的持有者，类似于角色身上持有组件，Outer为角色，会控制所创建对象的生命周期；

经过一些测试，Outer与对象的垃圾回收有关。即使通过 TSharedPtr 或 UPROPERTY 对 UObject 有很强的引用，如果Outer销毁，创建的所有对象也会消失。