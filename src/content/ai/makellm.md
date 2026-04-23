---
title: "つくりながら学ぶ！LLM 自作入門 (Compass Booksシリーズ)"
slug: "diyllm"
order: 4
source: "ai"
---

# Chapter1

## LLM とは

人間のようなテキストを理解、生成、反応するように設計されたニューラルネットワークのこと。
モデルとデータセットを指す。
Transformer というアーキテクチャを使っている。

大枠は機械学習のサブセットとなる。要は内包している。
この中にディープラーニングがある。3 層構造でニューラルネットワークを使いパターン抽象概念をモデル化する。
これはディープニューラルネットワークと呼ばれている。
大規模言語のモデルと、ディープラーニングを使うことで生成 AI というジャンルになっている。

LLM の最初は事前学習というステップがある。ベースモデルや基盤モデルをつかる。GPT3 とか。
文中の次の単語の予測をラベルとする、ラベル付けされてないテキストからなる大規模コーパスで事前学習をする
例)
input: 吾輩は猫である。名前は
-> 予測は、まだ　ない　？
-> 確率で最も高いものを選ぶ

## 事前学習とは

穴埋め問題を大量やってもらう。

例) 今日は良い\_\_\_です。
-> モデルは天気とかを予測
-> 正解と比較して学習

次に指示に従ったり、分類タスクを実行することで、ラベル付けされたより小さなデータセットでファインチューニングする

これはテキスト補完、書きかけ文章を完了させることができる。フューショットも対応可能
ここからさらに訓練ができる、これをファインチューニングと呼ぶ。これはインストラクションと分類の 2 つがある。

## インストラクション

input: 以下の文章をようやくしてください
output: ようやく結果
-> 指示にした能力を獲得(ChatGPT)

## 分類

input: スパムメールに該当するか
output: yes/no
-> こういうタスクに特化

## Trasnformer

ほぼ全ての LLM が Transformer を使っている。

流れは以下。

入力テキスト > 前処理(エンコーダに準備) > エンコーダ 入力を受けとりベクトル抽出する。 >
埋め込み > 部分出力テキスト > 入力テキスト > 前処理ステップ > デコーダ > 出力層

後継 BERT, GPT では訓練が違うらしい。後ほどちゃんと書く。
Bert はエンコーダ部分のみ。タスクに強い
GPT は手 s 吃おに強い。デコーダのみ。予測が次の入力となり出力となる。

---

# Chapter2

## やること

エンコーダーの前の処理を実装する。
テキストをトークン、サブトークンに分割する。LLM 用にベクトル表現にする
短編小説をトークン化する。

## 単語埋め込み

LLM などのディープニューラルネットワークは、生テキストをそのまま処理できない。
埋め込みと言われているが、ベクトル表現にすることで DNN が認識できるようになる。

### データの準備

・訓練するためのテキストを準備
・ここのワード、サブトークンに分割すると、ベクトル表現にエンコードできる
・この本では RAG ではなく、1 語ずつ生成することを学習すること

・ML では word2vec のような事前学習済みのモデルを利用する。埋め込みに
・GPT12228 です。次元が。

#### テキストをトークン化する

細かいステップで見てみる。
・入力テキスト
・トークン化されたテキスト
・トークン ID
・トークンうめこみ

まず文章を特定のルールで分解する

```
preprocessed = re.split(r'([,.:;?_!"()\']|--|\s)', raw_text)
preprocessed = [item.strip() for item in preprocessed if item.strip()]
```

これを整数表現に変換する、トークン ID を作成。
トークン ID にするということは、語彙を構築することである。
語彙は訓練データセットから構成されている。
これを使って、トークン ID にマッピング

```
all_words = sorted(set(preprocessed))
vocab_size = len(all_words)
print(vocab_size)
vocab = {token: integer for integer, token in enumerate(all_words)}
```

#### 未知の単語について

語彙に含まれてないとエラーになるのでなんとかする。
訓練されてないワードは<unk>とかに突っ込む

#### Chapter2 のサンプルコード

```
class SimpleTokenizer2:
    def __init__(self, vocab):
        self.str_to_int = vocab
        self.int_to_str = {i: s for s, i in vocab.items()}

    def encode(self, text):
        preprocessed = re.split(r'([,.?_!"()\']|--|\s)', text)
        preprocessed = [
            item.strip() for item in preprocessed if item.strip()
        ]
        preprocessed = [
            item if item in self.str_to_int
            else "<|unk|>" for item in preprocessed
        ]
        ids = [self.str_to_int[s] for s in preprocessed]
        return ids

    def decode(self, ids):
        text = " ".join([self.int_to_str[i] for i in ids])
        text = re.sub(r'\s([,.?_!"()\'])', r'\1', text)
        return text
```

#### GPT のトークナイザについて

<endoftext>を使っていて<unk>使わない。バイトペアエンコーディングを使っている。

### BPE

ChatGPT で利用されいるオリジナルのトークナイザ。
この本では tiktoken ライブラリを利用する。
未知の単語が来た場合は、小さなサブワードに分割して対応する。

例)
Akwirwier

> "Ak", "w", "ir", "w", " ", "ier"

### 埋め込み

埋め込みをするには、入力変数と目的変数のペアを作ること。
スライディングウィンドウアプローチをつかう。これをサンプリングデータとして利用

```
サンプル
for i in range(1, context_size + 1):
    context = enc_sample[:i]
    desired = enc_sample[i]
    print(context, "-->", desired)
```

###　コードサンプル

```
class GPTDatasetV1(Dataset):
    def __init__(self, txt, tokenizer, max_length, stride):
        self.input_ids = []
        self.target_ids = []
        token_ids = tokenizer.encode(txt)

        for i in range(0, len(token_ids) - max_length, stride):
            input_chunk = token_ids[i:i + max_length]
            target_chunk = token_ids[i + 1: i + max_length + 1]
            self.input_ids.append(torch.tensor(input_chunk))
            self.target_ids.append(torch.tensor(target_chunk))

    def __len__(self):
        return len(self.input_ids)

    def __getitem__(self, idx):
        return self.input_ids[idx], self.target_ids[idx]
```

### トークンの埋め込み

最後のステップ。tokenid をベクトル変換する必要がある。埋め込みそうの重みをランダムにする必要もある。これはルックアップ演算が行われる。

```
def sampleprint():
    input_ids = torch.tensor([2,3,5,1])
    vocab_size = 6
    output_size = 3

    torch.manual_seed(123)
    embedding_layer = torch.nn.Embedding(vocab_size,output_size)
    print(embedding_layer(input_ids))
```

```
# Embed instance
vocab_size = 50257
output_dim = 256
token_embedding_layer = torch.nn.Embedding(vocab_size, output_dim)

# dataloader
    dataloader = create_dataloader_v1(
        raw_text,
        batch_size=8,
        max_length=4,
        stride=4,
        shuffle=False
    )
    data_iter = iter(dataloader)
    inputs, targets = next(data_iter)
    print("Token ids:\n", inputs)
    print("Targets:\n", inputs.shape)

    token_embeddings = token_embedding_layer(inputs)
    print("token embeddings", token_embeddings.shape)
```

### 単語の位置について

SelfAttention の欠点としてトークン ID の位置、順序とかは関係ないこと。
これを位置情報を組み込めば助けになる。
方法は 2 つ。相対と絶対。

・絶対
トークンの埋め込みにユニークな値を利用。GPT モデルはこっち。

・相対
トークン間がどのくらい離れているか、ということができる。

```
# position embedding
context_length = max_length
pos_embedding_layer = torch.nn.Embedding(context_length, output_dim)
pos_embeddings = pos_embedding_layer(torch.arange(context_length))
print(pos_embeddings.shape)

input_embeddings = token_embeddings + pos_embeddings
print("input embeddings:", input_embeddings.shape)
```

## まとめ

入力埋め込みパイプラインは以下となる。

まず入力テキストが個々のトークンに分割、語彙を使ってトークン ID になる。
このトークン ID が埋め込みトークンになり位置情報埋め込みが加算される。
これでメイン層の入力となる。

---

# Chapter3

# attention

このセクションではここの話。
入力テキスト > 前処理ステップ > Self-Attention(ここ) > 後処理ステップ > 出力テキスト

## Self-Attention がない時代の問題点

・例えば翻訳をするツールを作っている。
・RNN を使っていた
・エンコーダで input を処理
・デコーダでテキスト生成

・RNN ではエンコーダで隠れ状態を全体から処理する。よくわからない。隠れ状態。どうやら圧縮表現をエンコードするらしい。とにかく input をメモリに入れる？
・RNN ではデコーダでこのメモリを元に生成する。
・RNN では長文だと忘れるという欠点があった。デコーダは直前の値にアクセスできない？これ何かに例えないと

### Bahdanau Attention の登場

2014 年に RNN 用のメカニズム開発。
・Attention ではデコーダが全ての input にアクセスできることが特徴
・入力の位置が他の位置との関連性を考慮できることが特徴。注意を払えることができる

### Self とは

Transhfomer アーキテクチャのすべての LLM の基礎となるもの。self とは 1 つの入力に対して異なる位置を関連づけることで Attention の重みを計算する、と言うことらしい。なんとなくはわかった。

### step1: ウォークスルー 「重みを持たない 単純な　 Self-Attention」

・まず重みを持たない Self-Attention を実装する
・まず概念として SA の目標はコンテキストベクトルを作ることである。その途中で重みを使うことである
・入力から 3 次元ベクトルのデータを扱っている。とにかくまず各トークンを 3 つの値で表現している。
・

### サンプル

テキスト: Your journey begins with one step
コード:

```
import torch

inputs = torch.tensor(
    [[0.43, 0.15, 0.89], # Your
     [0.55, 0.87, 0.66], # journey
     [0.57, 0.85, 0.64], # starts
     [0.22, 0.58, 0.33], # with
     [0.77, 0.25, 0.10], # one
     [0.05, 0.80, 0.55] ## step
    ]
)
```

### コンテキストベクトル

SA では重要。

### SA 構築の最初のステップは、スコアを出すこと

Attention スコアと呼ばれる中間値を計算すること。
これはトークン同士の関連度を数値すること。
実際にやることはベクトルの内積計算。大きいほど関連が強い。

```
query = inputs[1]
attn_scores_2 = torch.empty(inputs.shape[0])
for i, x_i in enumerate(inputs):
    attn_scores_2[i] = torch.dot(x_i, query)
```

やっていることは、全部かけて足す。これを配列で繰り返す。

```
[0.43, 0.15, 0.89] * [0.55, 0.87, 0.66] +
[0.55, 0.87, 0.66] * [0.55, 0.87, 0.66] +
[0.57, 0.85, 0.64] * [0.55, 0.87, 0.66] +
[0.22, 0.58, 0.33] * [0.55, 0.87, 0.66] +
[0.77, 0.25, 0.10] * [0.55, 0.87, 0.66] +
[0.05, 0.80, 0.55] * [0.55, 0.87, 0.66]
```

### 正規化

このスコアを正規化する。ソフトマックス関数を使うことで、極端な値を安定させることができる。
これをすると重みとなる。
Attetion の重みの総和が 1 となる。

```
def softmax_native(x):
    return torch.exp(x) / torch.exp(x).sum(dim=0)
```

メリットは常に正の数になること、重みが大きいほど重要度が高いと考えることができる。
正規化をして 1 にしないと、関連度がわかりにくい。

### step2: 全ての入力トークンについて Attention の重みをつける

```
attn_score = torch.empty(6,6)
for i, x_i in enumerate(inputs):
    for j, x_j in enumerate(inputs):
        attn_score[i, j] = torch.dot(x_i, x_j)
print(attn_score)
```

for 以外のアプローチも大事

### 次は訓練可能な重みを持った Self-Attention

Scaled Dot-Product Attention と呼ばれる。
最初に作ったものとの違いは、特定の入力要素に対応する入力ベクトルの加重和としてコンテキストベクトルを計算する。訓練可能な重み。

具体的には x に対してクエリ、キー、バリューベクトルを最初に計算する。
クエリベクトルは、x と重み行列の行列積で出せる

```
x_2 = inputs[1]
d_in = inputs.shape[1]
d_out = 2

torch.manual_seed(123)
W_query = torch.nn.Parameter(torch.rand(d_in, d_out), requires_grad=False)
W_key = torch.nn.Parameter(torch.rand(d_in, d_out), requires_grad=False)
W_value = torch.nn.Parameter(torch.rand(d_in, d_out), requires_grad=False)

query_2 = x_2 @ W_query
key_2 = x_2 @ W_key
value_2 = x_2 @ W_value
print(query_2)
```

クエリ、キー、バリューとして最適化をすることで訓練可能となる。
重みと重みパラメータは違っている。重みは関連度を図るもの。
重みパラメータはただのパラメータとなる。

ここからスコアを計算する

```
keys_2 = keys[1]
attn_score_22 = query_2.dot(keys_2)
print(attn_score_22)
```

最後にコンテキストベクトルを計算する

```
context_vec_2 = attn_weights_2 @ values
print(context_vec_2)
```

### 恐るな。casualattention で

Masked Attention とも呼ばれる。トークンを処理するたびに入力テキストの現在のトークンよりも後ろにある未来のトークンをマスクする。
input に対して Attention の重みを使ってコンテキストベクトルを計算するときに未来のトークンにアクセス不可にする。

### singleattention

### multiattention
