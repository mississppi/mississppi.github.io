---
title: "誤解されない名前をつける"
slug: "unambiguous-names"
tags: ["命名", "誤解されない", "第3章"]
source: "リーダブルコード 第3章"
order: 3
---

## 範囲を表す変数には明確な接頭語を
数値や要素の範囲を扱う場合、以下のような接頭語ルールに従うと、
読み手が意図を正確に理解しやすくなります：

min_, max_: 上限・下限（含むかどうか問わず）

first, last: 包含的な範囲（どちらも含まれる）

begin, end: 排他的な範囲（end は含まない）

```
const minAge = 18;
const maxAge = 65;

const firstIndex = 0;
const lastIndex = array.length - 1;

const begin = 0;
const end = 5; // 対象は 0〜4、5 は含まれない
```

特に begin / end というペアは、end が排他的であることを示唆できるため、
配列スライスや文字列操作などで役立ちます。


## ブール値は質問のように書く

ブール値の変数や関数には、is, has, can などの質問形の接頭語をつけることで、読みやすく、直感的になります。

```
const isAdmin = user.role === "admin";
const hasPermission = checkPermission(user);
const canAccess = isLoggedIn && hasPermission;

また、notX のような否定形の名前はなるべく避けましょう。
否定の否定は、読みづらく、バグの温床にもなります。

```
// ❌ 読みにくい
if (!notFound) { ... }

// ✅ 肯定形で
if (found) { ... }
