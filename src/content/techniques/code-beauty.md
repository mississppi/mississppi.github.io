---
title: "コードに美しさを持たせる"
slug: "code-beauty"
tags: ["美しさ", "整形", "第4章"]
source: "リーダブルコード 第4章"
order: 4
---

プログラムにおいて、美しさは軽視されがちですが、  
**読みやすいコードは、しばしば見た目が整っている**という特徴を持っています。

---

## 美しいコードは「読む気」にさせる

同じ機能のコードでも、**整ったコードと乱雑なコード**では、  
読み手の印象も、理解速度もまったく違います。

```js
// 乱雑なコード
let a=1; let b=2;
if(a<b){b=a;}

// 美しいコード
let a = 1;
let b = 2;

if (a < b) {
  b = a;
}
```

フォーマットが揃っていることで、**コードの構造が目に入りやすくなり、余計なストレスが減ります。**

---

## 類似したコードは似た見た目に

**似た処理が並ぶときは、同じフォーマットで揃える**ことで、パターンを読み取りやすくなります。

```js
// 一貫性のある揃え方
drawText("Name", x, y);
drawText("Age",  x, y + 1);
drawText("Job",  x, y + 2);
```

ズレやごちゃごちゃした記述は、**無意識に脳の負荷になる**ので避けましょう。

---

## 余白は情報を整理するツール

余白や空行は単なる「スペース」ではなく、**情報の区切り**を示す重要な記号です。  
グループごとに**意味のかたまりを明確に**すると、読み手の理解がスムーズになります。

```js
// 意味のまとまりを区切る
const isLoggedIn = checkLogin();

if (!isLoggedIn) {
  redirectToLogin();
  return;
}

// ユーザーがログインしていれば、プロフィールを表示
renderUserProfile();
```

---

## 書いた後、**少しだけ整える**

美しさとは、機能とは無関係に見えるけれど、  
**可読性と保守性を間接的に支える力**を持っています。

- スペースや空行を整える
- 類似コードの並びを揃える
- コメントの位置を整える

これだけで、**コード全体の信頼感が上がる**こともあります。

---

## まとめ

- 美しいコードは、読む前から「読みやすい」と感じさせる
- 類似した処理は、同じフォーマット・並びで統一する
- 空白や余白は、意味のグループ化に使う
- 書いた後、1〜2分で**見た目を整える癖**をつけよう
