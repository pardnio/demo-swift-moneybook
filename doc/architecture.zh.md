# demo-swift-moneybook - 架構

最後更新：2026-10-25

> 返回 [README](./README.zh.md)

## 概覽

```mermaid
graph TB
    App[AppDelegate] --> Scene[SceneDelegate]
    Scene --> Nav[UINavigationController]
    Nav --> Home[首頁 homeViewController]
    Home --> Pie[圓餅圖 homePieView]
    Home --> Days[每日明細 CollectionView]
    Home --> Month[月份選擇器]
    Home -->|加入 navigationController.view| Drawer[側邊選單 tabLeftView]
    Home -->|present| Calendar[日曆 calendarViewcontroller]
    Home -->|present| Add[新增 addViewController]
    Data[(全域範例資料 datas)] --> Days
    Ext[PPDExtension] -.-> Home
    Ext -.-> Calendar
    Ext -.-> Add
```

## Module: 啟動流程

`AppDelegate` 回傳預設場景設定，`SceneDelegate` 以純程式碼建立視窗與導覽堆疊。

```mermaid
graph TB
    subgraph 啟動流程
        A[AppDelegate @main] --> B[UISceneConfiguration Default Configuration]
        B --> C[SceneDelegate.scene willConnectTo]
        C --> D[UIWindow windowScene]
        D --> E[UINavigationController]
        E --> F[homeViewController]
    end
    Plist[Info.plist 場景設定] --> B
```

## Module: 首頁 homeViewController

首頁由捲動容器、毛玻璃月份選擇器與底部新增按鈕組成。

```mermaid
graph TB
    subgraph homeViewController
        Title[標題按鈕 yyyy年MM月]
        Left[左側 list.bullet]
        Right[右側 calendar]
        Scroll[UIScrollView] --> Stack[垂直 UIStackView]
        Stack --> Pie[homePieView]
        Stack --> Days[每日 UICollectionView]
        Days --> DayCell[homeCollectViewCell]
        DayCell --> Row[homeDayRowView]
        Scroll --> AddBtn[新增按鈕 +]
        Blur[毛玻璃 UIVisualEffectView] --> MonthGrid[月份 UICollectionView]
        MonthGrid --> MonthHeader[monthCollectViewHeader]
        MonthGrid --> MonthCell[monthCollectViewCell]
    end
    Data[(datas)] --> Days
    Title -->|navTitleTap| Blur
    Left -->|navLeftBar| Drawer[tabLeftView]
    Right -->|navRightCanendar| Calendar[calendarViewcontroller]
    AddBtn -->|addItem| Add[addViewController]
```

## Module: 圓餅圖 homePieView

以兩個裁切的半圓視圖旋轉組合出百分比。

```mermaid
graph TB
    subgraph homePieView
        Labels[月支出 / 月收入 標籤]
        Base[底圓 FCC852] --> RightClip[右半裁切容器]
        Base --> LeftClip[左半裁切容器]
        RightClip --> RightView[rightView 半圓]
        LeftClip --> LeftView[leftView 半圓]
        Base --> Center[中央白圓 百分比文字]
    end
    Percent[percent 參數] --> Angle[角度 = percent / 100 x 360]
    Angle --> RightView
    Angle --> LeftView
```

## Module: 日曆 calendarViewcontroller

依當月日期即時產生日曆格，下方以表格呈現明細。

```mermaid
graph TB
    subgraph calendarViewcontroller
        First[當月首日星期 firstDay]
        Last[當月天數 lastDate]
        Pre[上月最後一天 preLastDate]
        List[日期格清單 list]
        Grid[日期 UICollectionView] --> DateCell[calendarCollectionViewCell]
        Table[明細 UITableView] --> RowCell[dataTableViewCell]
    end
    DateExt[Date 擴充 startOfMonth / endOfMonth / endOfPreMonth] --> First
    DateExt --> Last
    DateExt --> Pre
    First --> List
    Last --> List
    Pre --> List
    List --> Grid
    Sample[(私有範例資料 datas)] --> Table
```

## Module: 新增 addViewController

上方為收入／支出分段控制，中間為分類網格，下方為數字鍵盤。

```mermaid
graph TB
    subgraph addViewController
        Seg[UISegmentedControl 收入 / 支出]
        Grid[分類 UICollectionView] --> Cell[addCollectionViewCell]
        Pad[numPadView]
    end
    Income[(incomeTypes)] --> Grid
    Cost[(costTypes 未使用)] -.-> Grid
    subgraph numPadView
        DateBar[日期列 前一天 / 日期 / 後一天]
        Field[描述 UITextField + 金額標籤]
        Keys[數字鍵 0-9 00 .]
        Ops[運算鍵 ÷ × + - AC ←]
        Enter[Enter 鍵]
    end
    Pad --> DateBar
    Pad --> Field
    Pad --> Keys
    Pad --> Ops
    Pad --> Enter
```

## Module: 側邊選單 tabLeftView

加在 `navigationController.view` 上的覆蓋層，寬度為螢幕 70%。

```mermaid
graph TB
    subgraph tabLeftView
        Dim[半透明遮罩]
        Panel[tabView 白色面板]
        Panel --> Header[頂部 FCC852 區塊]
        Panel --> Back[返回按鈕]
        Panel --> Head[頭像 head]
        Panel --> Name[名稱]
        Panel --> Mail[開啟信箱按鈕]
        Panel --> Social[Github / Linkedin / Facebook / Instagram]
    end
    Assets[(Assets.xcassets 圖示)] --> Head
    Assets --> Social
```

## Module: PPDExtension

一組回傳自身的 UIKit 擴充，所有畫面都以此鏈式組成。

```mermaid
graph TB
    subgraph PPDExtension
        Index[index.swift vw / vh / safeTop / safeBottom / _ 閉包]
        View[UIView 外觀 / child / layer / rotate]
        Layout[Autolayout Teq / Beq / Leq / Req / Xeq / Weq / Heq]
        Stack[UIStackView]
        Button[UIButton Configuration]
        Label[UILabel / Attribute]
        Collection[UICollectionView]
        Misc[UIColor / UIImage / Date / CGRect / CGSize / CGPoint / CAGradientLayer / UIVisualEffectView]
    end
    View --> Layout
    View --> Stack
    View --> Button
    View --> Label
    View --> Collection
    UIKit[UIKit] --> View
    Index --> Screens[各畫面與元件]
    Layout --> Screens
    Button --> Screens
```

## 資料流

```mermaid
sequenceDiagram
    participant User as 使用者
    participant Home as homeViewController
    participant Data as datas
    participant Cell as homeCollectViewCell
    participant Modal as calendar / add
    Home->>Data: numberOfItemsInSection 讀取 datas.count
    Home->>Cell: cellForItemAt 設定 date / total
    Cell->>Cell: 依 list 建立 homeDayRowView
    Home->>Home: viewWillAppear 以 contentSize 重設高度
    User->>Home: 點擊日曆或 + 按鈕
    Home->>Modal: present(animated: true)
```

## 狀態機

### 月份選擇器

```mermaid
stateDiagram-v2
    state "隱藏" as Hidden
    state "展開" as Shown
    [*] --> Hidden
    Hidden --> Shown: 點擊標題 / 面板下滑 0.3 秒後網格淡入
    Shown --> Hidden: 點擊標題 / 網格淡出 0.3 秒後面板上移
```

### 側邊選單

```mermaid
stateDiagram-v2
    state "滑入中" as SlidingIn
    state "顯示" as Visible
    state "已收合" as Dismissed
    [*] --> SlidingIn: navLeftBar 建立 tabLeftView
    SlidingIn --> Visible: 0.2 秒淡入並移至 x = 0
    Visible --> Dismissed: tabBack 淡出並移回 -tabWidth（仍留在視圖階層）
```

***

©️ 2023 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
