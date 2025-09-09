---
title: "めも"
slug: "memo"
order: 4
source: "swift"
---

# SwiftUI & CoreData めも

SwiftUI と CoreData を使って開発していく中でのメモ。思い出しやすいように、よく使うパターン・お作法をまとめています。

---

## 💡 `onAppear` とは？

View が画面に表示されたタイミングで何か処理を実行したいときに使う。

```swift
Text("Hello")
    .onAppear {
        print("items", items)
    }
```

---

## 🧠 CoreData の使い方

### 1. `viewContext` を環境にセット

CoreData を使うには `NSManagedObjectContext` を環境に渡しておく。

```swift
.environment(\.managedObjectContext, container.viewContext)
```

### 2. Context の保持

ViewModel や View で使うために定義しておく：

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

| 要素     | 内容                                    |
| -------- | --------------------------------------- |
| Entity   | どのエンティティを対象にするか → `Item` |
| 並び順   | `sortDescriptors`                       |
| 条件     | `predicate`                             |
| 件数制限 | `fetchLimit`                            |

---

## 🧩 ViewModel のお作法

MVVM の基本。データと UI の責務を分離する。

### ✅ ViewModel とは？

- **データとロジック**の置き場
- View はあくまで表示と操作のみを書く

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

| 要素              | 内容                                            |
| ----------------- | ----------------------------------------------- |
| enum 名           | 意味のある名詞 → `TaskIcon`（タスク用アイコン） |
| case              | 状態や種類を表す → `.delete`, `.done(Bool)`     |
| computed property | 見た目の制御に使う → `systemName`, `color`      |
| switch 文         | 各状態で処理を分岐                              |

---

## ⚙️ クロージャ（Closure）

```swift
let value = { return 42 }()
```

### ✅ 覚え方

> 「この関数、すぐ実行してその値くれ！」

---

## 🌀 `NSSortDescriptor` のおさらい

| 項目           | 内容                                             |
| -------------- | ------------------------------------------------ |
| 所属           | Foundation（Objective-C 由来）                   |
| 用途           | CoreData、NSArray、NSFetchRequest など           |
| Swift での役割 | CoreData のクエリでの並び替えに必須              |
| 代替           | `sorted {}` を使うのも可（ただし非効率なことも） |

---

## 📱 `.sheet(item:)` の仕組み

- `Identifiable` なオブジェクトを渡すことで、`sheet` を表示する形式。
- `nil → 非nil` になった瞬間に `sheet` が表示される。

```swift
.sheet(item: $editingTask) { task in
    // 編集用のビューなど
}
```

> ※ `Item` は `Identifiable` に準拠している（CoreData ではデフォルト）

---

## 🧬 `@State` ってなに？

> SwiftUI における「画面の状態」を保持するためのプロパティラッパー。

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

→ 親 View から子 View に状態を渡すときの基本パターン。

---

## init()とは

View が作られるタイミングで最初に 1 回だけ呼ばれる特別な関数です。

## init と stateobject のお作法

init で初期化するのがお作法

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

## react に例えながら値の更新通知について

@Publised をつけると、値の更新時に通知されるようになる

@ObservedObject は親 View から渡されるこれを監視する

```
    @ObservedObject var habit: Habit
```

# 日付処理で覚えておくこと

| 覚えるべき   | 内容                         | SwiftUI / iOS での具体例                     |
| ------------ | ---------------------------- | -------------------------------------------- |
| ① 日付の生成 | 今日、月初などの Date を作る | `Date()`, `Calendar.current.date(...)`       |
| ② 日付の比較 | 同じ日？今日？未来？         | `Calendar.current.isDate(_:inSameDayAs:)`    |
| ③ 日付の整形 | 表示用の文字列               | `DateFormatter().string(from:)`              |
| ④ 範囲の取得 | 月の日数、週の範囲など       | `calendar.range(of: .day, in: .month, for:)` |

意識しておくと今後楽になる

## dismiss とは

SwiftUI で用意されている 環境変数の 1 つで、
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

| Delete Rule   | 説明                                                                                 |
| ------------- | ------------------------------------------------------------------------------------ |
| **Nullify**   | リレーション先が削除されても、自分の参照だけ `nil` にする（安全でよく使う）          |
| **Cascade**   | リレーション先が削除されたら、自分自身も削除される（子が親に依存しているときに使う） |
| **Deny**      | リレーション先が使われている限り削除を許さない（削除失敗させたい時）                 |
| **No Action** | 何もしない（手動管理向け、普通は使わない）                                           |

# App ファイルから@StateObject -> .environmentObject で下に渡すってのはお作法

ViewModel はこうやって渡すのがお作法。毎回 viewmodel が生成されるため。

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
MVVM: 低レベル層はカスタムエラーを変換、中間層はカスタムエラーをキャッチ。UI 層は通知する。

### 3. エラーの種類に応じた適切な対応をする

意識: ユーザにどう伝えるか、アプリとしてどう回復継続するか
やること: 回復可能ならユーザの再試行を促す、デフォルト値を使う。不能なら安全に終了を促す。ログを出す。

### 4. エラーはなるべく早く発見、なるべく早く伝える

意識: 問題が起きたらすぐにそれを意識し、コード上の適切な場所でエラーとして表現する
やること: 早期リターンを徹底する

---

##　よく使うスタイルについて

###

| Delete Rule            | 説明                                                                                                                                                                | 使い方                                                                                                                                                             |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **resizable**          | View のサイズ調整。適用しない場合、Image は元の画像サイズで表示され、frame モディファイアなどでサイズを変更しようとしても無視されます。                             | 　 Image("...") .resizable()のように使います。                                                                                                                     |
| **Cascade**            | View のアスペクト比を維持しつつ、サイズを調整する。                                                                                                                 | Image("...") .resizable() .scaledToFit()のように使います。                                                                                                         |
| **Image のサイズ調整** | Image のサイズを調整する場合、resizable()を最初に適用し、その後に scaledToFit()や scaledToFill()、そして frame モディファイアを連鎖させるのが一般的なパターンです。 | resizable()はサイズ調整の「許可」を与え、scaledToFit()はサイズ調整の「方法」を指示します。両者を組み合わせることで、アイコンや画像が意図したサイズで表示されます。 |
| **No Action**          | 何もしない（手動管理向け、普通は使わない）                                                                                                                          |

---

#　 LLM の整理

###

### ニューラルネットワーク

ニューロンを模倣した計算モデル
情報を層で少しずつ変換して処理する。という特徴
段階的に抽象度を上げながら理解を深めていく。  
LLM は何百層にもなっていて、複雑なパターンが読み取れる

### LLM の進化

2 つの強みで進化してきた。

#### 1. 事前学習　 Pre-training

一般的なルールやパターン。百科事典的な

#### 2.　ファインチューニング

特定の目的に合わせて微調整する。

#### LLM の学習について

自己教師あり学習という方法。ラベルがない。テキストとのものでモデル訓練している。
また、このモデルをスケールさせればタスク性能も向上するという仮説で GPT-3 くらいまではきていた。

## 自然言語処理について

言葉をコンピュータに理解させる技術のことを指す。これは昔は専用のモデルが必要とされていた。
近年はそうでもない。汎用 LLM とも言われてきている。

### 汎用 LLM について

得意なこと

・テキスト生成
・質問応答
・翻訳・ようやく
・コード生成

## 機械学習との違い

そもそも、「データからルールを自動で学習する」技術。ただデメリット
・特定のタスクに対して特定のモデル。
・学習には大量ラベル月データが必要
・新規タスクは、ゼロからモデル構築

LLM は全く異なるスタンスとなる。
・事前学習で得ている言語の一般知識
・トランファーラーニングによるタスク適応
・スケーラブルなモデル

### トランスフォーマ

LLM は、従来のモデルである RNN, CNN とは別のアテンション概念を使っている。
Tranformer, Transfor ラーニングで構築。これが根本から違うらし。

---

# トランスフォーマ

RNN, LSTM だと長文が文脈保持が難しかったが、並列処理が可能で、単語間の関係も効率的に捉えることができる用になった。

## 主要構造

エンコーダ(BERT が特化)、でこーだ(GPT)、T5(両方)

##　最大の特徴
self-attention-mechanism と呼ばれる、各単語の関連を計算する仕組み。
とにかくこれが文脈を深く理解し、適切なテキスト生成のために欠かせない仕組みとなっている。
具体的には、入力テキストの各単語が、文中の他の単語とどの程度関連してるか、数値的に計算する。重みを調整することができる。

## Attention の仕組み

スケールドットプロダクトアテンションと呼ばれる処理について設名する。

Query: 注目する単語
Key: 他単語の情報、比較の対象
Value: 関連性に基づいて出力される意味情報

ここからが本題。処理の手順

1. クエリとキーの内積計算
2. ソフトマックス関数を使う。スコアを確率分布に変換
3. vaalue を重み付で加算。ベクトルを出力

さらにマルチヘッドアテンションを使い強化している。
これは以下となる

1. 複数の自己注意メカニズムをヘッドを並列に実行
   それぞれのクエリキーバリュー行列を用いる

2. それぞれのヘッドを結合、結合ベクトルを生成

3. 最終的に、統合ベクトルを活用して文章を処理

これで単語間の関係を複数の視点から同時に分析できる。高度なことができる。

## 時代を築いたモデル

| モデル名          | 説明                                                             |
| ----------------- | ---------------------------------------------------------------- |
| BERT Google(2018) | 初のトランスフォーマモデル。Encoder 型。Google 検索              |
| GPT(OpenAI)       | 自己回帰型モデル。文の前方(左側)の文脈のみで次の単語を予測する。 |
| T5 Google(2020)   | Encoder,Decoder トランスフォーマ。テキスト to テキスト           |

## 現在の LLM たち。特徴

| モデル名                    | 説明                                                                                                               |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| Claude4 ファミリー          | --                                                                                                                 |
| Claude4 Opus4 (Anthropic)   | 最大 20 万トークン以上の文脈保持、ツール連携、メモリ保持などもできる。用途はコード生成、複数分析のドキュメント処理 |
| Claude4 Sonnet4             | 3.7 の後継。　バランス型。対話型エージェント。FAQ システム。日常業務の要約・分析                                   |
| Gemini 2.5 ファミリー       | ---                                                                                                                |
| Gemini2.Pro                 | 100 マントークン以上の長文文脈処理に対応。数学、化学、コーディングで良き                                           |
| Gemini2.5Flash              | ノーマル用途に最適化された。日常的なユースケース                                                                   |
| Llama ファミリー            | ---                                                                                                                |
| Llama4 　 Scout             | Moe 構造。コード生成も。専門レポートのようやく、開発支援 AI                                                        |
| Mistral/Mixtral(Mistral.ai) | OSS,MOE 構造。                                                                                                     |
| CommandR+(Cohere)           | RAG（検索拡張生成)に最適化されたモデル。社内ナレッジ検索、法務・契約レビュー支援                                   |

# LLM のトレーニング方法

どうやって言葉を理解するのか。
インプット: ニュース、書籍、web、会話、コードなど.　　
データセットとも言われる。

ただしこれらはクリーンな状態のデータにする必要がある
・スパムの除去
・html タグの除去
・絵文字
・重複データの削除
・文法ミス、誤字修正

揺れも問題になる。Normalization する。

tokenization と呼ばれるが、単語分割する。
・スペースごとに区切る
・形態素解析。品詞単位
・サブワード。よく出現する部分単位で分割

ただし、バイアスかかるため偏ったデータは好ましくない

## トレーニングステップの概要

まず膨大なデータと計算資源がいる。具体的には次のステップ

1. 初期化-モデルのパラメータをランダムに設定する  
   ランダム初期化、Xaview/Glorot 初期化、He 初期化などがある

2. フォワードプロ派ゲーション-入寮を処理し、予測結果を生成
3. ロス計算-予測結果と正解データの誤差を測定する
4. バックプロ派ゲーション-誤差を下にパラメータ修正する
5. エポックの繰り返し-上記プロセスを何百万回も繰り返す。モデル改善
