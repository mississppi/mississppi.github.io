---
title: "テンプレ"
slug: "template"
order: 3
source: "swift"
---

# よく使う
100本ノックで使ってるもの


```
Formatter+Extensions.swift

import Foundation

extension DateFormatter {
    static let trackDate: DateFormatter = {
        let f = DateFormatter()
        f.dateStyle = .medium
        f.timeStyle = .none
        return f
    }()
}

```


```
PersistenceController

import CoreData

struct PersistenceController {
    static let shared = PersistenceController()

    let container: NSPersistentContainer

    init(inMemory: Bool = false) {
        container = NSPersistentContainer(name: "MusicLog") // ← .xcdatamodeld のファイル名と一致させること

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


```
ルートでviewmodelをinitして渡す。
とにかく@StateObjectで作れば大丈夫！！

import SwiftUI

@main
struct ShoppingListApp: App {
    let persistenceController = PersistenceController.shared
    
    @StateObject private var shoppingListViewModel: ShoppingListViewModel
    
    init() {
        let context = persistenceController.container.viewContext
        _shoppingListViewModel = StateObject(wrappedValue: ShoppingListViewModel(context: context))
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(\.managedObjectContext, persistenceController.container.viewContext)
                .environmentObject(shoppingListViewModel)
        }
    }
}
```