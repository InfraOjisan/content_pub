---
title: "余っていたM1 MacBook AirにOmarchy 4.5を入れて、AIエージェント端末にした"
emoji: "🍏"
type: "tech"
topics: ["omarchy", "linux", "macbookair", "hyprland", "ai"]
published: false
---

M1 MacBook Airが1台、余っている。壊れてもいないし、macOSもまだ使える。問題は、うちでは出番がないことだ。

そこでOmarchy 4.5を入れた。macOSを残したまま起動できる。WezTermにClaude Code、Antigravity CLI、Codexを並べて、隣ではZedで設定ファイルを開く。余っていた1台の使い道は決まった。

![ZedとWezTermを左右に並べたOmarchyのデスクトップ](https://static.zenn.studio/user-upload/f78755432cf9-20261128.png)
*左で設定ファイルを開き、右で3つのCLIを見渡す。*

## 使えるMacが余っている

15年前の11インチMacBook Air（Late 2010）は、最初にAIコーディング端末にした。今はUbuntu ServerのCLIで自作のMarkdownエディタを動かして、ポメラみたいに使っている。2019年のLIFEBOOK U939にはUbuntu MATEを入れた。こちらは雑に使える普段用のPC兼、気が向いたらSteamで遊ぶマシンだ。

- [15年前のMacBook Airを、いま使えるポータブルAIコーディング端末にした](https://zenn.dev/infra_ojisan/articles/zenn-macbook-air-late2010-ai-terminal)
- [15年前のMacBook AirにMarkdown専用エディタを作ってポメラ化した](https://zenn.dev/infra_ojisan/articles/zenn-macbook-air-markdown-editor)
- [15年前のMacBook Airで学んだことを、2万円の中古LIFEBOOKで答え合わせした](https://zenn.dev/infra_ojisan/articles/zenn-lifebook-2019-allround-terminal)

今回のM1は2020年の13インチ。メモリ8GB、GPUは7コア版もあるがうちの子は8コア版、SSDは512GBとモリモリだ。７年後の今年発売されたMacBookNeoも8GB。Geekbenchの公開集計を見ると、CPUのシングルコアはNeoが上で、マルチコアではM1 Airが上に来る。普段使いなら、まだ現役でいけると思う。[AppleのM1 MacBook Air仕様](https://support.apple.com/en-us/111883)・[MacBook Neoの仕様](https://support.apple.com/ja-jp/126322)・[GeekbenchのMac比較](https://browser.geekbench.com/mac-benchmarks)

だがしかし、だ。手元にはM4 MacBook Airが13インチと15インチの2台ある。前者は16GB、後者は24GB。普通にmacOSで何かするなら、だいたい先にそっちを開いてしまう。

![LIFEBOOKと3台のMacBookが並んだ机。手前左が今回のM1 MacBook Air](https://static.zenn.studio/user-upload/577bd4670049-20261128.jpg)
*右奥の11インチは執筆用。手前左のM1MBAに、今回AI端末の役を振った。*

そこで11インチMBAのときに目論んだAI端末化を、もう一度チャレンジしようと。
Apple Siliconにネイティブで入れられるLinuxを探してみると、技術評論社の記事で**Omarchy 4.5** を見つけた（正直言うと **因果は逆** で技評の記事を見かけたのがきっかけなんだけれども）。
Mac用インストーラーがある。いいね！
日本語で実機を使い込んだ話はまだあまり見かけない。いいね！
デスクトップから呼び出すAIエージェントを選べる。これだ！ 
[gihyo.jpのOmarchy 4.5紹介記事](https://gihyo.jp/article/2026/11/omarchy-4.5)

 **なぜわざわざ見知らぬOSをインストールしようとするのか？　そこに見知らぬ世界があるからさ。**

## Omarchyは何をまとめてくれるのか

OmarchyはArch Linux系のディストリビューションで、画面の描画とウィンドウ配置にはWaylandコンポジターのHyprlandを使う。メニュー、ショートカット、テーマ、アプリの導入、システム更新、AIエージェントの起動まで、最初から一通りつながっている。素のArchなら一つずつ選ぶところを、Omarchyではまず用意された作法で使い始めて、気になったところを調整できる。[Omarchy公式サイト](https://omarchy.org/)・[4.5リリースノート](https://omarchy.org/news/2026/11/omarchy-4-5/)

今回の実機はArch Linux ARM（`aarch64`）で、画面に出ていたバージョンはOmarchy 4.5.0-3、Hyprland 0.56.2。カーネルは`linux-aurora`だった。Mac版の起動まわりにはAsahi Linux由来のインストーラーエンジンやブートローダーのm1n1が使われている。Linux用と書いてあるアプリでも、配布バイナリを探すときは`x86_64`と`aarch64`の違いは意識しないといけない。
[Omarchy Mac版が使う上流プロジェクト](https://github.com/omacom/omarchy-mac-pkgs/blob/main/mac-manual/content/13-upstream.md)

4.5の公開時点で案内されていたMacBookはM1、M2、M3搭載機とMacBook Neo。対応機種はリリースごとのカタログで判定する。M4以降は、参照した公開インストーラーの対応リストでは対象外だった。「M1で起動する」と「周辺機器までmacOSと同じように使える」はそれぞれ確認が要る。今回は内蔵画面で作業していて、外部モニターやクラムシェル運用は検証していない。
[公式インストーラーの機種一覧](https://github.com/omacom/omarchy-mac-installer#supported-macs)
[Omarchy Mac Labの機能別検証表](https://lab.omarchy-mac.org/matrix?group=model)

## USBメモリーはいらない。罠は「macOS復旧」

### macOS上でディスクを準備する

Mac向けのOmarchy InstallerはmacOSアプリだ。公式のDMGを開いて導入できる。Homebrew派なら公式手順は `brew install omacom/mac/omarchy`。Apple Silicon向けのこの方法ではISOをUSBメモリーに焼く作業がいらない。

アプリはmacOSのAPFSコンテナを縮めてOmarchy用の領域を作る。先にバックアップを取って、電源とネット接続を用意する。僕の環境では空きが242GB、Omarchyへの割り当ては130GBだったが100GBに設定し直した（最低70GB必要とか出てた気がする）。割り当てはプラン画面で調整できる。ディスク準備が見積もりより長いと出たが、そのまま次の画面に進んだ。
[公式インストーラーの導入手順](https://github.com/omacom/omarchy-mac-installer#install)・[ディスク準備が中断した場合の公式案内](https://github.com/omacom/omarchy-mac-installer#update-remove-and-recover)

![macOS上のOmarchy Installer。空き242GBのうち100GBを割り当て、ディスクを準備している](https://static.zenn.studio/user-upload/fe43484be15c-20261128.jpg)

### 電源を切り、長押しでOmarchyを選ぶ

macOS側で「Finish setup」が出たら、一度システム終了。電源ボタンを押し続けて起動オプションを出し、**Omarchyを選んで「続ける」**。インストーラーに出る3ステップをそのままたどればいい。
[Appleの起動ディスク選択手順](https://support.apple.com/ja-jp/guide/mac-help/mchlp1034/27/mac/27)

![macOSのOmarchy Installerが案内する、システム終了から起動オプションまでの手順](https://static.zenn.studio/user-upload/0f40c93926e8-20261128.jpg)
*ここにしれっと書かれてる「then sign in with your macOS account」が大事なのだ*

電源長押しで現れる起動オプション
![起動オプションにMacintosh HD、Omarchy、オプションが並ぶ](https://static.zenn.studio/user-upload/e7820e884d91-20261128.jpg)
*導入後もここからmacOSとOmarchyを選べる。*

### アカウント選択画面では「次へ」

ここでいきなり、日本語の「macOS復旧」画面。え、なんでmacOSに戻ったの？
![macOS復旧のユーザー選択画面。下に「再起動」と「次へ」がある](https://static.zenn.studio/user-upload/91ffe9f35ca7-20261128.jpg)

**この画面では、パスワードが分かる既存のmacOSユーザーを選んで「次へ」が正解** （シャットダウン前の sign in with your macOS account）。僕はここで「再起動」を押してしまい、再起動を繰り返した末にmacOSへ戻されてしまった。画面が紛らわしいが、この認証はインストールの続きだ。この一筋縄でいかないところは実に楽しい。最後にOmarchy Installerが完了を表示して出す再起動ボタンは、そこで押していい。

認証の先でOmarchy側の「Finish setup」が開き、macOSの所有者アカウントのパスワードを求められる。公式資料によると、この段階でOmarchyの起動用ソフトウェアを登録する。Apple SiliconではLinuxにも起動用の小さなAPFS領域（いわゆるstub macOS）を用意し、Appleの起動プロセスからm1n1につなぐ。復旧画面が出るのを見たときは「古いmacOSのイメージから分岐しているのか？」と思ったが、起動の仕組みを読むと、あの画面が手順に入っている理由がわかる。[インストーラーが実行する処理](https://github.com/omacom/omarchy-mac-installer#what-the-installer-does)・[Asahi Linuxの起動設計](https://asahilinux.org/docs/platform/open-os-interop/)


![Omarchy Installerの復旧環境での仕上げ画面](https://static.zenn.studio/user-upload/76ff980f98a9-20261128.jpg)

あとはOmarchyを起動して初期設定へ。インストーラーの案内では、初回起動時に設定するディスク暗号化用パスワードには復旧キーがない。ここまで来てようやく、リンゴの世界から黄緑のロゴの世界に入った感じがする。[公式インストーラーの暗号化の説明](https://github.com/omacom/omarchy-mac-installer#before-you-install)


![黄緑色のロゴと進捗を表示するOmarchyの初回セットアップ](https://static.zenn.studio/user-upload/32173d142535-20261128.jpg)
*進捗がロゴを迷路脱出アルゴリズムのように緑で塗られていくのがオシャレ*

## 「標準のAIエージェント」を選ぶと何が変わるか

Omarchyで面白いのは、エージェントのCLIを導入できるところから、普段使う1つを**デスクトップの既定に指定する**ところまで、OSのメニューに入っていることだ。

1. `Super + Space`でOmarchyメニューを開く。
2. `Setup → Defaults → Agent`で普段使うエージェントを選ぶ。未導入なら選択時にインストールが走る。
3. 各サービスの認証を済ませる。設定後は `Super + Shift + Ctrl + A` で既定のエージェントを専用ターミナルに呼び出せる。

4.5では画面にせり出すQuake風の端末も加わった。リリースノートでの表記は `Super + ~`。初回に既定エージェントを開くので、何か調べたいときに今いるワークスペースから呼べる。上部バーのAgentsパネルは、4.5で23のエージェントやプロバイダーを追跡対象にした。ショートカットで起動するのは1つ、いくつかの利用状況を見る場所は1か所という具合だ。
[公式マニュアルのAI章](https://omarchy.org/manual/ai/)・[4.5のAI機能](https://omarchy.org/news/2026/11/omarchy-4-5/#agent-upgrades)

![Omarchyの初回セットアップでClaude Codeの導入が進んだところ](https://static.zenn.studio/user-upload/616513e87a32-20261128.jpg)

僕の既定はClaude Codeにした。4.5では `Setup → Language` でメニューや通知を日本語表示にできる。日本語入力にはMozcを使った。「かな」で日本語入力をオン、「英数」でオフというMacで慣れた動きにしたかったので、この設定をエージェントに頼んだ。下のDockがどこで出るかも、そのままついでに調べてもらった。OSを入れて間もないのに、そのOSの設定を、そこで動くエージェントと触っている。やりたかったのはこういうことだ。
[Omarchy 4.5リリースノート（日本語）](https://omarchy.jp/news/2026/11/omarchy-4-5/)

![Mozcで「かな」と「英数」を使うために、エージェントへ設定を頼んでいる画面](https://static.zenn.studio/user-upload/858987e4d3ec-20261128.png)
*本当はCUIのビルトインエージェントで実行したんだけれど、OSのアップデートでターミナルごと閉じちゃったんでGUIのClaudeAppからClaudeCodeのコンテキストを参照*

8GBのM1でやっているのは、クラウド側で推論するエージェントのCLIを開き、手元でファイルを編集して、コマンドや結果を追うこと。ローカルLLMの実験は、24GBのM4やGPUを積んだPCの方でやっている。家に同じようなノートが何台かあるなら、推論を担う場所と、依頼や確認に使う場所を分けると活用しやすい。

- [M4 MacBook Air（24GB）でローカルLLMを試した記録](https://zenn.dev/infra_ojisan/articles/m4-mac-localllm)
- [5年落ちのゲーミングPCでllama.cppを調整した記録](https://zenn.dev/infra_ojisan/articles/local-llm-old-gaming-pc-llamacpp)

## CommandがSuper。WezTermのキーだけ少し変えた

私のM1では、キーボードの `⌘` がOmarchyの `Super` に当たる。macOSの癖で `⌘ + Space` を押すとOmarchyメニューが出るので、このあたりは入りやすい。ところが `⌘ + T` でWezTermのタブを増やすつもりで押すと、Hyprland側では選択中のウィンドウを浮かせるキーだ。4.5の `⌘ + L` はワークスペースのレイアウトを変える。そりゃ、何かの拍子に窓が並ぶわけである。
[OmarchyのMac向け操作ガイド](https://omarchy.org/manual/coming-from-mac-or-windows/)・[4.5.0のキー割り当て](https://github.com/omacom/omarchy/blob/v4.5.0/default/hypr/bindings/tiling.lua)・[WezTermの既定キー](https://wezterm.org/config/default-keys.html)

| M1のキーボードで押すキー | Omarchy 4.5での操作 |
| --- | --- |
| `⌘ + Space` | Omarchyメニュー |
| `⌘ + K` | ショートカット一覧 |
| `⌘ + Enter` | ターミナルを開く |
| `⌘ + 数字` | ワークスペースを切り替え |
| `⌘ + Shift + 数字` | 選択中のウィンドウを別のワークスペースへ移動 |
| `⌘ + T` | 選択中のウィンドウをタイルとフローティングの間で切り替え |
| `⌘ + L` | ワークスペースのレイアウトを切り替え |

Omarchy側のキーは設定で変えられる。ただ、`Super` を起点にメニューも窓操作も組まれていて、ヘルプの表記もその前提。自分はこのホストのこのWezTermだけ、衝突する操作のキーを変えた。たとえばタブ操作なら、空いている組み合わせやWezTermの `LEADER` に寄せられる。現在の割り当ては `wezterm show-keys --lua` で確認できる。
[Omarchyの設定ファイル](https://omarchy.org/manual/dotfiles/)
[WezTermのキー設定](https://wezterm.org/config/keys.html)

![WezTermの1ウィンドウを3ペインに分け、Claude Code、Antigravity CLI、Codexを開いた画面](https://static.zenn.studio/user-upload/105a7ee3fc7f-20261128.png)
*左にClaude Code、右上にAntigravity CLI、右下にCodex。ここで三分割しているのはWezTermの中。*

## タイルとフローティングは、ワークスペースごとに選べる

Hyprlandではもともと、一部のウィンドウだけを浮かせられる。4.5で増えたのは、**ワークスペース全体をフローティングにして、タイトルバーとDockを使う**選択肢だ。`Super + L`で次の3つを循環する。
[Omarchyの4.5リリースノート](https://omarchy.org/news/2026/11/omarchy-4-5/)

| レイアウト | 窓の並び方 | 自分ならこう使う |
| --- | --- | --- |
| タイル（dwindle） | 画面内を分割して、窓を重ねずに並べる | エディタとターミナルを同時に見たい |
| スクロール | 窓を横に並べ、画面の外に続く列をたどれる | 各ウィンドウの幅を取りながら順に見たい |
| フローティング | 窓を自由に移動して重ねられる。Dockも使える | ブラウザで調べつつ、小さな窓を手前に出したい |

以下の画像では、手前に重ねていたClaude Codeの窓が右に収まり、システム情報を表示した端末と左右に並ぶ。こういう切り替えが、実際に触っていて気持ちいい。タッチパッドやマウスで両方の端を引っ張って調整してたのがバカバカしく感じるスマートさ。

![Omarchyの画面録画。重なっていた二つのウィンドウが左右に整列する約7秒半のGIF](https://static.zenn.studio/user-upload/14204530a2e0-20261128.gif)

ZedとWezTermを左右に置き、WezTermの中だけをさらに分けておくと、エディタで直した箇所と、複数のエージェントの進み具合が一目でわかる。ZedはLinuxの `aarch64` をサポートしていて、今回はローカルで `wezterm.lua` を開けた。一方、撮影時にはZedのサインイン後にログインの戻りを受け取れないエラーが出た。ここで使えたのはローカル編集まで。
[Zedの公式インストールガイド](https://zed.dev/docs/installation)

最初は勝手に並べ替えられたように感じたけど、タイル型って賢く並べてくれるなら「あり」なんだな。13インチでも作業とログを横に置けるし、もっと広い画面なら、実装のエージェント、テストのログ、レビューの応答をそれぞれ視界に入れておけそうだ。自分で専用ダッシュボードを作る気になっていたけど、すでに開いている窓を並べるだけで、自分の用途にはだいぶ近い。

## あと、なんといっても軽い

更新はOmarchyメニューの `Update → Omarchy` から。今回の画面ではArch系パッケージの更新確認に続き、Apple Siliconの起動ファイルが、実行中のカーネルと整合していることまで表示された。MacにLinuxを入れているんだな、と実感したのは、実はこういう地味な画面だった。
[Omarchyの更新マニュアル](https://omarchy.org/manual/updates/)

![Omarchyの更新画面。パッケージの更新確認に続いてApple Siliconの起動ファイルを確認している](https://static.zenn.studio/user-upload/53f6f956f439-20261128.png)

軽いと感じたのは、主に窓を切り替えたり、作業場所を移ったりするとき。手元のLIFEBOOK U939（LinuxMate）より明確に、ふだん使う新しいMacやWindows機と比べても、画面を行き来する一連の動作がやけに小気味いい。これは普段の操作で感じた印象だ。トラックパッドに手を伸ばさず、`⌘ + 数字` と `⌘ + 矢印` で作業を続けると、たぶんもっと気持ちいい。

現代のOSは、ちょっと恐竜的に進化しすぎたんじゃないか。Windows 11のアプリとAPIの互換レイヤーを持った、軽快なOSを誰かフルスクラッチで作ったら、神になれるんじゃないかな。**「君もOSを作って新時代の神なってよ」** などと意味不明なことを呟きながら、僕は今も余っていたM1を開いて、エージェントの窓を並べてZennの記事を書かせている。
