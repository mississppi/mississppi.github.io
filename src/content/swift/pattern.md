---
title: "MVCでパターン解説"
slug: "patter"
order: 4
source: "swift"
---

## Observerパターン: 

ViewがControllerを監視している構造

### コントローラ:
@Publishedをつけて監視したい情報を設定
```
class TaskController: ObservableObject {
    @Published var tasks: [Task] = []
}
```

### view側で
コントローラを呼ぶ際に@StateObjectで呼ぶ
```
@StateObject private var controller = TaskController()
```

---

## Strategyパターン

とにかく条件でロジックなどを切り替えたい場合に使う

用意するもの

1. インタフェースProtocol
```
protocol TaskViewStorategy {
    func render(task: Task) -> AnyView
}
```

2. viewを複数
```
struct CompactViewStorategy: TaskViewStorategy {
    func render(task: Task) -> AnyView {
        AnyView (
            Text(task.title)
                .font(.body)
        )
    }
}
```

3. 切替ロジック
```
func toggleViewStrategy() {
    print("toggled")
    print(viewStrategy)
    if viewStrategy is CompactViewStorategy {
        viewStrategy = DetailedViewStorategy()
    } else {
        viewStrategy = CompactViewStorategy()
    }
}
```

--- 

## Factoryパターンとバリデーション設計の導入記録

### このパターンは
生成の責務を分けたい時に使う
.createとかでカプセルか

### 今回の構成
factory 
```
struct TaskFactory {
    static let rules: [TitleValidationRule] = [
        NotEmptyRule()
    ]
    static func create(title: String, content: String) -> Result<Task, ValidationError> {
        let errors = rules
            .filter { !$0.validate(title) }
            .map { $0.errorMessage }
        
        guard errors.isEmpty else {
            return .failure(ValidationError(messages: errors))
        }
        
        return .success(Task(title: title, content: content))
    }
}

```

### バリデーションの ルール

```
struct NotEmptyRule: TitleValidationRule {
    let errorMessage = "タイトルを入力してください。"

    func validate(_ title: String) -> Bool {
        !title.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty
    }
}

```

### 一緒にバリデーションの設計もした
```
protocol TitleValidationRule {
    func validate(_ title: String) -> Bool
    var errorMessage: String { get }
}
```

### 4. Result型とエラーハンドリング
とにかくResult型を使うならお作法
```
struct ValidationError: Error {
    let messages: [String]
}
```

### コントローラ側で分けて使える

```
switch TaskFactory.create(title: title, content: content) {
case .success(let task):
    tasks.append(task)
    errorMessages = []
case .failure(let error):
    errorMessages = error.messages
}
```


## commandパターン使ってみた
用意するものは4つ

プロトコル
```
protocol RequestCommand {
    func execute()
    func cancel()
}

```

それを継承した本体。実体？
```
final class CurlCommand: RequestCommand {
    private var task: Task<Void, Never>?
    
    func execute() {
        guard task == nil else {return}
        task = Task {
            while !Task.isCancelled {
                await CurlRunner.shared.sendRequest()
                try? await Task.sleep(nanoseconds: 10 * 1_000_000_000)
            }
        }
    }
    
    func cancel() {
        task?.cancel()
        task = nil
    }
}

```

Task は非同期処理のためのスレッド的なもの。開始・終了を明示的に管理できる
```
Task {} ・・・即実行
task.cencel() ・・・停止
```

やりたい処理、今回はcurlでリクエスト
```
import Foundation

final class CurlRunner {
    static let shared = CurlRunner()
    
    private init() {}
    
    func sendRequest() async {
        print("Sending request...")
        let process = Process()
        process.executableURL = URL(fileURLWithPath: "/usr/bin/curl")
        process.arguments = ["https://mississppi.github.io/"]
        
        do {
            try process.run()
            process.waitUntilExit()
        } catch {
            print("failed to run curl")
        }
    }
}


```