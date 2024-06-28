---
title: UE5自定义资源（二）
date: 2024-06-26 20:52:14
tags: UE5
categories: UE5
sitemap: true
---
UE版本：5.2.1

ue4不适用
## 自定义资源图标
```C++
void FHtnAssetEditorModule::StartupModule()
{
	/* 其他操作 
    ------------*/

    //创建Slate样式集
	_styleSet = MakeShareable(new FSlateStyleSet(TEXT("HtnStyle")));
    //获取插件管理器，目的是获取Icon路径
	TSharedPtr<IPlugin>plugin = IPluginManager::Get().FindPlugin("DemoModule");
	FString contentDir = plugin->GetContentDir();
    //设置获取到路径为根路径
	_styleSet->SetContentRoot(contentDir);
    // FSlateImageBrush结构体封装一些图片信息
	FSlateImageBrush* thumbnailBrush = new FSlateImageBrush(_styleSet->RootToContentDir(TEXT("HtnUI02"), TEXT(".png")), FVector2D(128.0, 128.0));
	FSlateImageBrush* iconBrush = new FSlateImageBrush(_styleSet->RootToContentDir(TEXT("HtnUI02"), TEXT(".png")), FVector2D(128.0, 128.0));
    //将样式设置到自定义资源
	_styleSet->Set("ClassThumbnail.HierarchicalTaskNetwork", thumbnailBrush);
	_styleSet->Set("classIcon.HierarchicalTaskNetwork", iconBrush);
    //注册样式
	FSlateStyleRegistry::RegisterSlateStyle(*_styleSet);
}
```

>ClassThumbnail.[youAssetAame]和classIcon[youAssetAame]             
>youAssetAame不要带前缀U

## 新建节点

![alt text](image.png)

创建新的GraphSchema继承自UEdGraphSchema，再创建节点行为继承自FEdGraphSchemaAction

```C++
// .h
UCLASS()
class UHtnGraphSchema :public UEdGraphSchema
{
	GENERATED_BODY()

public:
	virtual void GetGraphContextActions(FGraphContextMenuBuilder& contextMenuBuilder) const override;
};

USTRUCT()
struct FNewNodeAction :public FEdGraphSchemaAction
{
	GENERATED_BODY()
public:
	FNewNodeAction(){}
	FNewNodeAction(FText inNodeCategory, FText inMenuDesc, FText inToolTip, const int32 inGrouping)
		:FEdGraphSchemaAction(inNodeCategory, inMenuDesc, inToolTip, inGrouping){}

	virtual UEdGraphNode* PerformAction(UEdGraph* parentGraph, UEdGraphPin* fromPin, const FVector2D location, bool bSelectNewNode);
};
//.cpp
void UHtnGraphSchema::GetGraphContextActions(FGraphContextMenuBuilder& contextMenuBuilder) const
{
	TSharedPtr<FNewNodeAction> newNodeAction(
		new FNewNodeAction(FText::FromString(TEXT("Nodes")), FText::FromString(TEXT("New Node")),
		FText::FromString(TEXT("Makes a new node")),0));

	contextMenuBuilder.AddAction(newNodeAction);
}

UEdGraphNode* FNewNodeAction::PerformAction(UEdGraph* parentGraph, UEdGraphPin* fromPin, const FVector2D location, bool bSelectNewNode)
{
	//必须使用自己节点，后文会提到
	UHtnEdGraphNode* result = NewObject<UHtnEdGraphNode>(parentGraph);
	result->NodePosX = location.X;
	result->NodePosY = location.Y;

	result->CreatePin(EEdGraphPinDirection::EGPD_Input, TEXT("Inputs"), TEXT("SomeInput"));
	result->CreatePin(EEdGraphPinDirection::EGPD_Output, TEXT("Outputs"), TEXT("Output1"));
	result->CreatePin(EEdGraphPinDirection::EGPD_Output, TEXT("Outputs"), TEXT("Output2"));

	parentGraph->Modify();
	parentGraph->AddNode(result, true, true);
	return result;
}
```
做完节点行为需要将自己的Schema替换原本的Schema

```C++
//在初始化界面的函数中
_workingGraph = FBlueprintEditorUtils::CreateNewGraph
(
	_workingAsset, NAME_None, UEdGraph::StaticClass(),	
	UHtnGraphSchema::StaticClass() // 用自己的UHtnGraphSchema，ue5一般都是用反射进行注册，UHtnGraphSchema要有UCLASS宏
);
```
注册完成后即可通过UHtnGraphSchema::GetGraphContextActions呼出节点选择框

## 节点行为

![alt text](image04.png)

创建自己的节点
```C++
//.h
UCLASS()
class HTNASSETEDITOR_API UHtnEdGraphNode : public UEdGraphNode
{
	GENERATED_BODY()
public:
	virtual FText GetNodeTitle(ENodeTitleType::Type titalType)const override{return FText::FromString("TestNodeTitle"); }
	virtual FLinearColor GetNodeTitleColor() const override { return FLinearColor(FColor::Green); }
	virtual bool CanUserDeleteNode()const override{ return true; }
	virtual void GetNodeContextMenuActions(class UToolMenu* menu, class UGraphNodeContextMenuContext* context) const override;
};
//.cpp
void UHtnEdGraphNode::GetNodeContextMenuActions(UToolMenu* menu, UGraphNodeContextMenuContext* context) const
{
	FToolMenuSection& section = menu->AddSection(TEXT("SectionName"), FText::FromString(TEXT("Custom Node Actions")));

	UHtnEdGraphNode* node = (UHtnEdGraphNode*)this;
	//右键行为
	section.AddMenuEntry(TEXT("AddPinEntry"), FText::FromString(TEXT("Add Pin")),
		FText::FromString(TEXT("Creates a new pin")),
		FSlateIcon(TEXT("HtnStyle"), TEXT("HtnAssetEditor.NodeAddPinIcon")),
		FUIAction(FExecuteAction::CreateLambda
		(
			[node]() {
				node->CreatePin(EEdGraphPinDirection::EGPD_Output, TEXT("Outputs"), TEXT("Another Output"));
				node->GetGraph()->NotifyGraphChanged();
				node->GetGraph()->Modify();
			}
		))
	);
	section.AddMenuEntry(TEXT("DeletePinEntry"), FText::FromString(TEXT("Delete Pin")),
		FText::FromString(TEXT("Delete the last pin")),
		FSlateIcon(TEXT("HtnStyle"), TEXT("HtnAssetEditor.NodeDeletePinIcon")),
		FUIAction(FExecuteAction::CreateLambda
		(
			[node]() {
				UEdGraphPin* pin = node->GetPinAt(node->Pins.Num() - 1);
				if (pin->Direction != EEdGraphPinDirection::EGPD_Input) {
					node->RemovePin(pin);
					node->GetGraph()->NotifyGraphChanged();
					node->GetGraph()->Modify();
				}
			}
		))
	);
	section.AddMenuEntry(TEXT("DeleteEntry"), FText::FromString(TEXT("Delete Node")),
		FText::FromString(TEXT("Delete the node")),
		FSlateIcon(TEXT("HtnStyle"), TEXT("HtnAssetEditor.NodeDeleteNodeIcon")),
		FUIAction(FExecuteAction::CreateLambda
		(
			[node]() {
				node->GetGraph()->RemoveNode(node);
			}
		))
	);
}

```

