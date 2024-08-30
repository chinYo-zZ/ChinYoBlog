---
title: UE 从C++写蓝图函数的一些有用的操作
date: 2024-08-12 23:40:20
tags: UE5
categories: UE5
sitemap: true
---

 ## UPARAM(ref)
<div align=center><img  alt="示例" src="image.png"/></div>

使用此宏可以在不将引用标记为const的情况下使其在蓝图中变成输入

不加const或者UPARAM(ref)的情况下蓝图会将引用变为<font color="#0080c0">输出</font> 

 ## AutoCreateRefTerm

 <div align=center><img  alt="示例" src="image1.png"/></div>

使用此说明符，蓝图会自动创建引用（在蓝图中<font color="#0080c0">const &</font> 没有输入的情况下会报错），不需要手动创建

未完待续。。。
