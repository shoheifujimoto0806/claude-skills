# Plus X スカウトスキル群

合同会社Plus X のヘッドハンティング業務を支援する Claude Skills リポジトリ。

## Skills 一覧

- `scout-prep` — 求人票からスカウト準備（検索条件・スカウトメール）を一括生成
- `scout-pm` — スカウト準備全体を統括するPMスキル（候補者プール管理含む）

## Claude Skills 公式ドキュメント

Skill の作成・改善時は以下を参照すること。

- [Extend Claude with skills](https://docs.anthropic.com/en/docs/claude-code/slash-commands) — Skills の作成・管理方法（メインドキュメント）
- [Equipping agents for the real world with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) — エンジニアリングブログ（設計思想・ベストプラクティス）
- [Introducing Agent Skills](https://www.anthropic.com/news/skills) — Skills 機能のアナウンス
- [The Complete Guide to Building Skills for Claude (PDF)](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf) — Skills 構築ガイド
- [Lessons from Building Claude Code: How We Use Skills](https://x.com/trq212/status/2033949937936085378) — Anthropic Thariq 氏によるスキル活用の実践知見（カテゴリ分類・作成Tips・配布方法）

## リポジトリ構成

ファイルやフォルダを追加・削除・移動したときは、このセクションを必ず更新すること。

```
CLAUDE.md                  — プロジェクト設定・リファレンス（このファイル）
OBJECTIVE.md               — プロジェクトの完了条件（フェーズ別）
skills/
  scout-prep/SKILL.md      — スカウト準備スキル
  scout-pm/SKILL.md        — スカウトPMスキル
.claude/
  skills/
    skill-creator/         — Skill作成・評価ツール（Anthropic公式）
```
