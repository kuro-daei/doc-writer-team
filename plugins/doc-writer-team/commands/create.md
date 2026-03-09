---
description: トピックをリサーチしてローカルディレクトリに構造化されたドキュメントセットを作成する
argument-hint: <topic> [--output-dir <path>]
allowed-tools: ["Agent"]
---

# doc-writer:create

専門エージェントチームを使ってトピックをリサーチし、構造化された Markdown ドキュメントセットを作成する。

## 使い方

```
/doc-writer:create <topic> [--output-dir <path>]
```

## 引数

ユーザーがこのコマンドを呼び出した引数: $ARGUMENTS

`$ARGUMENTS` を解析して以下を抽出する：
- **topic**: `--output-dir` の前にあるすべて（必須）
- **output-dir**: `--output-dir` の後の値（任意、デフォルト: `doc/`）

## 指示

このコマンドは `doc-writer` スキルに委譲する。editor-in-chief エージェントを以下のプロンプトで使用する：

```
トピックをリサーチしてドキュメントセットを作成してください。
トピック: [extracted topic]
出力ディレクトリ: [extracted output-dir or "doc/"]
```

ワークフローのロジック（リサーチ・構成承認・並列執筆・レビュー・保存）はすべて doc-writer スキルで定義された editor-in-chief エージェントが処理する。
