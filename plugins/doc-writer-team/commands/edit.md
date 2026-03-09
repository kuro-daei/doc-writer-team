---
description: 既存ドキュメントを指示に従って編集する（章の追加・削除・構成変更）
argument-hint: <files> <instructions> [--output-dir <path>]
allowed-tools: ["Agent"]
---

# doc-writer:edit

既存の Markdown ドキュメントを編集指示に従って変更する。章の追加・削除・構成変更などの大幅な変更に使う。

## 使い方

```
/doc-writer:edit <files> <instructions> [--output-dir <path>]
```

## 引数

ユーザーがこのコマンドを呼び出した引数: $ARGUMENTS

`$ARGUMENTS` を解析して以下を抽出する：
- **files**: 編集対象のファイルパス（スペース区切り、またはディレクトリパス）
- **instructions**: 編集指示（`--output-dir` の前まで）
- **output-dir**: 保存先（任意。デフォルト: 各ファイルを上書き）

## 指示

editor-in-chief エージェントを以下のプロンプトで使用する：

```
既存ドキュメントを編集してください（編集モード）。
対象ファイル: [extracted files]
編集指示: [extracted instructions]
出力先: [extracted output-dir or "上書き"]
```

ワークフロー B（編集）のロジックはすべて editor-in-chief エージェントが処理する。
