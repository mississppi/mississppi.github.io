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

## 1.1 クラス設計

## 1.1 モジュール設計

## 1.1 ユニットテスト

## 1.1 実装の進め方

## 1.1 レビュー

##
