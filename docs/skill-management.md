# スキル管理ガイド

人間向けのドキュメント。Claude は読まなくてよい。

## リポジトリ全体のフォルダ構成

```
CLAUDE.md                  — プロジェクト設定・リファレンス
OBJECTIVE.md               — プロジェクトの完了条件（フェーズ別）
data/                      — 候補者データ（Git 追跡対象）
  jobs/                    — 求人 x 媒体の単位でファイルを格納
  candidates/
    index.yaml             — 派生物: jobs から再生成される候補者インデックス
docs/                      — 人間向けドキュメント
.claude/
  skills/
    scout-pm/              — ユーザー向けスキル（唯一の frontmatter 付き）
    scout-prep/            — サブスキル（prep: 求人要件構造化〜スカウト文面生成）
    scout-operator/        — サブスキル（operator: 媒体上での検索・リスト操作）
    references/            — 複数スキル共有リファレンス
      platforms/
        _template/         — 新媒体追加時の雛形
        youtrust/          — YouTrust 固有の仕様・操作手順
    skill-creator/         — Skill 作成・評価ツール（Anthropic 公式）
```

## スキルの標準フォルダ構成

各スキルフォルダの構成:

```
.claude/skills/<skill-name>/
  SKILL.md                 — スキル本体（ユーザー向けは frontmatter + 本文、サブスキルは本文のみ）
  gotchas.md               — よくある失敗・注意事項（運用しながら追記）
  templates/               — テンプレートファイル（スキルによっては不要）
```

## SKILL.md の書き方

### ユーザー向けスキル（scout-pm）

frontmatter（`---` で囲んだ YAML ブロック）に `description` を記述する。description はスキル発火条件のトリガーとなる。本文にはフロー全体の制御ロジック、ゲートチェック、サブスキル参照、チェックポイント設計などを記載する。

```markdown
---
description: |
  スキルの説明文。どのような依頼で発火するかを記載。
---

# スキル名

本文...
```

### サブスキル（scout-prep, scout-operator）

frontmatter は書かない。本文のみ。scout-pm の SKILL.md 内から `Read` で参照される。

```markdown
# スキル名

本文...
```

## 新媒体の追加手順

1. `.claude/skills/references/platforms/_template/` をコピーして媒体名のフォルダを作成
   ```
   cp -r .claude/skills/references/platforms/_template/ .claude/skills/references/platforms/<新媒体名>/
   ```
2. `manifest.yaml` を媒体の仕様に合わせて編集
3. `platform.md` を媒体の操作手順に合わせて編集
4. `CLAUDE.md` のリポジトリ構成セクションを更新

## 新スキルの追加手順

1. `.claude/skills/<skill-name>/` フォルダを作成
2. `SKILL.md` を作成（サブスキルなら frontmatter なし、ユーザー向けなら frontmatter 付き）
3. `gotchas.md` を作成（初期内容は空でも可）
4. 必要に応じて `templates/` を作成
5. 親スキルの SKILL.md にサブスキル参照（Read パス指示）を追加
6. `CLAUDE.md` の Skills 一覧とリポジトリ構成セクションを更新

## 設計ルール

### PM + サブスキル構成

- ユーザーが直接呼び出すのは PM スキルのみ
- サブスキルは PM の SKILL.md 内から Read で参照される内部モジュール
- サブスキルには frontmatter を書かない（トリガー競合を防ぐ）

### 本文必須

- SKILL.md には必ず本文を記載する。frontmatter だけのスキルは禁止
- 本文にはワークフロー、入出力、参照先パスなど具体的な指示を含める

### Gotchas 運用

- 運用中に発見した注意事項・失敗パターンを gotchas.md に蓄積する
- SKILL.md の末尾で gotchas.md を参照する

### データ設計

- 正本は `data/jobs/<job-id>/` 配下（prep.yaml, <platform>.yaml）
- `data/candidates/index.yaml` は派生物であり、正本から再生成する
- data/ 配下は Git 追跡対象（.gitignore しない）

## 命名規約

- **スキル名**: `scout-*` のケバブケース（例: scout-pm, scout-prep, scout-operator）
- **媒体フォルダ**: 小文字ハイフン区切り（例: youtrust, recruit-direct-scout）
- **job-id**: `<企業名略称>-<ポジション略称>` のケバブケース（例: acme-backend-eng）
