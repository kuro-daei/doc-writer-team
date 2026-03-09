# 設計: 編集・校閲モード追加

## 概要

doc-writer-team プラグインを新規作成専用から、編集・校閲にも対応した汎用ドキュメント作業ツールに拡張する。どのモードでも4人のエージェント全員が役割を果たし、レビューは必須とする。

## モード定義

| モード | トリガー | 対象 | 主な変更 |
|---|---|---|---|
| **新規作成 (create)** | トピックを指定 | 新規ファイル | ゼロから生成 |
| **編集 (edit)** | 既存ファイル + 編集指示 | 既存ドキュメント | 章追加・削除・構成変更 |
| **校閲 (proofread)** | 既存ファイルのみ | 既存ドキュメント | 言い回し・誤字・一貫性改善 |

- 編集はユーザーが「何を変えるか」を指示する
- 校閲はエージェントが自律的に問題を発見して改善する

## 各モードでの4人の役割

### 新規作成
| ステップ | エージェント | 作業 |
|---|---|---|
| 1 | editor-in-chief | リクエスト把握 |
| 2 | deep-researcher | トピックをリサーチ |
| 3 | editor-in-chief | 構成提案・ユーザー承認待ち |
| 4 | technical-writer | 全ファイルを並列執筆 |
| 5 | reviewer | 全体レビュー（必須） |
| 6 | technical-writer | 指摘を修正 |
| 7 | editor-in-chief | 保存・完了報告 |

### 編集
| ステップ | エージェント | 作業 |
|---|---|---|
| 1 | editor-in-chief | 既存ファイル読み込み・編集指示を把握 |
| 2 | deep-researcher | 追加リサーチが必要か判断し、必要なら実施・不要なら「不要」と報告 |
| 3 | editor-in-chief | 編集計画をユーザーに提示・承認待ち |
| 4 | technical-writer | 編集実施 |
| 5 | reviewer | 編集後の全体レビュー（必須） |
| 6 | technical-writer | 指摘を修正 |
| 7 | editor-in-chief | 保存・完了報告 |

### 校閲
| ステップ | エージェント | 作業 |
|---|---|---|
| 1 | editor-in-chief | 既存ファイル読み込み・スコープ把握 |
| 2 | deep-researcher | 事実確認が必要か判断し、必要なら実施 |
| 3 | reviewer | 品質レビュー（必須） |
| 4 | technical-writer | 指摘を修正 |
| 5 | editor-in-chief | 保存・完了報告 |

## コマンド構成

```
/doc-writer:create <topic> [--output-dir <path>]
/doc-writer:edit --files <files> --instructions <instructions> [--output-dir <path>]
/doc-writer:proofread <files> [--focus <aspect>]
```

## 変更ファイル

| ファイル | 変更種別 | 内容 |
|---|---|---|
| `agents/editor-in-chief.md` | 更新 | 3モードのワークフロー追加 |
| `commands/edit.md` | 新規 | 編集コマンド定義 |
| `commands/proofread.md` | 新規 | 校閲コマンド定義 |
| `skills/doc-writer/SKILL.md` | 更新 | 編集・校閲のトリガーフレーズ追加 |
| `README.md` | 更新 | 使い方の更新 |
