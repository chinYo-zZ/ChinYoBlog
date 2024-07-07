---
title: UE5 使用Fromat创建格式化的UELog
date: 2024-06-28 13:49:22
tags: UE5
categories: UE5
sitemap: true
---

<div align=center><img  alt="简介" src="image.png"/></div>

```C++
#define MY_LOG(Format, ...) MyLog(TEXT(Format), __VA_ARGS__)

inline void FillArgs(FStringFormatOrderedArguments& Args)
{
    //终止递归
}

template<typename F, typename... R>
void FillArgs(FStringFormatOrderedArguments& Args, F&& First, R&&... Rest)
{
    Args.Add(Forward<F>(First));
    FillArgs(Args, Forward<R>(Rest)...);
}

template<typename... T>
void MyLog(const TCHAR* Format, T&&... Args)
{
    FStringFormatOrderedArguments OrderedArgs; 
    FillArgs(OrderedArgs, Forward<T>(Args)...); 
    FString Message = FString::Format(Format, MoveTemp(OrderedArgs));

    GEngine->AddOnScreenDebugMessage(-1, 5, FColor::Cyan, Message);
    UE_LOG(LogTemp, Display, TEXT("%s"), *Message);
}
```
<div align=center><img  alt="支持Fromat的参数" src="image1.png"/></div>

## 后续更新支持类->GetName()