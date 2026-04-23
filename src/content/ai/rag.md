---
title: "RAG"
slug: "rag"
order: 3
source: "ai"
---

## 分類と鄭義

今でも重要な論文: Retrieval_augmented Generation for Large Language Models: A Survey (arXiv: 2312.10997)

1. Naive RAG -　最も基本的な RAG
   質問　-> を関連文書を取得しプロンプトに付与 -> LLM で回答
   検索の質がそのまま品質となる。とりま基礎

2. Advanced RAG ー 精度を高める実務アプローチ
   Naive の直列に前処理と後処理が入る。検索命中率と提示根拠の質を底上げ。

   前処理:
   ・Query Rewriting: 曖昧質問を検索エンジに強い形に言い換え。
   ・Query Routing: 質問をカテゴリ判定して最適インデックスへ振り分け
   ・Query Expansion: 同義語・構文違いを補って取りこぼしをへらす

   後処理:
   ・Reranking: 並べ直し
   ・Summarization: 引用に耐える断片へ整形
   ・Fusion: ふくす文書の融合

3. Modular RAG - 部品化の設計で制度を積み上げる
   検索・再ランキング・書き換え・読解・記憶・推論を独立モジュールとして切り出して使う。
   いまはあまり利用され無い。Advanced RAG に取り込まれてリウ

## LLM as a Judge

2024 年くらいから LLM での評価も流行ってきた。

1. InContext Learning : Score, Yes/No, Pair, 選択肢で指示を LLM に与える
2. Model Selection : 評価者としての LLM を選択
3. Post_processing: 評価結果を整理、数値化や根拠分の抽出。
4. Evaluation:これらをまとめて良し悪し

#### パターンがある

#### 改善
