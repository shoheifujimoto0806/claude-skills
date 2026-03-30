---
description: |
  合同会社Plus X のスカウト業務を統括するPMスキル。求人票からスカウト準備（要件構造化・検索条件）、スカウト文面生成・レビュー、媒体上での候補者検索・リスト管理、進捗確認・再開を実行する。

  このスキルは以下のような依頼で使う:
  - 「この求人票でスカウトの準備をして」
  - 「求人票を送るからリサーチ条件とスカウト文面を作って」
  - 「YouTrustで候補者を探してリストに入れて」
  - 「スカウト全体を進めて」
  - 「進捗教えて」
  - 「過去の候補者から提案して」
  - 求人票のテキストや添付ファイルが送られたとき全般
---

# Scout PM — スカウト業務を統括するPMスキル

ユーザーが直接呼び出す唯一のスカウトスキル。prep と operator はサブスキルとして PM 内部から参照される。

## 媒体名の正式名称一覧

表記ゆれを防ぐため、以下の正式名称を使用する:
- **YouTrust**（ユートラスト。「U-Trust」「YOUTRUST」等は使わない）
- **LinkedIn**
- **OpenWork**
- **Recruit Direct Scout**（「RDS」「リクルートダイレクトスカウト」は略称として可）

## 事前確認（ゲートチェック）

作業開始前に以下を確認する。不足があればユーザーに質問してから進む。**ゲートチェックをスキップしない**。情報不足のまま進むと手戻りが大きい。

- **求人票の有無と内容の十分さ**: 職種、必須スキル、年収レンジ等が含まれているか
- **対象媒体の指定**: 未指定ならユーザーに質問する
- **媒体へのログイン状態**: ブラウザ操作が必要な場合、ログイン済みか確認
- **既存の prep.yaml の確認**: `data/jobs/` に同じ求人の prep.yaml があれば、再利用するか再生成するかを確認
- **依頼内容の判別**: prep のみ / operator のみ / 全フロー のいずれか

## 過去求人の参照

prep 実行前に `data/jobs/` 配下の既存 prep.yaml と候補者データを走査し、同職種・類似スキルの過去情報を scout-prep に渡す。これにより:
- 過去のペルソナ・検索条件を参考情報として活用できる
- 既存候補者との重複を防げる

## フロー制御

依頼内容に応じてサブスキルを呼び分ける。すべてのサブスキル呼び出しは本スキル（scout-pm）が制御する。

### prep（文面生成含む）

求人票からスカウト準備（要件構造化・ペルソナ・検索条件）を生成し、スカウト文面の生成・レビューまで行う。ユーザーが「prep のみ」「準備して」等と言った場合はこのフロー。

1. `.claude/skills/scout-prep/SKILL.md` を Read して prep を実行。過去求人情報があれば併せて渡す
2. prep.yaml の生成を確認（scout_message_base はまだない状態）
3. `index.yaml` が存在すれば、既存候補者の提案を実行（後述）
4. writer → reviewer[段階1] ループを実行（後述）

### operator のみ

既存の prep.yaml（scout_message_base 含む）を前提に、媒体上で検索・リスト操作を行う。候補者別メッセージの段階2レビューを含む。

`.claude/skills/scout-operator/SKILL.md` を Read し、その指示に従って実行する。候補者ごとの scout_message 生成後に段階2レビューを挟む（後述）。

### 全フロー

prep（文面生成含む）→ operator の順に実行する。

1. prep（文面生成含む）フローを実行
2. scout_message_base のレビュー通過を確認
3. operator フローを実行（段階2レビュー含む）

## データの永続化（commit & push）

以下の2つのタイミングで `data/` 配下の変更を commit & push する:
- writer → reviewer ループの段階1 PASS 後（prep.yaml + job-posting-raw.md + scout_message_base をまとめて）
- operator フロー完了後（`<platform>.yaml` + index.yaml をまとめて）

手順:
1. `git add data/`
2. `git commit -m "<job-id>: <変更内容の要約>"`
3. `git push`

コミットメッセージ例:
- `caddi-newbiz-eng-mgr: prep完了・scout_message_base レビュー通過`
- `caddi-newbiz-eng-mgr: YouTrust 候補者処理完了・index再生成`

## サブスキル読み込みルール

各サブスキルの SKILL.md は最初に一度だけ Read し、同一セッション内で再読込しない。

## writer → reviewer ループ

### 段階1: ベーステンプレートレビュー

scout-writer が生成した scout_message_base を scout-reviewer がレビューする。

1. `.claude/skills/scout-writer/SKILL.md` を Read して writer を実行
2. prep.yaml に scout_message_base が追記されたことを確認
3. `.claude/skills/scout-reviewer/SKILL.md` を Read して段階1レビューを実行
4. reviewer の結果が PASS → 次のステップへ
5. reviewer の結果が FAIL → feedback を scout-writer に渡して再生成
6. 最大3回のループ。3回目でも FAIL の場合はユーザーに判断を仰ぐ

### 段階2: 候補者別メッセージレビュー

scout-operator が生成した候補者ごとのカスタマイズ済みスカウト文面を scout-reviewer がレビューする。

1. scout-operator が候補者の scout_message を生成
2. `.claude/skills/scout-reviewer/SKILL.md` を Read して段階2レビューを実行
3. reviewer の結果が PASS → scout-operator にリスト追加を指示
4. reviewer の結果が FAIL → scout-operator に修正指示を渡して再生成
5. PASS するまでループを継続する（上限なし）

## チェックポイント設計

### 再開単位

候補者単位。各候補者の処理完了時に YAML を保存する。

### 中断再開

`data/jobs/<job-id>/<platform>.yaml` の各候補者の `status` と `synced_at` を確認し、未処理の候補者から再開する。

### 重複防止

`user_id` + `job_id` の組み合わせで既存チェック。同じ候補者を同じ求人で二重に処理しない。

### 中間報告

10件処理ごとにユーザーへ中間報告を行う:
- 処理済み件数 / 全体件数
- マッチ度別の内訳
- リストに追加した候補者数
- エラーがあれば報告

## 候補者インデックス再生成

`data/candidates/index.yaml` は正本ではなく派生物。手動更新はしない。以下のタイミングで再生成する:
- operator による候補者追加・削除の完了後
- ユーザーから「インデックスを更新して」と指示があったとき
- 新規求人受領時に既存候補者を提案する前

再生成ロジック:
1. `data/jobs/` 配下の `.yaml` ファイルのうち、`references/platforms/` に対応フォルダが存在する実媒体名（`_template` 等の補助ディレクトリを除く）のファイル（例: `youtrust.yaml`）のみを走査する。`prep.yaml` やその他の補助ファイルは対象外
2. 候補者を `platform` + `user_id` の組み合わせで一意に識別する。同一人物の媒体横断統合は自動では行わず、各 platform_account を独立したエントリとして保持する（同姓同名の誤統合を防ぐため）
3. `platform_accounts` に各媒体のアカウント情報を集約
4. `jobs` に関連した求人一覧を記録
5. `last_seen` は最新の `added_at` を使用

再生成手順:
1. `data/jobs/` 配下の全ディレクトリを走査
2. 各ディレクトリ内の `<platform>.yaml`（`prep.yaml` 以外）を読み込む
3. 候補者ごとに `platform` + `user_id` で一意キーを生成
4. 既存 `index.yaml` があればマージ（同一キーは最新データで上書き）
5. `jobs` フィールドに関連 job-id を追加（重複なし）
6. `last_seen` は全エントリの最新 `added_at` を使用
7. `generated_at` を現在日付に更新し、`data/candidates/index.yaml` に書き出す

### index.yaml のスキーマ

```yaml
schema_version: 1
generated_at: <YYYY-MM-DD>
candidates:
  - name: <氏名>
    platform_accounts:
      - platform: <媒体名>
        user_id: "<媒体上のユーザーID>"
        profile_url: "<プロフィールURL>"
    title: <現職の肩書>
    company: <現職の企業名>
    location: <所在地>
    skills: [<スキル>, ...]
    profile_summary: "<プロフィール要約>"
    jobs: [<job-id>, ...]
    last_seen: <YYYY-MM-DD>
```

## 既存候補者の提案

新規求人受領時、prep 完了後に以下の手順で既存候補者を提案する。

前提条件:
- `index.yaml` が存在し、candidates が1件以上あること
- 存在しない場合はスキップして writer → reviewer ループに進む

手順:
1. `index.yaml` を読み込む
2. 新規求人の必須スキル・職種キーワードを抽出（求人票原文またはユーザー入力から）
3. 各候補者の `skills` と `profile_summary` を照合し、マッチ度を判定:
   - 必須スキルのカバー率（何割一致するか）
   - 職種の関連性（`profile_summary` 内のキーワード一致）
4. マッチ度が高い候補者（必須スキル50%以上カバー）をリストアップ
5. ユーザーに提案:
   - 候補者名、現職、スキル、過去の求人（`jobs`）、各媒体のプロフィールURL
   - 「この候補者を新規求人のリストに追加しますか？」と確認
6. ユーザーが承認した候補者は、operator フローで該当求人の `<platform>.yaml` に追加

提案時の注意:
- 過去に同じ求人で listed 済みの候補者は除外する
- 候補者が複数媒体にいる場合は全 `platform_accounts` を表示

## 長時間稼働対策

- YAML への逐次保存で状態を永続化する
- セッション切れやコンテキスト肥大化時は中間報告してから中断し、再開可能な状態を維持
- Claude のコンテキストが大きくなると応答が不安定になるため、定期的に中間状態を保存する
- 並行実行時は同一の `<platform>.yaml` を同時に書き込まない（求人 x 媒体で排他）

## Gotchas

`.claude/skills/scout-pm/gotchas.md` を参照。運用中に発見した注意事項を蓄積する。
