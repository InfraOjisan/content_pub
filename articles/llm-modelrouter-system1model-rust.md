---
title: "流行りの意思決定モデル（System 1 Model：いわゆるJev的なやつ）を使ったモデルルーターをRustで作ってみた"
emoji: "🌐"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aiagent", "llm", "modelrouter", "rust", "jev","laya"]
published: false
---

---
title: "RustでローカルLLMルーターを作った：Layaの選択型判定でモデルを使い分ける"
emoji: "🚦"
type: "tech"
topics: ["rust", "llm", "ollama", "laya", "ai"]
published: false
---

LLMを使うタスクには、短い文章の整形から、長文の要約、画像の読み取り、複雑な設計・コーディングまで幅があります。これらを毎回同じモデルに送ると、軽い仕事にも高価なモデルを使ったり、逆に難しい仕事を小さなモデルへ送ったりします。

そこで、リクエストを受け取り、内容に応じて転送先を選ぶローカルのモデルルーターをRustで作りました。

ソースコード：[InfraOjisan/llm_router](https://github.com/InfraOjisan/llm_router)

## ローカルにルーターを置く意味

このルーターは `http://127.0.0.1:8787` で待ち受けます。アプリやエージェントはルーターにリクエストを送り、ルーターが設定に従ってOllamaなどのローカルLLM、画像対応モデル、外部APIを選びます。

```text
アプリ・エージェント
        ↓
ローカルのRust製ルーター
        ├─ 簡単な仕事 → ローカルLLM
        ├─ 要約・レポート → 日本語と長い文脈に強いモデル
        ├─ 画像・PDF → 対応する画像認識・OCRモデル
        └─ 複雑な設計・コード → 推論能力の高いモデル
```

利点は主に3つあります。

**コストを調整しやすい。** 簡単なタスクをローカルモデルで処理できれば、その分の外部API利用を減らせます。ただし、ローカル実行にもハードウェア、電力、運用のコストがあります。現時点で、このルーターによる削減率は測定していません。

**モデルの選び方を一か所で管理できる。** 転送先のURL、モデル名、ルーティング条件をJSONファイルで変更できます。各アプリにモデル選択ロジックを分散させずに済みます。外部APIのキーも設定ファイルへ直接書かず、環境変数から読み込みます。

**送信先を明示できる。** ローカルで処理したいタスクと外部APIを使うタスクを設定で分けられます。ただし、外部APIを指定した経路のデータは当然そのサービスへ送られます。「ルーターがローカルにある」ことと「すべての処理がローカルで完結する」ことは別です。

## なぜ意思決定モデルを使うのか

ルーティングに通常の生成LLMを使うこともできます。しかし、ルーターに必要な答えは長い文章ではありません。必要なのは、たとえば `simple`、`summary`、`ocr`、`deep` の**どれを選ぶか**です。

[JevのDecision Model API](https://jev-router.com/decision-model-api)や[Laya](https://github.com/NandhaKishorM/laya)は、このような選択肢を持つ質問を扱えます。Layaでは、入力文と選択肢の説明を渡すと、`choice` という構造化された答えと確信度が返ります。自由文を生成させてから「モデル名らしい文字列」を解析する処理が要りません。

このルーターでは、JSONに書いた自然言語の条件を選択肢として判定モデルへ渡します。

```json
{
  "id": "summary",
  "condition": "Summarization, reports, translation, or analysis of a long document, especially when Japanese fluency and context length matter more than deep reasoning.",
  "endpoint": "japanese"
}
```

判定結果の確信度が設定値に届かない場合や、判定サーバーに接続できない場合は、指定したフォールバック経路を使います。

Layaは文章を逐次生成するモデルではなく、選択型の判定を行うモデルです。そのためルーティング用途では低レイテンシが期待できます。ただし、**このルーターと判定モデルを組み合わせた実測レイテンシはまだ公開していません**。初回のモデル読み込みや実行環境によっても時間は変わります。

## なぜRustを使うのか

複雑なルーティングロジックはコード内に無く、意思決定モデルの判定結果に沿って送り先に中継するだけですから、レイテンシを下げるためのRustです。ルーティング内容を変更するには設定ファイルやモデルを変更するだけですのでビルドは初回しか必要ではありません。つまりRustの「変更ごとにビルド」という煩わしさはマスクされます。

## ルーティングの流れ

最初のリクエストでは、次の順に経路を決めます。

1. 画像・PDFを含む入力なら、設定した画像・OCR用経路へ送る
2. それ以外は、Laya互換の `/v1/systemone` APIで経路を選ぶ
3. 判定に失敗した場合や確信度が低い場合は、フォールバック経路を使う
4. 選ばれた転送先のモデル名でリクエストの `model` を置き換え、転送する

同じ `x-harness-id` と `x-session-id` の組は、最初に選んだ経路へ固定します。会話の途中でモデルが切り替わり、回答の性質が急に変わることを避けるためです。同時に届いた最初のリクエストも、一度だけ判定します。

## 動かし方

Rustのツールチェーンと、使いたいモデルのサーバーを用意します。設定例には、ローカルのLaya判定サーバー、Ollama、その他のOpenAI互換エンドポイントが書かれています。モデル名やURLは自分の環境に合わせて変更してください。

```bash
git clone https://github.com/InfraOjisan/llm_router.git
cd llm_router
cp config.example.json config.json
# config.json を編集
cargo run --release -- --check config.json
cargo run --release -- config.json
```

リクエストはOpenAI互換のチャットAPIへ送れます。ハーネスIDとセッションIDは必須です。

```bash
curl http://127.0.0.1:8787/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "x-harness-id: my-agent" \
  -H "x-session-id: conversation-123" \
  -d '{
    "model": "auto",
    "messages": [
      {
        "role": "user",
        "content": "この設計案の問題点を検討して"
      }
    ]
  }'
```

`POST /v1/responses` にも対応しています。ストリーミング応答はチャンク単位で転送します。応答ヘッダーの `x-router-route` で選ばれた経路を確認できます。

## 現時点での範囲

このルーター自身はOCRを実行しません。画像やPDFを対応する転送先へ送る仕組みです。PDFを処理できるかどうかは、転送先が受け付けるAPI形式にも依存します。

また、セッションの固定情報は現在メモリ内にあります。ルーターを再起動すると固定情報は失われます。設定のホットリロード、転送先障害時の自動切替、Mac・Windowsでのビルド確認も今後の課題です。

GitHub Actionsでは、Linux上の `cargo test` と `cargo build --release` が[成功しています](https://github.com/InfraOjisan/llm_router/actions/runs/36523890929)。速度やルーティング精度、実際のコスト差は、利用するモデルとタスクを決めて測定する必要があります。

## まとめ

モデルルーターの価値は、「常に最も強いモデルを使う」ことではなく、**必要な能力に合ったモデルを選べる状態を作ること**にあります。ローカルで処理できる仕事を増やしつつ、難しい仕事には適切なモデルを使う。その判断を、自由文の生成ではなく選択型の意思決定モデルに任せる構成を試しました。

コードと設定例は[GitHub](https://github.com/InfraOjisan/llm_router)で公開しています。