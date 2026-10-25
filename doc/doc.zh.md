# demo-swift-moneybook - 技術文件

最後更新：2026-10-25

> 返回 [README](./README.zh.md)

## 前置需求

- macOS 並安裝 Xcode 14.3 以上版本（專案 `LastUpgradeCheck = 1430`，需 iOS 16.4 SDK）
- Swift 5（`SWIFT_VERSION = 5.0`）
- iOS 16.4 以上的模擬器或實機（`IPHONEOS_DEPLOYMENT_TARGET = 16.4`，支援 iPhone 與 iPad）
- 無第三方相依套件（未使用 Swift Package Manager 或 CocoaPods）

## 安裝

### 從原始碼以 Xcode 執行

```bash
git clone https://github.com/pardnchiu/demo-swift-moneybook.git
cd demo-swift-moneybook
open Moneybook.xcodeproj
```

在 Xcode 選擇 `Moneybook` target 與任一 iOS 16.4+ 模擬器，按下 Run（`⌘R`）。

### 以命令列建置（模擬器）

```bash
git clone https://github.com/pardnchiu/demo-swift-moneybook.git
cd demo-swift-moneybook
xcodebuild -project Moneybook.xcodeproj -target Moneybook -sdk iphonesimulator build
```

### 安裝至實機

專案使用自動簽署（`CODE_SIGN_STYLE = Automatic`），並綁定原作者的 `DEVELOPMENT_TEAM` 與 Bundle Identifier `Pardn-Ltd.Moneybook`。安裝到自己的裝置前，於 Xcode 的 Signing & Capabilities 改為自己的 Team 與唯一的 Bundle Identifier。

## 使用方式

### 畫面

| | | | |
| - | - | - | - |
| ![首頁](../image/home.png) | ![日曆](../image/date.png) | ![月份選擇器](../image/month.png) | ![新增](../image/income.png) |
| ![新增](../image/cost.png) | ![側邊選單](../image/tab.png) | | |

### 基礎操作

App 啟動後由 `SceneDelegate` 以純程式碼建立視窗，根畫面為包在 `UINavigationController` 內的 `homeViewController`：

| 操作 | 結果 |
|------|------|
| 點擊導覽列標題（`yyyy年MM月`） | 展開／收合毛玻璃月份選擇器，標題箭頭旋轉 180 度 |
| 月份選擇器標頭的左右箭頭 | 年份 -1／+1 |
| 點擊左上 `list.bullet` | 從左側滑出個人側邊選單，點擊返回鍵收合 |
| 點擊右上 `calendar` | 以 modal 呈現當月日曆與明細列表 |
| 點擊底部 `+` 按鈕 | 以 modal 呈現新增畫面（分類網格 + 數字鍵盤） |

### 進階：修改範例資料

首頁的每日明細來自 `Moneybook/view/homeViewController.swift` 頂層的全域陣列 `datas`。直接改寫該陣列即可替換首頁內容，例如：

```swift
import UIKit

var datas: [[String:Any]] = [
	[
		"date": "2023/05/04 星期四",
		"total": -579,
		"list": [
			[
				"title": "午餐",
				"price": 120
			] as [String : Any],
			[
				"title": "交通",
				"price": 459
			] as [String : Any]
		]
	]
];
```

`cellForItemAt` 以 `as? String`／`as? Int` 讀取各欄位：型別不符的欄位會被略過並保留預設文字；`list` 中任一筆缺少 `title` 或 `price` 時，該日其後的明細不再渲染。

### 進階：調整圓餅比例

`homeViewController.viewDidLoad()` 以固定值建立圓餅圖：

```swift
let homePieView = homePieView(70);
```

參數為 0–100 的百分比，換算為角度後：右半圓先以 1.2 秒旋轉至該角度（上限 180 度），約 1.25 秒後左半圓再以 0.4 秒補上超過 180 度的部分。

### 進階：以 PPDExtension 組畫面

所有 UI 都以 PPDExtension 的鏈式方法撰寫。例如在 `UIViewController` 中加入一顆置中的圓形按鈕：

```swift
import UIKit

class sampleViewController: UIViewController {

	override func viewDidLoad() {
		super.viewDidLoad();

		let button = UIButton()
			.img(UIImage(sys: "plus"))
			.text(color: .white, align: .center)
			.bg(color: UIColor(hex: "F97473"))
			.border(color: .black, width: 2)
			.radius(30)
			.touch(down: self, #selector(tap));

		_=view.child([button]);

		_=button
			.Xeq(X: view)
			.Beq(B: view, -30)
			.Weq(60)
			.Heq(60);
	};

	@objc func tap() {
		self.present(addViewController(), animated: true);
	};
};
```

`UIImage(sys:)` 為 failable initializer，找不到 SF Symbol 時 `UIButton.img(_:)` 會改用 `_image`（`folder.badge.questionmark`）作為備援圖示。

## 偏好設定參考

### 建置設定

| 設定 | 值 |
|------|----|
| Target | `Moneybook` |
| Bundle Identifier | `Pardn-Ltd.Moneybook` |
| Deployment Target | iOS 16.4 |
| 裝置 | iPhone、iPad（`TARGETED_DEVICE_FAMILY = "1,2"`） |
| 版本 | `MARKETING_VERSION = 1.0`、`CURRENT_PROJECT_VERSION = 1` |
| 多場景 | 停用（`UIApplicationSupportsMultipleScenes = false`） |

### 畫面與元件

| 檔案 | 類別 | 職責 |
|------|------|------|
| `view/homeViewController.swift` | `homeViewController` | 首頁：圓餅圖、每日明細、月份選擇器、導覽列按鈕 |
| `view/home/homePieView.swift` | `homePieView` | 月支出／月收入標籤與動畫比例圓餅 |
| `view/home/homeCollectViewCell.swift` | `homeCollectViewCell` | 單日卡片（日期標頭 + 合計） |
| `view/home/homeDayRowView.swift` | `homeDayRowView` | 單筆明細列（標題 + 金額，非首列加分隔線） |
| `view/home/monthCollectViewCell.swift` | `monthCollectViewCell` | 月份按鈕格 |
| `view/home/monthCollectViewHeader.swift` | `monthCollectViewHeader` | 年份標頭與前後年切換 |
| `view/home/tabLeftView.swift` | `tabLeftView` | 左側滑出選單：頭像、名稱、信箱按鈕與社群按鈕 |
| `view/calendarViewcontroller.swift` | `calendarViewcontroller` | 當月日曆格與明細表格 |
| `view/calendar/calendarCollectionViewCell.swift` | `calendarCollectionViewCell` | 日期格 |
| `view/calendar/dataTableViewCell.swift` | `dataTableViewCell` | 明細列（時間、標題、金額） |
| `view/addViewController.swift` | `addViewController` | 新增畫面：收入／支出切換、分類網格、數字鍵盤 |
| `view/add/addCollectionViiewCell.swift` | `addCollectionViewCell` | 分類按鈕格 |
| `view/add/numPadView.swift` | `numPadView` | 日期列、描述輸入欄、數字與運算鍵、Enter 鍵 |

### 首頁資料結構（`datas`）

| 鍵 | 型別 | 用途 |
|----|------|------|
| `date` | `String` | 日卡片標頭文字 |
| `total` | `Int` | 日卡片右側合計；範例資料使用 `price` 鍵，因此合計維持預設文字 `-1234` |
| `list` | `[[String:Any]]` | 當日明細，每筆含 `title: String` 與 `price: Int` |

### 新增畫面分類

| 清單 | 項目 |
|------|------|
| `incomeTypes`（目前顯示） | 薪水、獎金、回饋、交易、股票、租金、投資、其他、新增 |
| `costTypes`（已定義、尚未接上） | 娛樂、日用品、房租、醫療、社交、禮物、3C、其他、自訂1–3、新增 |

### 配色

| Hex | 用途 |
|-----|------|
| `#F97473` | 主色：新增按鈕、選取狀態、今天、Enter 鍵、金額 |
| `#FCC852` | 輔色：圓餅底色、分段控制選取色、數字鍵盤背景、側邊選單頂部背景 |
| `#47B7DF` | 運算鍵、信箱按鈕 |

### PPDExtension 方法

| 擴充對象 | 方法 | 說明 |
|----------|------|------|
| 全域 | `vw`、`vh`、`safeTop()`、`safeBottom()` | 螢幕寬高與目前 key window 的 safe area |
| `NSObject` | `_ { }` | 對自身執行設定閉包後回傳自身 |
| `UIView` | `init(_:_:_:_:)`、`frame`、`x`、`y`、`alpha`、`radius`、`radius(_:corner:)`、`clip(view:)`、`bg(color:)`、`bg(image:)`、`shadow()`、`border(color:width:)`、`border(vert:width:)`、`child(_:)`、`layer(_:)`、`rotate(angle:)` | 外觀、子視圖、圖層與旋轉 |
| `UIView`（Auto Layout） | `Teq`、`Beq`、`Leq`、`Req`、`Xeq`、`Weq`、`Heq` | 建立並啟用對應錨點約束，自動關閉 `translatesAutoresizingMaskIntoConstraints` |
| `UIStackView` | `init(axis:align:fill:gap:)`、`padding(T:/L:/B:/R:)`、`child(_:)`、`child(unsort:)`、`child(_:_:)`、`rm(_:)` | 堆疊方向、對齊、分佈與 arranged subview |
| `UIButton` | `text`、`font`、`subFont`、`img`、`bg(image:mode:)`、`padding(L:/R:)`、`if(button:_:)`、`touch(down:_:)` | 基於 `UIButton.Configuration` 的文字、圖示與事件 |
| `UILabel` | `text(_:color:align:row:wrap:)`、`font(weight:size:)`、`attr(txt:/clr:/size:weight:/charSpacing:/offset:/align:/minH:)` | 一般與 attributed 文字 |
| `UICollectionView` | `init(_:_:_:_:_:)`、`proto(_:_:)`、`cell(_:_:)`、`cell(reuse:_:)`、`header(_:_:)`、`padding(vert:/horz:/T:/B:)` | delegate、註冊與 content inset |
| `UIVisualEffectView` | `init(style:)` | 以 `UIBlurEffect.Style` 建立毛玻璃 |
| `UIColor` | `init(hex:)`、`init(rgb:)` | `RGB`／`RRGGBB`／`RRGGBBAA` hex 與 `"r/g/b/a"` 字串 |
| `UIImage` | `init?(sys:)`、`init?(name:)` | SF Symbol 與 Asset 圖片 |
| `Date` | `startOfMonth`、`endOfMonth`、`endOfPreMonth`、`preMonth`、`nextMonth` | 日曆計算用的月份邊界 |

### 目前限制

此專案為 UI 實作範例：

- 所有資料皆為寫死的範例資料，未實作儲存或持久化
- 數字鍵盤、分類按鈕、側邊選單按鈕、日曆日期格與月份按鈕未綁定動作
- 新增畫面的收入／支出分段控制不會切換分類清單
- 月份選擇器固定標示 5 月，圓餅比例固定為 70%

***

©️ 2023 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
