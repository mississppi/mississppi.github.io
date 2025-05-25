---
title: "git stashの使い方（-uの違いと戻し方）"
slug: "stash"
tags: ["git", "stash", "一時退避"]
order: 1
source: "gitで困ったら"
---

## `git stash` と `git stash -u` の違い

| コマンド | 保存対象 |
|----------|-------------------------------|
| `git stash` | **変更した tracked ファイル** のみ |
| `git stash -u` | **tracked + untracked ファイル**（未追跡ファイルも含む） |

### trackedとは？

- `git add` して Git に登録されたファイル。
- 変更は stash できる。

### untrackedとは？

- 作ったけど `git add` していないファイル（`git status` で赤い文字になる）。
- `git stash` だけでは退避されない。
- `git stash -u` で退避できる。

---

## stash したときの戻し方

### 一番新しい stash を適用して削除

```sh
git stash pop
```

### 指定の stash を適用（削除しない）

```sh
git stash apply stash@{0}
```

### 指定の stash を削除

```sh
git stash drop stash@{0}
```

### 一覧から選ぶとき

```sh
git stash list
```

---

## 補足

- `git stash -a` は `.gitignore` されたファイルも含む（通常は使わない）
- `pop` は失敗すると変更が失われる可能性があるため、慎重に
- `apply` + `drop` で段階的に戻すのもアリ
