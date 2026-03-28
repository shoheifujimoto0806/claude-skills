# Plus X スカウトスキル群

合同会社Plus X のヘッドハンティング業務を支援する Claude Skills リポジトリ。

## Skills 一覧

- `scout-pm` — スカウト業務を統括するPMスキル（唯一のユーザー向けスキル。frontmatter 付き）
- `scout-prep` — サブスキル: 求人票からスカウト準備（要件構造化・検索条件）を生成
- `scout-writer` — サブスキル: スカウト文面の生成
- `scout-reviewer` — サブスキル: スカウト文面のレビュー（段階1: ベーステンプレート、段階2: 候補者別メッセージ）
- `scout-operator` — サブスキル: 媒体上での候補者検索・リスト管理（ブラウザ操作）

## 開発ルール

- 3行以上の実装・変更時は、作業をできるだけ小さく分割してサブエージェントで並列実装し、メインエージェントはレビューのみ行う

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
CLAUDE.md                          — プロジェクト設定・リファレンス（このファイル）
OBJECTIVE.md                       — プロジェクトの完了条件（フェーズ別）
data/                              — 候補者データ（Git 追跡対象）
  jobs/                            — 求人 x 媒体の単位でファイルを格納
  candidates/
    index.yaml                     — 派生物: jobs から再生成される候補者インデックス
docs/
  skill-management.md              — スキル管理ガイド（人間向け。Claude は読まなくてよい）
.claude/
  skills/
    scout-pm/                      — ユーザー向けスキル（唯一の frontmatter 付き）
      SKILL.md                     — フロー全体のオーケストレータ
      gotchas.md                   — フロー管理の注意事項
    scout-prep/                    — サブスキル（frontmatter なし）
      SKILL.md                     — 準備スキル本文
      gotchas.md                   — よくある失敗・注意事項
      templates/
        job-requirements.md        — 求人要件の構造化テンプレート
        persona.md                 — ターゲットペルソナ定義テンプレート
        search-conditions.md       — 検索条件共通テンプレート
    scout-writer/                  — サブスキル（frontmatter なし）
      SKILL.md                     — スカウト文面生成スキル本文
      gotchas.md                   — 文面生成の注意事項
      templates/
        scout-message.md           — スカウトメール共通テンプレート（scout-prep から移動）
    scout-reviewer/                — サブスキル（frontmatter なし）
      SKILL.md                     — スカウト文面レビュースキル本文
      gotchas.md                   — レビューの注意事項
    scout-operator/                — サブスキル（frontmatter なし）
      SKILL.md                     — 媒体操作スキル本文（検索・リスト管理）
      gotchas.md                   — 媒体操作の注意事項
    references/                    — 複数スキル共有リファレンス
      platforms/
        _template/                 — 新媒体追加時の雛形
          manifest.yaml            — 媒体仕様の構造化データ雛形
          platform.md              — 媒体リファレンスの雛形
        youtrust/                  — YouTrust 固有の仕様・操作手順
          manifest.yaml            — YouTrust 仕様
          platform.md              — YouTrust UI操作手順・注意事項
    skill-creator/                 — Skill作成・評価ツール（Anthropic公式）
```
