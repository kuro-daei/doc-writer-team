# doc-writer-team

Claude Code プラグイン。専門エージェントチームがトピックをリサーチし、構造化された Markdown ドキュメントセットを生成する。

## プロジェクト構造

```
.claude-plugin/
  marketplace.json     # マーケットプレイスメタデータ（名前: eiji-official）
plugins/
  doc-writer-team/     # プラグインルート
    .claude-plugin/
      plugin.json      # プラグインメタデータ
    agents/
      editor-in-chief.md   # オーケストレーター（チーム全体を指揮）
      deep-researcher.md   # Web検索でリサーチ
      technical-writer.md  # ドキュメント執筆・修正
      reviewer.md          # 品質・正確性レビュー
    commands/
      create.md            # /doc-writer:create スラッシュコマンド
    skills/
      doc-writer/
        SKILL.md           # スキル定義（会話経由のトリガー）
docs/plans/            # 設計ドキュメント
```

## エージェント定義の書き方

エージェントファイル（`plugins/doc-writer-team/agents/*.md`）はフロントマターと本文で構成される：

```yaml
---
name: agent-name
description: |
  いつこのエージェントを使うかの説明。
  <example> タグで使用例を含める。
model: inherit          # 親から継承（または claude-xxx-xxx を指定）
color: red              # UI表示色
tools: ["Agent", "Read", "Write"]  # 使用可能なツール
---
```

- `description` は Claude がエージェント選択に使うため、トリガー条件を明確に書く
- `tools` は最小限に絞る

## コマンド定義の書き方

コマンドファイル（`plugins/doc-writer-team/commands/*.md`）：

```yaml
---
description: コマンドの説明
argument-hint: <arg> [--option <value>]
allowed-tools: ["Agent"]
---
```

- `$ARGUMENTS` で引数を受け取る
- コマンド本文でエージェントへの委譲方法を記述

## スキル定義の書き方

スキルファイル（`plugins/doc-writer-team/skills/<skill-name>/SKILL.md`）：

```yaml
---
name: skill-name
description: トリガー条件の説明（日本語フレーズも含める）
version: 0.1.0
---
```

## ワークフロー

```
user → editor-in-chief
  → deep-researcher（リサーチ）
  → ユーザーに構成案を提示・承認待ち
  → technical-writer（各ファイルを執筆）
  → reviewer（全体レビュー）
  → technical-writer（フィードバック反映）
  → Write ツールでファイル保存
```

## 開発時の注意

- エージェント間のデータ受け渡しは editor-in-chief が担当（リサーチレポート、アウトライン、ドラフト）
- ユーザー承認ステップ（Step 3）は必須。スキップ不可
- デフォルト出力先は `doc/`
