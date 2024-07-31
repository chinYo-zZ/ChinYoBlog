---
title: UE5 GAS PossessedBy()和OnRep_PlayerState()
date: 2024-07-15 20:59:42
tags: UE5
categories: UE5
sitemap: true
---
在默认Pawn类中对角色进行多人游戏GAS组件初始化记录
```C++
void AALSCharacter::PossessedBy(AController* NewController)
{
	Super::PossessedBy(NewController);
	AChPlayerState* PS = GetPlayerState<AChPlayerState>();
	UKismetSystemLibrary::PrintString(GetWorld(), FString::Printf(TEXT("PossessedBy ------- %s --> %s"), *PS->GetName(), *NewController->GetName()),
		true, false, FLinearColor::Red, 100.f);
	UKismetSystemLibrary::DrawDebugSphere(GetWorld(), this->GetActorLocation(), 100.f, 12, FLinearColor::Red, 100.f);
	
}

void AALSCharacter::OnRep_PlayerState()
{
	Super::OnRep_PlayerState();
	UKismetSystemLibrary::PrintString(GetWorld(), FString::Printf(TEXT("OnRep_PlayerState-------%s"), *GetPlayerState()->GetName()),
		true, false, FLinearColor::Red, 100.f);
	UKismetSystemLibrary::DrawDebugSphere(GetWorld(), this->GetActorLocation(), 100.f, 12, FLinearColor::White, 100.f);

}
```

<div align=center><img  alt="结果" src="image.png"/></div>

PossessedBy()和OnRep_PlayerState()各执行了两次，PossessedBy()在服务端上执行，OnRep_PlayerState()在客户端上执行

其中OnRep_PlayerState()是因为Pawn中PlayerState变化触发的

**第一次修改**
<div align=center><img  alt="第一次修改PlayerState" src="image1.png"/></div>

**第二次修改**
<div align=center><img  alt="第二次修改PlayerState" src="image2.png"/></div>


然后调用客户端OnRep_PlayerState()进行客户端代理Pawn的修改，执行两次，两个Pawn

<div align=center><img  alt="PlayerState修改调用" src="image3.png"/></div>
