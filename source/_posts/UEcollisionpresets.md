---
title: UE 添加碰撞预设（collision presets）
date: 2024-08-01 00:23:11
tags: UE5
categories: UE5
sitemap: true
---

 **给项目添加碰撞预设**

<div align=center><img  alt="示例" src="image.png"/></div>

碰撞通道可以在引擎中添加

碰撞预设好像只能在ini文件中添加

```
+Profiles=(Name="Projectile",
CollisionEnabled=QueryOnly,
bCanModify=True,
ObjectTypeName="Projectile",
CustomResponses=((Channel="Pawn",Response=ECR_Overlap),
(Channel="Visibility",Response=ECR_Ignore),
(Channel="Camera",Response=ECR_Ignore),
(Channel="Ability",Response=ECR_Ignore),
(Channel="Projectile",Response=ECR_Ignore),(Channel="AbilityOverlapProjectile",Response=ECR_Overlap),
(Channel="Pickup",Response=ECR_Ignore)),
HelpMessage="Overlaps Pawns and AbilityOverlapProjectile")
```
可以根据参数名称去搜索枚举

比如CollisionEnabled，如下
<div align=center><img  alt="示例" src="image-1.png"/></div>