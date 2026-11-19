---
title: "APIキーはいつ失効させる？ LLMjacking時代のシークレット管理"
emoji: "🔑"
type: "tech"
topics: ["security", "llm", "githubactions", "cicd", "cloud"]
published: false
---

PoC用に発行したLLMのAPIキーを `.env` に入れ、`.gitignore` に追加する。動作確認が終わってクラウドの検証環境も削除した。そのキーは、発行元でまだ有効になっていないでしょうか。

APIキーは、設定ファイルや検証環境を消しても、発行元で有効な間は使えます。2026年には、CI/CDやクラウドサービスの環境変数が狙われ、そこからキーが盗まれた事例も公表されました。

開発側でまず決めたいのは、**本番の権限をどこまで渡すか、キーを何時間・何日使えるようにするか、使い終わったら誰が止めるか**です。最近の攻撃事例から、この3点を普段の開発にどう組み込むかを見ていきます。

## 公開GitHubでの漏えいは増え、実行環境も狙われている

GitGuardianが公開GitHubで2025年に新たに検出したシークレットは約2,865万件で、前年比34%増でした。AIサービス関連は約128万件で、前年比81%増です。これは認証情報などが**コード上に露出した件数**を示す調査です。攻撃側の動きでは、Wizが2026年上半期に観測したAIインフラを狙う活動が、2025年下半期のおよそ2倍でした。いずれも各社の観測範囲での数字です。 [GitGuardianの2026年調査](https://blog.gitguardian.com/the-state-of-secrets-sprawl-2026/) · [Wizの2026年上半期調査](https://www.wiz.io/blog/cloud-threat-highlights-h1-2026)

### 盗まれたGitHub認証情報から、CIのシークレットへ

2026年11月にAikidoが報告した「wfx-loot」では、攻撃者が盗んだGitHub認証情報を使い、書き込み権限のあるリポジトリのワークフローなどを改変しました。標的にはCIランナー上の環境変数やGitHub Actionsのシークレット、クラウドやAIサービスの認証情報が含まれます。非公開リポジトリでも、侵入されたアカウントの権限次第で同じ問題が起こります。 [Aikidoの事例分析](https://www.aikido.dev/blog/compromised-github-accounts-push-ci-secret-stealer-to-tens-of-thousands-of-repositories)

悪意ある依存パッケージを開発端末やCIで実行してしまう経路もあります。Microsoftが調査した「Shai-Hulud 2.0」では、改変されたnpmパッケージのインストールをきっかけに、保存済みのクラウド認証情報などが収集されました。依存関係をインストールするジョブで本番キーまで使える構成は、盗まれた場合の影響を広げます。 [MicrosoftのShai-Hulud 2.0調査](https://www.microsoft.com/en-us/security/blog/2025/12/09/shai-hulud-2-0-guidance-for-detecting-investigating-and-defending-against-the-supply-chain-attack/)

### 環境変数に入れたAPIキーも盗まれる

Fireworks AIは2026年10月、評価ジョブに渡した一部顧客の環境変数に不正アクセスがあったと公表しました。同月のRailwayへの攻撃では、プラットフォーム内部に侵入した第三者が一部の環境変数の内容を復号し、盗まれたキーによる不正利用額は推計約74万ドルに達しました。 [Fireworks AIのセキュリティ公表](https://fireworks.ai/security-bulletins) · [Railwayの事後報告](https://blog.railway.com/p/incident-report-october-security-incident)

`.gitignore` が効くのはGitへの誤追加です。アプリやCIジョブに渡された値は、その実行環境で動くコードから読み取られ得ます。シークレット管理サービスを使う場合も、**どのジョブが、どの値を取得できるか**まで設計する必要があります。 [Google Cloudのシークレット管理指針](https://docs.cloud.google.com/secret-manager/docs/best-practices)

### LLMjackingは請求と利用枠に現れる

LLMjackingの典型例では、攻撃者が盗んだAPIキーやクラウド認証情報で被害者のLLM利用枠を使い、推論アクセスを転売します。Google Threat Intelligence GroupとMandiantは、**1日で10万ドル超**の不正請求や、アクセスを止めるまでに**累計100万ドル**に達した事例を報告しています。ランサムウェアのようにファイルが暗号化されなくても、請求や利用枠に被害が出ます。クラウドの権限が広ければ、追加キーの発行や別のデータへのアクセスにもつながります。 [Google／MandiantのLLMjacking調査](https://cloud.google.com/blog/topics/threat-intelligence/llmjacking-monetizing-compromised-cloud-models)

## 発行から失効までを開発フローに入れる

### 開発用と本番用のキーを分ける

キーは環境と用途ごとに発行します。サービスが対応していれば、開発・検証・本番でAPIプロジェクトやクラウドアカウントも分けます。たとえばPoCで推論だけを試すなら、開発用プロジェクトで推論に必要な権限だけを付け、管理操作や保存データへのアクセスは外します。本番キーは開発端末や外部PRのテスト、AIコーディングツールに渡さない設計にします。こうしておくと、開発用キーを失効させるときに本番を巻き込まずに済みます。 [Google Cloudのシークレット管理指針](https://docs.cloud.google.com/secret-manager/docs/best-practices) · [AWS IAMのベストプラクティス](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

### 無期限のキーは原則発行しない

クラウドやCI/CDで対応しているなら、OIDCやワークロードIDを使い、実行時に必要な期間だけ使える認証情報を取得します。外部APIで期限付きのキーを発行できるなら、用途に必要な期間を設定します。有効期限を設定できないサービスでは、発行時点で**キーのID・用途・利用先・責任者・失効予定日**を記録し、発行元で無効化するタスクを作ります。管理台帳にはキーの値を入れません。 [OWASPのシークレット管理指針](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html) · [GitHubのOIDC活用ガイド](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments)

**PoCや一時環境の完了条件に、専用キーが発行元で失効済みであることを入れます。** 作業が予定より早く終わったら、期限まで待たずに失効させます。本番で継続利用する固定キーは、平時の交換であれば新キーの動作を確認してから旧キーを失効させます。漏えいが疑われる場合は、切り替えに時間がかかっても、旧キーの停止を優先します。 [OWASPのシークレット管理指針](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html) · [GitHubの漏えい対応手順](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)

### ジョブに渡す秘密を減らす

- **Git**には、秘密値を抜いた `.env.example` や設定テンプレートを置きます。非公開リポジトリでもプッシュ時のシークレット検査と既存履歴のスキャンを有効にします。 [GitHubの漏えい対策](https://docs.github.com/en/code-security/concepts/secret-security/secret-leakage-risks) · [OWASPの開発時シークレット管理指針](https://owasp.github.io/DevSecOpsGuideline/2-Process/2-2-Develop/2-2-1-Pre-commit/2-2-1-2-Secrets-Management/)
- **CI/CD**では、外部PRのテストや依存関係のインストールと、本番デプロイを分けます。本番シークレットは承認済みのデプロイジョブだけに渡し、ワークフローの変更をレビュー対象にします。外部のGitHub Actionsは検証済みのコミットSHAに固定します。 [GitHub Actionsのセキュリティ指針](https://docs.github.com/en/actions/reference/security/secure-use)
- **本番アプリ**が使う固定キーは、必要なワークロードだけがシークレット管理サービスから取得できるようにします。環境変数で渡す必要がある場合は、渡すプロセスを絞り、値がデバッグ出力やログに残らないようにします。 [Google Cloudのシークレット管理指針](https://docs.cloud.google.com/secret-manager/docs/best-practices)

### 請求が跳ね上がる前に気づけるようにする

環境やプロジェクトごとに、リクエスト数、トークン消費、普段使わないモデルの呼び出し、請求額の急増を監視します。クラウド側では、新しいAPIキーやサービスアカウントの作成、権限付与も検知対象にします。レート制限や利用上限を設定できる場合は活用し、予算設定が**通知だけか、リクエストを実際に止めるか**を確認しておきます。 [Google／Mandiantの検知・緩和策](https://cloud.google.com/blog/topics/threat-intelligence/llmjacking-monetizing-compromised-cloud-models)

## 漏えいを見つけたら、履歴の掃除より先にキーを止める

1. **発行元で無効化する。** 誤コミットや侵害の疑いがあれば、悪用の証拠を待たずに対象キーを止めます。侵害された端末やCIランナーがある場合は並行して隔離します。 [GitHubの漏えい対応手順](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)
2. **同じ環境から取れた認証情報と、侵入後に作られたアクセス手段を確認する。** API利用・請求・監査ログを保全し、DBやパッケージレジストリの認証情報、セッション、新規キーや権限付与も調べます。影響を受けた認証情報や不正に追加されたアクセス手段も無効化します。 [Google／Mandiantの対応指針](https://cloud.google.com/blog/topics/threat-intelligence/llmjacking-monetizing-compromised-cloud-models)
3. **侵入経路を塞いでから新キーを配布する。** 旧キーの失効と新キーでの正常動作を確認し、必要に応じてGit履歴やログ、ビルド成果物に残った値を除去します。 [GitHubの履歴からの削除手順](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository) · [OWASPのインシデント対応指針](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)

## PoCのチケットに書ける完了条件

たとえば、PoCのチケットに次の一文を入れておきます。

> 検証の担当者が、発行元の管理画面または管理APIでPoC専用キーの失効を確認し、チケットにキーIDと確認日時を記録したら完了。
