---
title: "くそめも"
slug: "memo"
order: 4
source: "swift"
---

# SwiftUI & CoreData めも

SwiftUIとCoreDataを使って開発していく中でのメモ。思い出しやすいように、よく使うパターン・お作法をまとめています。

---

## 💡 `onAppear` とは？

Viewが画面に表示されたタイミングで何か処理を実行したいときに使う。

```swift
Text("Hello")
    .onAppear {
        print("items", items)
    }
```

---

## 🧠 CoreDataの使い方

### 1. `viewContext` を環境にセット
CoreDataを使うには `NSManagedObjectContext` を環境に渡しておく。

```swift
.environment(\.managedObjectContext, container.viewContext)
```

### 2. Contextの保持
ViewModelやViewで使うために定義しておく：

```swift
private let context: NSManagedObjectContext
```

### 3. データ取得：`NSFetchRequest`

```swift
let request: NSFetchRequest<Item> = Item.fetchRequest()
request.sortDescriptors = [
    NSSortDescriptor(keyPath: \Item.createdAt, ascending: true)
]
```

#### 🔍 `NSFetchRequest<Item>` の構成

| 要素        | 内容                                      |
|-------------|-------------------------------------------|
| Entity      | どのエンティティを対象にするか → `Item` |
| 並び順      | `sortDescriptors`                         |
| 条件        | `predicate`                               |
| 件数制限    | `fetchLimit`                              |

---

## 🧩 ViewModelのお作法

MVVMの基本。データとUIの責務を分離する。

### ✅ ViewModelとは？

- **データとロジック**の置き場
- Viewはあくまで表示と操作のみを書く

```swift
📦 ViewModel
├─ データ保持（tasks）
├─ CoreData操作（fetch, add, delete）
└─ 状態変化を @Published で通知

🖼️ View
├─ @StateObject で ViewModel を持つ
├─ 表示（List）
└─ UI操作（ボタン）→ ViewModelの関数を呼ぶ
```

---

## 🗂️ `enum`の使い方

### ✅ ポイントまとめ

| 要素                  | 内容                                            |
|-----------------------|-------------------------------------------------|
| enum名                | 意味のある名詞 → `TaskIcon`（タスク用アイコン）|
| case                  | 状態や種類を表す → `.delete`, `.done(Bool)`    |
| computed property     | 見た目の制御に使う → `systemName`, `color`     |
| switch文              | 各状態で処理を分岐                              |

---

## ⚙️ クロージャ（Closure）

```swift
let value = { return 42 }()
```

### ✅ 覚え方

> 「この関数、すぐ実行してその値くれ！」

---

## 🌀 `NSSortDescriptor` のおさらい

| 項目     | 内容                                           |
|----------|------------------------------------------------|
| 所属     | Foundation（Objective-C由来）                  |
| 用途     | CoreData、NSArray、NSFetchRequestなど          |
| Swiftでの役割 | CoreDataのクエリでの並び替えに必須          |
| 代替     | `sorted {}` を使うのも可（ただし非効率なことも）|

---

## 📱 `.sheet(item:)` の仕組み

- `Identifiable` なオブジェクトを渡すことで、`sheet` を表示する形式。
- `nil → 非nil` になった瞬間に `sheet` が表示される。

```swift
.sheet(item: $editingTask) { task in
    // 編集用のビューなど
}
```

> ※ `Item` は `Identifiable` に準拠している（CoreDataではデフォルト）

---

## 🧬 `@State` ってなに？

> SwiftUIにおける「画面の状態」を保持するためのプロパティラッパー。

```swift
@State private var name: String = ""
```

- ユーザー操作で変わる値に使う
- 値が変わると画面が自動的に再描画される

### ✅ 使用例

```swift
TextField("名前", text: $name)
Text("こんにちは、\(name)さん！")
```

---

## 🔁 状態のやりとりの図解

```text
@State var name
   ↓
$name（Binding）
   ↓
子Viewの @Binding var name
```

→ 親Viewから子Viewに状態を渡すときの基本パターン。

---

## init()とは
View が作られるタイミングで最初に1回だけ呼ばれる特別な関数です。

## initとstateobjectのお作法
initで初期化するのがお作法

```
struct MyView: View {
    @StateObject var viewModel: MyViewModel

    init() {
        _viewModel = StateObject(wrappedValue: MyViewModel())
    }
}
```

## よく使う「ソート」の書き方

```
return logs.sorted { ($0.date ?? .distantPast) > ($1.date ?? .distantPast) }
```
✅ .distantPast ってなに？
Swift の Date 型が持ってる めっちゃ昔の日付（紀元前レベル）。

「ログの配列を、date の値（または昔の日）で比較して、新しい順に並べて返してる」

✅ 応用ポイント
sorted { $0.name ?? "" < $1.name ?? "" } → 名前順にも使える

filter や map と組み合わせるとさらに便利

## NavigationLink
次の画面に行くためのボタン

```
✅ 最小の例
swift
コピーする
編集する
NavigationLink("次へ") {
    Text("これは遷移先の画面です")
}
```

## reactに例えながら値の更新通知について

@Publisedをつけると、値の更新時に通知されるようになる

@ObservedObjectは親Viewから渡されるこれを監視する
```
    @ObservedObject var habit: Habit
```

# 日付処理で覚えておくこと

| 覚えるべき   | 内容              | SwiftUI / iOSでの具体例                           |
| ------- | --------------- | -------------------------------------------- |
| ① 日付の生成 | 今日、月初などのDateを作る | `Date()`, `Calendar.current.date(...)`       |
| ② 日付の比較 | 同じ日？今日？未来？      | `Calendar.current.isDate(_:inSameDayAs:)`    |
| ③ 日付の整形 | 表示用の文字列         | `DateFormatter().string(from:)`              |
| ④ 範囲の取得 | 月の日数、週の範囲など     | `calendar.range(of: .day, in: .month, for:)` |

意識しておくと今後楽になる

## dismissとは

SwiftUI で用意されている 環境変数の1つで、
シート（.sheet）やポップオーバーなどで表示された画面を 閉じる処理として使います。

```
Button("保存") {
    // データ保存処理...

    dismiss() // ← 表示されている画面（シート）を閉じる
}
```


# ✅ ミニマム版 PersistenceController.swift

```
struct PersistenceController {
    static let shared = PersistenceController()

    let container: NSPersistentContainer

    init(inMemory: Bool = false) {
        container = NSPersistentContainer(name: "MealLog") // ← .xcdatamodeld のファイル名と一致させること

        if inMemory {
            container.persistentStoreDescriptions.first?.url = URL(fileURLWithPath: "/dev/null")
        }

        container.loadPersistentStores { _, error in
            if let error = error as NSError? {
                fatalError("Core Data store failed: \(error), \(error.userInfo)")
            }
        }

        container.viewContext.automaticallyMergesChangesFromParent = true
    }
}
```



## Delete Rule
| Delete Rule   | 説明                                         |
| ------------- | ------------------------------------------ |
| **Nullify**   | リレーション先が削除されても、自分の参照だけ `nil` にする（安全でよく使う）  |
| **Cascade**   | リレーション先が削除されたら、自分自身も削除される（子が親に依存しているときに使う） |
| **Deny**      | リレーション先が使われている限り削除を許さない（削除失敗させたい時）         |
| **No Action** | 何もしない（手動管理向け、普通は使わない）                      |

# Appファイルから@StateObject -> .environmentObjectで下に渡すってのはお作法
ViewModelはこうやって渡すのがお作法。毎回viewmodelが生成されるため。

```
// Appファイル
@main
struct MyApp: App {
    @StateObject var viewModel = MyViewModel()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(viewModel)
        }
    }
}
```

```
// 子ビュー
struct ContentView: View {
    @EnvironmentObject var viewModel: MyViewModel
    ...
}
```

```
Appで定義
|
|- .environmentObjectに渡してあげる
|-- ContentView
|--- @EnvironmentObjectで受け取れる
```

---

## 設計 - エラーハンドリング

### 1. 失敗しそうな操作は「適切に伝える」
意識: エラーが発生する可能性があるなら、黙殺しない。必ず呼出元にその事実を伝える。
実装: エラー/失敗の可能性を型で明示することもある
```
Optional
Result<Success, Failuer>
Result<[None], DataServiceError>
```

### 2. エラーは「最も適切に処理できる場所」でキャッチをする
意識: エラー発生場所で全て処理しない。最も有意義に解釈しできて回復もでき、ユーザへ情報を提供できるレイヤーまで伝搬させることもある
MVVM: 低レベル層はカスタムエラーを変換、中間層はカスタムエラーをキャッチ。UI層は通知する。


### 3. エラーの種類に応じた適切な対応をする
意識: ユーザにどう伝えるか、アプリとしてどう回復継続するか
やること: 回復可能ならユーザの再試行を促す、デフォルト値を使う。不能なら安全に終了を促す。ログを出す。


### 4. エラーはなるべく早く発見、なるべく早く伝える
意識: 問題が起きたらすぐにそれを意識し、コード上の適切な場所でエラーとして表現する
やること: 早期リターンを徹底する