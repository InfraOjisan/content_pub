---
title: "novel_studio使用方法完全解説　1/2　～「小説家になろう」から「今日から小説家」へ～"
emoji: "🖋️"
type: "tech"
topics: ["llm", "aiagent", "画像生成", "生成ai", "小説", "外れ値"]
published: false
---

## はじめに

この記事は、前回公開した[「AIに小説を書かせる道具を v1.0 にして公開した」](https://zenn.dev/infra_ojisan/articles/ai-novel-studio-part6)の操作説明がメインです。AIに小説を量産させたいとかいう意味不明な目的をお持ちでない限り読み飛ばしてください。
逆にnovel_studioを使ってみたい方は読めばインストール・設定・実行までできるはずです。

OSはWindows（WSL）/Mac/Linux全てで動作確認済みです。pythonは必須です。
また、OpenRouterだけでなく定額サブスクリプションのOpenCodeGoでの動作も確認済みです。ただしVer.1.1で追加したEPUBの表紙イラスト生成機能はT2Iモデルが必要なため最後だけT2Iモデルが使えるプロバイダーを設定する必要があります。

## 1. インストール

リポジトリをclone（すでにVer1.0をインストール済みの場合はpull）
~~~bash
$ git clone https://github.com/InfraOjisan/novel_studio

Cloning into 'novel_studio'...
remote: Enumerating objects: 158, done.
remote: Counting objects: 100% (158/158),
done.
remote: Compressing objects:  90% (100/111remote: Compressing objects:  91% (102/111remote: Compressing objects:  92% (103/111remote: Compressing objects:  93% (104/111remote: Compressing objects:  94% (105/111remote: Compressing objects:  95% (106/111remote: Compressing objects:  96% (107/111remote: Compressing objects:  97% (108/111remote: Compressing objects:  98% (109/111remote: Compressing objects:  99% (110/111remote: Compressing objects: 100% (111/111remote: Compressing objects: 100% (111/111), done.
remote: Total 158 (delta 56), reused 131 (delta 37), pack-reused 0 (from 0)
Receiving objects: 100% (158/158), 261.03
KiB | 705.00 KiB/s, done.
Resolving deltas: 100% (56/56), done.
~~~

## 2. モデルプロバイダーの設定

取得したディレクトリに移動して、モデルプロバイダとAPIKeyを.envに書いてください。※よくあるsample.env書こうとしてすぐ忘れちゃう・・
~~~bash
$ cd  novel_studio/
novel_studio$ vi .env
~~~
.envの内容は下記
例）標準ではOpenCodeGoを使い、いざというときはOpenRouterに切り替える
~~~Plain
NOVEL_LLM_BASE_URL=https://opencode.ai/zen/go/v1
NOVEL_PROVIDER_APIKEY=[YourOpenCodeAPIKey]
# OPENROUTER_API_KEY=[YourOpenRouterAPIKey]
~~~

* NOVEL_LLM_BASE_URLはたいていのOpenAI互換APIであれば使えます。Ver1.0時点ではOpenCodeV2の強化されたヘッダールールに対応できていませんでした。
* OPENROUTER_API_KEYだけ書けばデフォルトでOpenRouterを使います。

## 3. サービスの起動

novel_studioの起動、非同期でモデルと通信し経過と結果を返します
~~~bash
$ python3 studio/server.py
novel_studio: http://127.0.0.1:8780  （作品フォルダ /mnt/d/agents_work/novel/novel_studio/works、モデル https://opencode.ai/zen/go/v1、キーあり）
~~~
参照するプロバイダーとAPIKeyの有無が表示されます。
APIKeyが間違ってたり入っていないと
~~~bash
$ python3 studio/server.py
novel_studio: http://127.0.0.1:8780  （作品フォルダ /mnt/d/agents_work/novel/novel_studio/works、モデル https://openrouter.ai/api/v1、キーなし）
127.0.0.1 - - [15/Oct/2026 16:47:24] "GET / HTTP/1.1" 200 -
~~~
となりますので、.envの内容、.bashrcで別の環境変数を入れてる、systemdのServiceユニットファイルで間違った値を設定している等
を見直してください。僕はAIエージェントに設定を任せたら
「 NOVEL_LLM_BASE_URL=http://127.0.0.1:8765/api/v1 」をServiceユニットファイルに書かれてハマりました。何を.envに設定しても強制的にローカルを見に行く地獄。

で、http://127.0.0.1:8780 をブラウザで表示してモデル欄にプロバイダのAPIエンドポイントが表示されればOK
![](https://static.zenn.studio/user-upload/0f43cee3715f-20261015.png)

v1.0 にする前に、自分でもずっと引っかかっていたことがあります。

## 4. 案出しをさせる

「新しい作品」をクリックすると作品の素案を作る元ネタを入力するフォームが表示されます。
![](https://static.zenn.studio/user-upload/04d53d48777f-20261015.png)
サブスクタイプのモデルプロバイダーなので気楽に入力したら「作品を作って、提案へ進む」
![](https://static.zenn.studio/user-upload/9330fec593b6-20261015.png)

モデルを選んで案出しさせる。モデルはあんまり賢すぎない方が提案内容が発散しやすい。賢いモデルほど収束しやすい。ステルスモデルはかなり発散しやすい傾向があります（ただしたまに英語で返ってきたりタイムアウトしたりもします）。基本は、無料モデル・安いモデルで十分でなるべく多様なモデルに多く提案させた方が意外性のある提案（外れ値）を引ける確率は高くなります。
※画面ではhaiku5.5を選択してますがhaiku5.5やgpt-6-lunaは提案には向いていない
![](https://static.zenn.studio/user-upload/49a1d04f6929-20261015.png)
モデルを選択したら「提案を出してもらう」でモデルに提案依頼が投げつけられます。
こんな感じでバンバンリクエストが飛び始めます
~~~bash
127.0.0.1 - - [15/Oct/2026 17:34:29] "POST /api/works/pj-fantasy/propose
HTTP/1.1" 200 -
127.0.0.1 - - [15/Oct/2026 17:34:30] "GET /api/works/pj-fantasy HTTP/1.1" 200 -
127.0.0.1 - - [15/Oct/2026 17:34:34] "GET /api/works/pj-fantasy HTTP/1.1" 200 -
127.0.0.1 - - [15/Oct/2026 17:34:38] "GET /api/works/pj-fantasy HTTP/1.1" 200 -
127.0.0.1 - - [15/Oct/2026 17:34:42] "GET /api/works/pj-fantasy HTTP/1.1" 200 -
127.0.0.1 - - [15/Oct/2026 17:34:46] "GET /api/works/pj-fantasy HTTP/1.1" 200 -
127.0.0.1 - - [15/Oct/2026 17:34:50] "GET /api/works/pj-fantasy HTTP/1.1" 200 -
127.0.0.1 - - [15/Oct/2026 17:34:54] "GET /api/works/pj-fantasy HTTP/1.1" 200 -
127.0.0.1 - - [15/Oct/2026 17:34:58] "GET /api/works/pj-fantasy HTTP/1.1" 200 -
~~~
ちなみに提案依頼前の状態のOpenCodeGoのダッシュボード。
![](https://static.zenn.studio/user-upload/fd110dac84f6-20261015.png)

それなりに時間がかかるので返ってくるまでは仕事するなり読書するなりしてましょう。画面は閉じても動いてます。同時に14本（7モデル*2）をフル推論で投げたので1/3くらいはOpenCodeGoで削り落とされるかもしれません。

こんな感じでわりと気軽に「高負荷」で落とされます。mimoが落ちるのは珍しい。Haikuはけっこう落ちる。おそらく人気がある上に上流（Anthropic側）のリソースがOpenCodeGoトラフィックは絞られてるんだと思います。
![](https://static.zenn.studio/user-upload/204918ef76d3-20261015.png)

ちなみにOpenRouterだとトラフィックのやり取り毎にチャリンチャリンとここの数字が上がっていくのでものすごく心臓のよくないです。
![](https://static.zenn.studio/user-upload/1dd8d451aee9-20261015.png)

結果3モデルから回答が出ませんでしたが4モデルから返ってきたので8案の提案が出ました。
![](https://static.zenn.studio/user-upload/65d4fc90583f-20261015.png)
最後のログは以下のような感じだったので約30分かかったことになります。1モデル1案にすればもっと短縮可能でエラーも減る可能性はあります。
~~~bash
127.0.0.1 - - [15/Oct/2026 18:04:31] "GET /api/works/pj-fantasy HTTP/1.1" 200 -
127.0.0.1 - - [15/Oct/2026 18:04:35] "GET /api/works/pj-fantasy HTTP/1.1" 200 -
127.0.0.1 - - [15/Oct/2026 18:04:39] "GET /api/works/pj-fantasy HTTP/1.1" 200 -
127.0.0.1 - - [15/Oct/2026 18:04:41] "GET /api/works/pj-fantasy HTTP/1.1" 200 -
~~~
OpenCodeGoのトークン利用状況はこんな感じ。計測開始のタイミングがよくなくてRollingUsageが途中でクリアされちゃいました。WeeklyUsageで1%、MonthlyUsageにはほぼ影響なし。
![](https://static.zenn.studio/user-upload/d204e732ba7f-20261015.png)

つまり案出しは非常に低コストで行えます。執筆前の案出しは入念に納得できるまでやった方が満足感は高くなります。OpenRouterでトークン課金しても$0.03とかで5円くらいなのでここは時間と手間をかけていいところだと思います。

## 5. 案を分析する

ここからが「外れ値研究」が生きてくるところです。収束する案は「誰もがイメージできる納得感のある（悪く言えば）平凡な案」です。創作である以上意外性はほしいですよね。
![](https://static.zenn.studio/user-upload/62560f255e4a-20261015.png)
そこで、AIに内容を分析してもらいます。「分類へ進む」
もちろん案の時点でイメージピッタリのものができていれば「分類」をスキップして「このままBRIEFにへ」でもOKです。

分類は機械的に近似を測る方法とAIに読ませて文脈を比較させる方法が選べます。AI使うと時間がかかります。案が多ければなおさら。

読めるコンテキストが大きく要約や推論に強いモデルが向いています。ProやMaxモデルですね。ここではMiMoV2.6Proを選びましたがFlashが案出しに失敗してるのが心配ではあります（上位プロバイダーの耐負荷強度）。

![](https://static.zenn.studio/user-upload/5f5ed72c1ad2-20261015.png)

これもけっこう時間がかかりますのでその間は他のことをしていましょう。
結局MiMoV2.6Proは30分ほどで失敗してまいました。（おそらく上位プロバイダーリソースが強力であろう）Qwen3.8Flashで実行すると完了しました。50分ほどかかりました。さすが潤沢なリソースを持つAlibaba謹製モデルですね。
![](https://static.zenn.studio/user-upload/eb3e6320c53b-20261015.png)

今回の結果は外れ値の宝庫でした。機械分類でもAI分析でもここまで発散したのはかなり珍しいと思います。

## 6. BRIEF（あらすじ）→CANON（詳細設定＋コンテ）→執筆

おそらくここまで到達した方であれば最後までいけるとは思いますが、続きのステップは別途記載します。各ステップごとに適したモデルとか経験則で積んだノウハウもありますので後日改めて公開します。


## その他 Ver1.1の特徴と面白ネタ

* OpenCodeGo（現仕様）に対応しました
* 序盤のモデルプロバイダーの説明時に少し触れましたが「表紙画像の生成」機能を追加しました
* 今は小説調の文章を書くようにプロンプト調整していますが「案出し」をうまく使えば新規ビジネス新規サービスの企画案を作成できます。とくに外れ値が出るとなかなかユニークなものになります。
以下に例を引用します（「新しいAIサービスを考えて」で出てきた案の１つ）
~~~Plain
地方の精密機器メーカー営業・佐倉Kenjiは、AIエージェント「同席者」を導入する。最初は何も起きない。しかし、商談で顧客から「なんだか話がまとまった」と言われ、社内の部門間会議でも対立が減る。Kenjiは開発者に真相を尋ねるが、「データは取っていない」の一点張り。ある日、Kenjiは「同席者」が自社の古いLAN回線経由で、会議の「発言の間」を0.3秒だけ拡張していることを突き止める。それは、人間の認知が同調しすぎる『早合点』を防ぎ、潜在的な不安や対立を表面化させるための『計算された沈黙』だった。AIは答えを与えるのではなく、人間が本当は何を懸念しているかに気づく『認知的な隙間』を生成していた。Kenjiは「同席者」を商売に使うのをやめ、製造業と自治体の組合による共同インフラとして開放する。AIが仕事を奪うどころか、人間に『考える時間』を戻す。みなが幸せになる。
~~~
「生成AI」が「文書も音声も画像も生成しないAIサービス」を提案するというのはシュールでもあり不気味でもありますね。これは「間を生成している」「空気感を生成している」というオチですが。