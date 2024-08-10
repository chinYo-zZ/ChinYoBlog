---
title: UE 优化方面的内容
date: 2024-07-01 21:26:55
tags: UE5
categories: UE5
sitemap: true
---
## 瓶颈定位 
![alt text](image.png)
 - Game Thread 首先会对整个游戏世界进行逻辑层面的计算与模拟(e.g.Spawn 多少个新的 actor、每个 actor 在这一帧位于何处、角色移动、动画状态等等)，所有这些信息会被输送到 Draw Thread
 - Draw Thread(也叫 Rendering Thread) 会根据这些信息，剔除(Culling)掉不需要显示的部分(e.g. 处于屏幕外的物体)，接着创建一个列表，其中包含了渲染每个物体必备的关键信息(e.g. 如何被着色、映射哪些纹理等等)，再将这个列表输送给 GPU Thread
 - GPU Thread 在获取了这个列表之后，会计算出每个像素最终需要如何被渲染在屏幕上，形成这一帧的画面
综上，对于每一帧来说，这三者的执行顺序依次为：Game Thread → Draw Thread → GPU Thread


**一帧的总耗时，取决于三者中开销最严重、即耗时最长的线程**

 ## CPU 优化

 1. 替换tick
  - 时间线组件
  - 循环计时器
  - 远距离时禁用tick或者tick间隔延长
  - 事件驱动

 2. 编码
 - 对象池
 - 复杂运算用c++
 - 多线程进行数据计算（不要引用对象）

 3. C++
 - 如果返回类型带有const，& 可以用auto防止临时对象构造
 - 循环中复用临时变量
 - 在构造函数中使用初始化列表
 - 利用好constexpr编译期计算