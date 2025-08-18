---
title: "Storyboard使わないAppkitの開発"
slug: "appkit"
order: 5
source: "swift"
---

## お作法

1. Main.stoaryboardを削除
2. buildの設定から設定を削除
3. main.swiftを作成
4. AppDelegate.swiftを編集

### main.swiftを作成
```
import Cocoa

let appDelegate = AppDelegate()
NSApplication.shared.delegate = appDelegate
NSApplication.shared.run()
```

### AppDelegate.swiftを編集

```
import Cocoa

class AppDelegate: NSObject, NSApplicationDelegate {
    var window: NSWindow!

    func applicationDidFinishLaunching(_ notification: Notification) {
        window = NSWindow(
            contentRect: NSMakeRect(0, 0, 400, 300),
            styleMask: [.titled, .closable, .resizable],
            backing: .buffered,
            defer: false
        )
        window.center()
        window.title = "Blank Window"
        window.makeKeyAndOrderFront(nil)
    }

    func applicationShouldTerminateAfterLastWindowClosed(_ sender: NSApplication) -> Bool {
        return true
    }
}
```

## selectboxをいちから構築

```
インスタンスを作成
let popupButton = NSPopUpButton()

選択肢を追加
popupButton.addItems(withTitles: ["Swift", "Python", "C"])

イベントハンドラを登録する
popupButton.target = self
popupButton.action = #selector(languageSelected(_:))
//action: 実行したい関数を #selector(...) で指定


4. 対応する関数を定義する
@objc func languageSelected(_ sender: NSPopUpButton) {
    let selected = sender.titleOfSelectedItem ?? ""
    print("選ばれた言語: \(selected)")
}

```


## loadView, viewDidLoadについて

loadView: 1回のみ
viewDidLoad: viewが構築されるたびに呼ばれる

