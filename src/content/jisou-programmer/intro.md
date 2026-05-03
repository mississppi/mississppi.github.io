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

## 4.1 プロジェクト構成

79. 本番環境はシンプルな仕組みで構築する

慣れているからいつもの構成は、よくない。

best:
機能をシンプルに。必要最小限で揃える。セキュリティ上の心配も増えるので。
個人環境と本番環境を統一することに固執しない。代表的な組み合わせ

#### OS提供のPython + venv + pip

#### 公式Dockerイメージ + pip

これらは単機能で仕組みの理解が簡単、セキュリティ更新が適用しやすい

80. OSが提供するPythonを使う
    ・セキュリティ更新情報が発信されている
    ・セキュリティ更新がある、apt, yumコマンドがわかるようになっている
    ・最新利用できない

best:
OSが提供するPythonを使う。Ubuntuはaptできるもの。RHELはyum、dnfができるもの。
セキュリティ更新パッチも利用できる。

81. OS標準以外のPythonを使う
    ・好きに選べる
    ・更新確認は独自で

Best:
デメリットがセキュリティ更新は自分ですること。

82. Docker公式提供を使う。
    ・比較的簡単い最新がつかえる

83. Pythonの仮想環境を使う
    ・仮想化した環境にインストールするため、OSのPythonを変更しなくていい
    ・Dockerの場合、仮想化が二重化され冗長

    Best:
    ・プログラム実行環境をPython本体から切り離す。
    ・仮想化しないと、古いファイル、依存古いライブラリが残る。
    ・Dockerなどのコンテナを利用してる場合、基本的にPythonの仮想環境不要。

84. リポジトリのルートディレクトリはシンプルに構成する

    Bad:
    ・直下に全て置いてる

    Best:
    ・リポジトリの主目的にあったもの。注目してほしいファイル、ディレクトリだけ。
    ・README.mdには目次程度を書いておく。詳細書きすぎず、配下のmdへ参照させる
    ・COMPOSE_FILE環境変数で設定ファイルを指定してもよし、Makefileにしてもよし。

85. 設定ファイルを環境別に分割する

    Best:
    ・設定ファイルと共通部分と環境依存部分に分割する

```
base.py

imoprt os
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
DEBUG = True
ALLOWED_HOSTSW = []
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    ...
    'mypp',
]
# MIDDLEWARE = [...]
DATABASES = [
    'default': {
        'ENGINE': 'jango.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
]
...

```

```
local.py

from .base import
```

```
staging.py

from .base import
DEBUG = False
ALLOWED_HOSTS = ['wwww.example.com']
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydatabse',
        'USER': 'mydatabaseuser',
        'PASSWORD': 'mypassword',
        'HOST': '127.0.0.1',
        'POST': '5432',
    }
}
```

これで

86. 状況依存の設定を環境変数に分離する

    Bad:
    ・多段継承して設定ファイルを管理する
    ・次第に管理できなくなる
    ・状況依存の設置値を設定ファイルで管理すると、一時的な設定変更のためにファイルを書き換え、その都度戻す必要がある。戻し忘れが発生したりする。

    Best:
    ・状況依存の設定値をコードから分離する。環境変数で設定する。
    ・django-environ, python-decoupleでenv使える
    ・デメリットは環境変数が増える。
    ・環境依存設定は、「設定ファイルを環境別に分割管理」
    ・状況依存の設定は「環境変数で管理」

```
これでDEBUGなければFalseとして動作する。

import os
DEBUG = bool(os.environ.get('DEBUG', False))
```

```
DEBUG=True
ALLOWED_HOSTS=127.0.0.1,localhost
INTERNAL_IPS=127.0.0.1
USE_SILD=True
DATABASE_URL=sqlite://db.sqlite3
```

```
DEBUG=False
ALLOWED_HOSTS=www.example.com
INTERNAL_IPS=
USE_SILD=
DATABASE_URL=postgres://mydatabaseuser:mypassword@127.0.0.1:5432/mydatabase
```

```

setting.py

imoprt os
import environ

BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(**file**)))
env = environ.Env()
env.reqd_env() # 現在のディレクトリか上位にある.envを読み込み、環境変数に設定する

# env.bool()は、'true','on','ok','y','yes','1'を真として扱う
DEBUG = env.bool('DEBUG')

# env.list()は環境変数から取得した文字列をカンマ区切りでリストに変換

ALLOWED_HOSTS = env.list('ALLOWED_HOSTS')
if DEBUG:
INTERNAL_IPS = env.list('INTERNAL_IPS')
INSTALLED_APPS = [
'django.contrib.admin',
...
'myapp'
]

# MIDDLEWARE = [...]

if env.bool('USE_SILK', default=False):
INSTALLED_APPS.append('slik')
MIDDLEWARE.append('slik.middleware.SilkyMiddleware')

DATABASES = {
'default': env.db_url('DATABASE_URL')
}
...

```

87. 設定ファイルもバージョン管理しよう

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

101ファイルを格納するディレクトリを分散させる

best:
分散にはいくつかある。

・元にあるデータのIDを利用してディレクトリを分ける

```
/receipts/123/receipt-20190718-123456.pdf
/receipts/124/receipt-20190718-123456.pdf
```

メリット: わかりやすい
デメリット: レコード数分だけディレクトリができる。また速度低下の原因になる

・ファイル名からハッシュを作成して、3文字のディレクトリで分ける

メリット: ディレクトリを抑えることができる
デメリット: 計算が発生。同一ファイル名がないことが前提

102. 一時的な作業ファイルは一時ファイル置き場に作成する

作成処理中の位置値ファイルを最終的な保存ようディレクトリに作成はしてはいけない。
一時ファイルと同等に扱ってはいけない。エラーになったら残骸になる。

best:
一時ファイル置き場をつくる。定期的な不要ファイルをクリーンアップも必要

103. 一時的な作業ファイルには絶対に競合しない名前を使う

具体的なbad:
秒レベルでの競合してしまった。ファイル名競合。

best:
Pythonのtempfileモジュールを使う。

```
import tempfile
with tempfile.NamedTemporaryFile(prefix='receipt-') as f:
    f.write(data)
    send_file(f.name)
```

104. セッションデータはRDB or KVSを使う

best:
・RDB: 手軽。同時書き込み負荷が高い
・KVS: 読み書き高速で自動削除機能もある

## 4.6 ネットワーク

105. 127.0.0.1と0.0.0.0の違い

127.0.0.1:8000を使うことをバインドという。127.0.0.1はローカルループバックという、
そのコンピュータ内でのみアクセス可能なネットワークインタフェースのこと。
このループバックにバインドした場合、同じコンピュータ内からは、
http://127.0.0.1:8000にアクセスしてWebアプリを表示できるが、
コンピュータ外から接続できない。

best:
サービスを提供したいIPアドレスにバインドすること。
ローカル開発環境以外で起動したWebサーバにアクセスする場合は、どのネットワークインターフェースにバインドするか指定が必要
コンピュータ外と直接通信するには、コンピュータ外との通信用ネットワークインタフェースのIPにバインドして起動する。
python manage.py runserver 192.168.99.1:8000
0.0.0.0にバインドすることで全てのネットワークインタフェースと接続できる

・外部からアクセス可能にするには
eth0などのコンピュータ外部と繋がっているネットワークインタフェースのIPを調べてバインドする。

・全てのネットワークにバインドしたい場合
0.0.0.0を指定する。ただ外部からもアクセス可能なので十分気を付ける

106. ssh port forwardingによるリモートサーバアクセス

best:
ssh port forwardingは、ssh接続を利用して、外部のネットワークから直接通信できないポートへの接続を可能にする技術。
対象サーバと直接http通信できない場合であっても、そのサーバにsshできれば、ssh port forwardingで、任意ポートと通信できる。

```
接続先はserver.example.comです。
接続元PCのポートを接続先のserver.example.comから見てlocalhost:80に接続するようにトンネルを作成してる。
ssh server.example.com -L 8000:localhost:80
```

```
ゲートウェイとあるサーバserver.example.comに接続できる場合は、
これで192の8000に接続できる。ただ、セキュリティ的にできない場合もある。
ssh server.example.com -L 8000:192.168.99.1.:8000
```

107. リバースプロキシ
     Gunicorn + Djangoを直接ネットに公開した場合、静的ファイルとかもあるので遅い

best:
WebサーバとしてApache, Nginxなどを設置し、Webアプリサーバにリバースプロキシで接続する。

転送元のIPなどは、X-FORWRDED-HOST, X-FORWARDED-PROTOなどを組み立てて使う。

108. Unixドメインソケットによるリバプロ

　　
