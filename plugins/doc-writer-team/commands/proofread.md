---
description: 既存ドキュメントを校閲して文章品質を改善する
argument-hint: <files> [--focus <aspect>]
allowed-tools: ["Agent"]
---

# doc-writer:proofread

既存の Markdown ドキュメントを校閲する。言い回し・誤字・一貫性・読みやすさを改善する。

## 使い方

```
/doc-writer:proofread <files> [--focus <aspect>]
```

## 引数

ユーザーがこのコマンドを呼び出した引数: $ARGUMENTS

`$ARGUMENTS` を解析して以下を抽出する：
- **files**: 校閲対象のファイルパス（スペース区切り、またはディレクトリパス）
- **focus**: 校閲の焦点（任意。例: "誤字のみ"、"一貫性"、"読みやすさ"）

## 指示

editor-in-chief エージェントを以下のプロンプトで使用する：

```
既存ドキュメントを校閲してください（校閲モード）。
対象ファイル: [extracted files]
校閲の焦点: [extracted focus or "全般"]
```

ワークフロー C（校閲）のロジックはすべて editor-in-chief エージェントが処理する。
