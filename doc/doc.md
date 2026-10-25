# demo-swift-moneybook - Documentation

Last updated: 2026-10-25

> Back to [README](../README.md)

## Prerequisites

- macOS with Xcode 14.3 or later (the project has `LastUpgradeCheck = 1430` and needs the iOS 16.4 SDK)
- Swift 5 (`SWIFT_VERSION = 5.0`)
- An iOS 16.4+ simulator or device (`IPHONEOS_DEPLOYMENT_TARGET = 16.4`, iPhone and iPad)
- No third-party dependencies (no Swift Package Manager or CocoaPods)

## Installation

### From Source with Xcode

```bash
git clone https://github.com/pardnchiu/demo-swift-moneybook.git
cd demo-swift-moneybook
open Moneybook.xcodeproj
```

Select the `Moneybook` target and any iOS 16.4+ simulator in Xcode, then press Run (`⌘R`).

### Command-line Build (Simulator)

```bash
git clone https://github.com/pardnchiu/demo-swift-moneybook.git
cd demo-swift-moneybook
xcodebuild -project Moneybook.xcodeproj -target Moneybook -sdk iphonesimulator build
```

### Install on a Device

The project uses automatic signing (`CODE_SIGN_STYLE = Automatic`) bound to the original author's `DEVELOPMENT_TEAM` and the bundle identifier `Pardn-Ltd.Moneybook`. Before installing on your own device, switch to your own team and a unique bundle identifier under Signing & Capabilities in Xcode.

## Usage

### Screens

| | | | |
| - | - | - | - |
| ![Home](../image/home.png) | ![Calendar](../image/date.png) | ![Month Picker](../image/month.png) | ![Add](../image/income.png) |
| ![Add](../image/cost.png) | ![Drawer](../image/tab.png) | | |

### Basic

On launch, `SceneDelegate` builds the window in code and sets `homeViewController`, wrapped in a `UINavigationController`, as the root screen:

| Action | Result |
|--------|--------|
| Tap the navigation title (`yyyy年MM月`) | Opens or closes the blurred month picker and rotates the title arrow 180 degrees |
| Tap the arrows in the month picker header | Year -1 / +1 |
| Tap `list.bullet` at the top left | Slides in the profile drawer from the left; the back button closes it |
| Tap `calendar` at the top right | Presents the current-month calendar and entry list modally |
| Tap the `+` button at the bottom | Presents the add screen (category grid + keypad) modally |

### Advanced: Change the Sample Data

The daily entries on the home screen come from the top-level global array `datas` in `Moneybook/view/homeViewController.swift`. Rewrite that array to replace the home content, for example:

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

`cellForItemAt` reads each field with `as? String` / `as? Int`: a field with the wrong type is skipped and the label keeps its default text; when any item in `list` lacks `title` or `price`, the remaining items of that day are not rendered.

### Advanced: Adjust the Pie Percentage

`homeViewController.viewDidLoad()` creates the pie with a fixed value:

```swift
let homePieView = homePieView(70);
```

The argument is a 0–100 percentage converted to an angle: the right half-disc rotates to that angle (capped at 180 degrees) over 1.2 seconds, and about 1.25 seconds later the left half-disc rotates the part beyond 180 degrees over 0.4 seconds.

### Advanced: Compose Views with PPDExtension

Every screen uses PPDExtension's chainable methods. For example, add a centered round button to a `UIViewController`:

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

`UIImage(sys:)` is a failable initializer; when the SF Symbol is missing, `UIButton.img(_:)` falls back to `_image` (`folder.badge.questionmark`).

## Preferences Reference

### Build Settings

| Setting | Value |
|---------|-------|
| Target | `Moneybook` |
| Bundle Identifier | `Pardn-Ltd.Moneybook` |
| Deployment Target | iOS 16.4 |
| Devices | iPhone, iPad (`TARGETED_DEVICE_FAMILY = "1,2"`) |
| Version | `MARKETING_VERSION = 1.0`, `CURRENT_PROJECT_VERSION = 1` |
| Multiple Scenes | Disabled (`UIApplicationSupportsMultipleScenes = false`) |

### Screens and Components

| File | Class | Responsibility |
|------|-------|----------------|
| `view/homeViewController.swift` | `homeViewController` | Home: pie, daily entries, month picker, navigation bar buttons |
| `view/home/homePieView.swift` | `homePieView` | Monthly expense / income labels and the animated percentage pie |
| `view/home/homeCollectViewCell.swift` | `homeCollectViewCell` | Day card (date header + total) |
| `view/home/homeDayRowView.swift` | `homeDayRowView` | Entry row (title + amount, separator on non-first rows) |
| `view/home/monthCollectViewCell.swift` | `monthCollectViewCell` | Month button cell |
| `view/home/monthCollectViewHeader.swift` | `monthCollectViewHeader` | Year header with previous / next year switch |
| `view/home/tabLeftView.swift` | `tabLeftView` | Left drawer: avatar, name, mail button, social buttons |
| `view/calendarViewcontroller.swift` | `calendarViewcontroller` | Current-month calendar grid and entry table |
| `view/calendar/calendarCollectionViewCell.swift` | `calendarCollectionViewCell` | Day cell |
| `view/calendar/dataTableViewCell.swift` | `dataTableViewCell` | Entry row (time, title, amount) |
| `view/addViewController.swift` | `addViewController` | Add screen: income / expense switch, category grid, keypad |
| `view/add/addCollectionViiewCell.swift` | `addCollectionViewCell` | Category button cell |
| `view/add/numPadView.swift` | `numPadView` | Date bar, description field, digit and operator keys, Enter key |

### Home Data Structure (`datas`)

| Key | Type | Purpose |
|-----|------|---------|
| `date` | `String` | Day card header text |
| `total` | `Int` | Day card total on the right; the sample data uses the `price` key, so the total keeps its default text `-1234` |
| `list` | `[[String:Any]]` | Entries of the day, each with `title: String` and `price: Int` |

### Add Screen Categories

| List | Items |
|------|-------|
| `incomeTypes` (displayed) | 薪水, 獎金, 回饋, 交易, 股票, 租金, 投資, 其他, 新增 |
| `costTypes` (defined, not wired) | 娛樂, 日用品, 房租, 醫療, 社交, 禮物, 3C, 其他, 自訂1–3, 新增 |

### Color Palette

| Hex | Usage |
|-----|-------|
| `#F97473` | Primary: add button, selected state, today, Enter key, amounts |
| `#FCC852` | Secondary: pie base, segmented control selection, keypad background, drawer header background |
| `#47B7DF` | Operator keys, mail button |

### PPDExtension Methods

| Extends | Methods | Description |
|---------|---------|-------------|
| Global | `vw`, `vh`, `safeTop()`, `safeBottom()` | Screen width / height and the safe area of the current key window |
| `NSObject` | `_ { }` | Runs a configuration closure on itself and returns itself |
| `UIView` | `init(_:_:_:_:)`, `frame`, `x`, `y`, `alpha`, `radius`, `radius(_:corner:)`, `clip(view:)`, `bg(color:)`, `bg(image:)`, `shadow()`, `border(color:width:)`, `border(vert:width:)`, `child(_:)`, `layer(_:)`, `rotate(angle:)` | Appearance, subviews, layers, and rotation |
| `UIView` (Auto Layout) | `Teq`, `Beq`, `Leq`, `Req`, `Xeq`, `Weq`, `Heq` | Creates and activates the anchor constraint and turns off `translatesAutoresizingMaskIntoConstraints` |
| `UIStackView` | `init(axis:align:fill:gap:)`, `padding(T:/L:/B:/R:)`, `child(_:)`, `child(unsort:)`, `child(_:_:)`, `rm(_:)` | Axis, alignment, distribution, and arranged subviews |
| `UIButton` | `text`, `font`, `subFont`, `img`, `bg(image:mode:)`, `padding(L:/R:)`, `if(button:_:)`, `touch(down:_:)` | Title, image, and events on top of `UIButton.Configuration` |
| `UILabel` | `text(_:color:align:row:wrap:)`, `font(weight:size:)`, `attr(txt:/clr:/size:weight:/charSpacing:/offset:/align:/minH:)` | Plain and attributed text |
| `UICollectionView` | `init(_:_:_:_:_:)`, `proto(_:_:)`, `cell(_:_:)`, `cell(reuse:_:)`, `header(_:_:)`, `padding(vert:/horz:/T:/B:)` | Delegates, registration, and content inset |
| `UIVisualEffectView` | `init(style:)` | Creates a blur view from a `UIBlurEffect.Style` |
| `UIColor` | `init(hex:)`, `init(rgb:)` | `RGB` / `RRGGBB` / `RRGGBBAA` hex and `"r/g/b/a"` strings |
| `UIImage` | `init?(sys:)`, `init?(name:)` | SF Symbols and asset images |
| `Date` | `startOfMonth`, `endOfMonth`, `endOfPreMonth`, `preMonth`, `nextMonth` | Month boundaries for the calendar |

### Current Limitations

This project is a UI implementation demo:

- All data is hard-coded sample data; no storage or persistence exists
- The keypad, category buttons, drawer buttons, calendar day cells, and month buttons have no actions bound
- The income / expense segmented control on the add screen does not switch the category list
- The month picker always highlights May, and the pie percentage is fixed at 70%

***

©️ 2023 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
