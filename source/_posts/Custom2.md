---
title: UE5自定义资源（二）
date: 2024-06-26 20:52:14
tags:
---
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

