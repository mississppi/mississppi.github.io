---
layout: post
title: Wordpressの記事作成後に外部連携処理を作成
description: Wordpressの記事作成後、外部システムへ連携するために必要なフック、処理をレポートします。記事ではFacebookへの連携。redisへの連携。jsonファイルへ出力を検証しました。
postHero: img/thumb4.jpg
---

# はじめに

今回は記事作成後になんらかのシステムへ記事を連携させるための処理を、
プラグインとして作ってみたので内容をまとめます。

# Facebookへの連携

今回はFacebookへの連携してみます。
(準備としてはFacebookの開発者登録はすでに済ませて、認証情報は取得済みの状態とします)

まずはcomposerでライブラリを導入します。

```
//composer.json
{
    "require": {
        "facebook/graph-sdk": "^5.1"
    }
}

```

フックはwp_insert_postを利用します。
wp_insert_postは、wp-includes/post.phpのwp_insert_post関数に定義されています。
タイミングとしてはreturnされる直前となっています。

連携処理については以下の流れです。

・ Facebookの認証情報を変数として使う  
・ /me/feedへリクエストします。

```
    public function __construct()
    {
        add_action('wp_insert_post', [$this,'post_to_facebook'], 10, 3);
    }

    /**
     * Facebookに投稿するメソッド
     *
     * WordPressの投稿が挿入または更新されると呼び出される。指定条件に基づき
     * Facebookに投稿を行います。
     *
     * @param int $post_id 投稿ID
     * @param WP_Post $post 投稿オブジェクト
     * @param bool $update 投稿が更新された場合はtrue、それ以外はfalse
     * @return void
     */
    public function post_to_facebook($post_id, $post, $update )
    {
            $fb = new \Facebook\Facebook([
                'app_id' => '',
                'app_secret' => '',
            ]);

            // Facebookに投稿するためのデータ
            $postData = [
                'message' => 'Hello', // 投稿メッセージ
            ];

            $accessToken = '';

            // Facebook Graph APIリクエスト
            try {
                // フィードに投稿
                $response = $fb->post('/me/feed', $postData, $accessToken);

                // レスポンスを取得
                $graphNode = $response->getGraphNode();
                
                // 投稿IDを取得
                $postId = $graphNode['id'];

                echo '投稿が成功しました。投稿ID: ' . $postId;
            } catch (Facebook\Exceptions\FacebookResponseException $e) {
                echo 'Facebook API エラー: ' . $e->getMessage();
            } catch (Facebook\Exceptions\FacebookSDKException $e) {
                echo 'Facebook SDK エラー: ' . $e->getMessage();
            }
    }
```

# redisへの連携

つぎにredisへの連携も試してみました。環境はdockerで用意しました。
```
  redis:
    image: "redis:latest"
    ports:
      - "6379:6379"
    volumes:
      - "./data/redis:/data"
```

利用するフックはwp_insert_postです。
ヘルパーとしてCRUD機能を作りました。


```
<?php
/* 
 * redis操作クラス
 */
class RedisHelper{
    private $redis;

    /**
     * RedisHelper コンストラクタ
     * 
     * @param string $host redisのホスト
     * @param int $port redisのポート
     **/
    public function __construct($host = 'redis', $post = 6379)
    {
        $this->redis = new Redis();
        $this->redis->connect($host, $post);
    }

    /**
     * データを格納します
     *
     * @param srting $key キー
     * @param mixed $value データ
     * @return bool 成功時にtrue、失敗でfalse
     **/
    public function set($key, $value){
        $value = json_encode($value);
        return $this->redis->set($key, $value);
    }

    /**
     * redisからデータを取得します
     *
     * @param int $key
     * @return string 
     */
    public function get($key)
    {
        $value = $this->redis->get($key);
        return $value ? json_decode($value, true) : '';
    }

    /**
     * データを更新します
     *
     * @param string $key
     * @param string $value
     * @return void
     */
    public function update($key, $value)
    {
        if($this->redis->exists($key)){
            $this->redis->set($key, $value);
        }
    }

    /**
     * データを削除します
     *
     * @param string $key
     * @return void
     */
    public function delete($key)
    {
        $this->redis->del($key);
    }
}
```

このクラスをつかってredisへデータ投入することができます。

```
    public function create_redis_data($id, $post)
    {
        $redis = new RedisHelper();
        $redis->set('post_'.$id, $post->post_content);
    }
```

# jsonファイルへ連携
外部ではありませんが、jsonファイルへ出力する方法も検証してみました。
フックは、transition_post_statusを利用します。
transition_post_statusはwp_insert_postよりも早いタイミングで実行されます。

## 内部でwpapiへのリクエストを実施
利用する関数はwp_remote_getでリクエストし、wp_remote_retrieve_bodyを使ってデータを取り出します。
対象はslugを使った1件で、APIでgetした結果をjsonにして出力します。
また今回使ったフックはステータス変更を検知できるので、auto-draftを除外すると良いでしょう。

```
    /**
     * JSONデータを生成・保存するメソッド
     *
     * WordPressの投稿が公開されたときに呼び出され、指定された投稿の情報をAPIから取得し、
     * JSON形式でファイルに保存します。
     *
     * @param string $new_status 投稿の新しいステータス
     * @param string $old_status 投稿の以前のステータス
     * @param WP_Post $post 投稿オブジェクト
     * @return void
     */
    public function outputJson($new_status, $old_status, $post)
    {
        //自動下書きはreturn
        if($post->post_status === 'auto-draft'){
            return;
        }

        //リクエストを作成します
        $slug = $post->post_name;
        $url = "http://localhost:80/wp-json/wp/v2/posts?slug=$slug";

        try {
            $response = wp_remote_get($url);
            if(is_wp_error($response)){
                // $msg = $response->get_error_message();
            } else {
                $api_data = wp_remote_retrieve_body($response); // APIからのデータ
                $decoded_data = json_decode($api_data);
                if ($decoded_data) {
                    // JSONデータを生成して保存
                    $data = json_encode($decoded_data, JSON_PRETTY_PRINT);
                    $json_file_path = dirname(__FILE__) . '/output.json';
                    file_put_contents($json_file_path, $data);
                } else {
                    // データが正しくデコードできない場合のエラーハンドリング
                }
            }

        } catch (Exception $e) {
            // 例外のエラーハンドリング
        }
        return;
    }
```

これで同ディレクトリにjsonファイルを出力することができました。