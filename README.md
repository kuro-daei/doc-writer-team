# doc-writer-team

ドキュメントの作成・編集・校閲を行う Claude Code プラグイン。専門 AI エージェントチームが連携して動作し、どのモードでも4人全員が役割を果たす。

## エージェント

| エージェント | 役割 |
|---|---|
| **Editor-in-Chief** | チームを指揮し、ワークフローを管理し、最終ファイルを保存する |
| **Deep Researcher** | Web 検索で情報収集・事実確認を行う（リサーチ不要の場合はその判断を報告） |
| **Technical Writer** | ドキュメントを執筆・編集・修正する |
| **Reviewer** | ドキュメントの品質・正確性・明瞭さをレビューする（全モードで必須） |

## モード

### 新規作成
トピックをリサーチして新しいドキュメントセットをゼロから作成する。

### 編集
既存ドキュメントに章の追加・削除・構成変更などの大幅な変更を加える。

### 校閲
既存ドキュメントの言い回し・誤字・一貫性・読みやすさを改善する。

## ワークフロー

### 新規作成
```
ユーザー → Editor-in-Chief
  → Deep Researcher（リサーチ）
  → [構成提案・ユーザー承認]
  → Technical Writer（各ファイルを並列執筆）
  → Reviewer（全体レビュー・必須）
  → Technical Writer（修正）
  → 保存
```

### 編集
```
ユーザー → Editor-in-Chief（既存ファイル読み込み）
  → Deep Researcher（追加リサーチ要否を判断・実施）
  → [編集計画・ユーザー承認]
  → Technical Writer（編集）
  → Reviewer（レビュー・必須）
  → Technical Writer（修正）
  → 保存
```

### 校閲
```
ユーザー → Editor-in-Chief（既存ファイル読み込み）
  → Deep Researcher（事実確認リサーチ要否を判断・実施）
  → Reviewer（品質レビュー・必須・主役）
  → Technical Writer（修正）
  → 保存
```

## 使い方

### スラッシュコマンド

```bash
# 新規作成
/doc-writer:create <topic> [--output-dir <path>]

# 編集
/doc-writer:edit <files> <instructions> [--output-dir <path>]

# 校閲
/doc-writer:proofread <files> [--focus <aspect>]

# 例:
/doc-writer:create "Claude Code プラグイン開発"
/doc-writer:create "Rust の非同期プログラミング" --output-dir docs/rust-async/
/doc-writer:edit doc/index.md "導入セクションを追加して"
/doc-writer:edit "doc/index.md doc/concepts.md" "全体の構成を見直して概念の章を先に持ってきて"
/doc-writer:proofread doc/
/doc-writer:proofread doc/index.md --focus "誤字と一貫性"
```

### 会話

Claude に話しかけるだけでOK：
- 新規作成: 「〇〇についてドキュメントを作って」「〇〇を調査して doc/ 以下にまとめて」
- 編集: 「doc/index.md に導入章を追加して」「〇〇の構成を変えて」
- 校閲: 「doc/ 以下を校閲して」「この文章を見直して改善して」

## 出力

ドキュメントは指定ディレクトリ（デフォルト: `doc/`）に Markdown ファイルとして保存される：

```
doc/
├── index.md
├── getting-started.md
├── concepts.md
└── ...
```
