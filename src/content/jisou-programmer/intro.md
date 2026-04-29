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

コントローラでは値の入出力と、処理全体の制御のみを行うべき。

webプリは以下のような責務を持っている

・Form：入力のバリデーションチェック
・HTMLの画面に入力フォームを表示
・フォームから送信されたデータを検証する

・Template: 値の描画
・テンプレート値から、HTMLを描画してブラウザ上の画面を表示する
・数字のコンマ区切りや、１００文字以上は。。。で略す。のような表示上の処理をする

・Model：データの保存
・DBに情報を永続化する
・永続化されたデータを条件をして取得する
・税込価格の取得、公開済み商品の取得というよくある処理をモデルのプロパティーやQuerySetに実装する

## 1.2 クラス設計

12. 辞書でなくクラスを定義する

bad

```
def get_fullname(user):
    return user['last_name'] + user['first_name]

とにかくdefでやろうとする
```

```
@dataclass
class User:
    last_name: str
    first_name: str
    birthday: date
```

「特定のキーをもつ辞書」が悪い理由
・特定キーが存在しているかチェックが必要になることがある
・他の形式の辞書で使えない関数なので再利用性が低い
・再利用性が低い割に色々受けいれられるので

クラスのメリット
・REPLでクラス名が表示される
・isinstanceでチェックできる
・型アノテをすると、指定したクラスが引数に渡されるかをチェックできる
・IDEで動的解析をするときに、クラスの定義者にジャンプしたり、関数の入出力をクラスのインスタンスで制限できる
・クラス名でコード検索すれば、そのクラスが利用される処理をすぐ見つけられる

13. dataclassを使う

```
def __init__()
    これがめんどい
```

14. 別メソッドに値を渡すためだけに属性を設定しない

bad

```
class User:
    def__init__():
        self.username = username
        self.age = None

    def calc_age(self):
        計算処理
        self.age = age

    def age_display(self):
        return f"{self.age}"
```

先にage_displayを呼び出すとNoneが出てしまう。
別のメソッドに値を渡すためだけに属性設定はやめる

```
class User:
    def __init__(self, username, birthday):
        self.=　

    @property
    def age(self):
        ageの計算
        return age

    def age_display(self ):
        return f"{self.age}"
```

@propertyとは
・メソッドを属性のように使える
・user.ageで呼び出せる
・計算を隠蔽できる
・必要な時だけ計算
・バリデとか後から追加

15. インスタンスを作る関数をクラスメソッドにする

bad:
このクラスを使う別のモジュールから、Productクラスとretrieve_protuct関数を
インポートする必要がある。

```
@dataclass
class Product:
    id: int
    name: str

def retrieve_product(id):
    res = requests.get(f'/api/products/{id}')
    data = res.json()
    return Product(
        id=data['id']
        name=data['name']
    )
```

best:
・外部APIからrequestする処理は分離するといい。
・常にAPIから取得すべきデータである場合は、クラスメソッドにすると使いやすい。

```
@dataclass
class Product:
    id: int
    name: str

    @classmethod
    def retrieve(cls, id: int) -> 'Product':
        data = retrieve_product_detail(id)
        return cls(
            id=data['id'],
            name=data['name'],
        )


```

## 1.3 モジュール設計

16. utils.pyのような汎用は避ける

best:
・データをフィルタするならmodels.pyなど
・リクエストはrequest.py
・クラス作る場合、意味が求められるので、utils.pyとかふむき

17. ビジネスロジックをモジュールに分割する

bad

```
View関数をまとめるviews.py(コントローラ)にView関数でない関数も記載する

def render_purchase_mail():

def purchase():

```

best

```
payment.pyとする

def render_purchase_mail():

def purchase():
```

18. モジュール名のおすすめ

best

```
- api (外部APIにアクセスする処理)
-- __init__.py
-- item.py(商品のAPI)
-- user.py(ユーザのAPI)

- commands(コマンドラインツールのサブコマンド)
-- list.py(商品一覧のコマンドのin/out)
-- purchase.py(購入コマンドのin/out)

- consts.py(定数)
- main.py
- models.py(商品、ユーザのデータを永続化するクラスをまとめる)
- purchase.py(商品購入する処理)
- validators.py(command LINEからのinputをチェックする処理)
```

認証: authentication.py
認可: permission.py / authorization.py
バリデーション: validations.py / validators.py
例外: exceptions.py

## 1.1 ユニットテスト

## 1.1 実装の進め方

## 1.1 レビュー

##

## 4.2 サーバ構成

88. 共有ストレージを用意しよう

サーバが複数台あったときアップロードしたファイルなどを共有データをとこに置くか。

best:
・アップしたファイルを集約して管理するような共有ストレージとなるサーバを用意
・現実的にはクラウドの巣レージを利用した運用・コストを抑えられる。

89. CDNを使おう

静的ファイルはキャッシュ化してCDNで配信しよう。

https://blog.redbox.ne.jp/what-is-cdn.html

90. KVSを利用しよう

bad
・一覧系データをメモリに乗せてしまった。

best
・KVSを利用して、Redisを使った

注意:
RDB側チューニングで高速化できればKVSは後回しでいい。

91. 時間のかかる処理は非同期
    リアルタイムで処理が必要な部分で時間がかかる場合、非同期化したほうがいい。

・ThreadPoolExecutor : 別スレッド
・ayncio
・Celery: ジョブキューシステム(ジョブ単位でキューに積んでおく。)

92. タスク非同期処理
    Gunicornのワーカプロセスなど自動的に再起動されるプロセス上で、スレッドやこプロセスはNG。

Gunicorは一定時間でワーカが再起動される設定がある。意図しないタイミングでされる。

スレッドは管理する実装が必要。

best:
・自作はしないようにする。
・Celery
・Django
・APCheduler

## 4.3 プロセス設計

93. サービスマネージャでプロセス管理する

manage.py runserverコマンドもアプリサーバ機能を提供するが簡易的機能。
並行処理を持ってない。js,cssとかを並行で処理できない。常駐には向いてない

best:
サービスマネージャとwebアプリケーションサーバを使用する。
・サービスマネージャ:
常駐デーモンの管理、異常終了の再起動とか。LinuxのSystemedが標準的に利用される。

・Webアプリサーバは Gunicorn ,uWSGIなどが一般利用される。
複数プロセス起動の並列処理機能、死活監視などを提供。動作が早い

たとえばGunicornは、
・worker x 4
・リクエスト1000回ごとにメモリ、リソース解放
・応答しないプロセスは再起動
・ログもだせる
・Gunicorn > systemedを起動

94. デーモンは自動起動させておこう

bad:
webアプリをsystemedでデーモン化したので、プロセスkillしても再起動されるが、
セキュリティパッチの適用で再起動したものの、起動しない

best:
systemctlで自動機能の設定をしておく

95.Celeryのタスクにはプリミティブなデータを渡す

bad:
モデルデータをオブジェクトそのままにCelrarlyを渡す。負荷をかけることがある。

best:
int,strで渡す。

## 4.4 ライブラリ

96. 要件から適切なライブラリを選ぶ

best:

・要件を満たすライブラリを探す
要件をしっかり正しく把握する。メンテに苦労したりする。
リスト化して本当に必要か。吟味する。

・先行事例を確認する

・枯れているライブラリを利用する
Awesome Python, DjangoPckagesなどサイト見てみる

・ライセンスを確認する
商用は有償みたいなことがあるのでしっかり確認

・オフィシャルかどうか調べておく

・注意すること
テストがない。
CIが設定されてない。
サンプル、ドキュメントがない
リリースノートがない
半年くらい開発止まっている
パッケージの作り方がガイドラインに従ってない
issueを放置している。
ライセンス明記してない
ただフォークされている

98. 更新
    セキュリティパッチはなるはやで適用しよう
    開発用ツールは極端に言えばあげない。

## 4.5 リソース設計

100. ファイルパスはプログラムからの相対パスで組み立てる

bad:

```
import csv
from pathlib import Path

CSV_PATH = Path('target.csv')

with CSV_PATH.open(mode='r') as fp:
    reader = csv.reader(fp)
    for row in reader:
        print(row)
```

これでprojectディレクトリに移動して実行するとファイルが見つからない。

best:
どこからプログラムを起動されても適切に動くようにパスを組み立てる

```
import csv
from pathlib import Path

# 起点となるプログラムがあるパス
here = Path(__file__).parent
CSV_PATH = here / 'target.csv'

with CSV_PATH.open(mode='r') as fp:
    reader = csv.reader(fp)
    for row in reader:
        print(row)

```

**file**を利用することでそのファイルへのパスを取得できる。
そこから現在のディレクトリのパスを取得して、想定的な外部ファイルまでのパスを組み立てれば、どこから実行されてもファイルが見つかる。

```
djangoならこれを使う
CSV_PATH = Path(settings.BASE_DIR, 'data', 'target.csv')
```
