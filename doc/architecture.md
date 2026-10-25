# demo-swift-moneybook - Architecture

Last updated: 2026-10-25

> Back to [README](../README.md)

## Overview

```mermaid
graph TB
    App[AppDelegate] --> Scene[SceneDelegate]
    Scene --> Nav[UINavigationController]
    Nav --> Home[homeViewController]
    Home --> Pie[homePieView]
    Home --> Days[Daily Entries CollectionView]
    Home --> Month[Month Picker]
    Home -->|added to navigationController.view| Drawer[tabLeftView Drawer]
    Home -->|present| Calendar[calendarViewcontroller]
    Home -->|present| Add[addViewController]
    Data[(Global Sample Data datas)] --> Days
    Ext[PPDExtension] -.-> Home
    Ext -.-> Calendar
    Ext -.-> Add
```

## Module: Launch

`AppDelegate` returns the default scene configuration, and `SceneDelegate` builds the window and navigation stack in code.

```mermaid
graph TB
    subgraph Launch
        A[AppDelegate @main] --> B[UISceneConfiguration Default Configuration]
        B --> C[SceneDelegate.scene willConnectTo]
        C --> D[UIWindow windowScene]
        D --> E[UINavigationController]
        E --> F[homeViewController]
    end
    Plist[Info.plist Scene Manifest] --> B
```

## Module: homeViewController

The home screen consists of a scroll container, a blurred month picker, and a bottom add button.

```mermaid
graph TB
    subgraph homeViewController
        Title[Title Button yyyy年MM月]
        Left[Left list.bullet]
        Right[Right calendar]
        Scroll[UIScrollView] --> Stack[Vertical UIStackView]
        Stack --> Pie[homePieView]
        Stack --> Days[Daily UICollectionView]
        Days --> DayCell[homeCollectViewCell]
        DayCell --> Row[homeDayRowView]
        Scroll --> AddBtn[Add Button +]
        Blur[Blur UIVisualEffectView] --> MonthGrid[Month UICollectionView]
        MonthGrid --> MonthHeader[monthCollectViewHeader]
        MonthGrid --> MonthCell[monthCollectViewCell]
    end
    Data[(datas)] --> Days
    Title -->|navTitleTap| Blur
    Left -->|navLeftBar| Drawer[tabLeftView]
    Right -->|navRightCanendar| Calendar[calendarViewcontroller]
    AddBtn -->|addItem| Add[addViewController]
```

## Module: homePieView

Two clipped half-disc views rotate together to form the percentage.

```mermaid
graph TB
    subgraph homePieView
        Labels[Monthly Expense / Income Labels]
        Base[Base Disc FCC852] --> RightClip[Right Half Clip]
        Base --> LeftClip[Left Half Clip]
        RightClip --> RightView[rightView Half-disc]
        LeftClip --> LeftView[leftView Half-disc]
        Base --> Center[Center White Disc Percentage Text]
    end
    Percent[percent Argument] --> Angle[Angle = percent / 100 x 360]
    Angle --> RightView
    Angle --> LeftView
```

## Module: calendarViewcontroller

The calendar grid is generated from the current month at runtime, with an entry table below.

```mermaid
graph TB
    subgraph calendarViewcontroller
        First[First Weekday firstDay]
        Last[Days in Month lastDate]
        Pre[Previous Month Last Day preLastDate]
        List[Day Cell List list]
        Grid[Date UICollectionView] --> DateCell[calendarCollectionViewCell]
        Table[Entry UITableView] --> RowCell[dataTableViewCell]
    end
    DateExt[Date Extension startOfMonth / endOfMonth / endOfPreMonth] --> First
    DateExt --> Last
    DateExt --> Pre
    First --> List
    Last --> List
    Pre --> List
    List --> Grid
    Sample[(Private Sample Data datas)] --> Table
```

## Module: addViewController

An income / expense segmented control sits on top, the category grid in the middle, and the keypad at the bottom.

```mermaid
graph TB
    subgraph addViewController
        Seg[UISegmentedControl 收入 / 支出]
        Grid[Category UICollectionView] --> Cell[addCollectionViewCell]
        Pad[numPadView]
    end
    Income[(incomeTypes)] --> Grid
    Cost[(costTypes Unused)] -.-> Grid
    subgraph numPadView
        DateBar[Date Bar Previous / Date / Next]
        Field[Description UITextField + Amount Label]
        Keys[Digit Keys 0-9 00 .]
        Ops[Operator Keys ÷ × + - AC ←]
        Enter[Enter Key]
    end
    Pad --> DateBar
    Pad --> Field
    Pad --> Keys
    Pad --> Ops
    Pad --> Enter
```

## Module: tabLeftView

An overlay added to `navigationController.view`, 70% of the screen width.

```mermaid
graph TB
    subgraph tabLeftView
        Dim[Translucent Dim Layer]
        Panel[tabView White Panel]
        Panel --> Header[Top FCC852 Block]
        Panel --> Back[Back Button]
        Panel --> Head[Avatar head]
        Panel --> Name[Name]
        Panel --> Mail[Open Mail Button]
        Panel --> Social[Github / Linkedin / Facebook / Instagram]
    end
    Assets[(Assets.xcassets Icons)] --> Head
    Assets --> Social
```

## Module: PPDExtension

A set of self-returning UIKit extensions that every screen is chained from.

```mermaid
graph TB
    subgraph PPDExtension
        Index[index.swift vw / vh / safeTop / safeBottom / _ Closure]
        View[UIView Appearance / child / layer / rotate]
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
    Index --> Screens[Screens and Components]
    Layout --> Screens
    Button --> Screens
```

## Data Flow

```mermaid
sequenceDiagram
    participant User
    participant Home as homeViewController
    participant Data as datas
    participant Cell as homeCollectViewCell
    participant Modal as calendar / add
    Home->>Data: numberOfItemsInSection reads datas.count
    Home->>Cell: cellForItemAt sets date / total
    Cell->>Cell: Builds homeDayRowView from list
    Home->>Home: viewWillAppear resets height from contentSize
    User->>Home: Tap calendar or + button
    Home->>Modal: present(animated: true)
```

## State Machine

### Month Picker

```mermaid
stateDiagram-v2
    [*] --> Hidden
    Hidden --> Shown: Tap title / panel slides down, grid fades in after 0.3s
    Shown --> Hidden: Tap title / grid fades out, panel slides up after 0.3s
```

### Drawer

```mermaid
stateDiagram-v2
    [*] --> SlidingIn: navLeftBar creates tabLeftView
    SlidingIn --> Visible: 0.2s fade in and move to x = 0
    Visible --> Dismissed: tabBack fades out and moves back to -tabWidth (stays in view hierarchy)
```

***

©️ 2023 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
