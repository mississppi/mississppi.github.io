---
title: "aws"
slug: "aws"
order: 1
source: "docker"
---

## aws

---

## DocDB と MongoDB の設計

参考ページ
https://www.mongodb.com/ja-jp/docs/v6.0/core/data-modeling-introduction/

---

## ローカルで Lambda をデプロイした

・sam deploy --guided でデプロイできる
・helloworld-HelloWorldFunction-pvRtm6gdlAnL という構成で生成されちゃう。
これを制御するには、template.yaml の FunctionName で指定する
・build, deploy で update できた
・apigateway もデプロイされてしまった。これは Event で apigateway を指定していたから
・Events を除去すれば単体アップ可能

## レイヤーを作ってアタッチしたい

・layer をローカルに普通に作る
・pip install する
・template.yaml で定義する。
・lambda にアタッチする
・普通に build, deploy すれば行けた！
・これで requirements.txt 削除できる

```
  CommonLibsLayer:
    Type: AWS::Serverless::LayerVersion
    Properties:
      LayerName: common-libs-layer
      ContentUri: my-layer/
      CompatibleRuntimes:
        - python3.12

    ラムダ
    Properties:
      CodeUri: hello_world/
      Handler: app.lambda_handler
      Runtime: python3.12
      Layers:
        - !Ref CommonLibsLayer #
```

## githubaction で samdeploy する

・デプロイがコケる。toml が悪いみたい。false にすれば進むみたい

```
[default.deploy.parameters]
capabilities = "CAPABILITY_IAM"
confirm_changeset = false
resolve_s3 = true
```

# codepipeline でデプロイしてみる

・普通に sam init する
・でも workflow 作らない
・github 側でブランチを aws 側に接続できるよう許可しておく
・buildspec.yml を作っておく

---

・よくわからんけどいったん cloudformation にしておく
・codepipe でパイプラインをぽちぽちで作った。
・S3 に置き場所を作る必要がある
・ソース、ビルド、デプロイを用意しないといけない。ビルドもできない
・packaged.yml にする必要がある
・samconfig.toml でで S3 のリゾルブする必要がある
・ロールが 2 つある。それぞれに。権限をつける
・pipeline 側で色々設定する。
・成功した。
・パイプライン自体を削除しても、リソースは削除されない

---

# api キーを定義してみる

・キーとリソースとか定義する

```
  # APIキーのリソースを定義
  ApiKey:
    Type: AWS::ApiGateway::ApiKey
    Properties:
      Name: RecipesApiKey # コンソールで表示されるキーの名前
      Enabled: true

  # APIの利用計画を定義
  ApiUsagePlan:
    Type: AWS::ApiGateway::UsagePlan
    Properties:
      UsagePlanName: UsagePlan # コンソールで表示される計画の名前
      ApiStages:
        - ApiId: !Ref ServerlessRestApi # SAMが自動で作成するAPIのIDを参照
          Stage: Prod # デプロイされるステージ名

  # 作成したAPIキーと利用計画を紐付け
  ApiUsagePlanKey:
    Type: AWS::ApiGateway::UsagePlanKey
    Properties:
      KeyId: !Ref ApiKey
      KeyType: API_KEY
      UsagePlanId: !Ref ApiUsagePlan
```

・リリースしたタイミングでキーが生成される
・git であやまってコミットしてしまった。隠しファイルとか

```
//stream状況確認
aws dynamodb describe-table --table-name movies --query 'Table.StreamSpecification'


```
