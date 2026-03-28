# Scout Reviewer — スカウト文面のレビュー

このスキルは scout-pm から呼び出されるサブスキル。単体では発火しない。

## 役割

スカウト文面をレビューし、PASS/FAIL を判定する。段階1（ベーステンプレート）と段階2（候補者別メッセージ）の2つのモードを持つ。

## 段階1: ベーステンプレートレビュー

scout-writer が生成した scout_message_base をレビューする。

### 入力

`data/jobs/<job-id>/prep.yaml` から以下を読み込む:
- `requirements`: 求人要件（title, must_have, nice_to_have, salary_range, notes）
- `scout_message_base`: レビュー対象の文面

`data/jobs/<job-id>/job-posting-raw.md`: 求人票の原文（事実の正確性チェックの正本。ファイルが存在しない場合は requirements のみで照合する）

prep.yaml の他のフィールド（personas, evaluation_framework, search_conditions）は段階1では不要。

### レビュー基準

| 基準 | チェック内容 |
|------|------------|
| **事実の正確性** | 求人票原文（job-posting-raw.md）に記載のない事実が文面に含まれていないか。原文が正本。存在しない場合は requirements で代替 |
| **求人内容との整合性** | ポジション説明・年収レンジ・業務内容が requirements と矛盾しないか |
| **日本語の自然さ** | 敬語の重複、不自然な接続、冗長な表現がないか |

プレースホルダー（`{着目ポイント}`, `{候補者名}`, `{日程調整セクション}`）はレビュー対象外。

## 段階2: 候補者別メッセージレビュー

scout-operator が生成した候補者ごとのカスタマイズ済みスカウト文面をレビューする。

### 入力

- `data/jobs/<job-id>/prep.yaml` の `requirements`: 求人要件
  （prep.yaml の他のフィールドは段階2では不要）
- `data/jobs/<job-id>/job-posting-raw.md`: 求人票の原文（求人内容との整合性チェック用。存在しない場合は requirements のみ）
- `data/jobs/<job-id>/<platform>.yaml` の候補者エントリから以下のフィールドをプロフィールとして参照:
  - `profile_summary`: プロフィール要約
  - `skills`: スキル一覧
  - `title`: 現職の肩書
  - `company`: 現職の企業名
  - `match_notes`: マッチ理由（プロフィール照合時に、着目ポイントの根拠がプロフィールに存在するかの補助情報として使用）
- `scout_message`: レビュー対象のカスタマイズ済み文面

### レビュー基準

| 基準 | チェック内容 |
|------|------------|
| **プロフィール照合** | 着目ポイントに候補者プロフィールに記載のない経歴・スキルが含まれていないか |
| **求人内容との整合性** | カスタマイズ後の文面全体が求人内容と矛盾しないか |
| **日本語の自然さ** | 着目ポイント埋込後の文面が自然に読めるか |

## 出力形式

呼び出し元（scout-pm）への返却。ファイルには保存しない。

```yaml
review_result:
  stage: 1 | 2           # 1: ベーステンプレート, 2: 候補者別メッセージ
  verdict: PASS | FAIL
  criteria:
    factual_accuracy: PASS | FAIL      # 段階1のみ
    profile_accuracy: PASS | FAIL      # 段階2のみ
    requirement_alignment: PASS | FAIL
    japanese_naturalness: PASS | FAIL
  feedback: "<FAIL時の修正指示。verdict が PASS の場合は null>"
```

### feedback の書き方

FAIL 時の feedback は、修正担当スキル（段階1: scout-writer、段階2: scout-operator）が即座に修正できるよう、以下を含める:
- **問題箇所の引用**: 文面のどの部分が問題か
- **問題の理由**: どのレビュー基準に違反しているか
- **修正方針**: どう直すべきかの具体的な指示

## Gotchas

`.claude/skills/scout-reviewer/gotchas.md` を参照。運用中に発見した注意事項を蓄積する。
