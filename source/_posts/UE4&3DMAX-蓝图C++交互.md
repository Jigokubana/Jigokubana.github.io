---
title: 蓝图与C++交互
date: 2020-10-11 20:31:22
categories: 
- UE4&3DMAX
---

# 蓝图Widget与C++代码交互-反射

1. 创建UserWidget的C++子类 MyUserWidget
```c++
// 用于初始化
virtual bool Initialize() override;
// 绑定到按钮的点击事件
UFUNCTION() // 必须加不然会导致无反应 日志报错
void ButtonLoginEvent();

// 通过反射获取实例 需要名称保持一致
UPROPERTY(Meta = (BindWidget))
UButton* ButtonLogin;
UPROPERTY(Meta = (BindWidget))
UTextBlock* TextBlockUsername; // 一个文本框

bool UMyUserWidget::Initialize()
{
    if (!Super::Initialize()) // 会报错提示不存在 无影响
    {
        return false;
    }

    // 进行点击函数绑定
    ButtonLogin->OnClicked.__Internal_AddDynamic(this, &UMyUserWidget::ButtonLoginEvent, FName("ButtonLoginEvent"));

    return true;
}

// 获取并显示 文本框文字
void UMyUserWidget::ButtonLoginEvent()
{
    if (GEngine)
    {
        FString Username = TextBlockUsername->GetText().ToString();

        GEngine->AddOnScreenDebugMessage(-1, 20, FColor::Yellow, Username);
    }
}
```
2. 创建需要绑定的蓝图NewWidgetBlueprint
3. 打开蓝图后点击File->Reparent Blueprint选择MyUserWidget作为父类
4. 添加名为ButtonLogin的按钮和TextBlockUsername的文本框 用于反射
5. 创建HUD的C++子类MyHud
6. 创建MyHud的蓝图BP_MuHud
7. 将MyHud的蓝图设置为 GameMode的HUDClass, 这样启动的时候就能创建MuHud蓝图的实例
```c++
// 从BP_MuHud中选择 NewWidgetBlueprint蓝图
UPROPERTY(EditAnywhere, Category = "UserWidget")
TSubclassOf<class UMUserWidget> WidgetClass;

// 启动时创建蓝图实例并显示 设置输入方式
void AMyHUD::BeginPlay()
{
    Super::BeginPlay();

    UUserWidget* Widget = CreateWidget<UMyUserWidget>(GetWorld(), WidgetClass);
    if (Widget != nullptr)
    {
        Widget->AddToViewport();
    }

    APlayerController* MyPlayerController = Cast<APlayerController>(UGameplayStatics::GetPlayerController(GetWorld(), 0));

    // 输入只对UI
    if (MyPlayerController != nullptr)
    {
        MyPlayerController->bShowMouseCursor = true;
        FInputModeUIOnly InputMode;
        MyPlayerController->SetInputMode(InputMode);
    }
}
```