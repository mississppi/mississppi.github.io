---
title: "自走プログラマ - はじめに"
order: 1
---

# 自走プログラマ

「自走プログラマ」の読書メモをここに記載します。

## 概要

---

## 1.1 関数設計

1. 関数名は処理内容を想像できる名前にする

bad

```
def item_csv(item):
    with open()
```

best
・動詞にする例: get_item_list, calc_tax, is_member

・取得できるものの名刺にする: current_date, as_dict
ある情報を取得するが明確な場合のみ

・役割で関数名にする: json_parse
入出力が想像できるのが重要

2. 狭い単語使おう

```
def fetch_latest_news():

def calc_tax_including(price):

def aggregate_sum_price(items):
```

3. 関数名から想像できる型の戻り値を返す

isならtrue/falseを返そう

```
def is_valid(name):
```

4. 副作用のない関数にまとめる

```
def is_valid(name):
    ここでは何も書き換えるのはよしましょう
```

5. 意味づけできるまとまりで関数にする

```
商品一覧を読み込む
def read_items():
    with open(...) as f:
        reader = csv.reader(f)
        for row in reader:
            name = row[0]

買い合せ対象商品の場合True
def is_addon_price(price):
    return price < 100

def main():
    items = read_items()
    for name, price in items:
        if is_addon_price(price)
            print(name)

```

6. リスト、辞書をデフォルト引数にしない

bad

```
def foo(valus=[]):

```

デフォルト引数にすると呼び出す度に値が変わる。
とにかくリスト、辞書をデフォルト引数にしてはだめ！
noneにして関数内で空を詰める

best

```
def foo(values=None):
    values = values or []
    values.append("hi")
    return values
```

7. コレクションを引数にしない。int, strを受け取る

bad

```
def calc_tax_included(item, tax_rate=0.1):
```

毎度itemにキーを持たせないと

best

```
def calc_tax_included(price, tax_rate=0.1):

```

8. インデックス番号に意味を持たせない

クラスを使おう

```
@dataclass
class Sale:
    sale_id: int

def validate_sales(sale):
    if not item_exists(sale.item_id):
        raise...
```

明治でインデックス番号がいるときはenumerateを

```
for n, item in enumerate(items, start=1):
    print()
```

9. 関数の引数に可変超引数を乱用しない
   \*args, \*\*kargsは乱用しない
   どんな値が来てもいい関数のみにする

10. コメントになぜを書く

・なぜそう書くのかをコメントする
・なぜこう処理しないのか、の説明と考えてもよし。過去経緯や外部ライブラリの都合で合理的でない処理の時に「なぜこのような処理をしないのか」を書いておく

11. コントローラには処理を書かない

```
def do something(users):
    """ hogehogeする処理
    複数ユーザんに対してこの関数を行う。
    --の場合に--なので、ユーザのデータを変更する必要がある。
    """

    # SQL実行回数を減らすため、このループは別関数に分離せず処理する
```

## 1.1 クラス設計

## 1.1 モジュール設計

## 1.1 ユニットテスト

## 1.1 実装の進め方

## 1.1 レビュー

##
