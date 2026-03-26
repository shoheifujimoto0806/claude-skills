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

1. prep.yaml から検索条件を読み込む
2. manifest.yaml で媒体のフィルタ項目を確認
3. 共通キーワードを媒体固有フィルタにマッピング
4. まず broad_keywords で検索し母集団の規模を確認
5. 多すぎる場合は narrow_keywords で絞り込む
6. 検索結果0件の場合は条件を緩めて再検索

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
