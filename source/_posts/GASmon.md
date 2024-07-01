---
title: UE5 GAS PlayMontageAndWait源码解析
date: 2024-06-01 20:06:36
tags: UE5
categories: UE5
sitemap: true
---
# PlayMontageAndWait源码解析

<div align=center><img  alt="蓝图节点" src="image.png"/></div>


声明的四个委托分别对应四个异步Exec
```C++
//声明动态多播委托
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FMontageWaitSimpleDelegate);

UPROPERTY(BlueprintAssignable)
FMontageWaitSimpleDelegate	OnCompleted;

UPROPERTY(BlueprintAssignable)
FMontageWaitSimpleDelegate	OnBlendOut;

UPROPERTY(BlueprintAssignable)
FMontageWaitSimpleDelegate	OnInterrupted;

UPROPERTY(BlueprintAssignable)
FMontageWaitSimpleDelegate	OnCancelled;
```
蓝图函数界面根据此函数创建
```C++
UFUNCTION(BlueprintCallable, Category="Ability|Tasks", meta = (DisplayName="PlayMontageAndWait",
	HidePin = "OwningAbility", DefaultToSelf = "OwningAbility", BlueprintInternalUseOnly = "TRUE"))
static UAbilityTask_PlayMontageAndWait* CreatePlayMontageAndWaitProxy(UGameplayAbility* OwningAbility,
	FName TaskInstanceName, UAnimMontage* MontageToPlay, float Rate = 1.f, FName StartSection = NAME_None, bool bStopWhenAbilityEnds = true, float AnimRootMotionTranslationScale = 1.f, float StartTimeSeconds = 0.f);
```
通过这个静态函数创建UAbilityTask_PlayMontageAndWait对象，并根据蓝图写入的数据赋值给成员变量
```C++
UAbilityTask_PlayMontageAndWait* MyObj = NewAbilityTask<UAbilityTask_PlayMontageAndWait>(OwningAbility, TaskInstanceName);
MyObj->MontageToPlay = MontageToPlay;
MyObj->Rate = Rate;
MyObj->StartSection = StartSection;
MyObj->AnimRootMotionTranslationScale = AnimRootMotionTranslationScale;
MyObj->bStopWhenAbilityEnds = bStopWhenAbilityEnds;
MyObj->StartTimeSeconds = StartTimeSeconds;
```
绑定一些函数，通过动画实例绑定montage和委托，动画播放时触发委托，委托通知绑定的函数，在绑定的函数里再广播给蓝图准备的四个Exec
```C++
//UAbilityTask_PlayMontageAndWait::Activate()
InterruptedHandle = Ability->OnGameplayAbilityCancelled.AddUObject(this, &UAbilityTask_PlayMontageAndWait::OnMontageInterrupted);

BlendingOutDelegate.BindUObject(this, &UAbilityTask_PlayMontageAndWait::OnMontageBlendingOut);
AnimInstance->Montage_SetBlendingOutDelegate(BlendingOutDelegate, MontageToPlay);

MontageEndedDelegate.BindUObject(this, &UAbilityTask_PlayMontageAndWait::OnMontageEnded);
AnimInstance->Montage_SetEndDelegate(MontageEndedDelegate, MontageToPlay);

```
外部取消播放蒙太奇
```C++
void UAbilityTask_PlayMontageAndWait::ExternalCancel()
{
	if (ShouldBroadcastAbilityTaskDelegates())
	{
        // 蓝图委托
		OnCancelled.Broadcast();
	}
	Super::ExternalCancel();
}
```
再绑定前会先调用播放蒙太奇函数
```C++
if (ASC->PlayMontage(Ability, Ability->GetCurrentActivationInfo(), MontageToPlay, Rate, StartSection, StartTimeSeconds) > 0.f)
{
	//...
}
```
asynctask没找到具体逻辑在哪

剩下的一些逻辑挺好理解的