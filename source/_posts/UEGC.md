---
title: UE 垃圾回收的执行时机
date: 2024-05-27 23:10:03
tags: UE5
categories: UE5
sitemap: true
---
需要测试的有以下类型
```C++
class  NonUObejct
{
public:
	NonUObejct();
	virtual ~NonUObejct();
};

UCLASS()
class UTestObject : public UObject 
{
	GENERATED_BODY()

public:
	UTestObject();
	virtual ~UTestObject();
};

USTRUCT()
struct FTestStruct
{
	GENERATED_BODY()
public:
	UPROPERTY()
	UTestObject* TestClass = nullptr;
};

UINTERFACE()
class UTestInterface : public UInterface 
{
	GENERATED_BODY()
};

class ITestInterface
{
	GENERATED_BODY()
public:
	virtual void TestMethod() = 0;
};

UCLASS()
class UObjectWithInterface : public UObject , public ITestInterface
{
	GENERATED_BODY()
public:
	UObjectWithInterface();
	virtual ~UObjectWithInterface();
	virtual void TestMethod() override;
};
```
对应CPP
```C++
NonUObejct::NonUObejct()
{
	UE_LOG(LogTemp, Warning, TEXT("NonUobject created"));
}

NonUObejct::~NonUObejct()
{
	UE_LOG(LogTemp, Warning, TEXT("NonUobject destroyed"));
}

UTestObject::UTestObject()
{
	UE_LOG(LogTemp, Warning, TEXT("UTestObject created"));
}

UTestObject::~UTestObject()
{
	UE_LOG(LogTemp, Warning, TEXT("UTestObject destroyed"));
}

UObjectWithInterface::UObjectWithInterface()
{
	UE_LOG(LogTemp, Warning, TEXT("UObjectWithInterface created"));
}

UObjectWithInterface::~UObjectWithInterface()
{
	UE_LOG(LogTemp, Warning, TEXT("UObjectWithInterface destroyed"));
}

void UObjectWithInterface::TestMethod()
{
	UE_LOG(LogTemp, Warning, TEXT("Interface : TestMethod()"));
}
```
> \
> gc.CollectGarbageEveryFrame 1 命令可以每帧执行垃圾回收 
> 
>    

## 一、UPROPERTY()标记
在玩家控制器中
```C++
UCLASS()
class TESTGC_API ATestPlayerController : public APlayerController
{
	GENERATED_BODY()
public:
	ATestPlayerController();
	virtual ~ATestPlayerController();
	virtual void BeginPlay()override;
private:
	UPROPERTY()
	UTestObject* _TestObject = nullptr;
};
//cpp
void ATestPlayerController::BeginPlay()
{
	Super::BeginPlay();
    //没有给参数Outer！
	_TestObject = NewObject<UTestObject>();
}
```
开始运行
```C++
LogTemp: Warning: UTestObject created
```
结束运行
```C++
LogTemp: Warning: UTestObject destroyed
```
玩家控制器消失，可以销毁_TestObject，关系为玩家控制器持有_TestObject

---
创建后马上设置为空
```C++
void ATestPlayerController::BeginPlay()
{
	Super::BeginPlay();
    //没有给参数Outer！
	_TestObject = NewObject<UTestObject>();
    _TestObject = nullptr;
}
```
```C++
LogTemp: Warning: UTestObject created
...
LogTemp: Warning: UTestObject destroyed
```
此时没有类引用_TestObject，垃圾回收销毁_TestObject


## 二、Outer引用

```C++
void ATestPlayerController::BeginPlay()
{
	Super::BeginPlay();
    //this给参数Outer,相当于持有者为玩家控制器
	_TestObject = NewObject<UTestObject>(this);
    UTestGameInstance* gameInstance = GetGameInstance<UTestGameInstance>();
    gameInstance->Object = _TestObject;

    UGameplayStatics::OpenLevel(gameInstance, TEXT("otherMap"));
}
```
切换关卡后
```C++
LogTemp: Warning: UTestObject destroyed
```
为什么有游戏实例的引用，还是被销毁了，原因是Outer参数将玩家控制器设置为持有者，玩家控制器被销毁，连带持有的对象也被销毁了
如果去掉this，切换关卡后将不会销毁，因为被游戏实例引用，并且没有父级


## 三、未继承Object

```C++
private:
	UPROPERTY()
	UTestObject* _TestObject = nullptr;
    UPROPERTY()
    NonUObejct* Non = nullptr;
};
```
直接报错，不继承Object只能使用智能指针管理内存

## 四、数组GC
```C++
private:
	UPROPERTY()
	TArray<UTestObject*> TArray;
};

void ATestPlayerController::BeginPlay()
{
    TArray.Add(NewObject<UTestObject>());
    TArray.Add(NewObject<UTestObject>());
    TArray.Add(NewObject<UTestObject>());

}
```
运行停止后，三次销毁
```C++
void ATestPlayerController::BeginPlay()
{
    TArray.Add(NewObject<UTestObject>());
    TArray.Add(NewObject<UTestObject>());
    TArray.Add(NewObject<UTestObject>());
    TArray.Empty();
}
```
依旧是三次销毁，说明数组也会GC

## 五、结构体

```C++
private:
	UPROPERTY()
	FTestStruct _TStruct;
};

void ATestPlayerController::BeginPlay()
{
    _TStruct.TestClass = NewObject<UTestObject>();

    _TStruct.TestClass = nullptr;
}
```
创建后马上销毁

## 六、接口

使用接口指针储存类对象必须使用TScriptInterface<>模版才能实现GC，否则会报错，数组同理
```C++
private:
	UPROPERTY()
	TScriptInterface<ITestInterface> _Inter = nullptr;
};

void ATestPlayerController::BeginPlay()
{
    _Inter = NewObject<UObjectWithInterface>();
}
```