---
title: "無関係な下位問題を分離する"
slug: "separate-subproblems"
tags: ["分離", "本質", "第10章"]
source: "リーダブルコード 第10章"
order: 10
---

複雑な処理を1つの場所に詰め込むと、コードの意図が見えづらくなります。  
この章では「**本質的な処理に集中するために、無関係な下位の問題を分離する**」という考え方を紹介しています。

---

## 本質と下位問題を分ける

処理の中には、**“本質”ではないけれど必要な作業（下位問題）**が多く含まれます。  
例：
- ファイルの読み込み
- フォーマットの整形
- 文字列のパース
- ログ出力 など

それらをすべて一箇所に書いてしまうと、**本当に大事な処理が埋もれてしまいます。**

---

## 例：読みづらいコード

```js
const fs = require("fs");
const data = fs.readFileSync("nums.txt", "utf8");
const numbers = data.split(",").map(Number);
const total = numbers.reduce((a, b) => a + b, 0);
const average = total / numbers.length;
```

**読み手は、読み込み・分解・計算・平均…と目を右往左往させられます。**

---

## 本質を引き立てる書き方

### ➤ 下位問題を関数に切り出す：

```js
function readNumbersFromFile(filename) {
  const data = fs.readFileSync(filename, "utf8");
  return data.split(",").map(Number);
}

function calculateAverage(numbers) {
  const total = numbers.reduce((a, b) => a + b, 0);
  return total / numbers.length;
}

const numbers = readNumbersFromFile("nums.txt");
const average = calculateAverage(numbers);
```

### ✅ メリット
- **何をしているかが一目でわかる**
- **個々の関数はテストしやすい**
- **読み手の脳への負荷が減る**

---

## 上から下へ「本質→詳細」の構成にする

まず「やりたいこと」を書き、それから下に「どうやってやるか（下位問題）」を定義する構成にすると、  
**読み手は安心して読み進めることができます。**

```js
// 上から読むだけで概要がつかめる
main() {
  const numbers = readNumbersFromFile("nums.txt");
  const average = calculateAverage(numbers);
  console.log(average);
}

// 下で詳細を定義
function readNumbersFromFile(...) { ... }
function calculateAverage(...) { ... }
```

---

## まとめ

- 本質的でない処理（ファイルI/O、整形など）は「下位問題」として切り出そう
- 下位問題を分けることで、本質の意図が明確になる
- コードを「本質→詳細」の順で書くことで、読みやすさと理解のしやすさが大きく向上する
