# Scout Writer — スカウト文面の生成

このスキルは scout-pm から呼び出されるサブスキル。単体では発火しない。

## 役割

prep.yaml の求人要件・ペルソナ・評価フレームワークを読み込み、スカウトメール共通文面（scout_message_base）を生成して prep.yaml に追記する。

## 入力

`data/jobs/<job-id>/prep.yaml` から以下を読み込む:
- `requirements`: 求人要件（title, must_have, nice_to_have, salary_range, notes）
- `personas`: ターゲットペルソナ（background, career_motivation）
- `evaluation_framework`: 候補者評価フレームワーク（attention_point_rules）

## 出力

prep.yaml に `scout_message_base` フィールドを追記（既存の場合は上書き）する。

## 絶対に守るルール

`.claude/skills/references/confidentiality-rules.md` を参照し、すべてのルールを適用する。

加えて、以下の文面固有ルール:
- `{候補者名}` のようなプレースホルダーを使い、パーソナライズすべき箇所を明示する
- 着目ポイント `{着目ポイント}` はプレースホルダーとして記載し、実際のリサーチ時に候補者ごとに作成する

テンプレート: `.claude/skills/scout-writer/templates/scout-message.md` を参照。

## 修正モード

scout-reviewer から修正指示（feedback）を受け取った場合に使用する。

1. prep.yaml の現在の `scout_message_base` を読み込む
2. feedback の内容に従い、問題箇所を修正する
3. 修正後の `scout_message_base` で prep.yaml を上書きする

修正時も「絶対に守るルール」はすべて適用する。

## Gotchas

`.claude/skills/scout-writer/gotchas.md` を参照。運用中に発見した注意事項を蓄積する。
