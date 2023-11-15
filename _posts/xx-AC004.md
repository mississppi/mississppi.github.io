---
layout: post
title: AmazonConnectのソフトフォンをcloudfront経由でアクセスする
description: AmazonConnectのソフトフォンをcloudfront経由でアクセスする
postHero: img/phone.jpg
---

# はじめに

AmazonConnectのソフトフォンをカスタマイズしました。

# cloudfrontとS3を用意

```

```

# lambdaでコンタクトデータを使う方法
コンタクトフローからlambdaを呼び出すと、デフォルトでコンタクトデータを利用することができます。
eventにDefaultというキーで利用することができます。  
今回コールされた電話番号が必要なので、event['Details']['ContactData']['SystemEndpoint']['Address'] を利用します。

```
import json
import boto3
from datetime import datetime
from boto3.dynamodb.conditions import Key

def lambda_handler(event, context):
    incoming_phone_number = event['Details']['ContactData']['SystemEndpoint']['Address'] 
    open = check_open(incoming_phone_number)
    print(open)
    return {
        'open': open
    }

def check_open(incoming_phone_number):
    dynamodb = boto3.resource('dynamodb', region_name='ap-northeast-1', aws_access_key_id='', aws_secret_access_key='')
    table_name = 'holiday_schedule'
    table = dynamodb.Table(table_name)
    
    #全取得
    # response = table.scan()
    # items = response.get('Items', [])
    
    # for day in items:
    #     print(day['phone_number'])
    
    #現在日時取得
    current_date = datetime.now()
    search_date = current_date.strftime("%m/%d")
    
    #1件取得
    response = table.query(
        KeyConditionExpression=Key('phone_number').eq(incoming_phone_number)    
    )
    items = response.get('Items', [])
    holidays = [item.get('holiday') for item in items]
    
    open_flag = "enable"
    for item in items:
        holiday = item.get('holiday')
        if holiday == search_date:
            open_flag = "disable"
            break
    
    return open_flag
```

# ブロックで値を判定する方法
コンタクト属性を確認するブロックを利用することで、値を判定することができます。
Lambdaでは外部としてのキーを指定します。

<img src="/post_img/6.png">