---
title: UE5 GameplayTag图鉴
date: 2024-07-28 22:42:50
tags: UE5
categories: UE5
sitemap: true
description: "大健康大开挖大我觉得那金娃"
---


- [GameplayTag](#idGameplayTag)

# GameplayTag

* **RequestGameplayTag（静态）**

```c++
static FGameplayTag RequestGameplayTag(
const FName& TagName,
 bool ErrorIfNotFound=true);
```

从已有的tag集合中获取指定的tag

* **IsValidGameplayTagString（静态）**

```c++
static bool IsValidGameplayTagString(
const FString& TagString, 
FText* OutError = nullptr, 
FString* OutFixedString = nullptr);
```

判断输入的tag是否有效，如果无效则注册tag

## 匹配函数

* **MatchesTag**

```c++
bool MatchesTag(
const FGameplayTag& TagToCheck) const;
```
![alt text](image.png)
是否匹配父标记

* **MatchesTagExact**

```c++
bool MatchesTagExact(
const FGameplayTag& TagToCheck) const
```

判断是否完全匹配，可以直接使用双等号，重载了运算符，效果是一样的

    蓝图中使用bool变量选择

![alt text](image-2.png)

* **MatchesTagDepth**

```c++
int32 MatchesTagDepth(
const FGameplayTag& TagToCheck) const;
```
![alt text](image-1.png)
根据标签匹配数量返回整数













