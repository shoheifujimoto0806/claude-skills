# Scout Operator — 媒体上での候補者検索・リスト管理

このスキルは scout-pm から呼び出されるサブスキル。単体では発火しない。

媒体上の全操作を担当する: 検索、プロフィール確認、リスト作成、リスト追加/削除。ブラウザ操作（claude-in-chrome）を使用する。

## ブラウザ操作

Chrome 拡張（claude-in-chrome）を使用する。ログイン済みブラウザが前提。

## 媒体リファレンスの参照

操作対象の媒体が決まったら、以下の手順でリファレンスを読み込む:

1. `.claude/skills/references/platforms/` を `ls` で確認し対象媒体フォルダを特定
2. `.claude/skills/references/platforms/<platform>/manifest.yaml` を Read して仕様確認
3. `.claude/skills/references/platforms/<platform>/platform.md` を Read して操作手順確認

manifest.yaml には媒体の利用可能アクション、制約（文字数上限、検索フィルタ等）が構造化されている。platform.md には UI 操作の具体的な手順が記載されている。

## 入力

`data/jobs/<job-id>/prep.yaml` の検索条件（共通キーワード）を読み込む。

prep.yaml の `search_conditions` には媒体非依存の共通キーワードが格納されている:
- `keywords`: メインの検索キーワード
- `broad_keywords`: 広め条件用
- `narrow_keywords`: 絞り込み条件用

これらを manifest.yaml の `constraints.search_filters` に基づいて媒体固有フィルタにマッピングする。

## 検索フロー

### ステップ1: 検索条件の準備

prep.yaml の `search_conditions` と manifest.yaml の `constraints.search_filters` を読み、以下を決定する:
- 使用するフリーワード（broad → narrow の順で試す）
- 設定するフィルタ項目（platform.md のフィルタ操作手順を参照）

### ステップ2: フィルタの設定

platform.md の「フィルタ操作」セクションに従い、ブラウザ上でフィルタを設定する:
1. 転職意欲フィルタ: 「まったく考えていない」を除外する設定を推奨
2. 職種フィルタ: prep.yaml のペルソナ target_titles に基づいて設定
3. フリーワード: まず broad_keywords で入力
4. その他フィルタ: 求人要件に応じて（希望の働き方、希望の働く場所等）

各フィルタの具体的な操作方法は platform.md を参照すること。

### ステップ3: 母集団の確認と絞り込み

1. 検索実行後、ヒット件数を確認
2. 1,000件以上: フィルタを追加するか narrow_keywords に切り替え
3. 100〜999件: 適切な母集団。プロフィール確認に進む
4. 10〜99件: やや少ないが進行可能
5. 0件: フィルタを緩めて再検索（転職意欲を「すべて」に、フリーワードを減らす等）

### ステップ4: 検索条件の記録

使用したフリーワードとフィルタ設定を `<platform>.yaml` の `source_query` と `search_filters_used` に記録する。再現性のために、フリーワードだけでなく主要なフィルタ設定も含める。

## プロフィール確認とマッチ度評価

検索結果から候補者のプロフィールを確認し、以下の手順でマッチ度を5段階で評価する。

1. prep.yaml の `evaluation_framework` を読み込む
   - `score_criteria`: スコア別（1〜5）の判定基準を確認
   - `job_specific_criteria`: この求人固有の判断基準を確認
   - `bonus_factors`: 加点要素を確認
   - `attention_point_rules`: 着目ポイント生成ルールを確認
2. 候補者のプロフィールを `score_criteria` と `job_specific_criteria` に照らしてベーススコアを決定
3. `bonus_factors` に該当する要素があれば加点を適用
4. `attention_point_rules` に基づいて、スカウト文カスタマイズ用の着目ポイントを生成

- マッチ度4以上の候補者: リストに追加し、着目ポイントを作成してスカウト文をカスタマイズ
- マッチ度3以下: スキップするか、判断を scout-pm に委ねる

## 候補者データの保存

候補者データは `data/jobs/<job-id>/<platform>.yaml` に追記する。

### スキーマ

```yaml
schema_version: 1
job_id: <job-id>
platform: <platform>
list_id: "<媒体上のリストID>"
list_name: "<リスト名>"
source_query: "<実際に使用した検索クエリ>"
search_filters_used:             # 実際に使用したフィルタ設定
  転職意欲: "<設定値>"
  職種: "<設定値>"
  フリーワード: "<検索ワード>"
hit_count: <検索ヒット件数>
updated_at: <YYYY-MM-DD>
candidates:
  - user_id: "<媒体上のユーザーID>"
    name: <氏名>
    profile_url: "<プロフィールURL>"
    title: <現職の肩書>
    company: <現職の企業名>
    location: <所在地>
    skills: [<スキル>, ...]
    profile_summary: "<プロフィール要約>"
    match_score: <1-5>
    match_notes: "<マッチ理由>"
    scout_message: |
      <カスタマイズされたスカウト文面>
    status: draft              # draft / listed / removed
    added_at: <YYYY-MM-DD>
    synced_at: null             # 媒体上のリストに反映した日時
    attempt_count: 0
    last_error: null
    run_id: null                # scout-pm セッションの識別子。中断再開時にどのセッションで処理されたかを追跡する。形式: YYYY-MM-DD-HHmm
```

### status の遷移

- **draft**: プロフィール確認済み、YAML に記録済み
- **listed**: 媒体上のリストに追加済み（synced_at を記録）
- **removed**: リストから削除済み

## リスト管理

### リスト作成

- 最初の候補者を追加する際にリストを新規作成
- リスト名は求人のポジション名で作成する
- list_id と list_name を `<platform>.yaml` に記録

### 候補者追加

- マッチ度4以上の候補者を優先的にリストに追加
- 追加後に status を `listed` に更新し synced_at を記録
- 追加のたびに YAML を保存（チェックポイント）

### 候補者削除

- 不要な候補者をリストから削除
- status を `removed` に更新

## セッション管理

- セッション切れの検知: ログインページへのリダイレクトで判断
- セッション切れ時: ユーザーに再ログインを依頼し、完了を確認してから操作を再開

## エラーハンドリング

- **リトライ可能なエラー**（ネットワークタイムアウト、一時的なUI応答なし等）: 3回まで自動リトライ。attempt_count で追跡
- **リトライ不可能なエラー**（権限不足、候補者ページ削除等）: last_error に記録し、ユーザーに報告して次の候補者に進む
- すべてのエラーは YAML に記録して永続化する

## チェックポイント

- 各候補者の処理完了時に YAML を保存
- 中断再開: `<platform>.yaml` の各候補者の `status` と `synced_at` を確認し、未処理の候補者から再開
- 重複防止: `user_id` + `job_id` の組み合わせで既存チェック

## Gotchas

`.claude/skills/scout-operator/gotchas.md` を参照。運用中に発見した注意事項を蓄積する。
