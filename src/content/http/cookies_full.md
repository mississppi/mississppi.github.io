---
title: "HTTP Cookieの基本"
slug: "cookies"
order: 3
source: "Real World HTTP"
---

## Cookieとは？

Cookieは、**Webブラウザがサーバーから受け取って保存し、次回以降のリクエストに自動で付与する仕組み**です。  
主にセッション管理、ユーザー識別、トラッキングなどに使われます。

---

## Cookieの構造

レスポンスの `Set-Cookie` ヘッダーでサーバーから送られます。

```
Set-Cookie: session_id=abc123; Path=/; HttpOnly; Secure; Max-Age=3600
```

| 属性名 | 説明 |
|--------|------|
| `session_id=abc123` | キーと値（名前と値） |
| `Path=/` | Cookieを送信するパスの指定 |
| `HttpOnly` | JavaScriptでアクセスできないようにする（XSS対策） |
| `Secure` | HTTPS接続時のみ送信される |
| `Max-Age=3600` | Cookieの有効期限（秒） |

---

## Cookieの属性とは？

「属性」とは、Cookieの**振る舞いや制限を制御する追加情報**です。  
`HttpOnly` や `Secure` など、Cookieをどう扱うべきかブラウザに伝えます。

属性を「つける or つけない」は、**サーバーが `Set-Cookie` ヘッダーでコントロール**します。

### 例1（属性あり）:

```http
Set-Cookie: session_id=abc123; HttpOnly; Secure; Max-Age=3600
```

この場合は以下のようなルールが働きます：

- JavaScriptからアクセスできない
- HTTPSでのみ送られる
- 1時間で失効

### 例2（属性なし）:

```http
Set-Cookie: session_id=abc123
```

最低限の制限のみで送られるため、セキュリティリスクが増します。

### ✅ まとめ

| 誰が設定する？ | 何を設定する？ | 目的 |
|----------------|----------------|------|
| サーバー | Cookieの属性 | 安全性・制限・寿命などを制御 |
| ブラウザ | 属性に従って送信 | 次回のリクエスト時に自動で送る |

---

## リクエスト時の動作

ブラウザは、保存されたCookieを**自動的に対象サーバーへ送信**します：

```
Cookie: session_id=abc123
```

---

## 実務での活用例

- ログインセッションの保持（セッションクッキー）
- ABテストやユーザー識別
- ショッピングカートの状態保持

---

## よくある注意点

- `HttpOnly` を設定しないと、XSSでCookieが盗まれる可能性あり
- `Secure` を設定しないと、HTTP通信時に盗まれる可能性あり
- `SameSite` 属性を指定しないと、クロスサイトリクエスト（CSRF）攻撃の対象になりやすい

---

## Cookieの属性一覧

| 属性 | 説明 |
|------|------|
| `Expires` / `Max-Age` | 有効期限 |
| `Domain` | Cookieを送信するドメイン |
| `Path` | Cookieを送信するパス |
| `Secure` | HTTPS限定 |
| `HttpOnly` | JSから読み取れない |
| `SameSite` | クロスサイト制限 (`Lax`, `Strict`, `None`) |

---

## Cookieとセッションの関係

多くのWebアプリでは、**セッションIDをCookieで保持**し、サーバーがそのIDに基づいてユーザー状態を管理します。

---

## 補足

- `Set-Cookie` は **レスポンスヘッダー**
- `Cookie` は **リクエストヘッダー**
- `localStorage` や `sessionStorage` とは用途・性質が異なる
