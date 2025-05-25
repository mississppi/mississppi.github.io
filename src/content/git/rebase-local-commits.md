---
title: "ローカルでコミットメッセージをまとめる方法"
slug: "rebase-local-commits"
tags: ["git", "rebase", "コミット整理"]
order: 2
source: "gitで困ったら"
---

## 状況

コミットを細かく行っていたが、プルリク提出前に**1つのまとまった意味のあるコミット**にしたい。

---

## やり方：`git rebase -i`

以下の手順でローカルコミットをまとめられる。

---

### ① 過去のコミットを編集モードで開く

```sh
git rebase -i HEAD~3
```

※ ここでは直前の3コミットを対象にする。

---

### ② 編集画面で `pick` を `squash` に変更

```sh
pick ghi789 implement header
squash def456 add footer
squash abc123 fix typo
```

---

### ③ メッセージを整理

```txt
Implement layout

# 以下の元コミットメッセージは削除してOK
# add footer
# fix typo
```

---

### ④ `git log` で確認

```sh
git log --oneline

# 出力例:
xyz999 Implement layout
```

---

## 💡 注意点

- push 前にやるのが基本
- すでに push 済みの場合は `git push --force-with-lease` が必要
- 意図の違うコミット（バグ修正とUI修正など）は**無理にまとめない**

---

## 補足：GIFで確認したい場合

![git rebase steps](/images/git-rebase-steps.gif)

---

## まとめ

- `git rebase -i` はコミット履歴を整える便利なコマンド
- プルリク前に履歴を読みやすくすると、レビューがスムーズ
- squash, fixup, reword などのキーワードも活用しよう
