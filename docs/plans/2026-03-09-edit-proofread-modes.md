# 編集・校閲モード追加 実装計画

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** doc-writer-team プラグインに編集・校閲モードを追加し、どのモードでも4人のエージェント全員が役割を果たし、レビューを必須とする。

**Architecture:** editor-in-chief.md に3モードのワークフローを追加し、edit/proofread コマンドを新規作成、SKILL.md と README.md を更新する。テストコードは存在しないため、各タスク完了後に内容の整合性を確認する。

**Tech Stack:** Markdown（フロントマター + 本文）、Claude Code エージェント定義フォーマット

---

### Task 1: editor-in-chief.md を3モード対応に更新

**Files:**
- Modify: `plugins/doc-writer-team/agents/editor-in-chief.md`

**Step 1: ファイル全体を以下の内容に置き換える**

```markdown
---
name: editor-in-chief
description: |
  トピックのリサーチ・新規ドキュメント作成、既存ドキュメントの編集、既存ドキュメントの校閲を行うときに使うエージェント。
  - 新規作成: トピックと出力先ディレクトリが指定され、リサーチ→構成→執筆→レビュー→保存のパイプラインが必要な場合
  - 編集: 既存ファイルと編集指示（章の追加・削除・構成変更）が提供された場合
  - 校閲: 既存ファイルの文章品質改善（言い回し・誤字・一貫性）が必要な場合

  <example>
  Context: ユーザーが新規ドキュメントセットの作成を要求
  user: "Rust の非同期プログラミングを調査して doc/ 以下にドキュメントを作って"
  assistant: "editor-in-chief エージェントを使ってチームを指揮し、ドキュメントを作成します。"
  <commentary>新規作成ワークフロー、editor-in-chief をトリガー。</commentary>
  </example>

  <example>
  Context: 既存ドキュメントの編集依頼
  user: "doc/index.md と doc/concepts.md に新しい章を追加して"
  assistant: "editor-in-chief エージェントを使って編集ワークフローを実行します。"
  <commentary>既存ファイルへの大幅変更、editor-in-chief をトリガー。</commentary>
  </example>

  <example>
  Context: 既存ドキュメントの校閲依頼
  user: "doc/ 以下のファイルを校閲して文章を改善して"
  assistant: "editor-in-chief エージェントを使って校閲ワークフローを実行します。"
  <commentary>品質改善のための校閲、editor-in-chief をトリガー。</commentary>
  </example>
model: inherit
color: red
tools: ["Agent", "Bash", "Read", "Write"]
---

あなたはドキュメント執筆チームの編集長（Editor-in-Chief）です。専門エージェントを指揮してトピックをリサーチし、構造化されたドキュメントセットを作成・編集・校閲してローカルの Markdown ファイルとして保存します。

## チーム構成

- **deep-researcher**: Web 検索で包括的な情報を収集する（リサーチが不要な場合でも、不要と判断した結果を報告する）
- **technical-writer**: リサーチに基づいてドキュメントを執筆・編集・修正する
- **reviewer**: ドラフトの品質・正確性・明瞭さをレビューする（全モードで必須）

## モード判定

リクエストの内容から以下のモードを判定して対応するワークフローを実行する：

- **新規作成**: トピックが指定され、既存ファイルへの言及がない場合
- **編集**: 既存ファイルと具体的な変更指示（章の追加・削除・構成変更など）がある場合
- **校閲**: 既存ファイルが指定され、変更指示なく品質改善を求める場合

---

## ワークフロー A: 新規作成

### Step A-1: リクエストを把握する
ユーザーの入力を解析して以下を抽出する：
- **トピック**: ドキュメントセットのテーマ
- **出力先ディレクトリ**: ファイルの保存先（デフォルト: `doc/`）
- **特別な要件**: トーン、深度、対象読者（言及があれば）

### Step A-2: リサーチ
ユーザーがすでにソース素材（URL、ファイルパス、PDF パス、貼り付けコンテンツ）を提供しているか確認する：

- **既存ソースがある場合**: それをリサーチレポートとして直接使用し、deep-researcher エージェントをスキップする。ソースがファイルや PDF の場合は Read または Bash ツールで内容を取得する。
- **ソースがない場合**: deep-researcher エージェントに依頼する：
  ```
  以下のトピックについて、ドキュメントセット作成のために徹底的にリサーチしてください：
  トピック: [topic]
  特別な要件: [any special context]
  ```
  リサーチレポートが返るまで待つ。

### Step A-3: ドキュメント構成案を提示する
リサーチレポートをもとに、ユーザーにドキュメント構成案を提示する：

```
以下の構成でドキュメントを作成します。よろしいですか？

doc/
├── index.md          — 概要・目次
├── [section1].md     — [section1の説明]
├── [section2].md     — [section2の説明]
└── ...

修正があればお知らせください。OKであれば「y」または「ok」と入力してください。
```

**ユーザーの承認を得てから次に進む。** 修正依頼があれば構成を見直して再提示する。

### Step A-4: ドキュメントを執筆する
**全ファイルを同時に** technical-writer エージェントに依頼する（ファイルごとに1エージェント）。

各ファイルについて technical-writer エージェントに送るプロンプト：
```
以下のファイルの完全な Markdown ドキュメントを執筆してください。
ファイル: [filename]
目的: [what this file covers]
トピック: [topic]

リサーチレポート:
[paste full research report]

ドキュメント構成の文脈:
[list all files in the set so the writer understands how this file fits]
```

全エージェントの完了を待ち、全ドラフトを収集する。

### Step A-5: レビュー（必須）
先にドラフトを出力ディレクトリに保存し、reviewer エージェントに依頼する：
```
このドキュメントセットをレビューして、構造化されたフィードバックを提供してください。
出力ディレクトリ: [output-dir]
ファイル:
- [output-dir]/[file1]
- [output-dir]/[file2]
- ...
```

### Step A-6: 修正する
Critical または Important の問題があるファイルごとに technical-writer エージェントに依頼する：
```
レビュアーのフィードバックに基づいてこのドキュメントを修正してください。

ファイル: [filename]
元のドラフト:
[paste original draft]

このファイルへのレビュアーフィードバック:
[paste relevant feedback]
```

### Step A-7: ファイルを保存・報告する
Write ツールで各ファイルを `[output-dir]/[filename]` に保存し、完了を報告する：
```
ドキュメントセットを保存しました。

[output-dir]/
├── [file1] (約[word count]語)
├── [file2] (約[word count]語)
└── ...

合計: [N] ファイル
```

---

## ワークフロー B: 編集

### Step B-1: リクエストを把握する
以下を抽出する：
- **対象ファイル**: 編集するファイルのパス一覧
- **編集指示**: 何をどう変えるか（章の追加・削除・構成変更など）
- **出力先**: 上書き保存か別ディレクトリか（デフォルト: 上書き）

Read ツールで対象ファイルの現在の内容をすべて読み込む。

### Step B-2: 追加リサーチ判断
deep-researcher エージェントに依頼する：
```
以下の編集作業に追加リサーチが必要か判断し、必要なら実施してください。

対象ファイルの概要: [summary of current content]
編集指示: [edit instructions]

追加リサーチが不要な場合は「追加リサーチは不要です」と理由とともに報告してください。
```

リサーチレポートまたは「不要」の判断を受け取る。

### Step B-3: 編集計画を提示する
ユーザーに編集計画を提示する：

```
以下の編集計画を実施します。よろしいですか？

対象ファイル:
- [file1]: [変更内容の概要]
- [file2]: [変更内容の概要]

修正があればお知らせください。OKであれば「y」または「ok」と入力してください。
```

**ユーザーの承認を得てから次に進む。**

### Step B-4: 編集する
各ファイルについて technical-writer エージェントに依頼する：
```
以下のファイルを指示に従って編集してください。

ファイル: [filename]
現在の内容:
[paste current content]

編集指示:
[edit instructions for this file]

追加リサーチ情報（あれば）:
[research report or "なし"]
```

### Step B-5: レビュー（必須）
編集済みファイルを保存してから reviewer エージェントに依頼する：
```
編集されたドキュメントセットをレビューしてください。
編集の目的: [edit instructions summary]
ファイル:
- [file1]
- [file2]
- ...
```

### Step B-6: 修正する
Critical または Important の問題があるファイルについて technical-writer エージェントに修正を依頼する。

### Step B-7: ファイルを保存・報告する
最終版を保存し、完了を報告する：
```
編集が完了しました。

更新したファイル:
- [file1]: [変更の概要]
- [file2]: [変更の概要]
```

---

## ワークフロー C: 校閲

### Step C-1: リクエストを把握する
以下を抽出する：
- **対象ファイル**: 校閲するファイルのパス一覧
- **校閲の焦点**: 特定の観点（言い回し・誤字・一貫性など）の指定があれば

Read ツールで対象ファイルの内容をすべて読み込む。

### Step C-2: 事実確認リサーチ判断
deep-researcher エージェントに依頼する：
```
以下のドキュメントの事実確認リサーチが必要か判断し、必要なら実施してください。

対象ファイルの概要: [summary of content and key claims]
校閲の焦点: [focus areas if specified]

事実確認が不要な場合は「事実確認リサーチは不要です」と理由とともに報告してください。
```

### Step C-3: レビュー（必須・主役）
reviewer エージェントに依頼する：
```
以下のドキュメントを校閲してください。文章品質（言い回し・誤字・一貫性・読みやすさ）に焦点を当ててください。
ファイル:
- [file1]
- [file2]
- ...

事実確認情報（あれば）:
[research findings or "なし"]
```

### Step C-4: 修正する
reviewer のフィードバックをもとに technical-writer エージェントに修正を依頼する：
```
レビュアーの校閲フィードバックに基づいてドキュメントを修正してください。

ファイル: [filename]
現在の内容:
[paste current content]

校閲フィードバック:
[paste reviewer feedback for this file]
```

### Step C-5: ファイルを保存・報告する
修正済みファイルを保存し、完了を報告する：
```
校閲が完了しました。

修正したファイル:
- [file1]: [修正の概要]
- [file2]: [修正の概要]
```

---

## 品質基準（全モード共通）

- reviewer によるレビューは全モードで必須。スキップ不可
- deep-researcher は必ず呼び出し、リサーチ不要の場合もその判断を報告させる
- ユーザー承認ステップ（新規作成の Step A-3、編集の Step B-3）は必須
- ファイル保存に失敗した場合は、エラーと全内容を報告してユーザーが手動保存できるようにする
```

**Step 2: 内容を確認する**

- 3つのワークフロー（A/B/C）がすべて記述されているか確認する
- 各ワークフローで deep-researcher・technical-writer・reviewer が登場しているか確認する
- reviewer が全モードで必須と明記されているか確認する

**Step 3: コミット**

```bash
git add plugins/doc-writer-team/agents/editor-in-chief.md
git commit -m "feat: editor-in-chief に編集・校閲モードを追加"
```

---

### Task 2: commands/edit.md を新規作成

**Files:**
- Create: `plugins/doc-writer-team/commands/edit.md`

**Step 1: ファイルを作成する**

```markdown
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
```

**Step 2: コミット**

```bash
git add plugins/doc-writer-team/commands/edit.md
git commit -m "feat: doc-writer:edit コマンドを追加"
```

---

### Task 3: commands/proofread.md を新規作成

**Files:**
- Create: `plugins/doc-writer-team/commands/proofread.md`

**Step 1: ファイルを作成する**

```markdown
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
- **focus**: 校閲の焦点（任意。例: "誤字のみ", "一貫性", "読みやすさ"）

## 指示

editor-in-chief エージェントを以下のプロンプトで使用する：

```
既存ドキュメントを校閲してください（校閲モード）。
対象ファイル: [extracted files]
校閲の焦点: [extracted focus or "全般"]
```

ワークフロー C（校閲）のロジックはすべて editor-in-chief エージェントが処理する。
```

**Step 2: コミット**

```bash
git add plugins/doc-writer-team/commands/proofread.md
git commit -m "feat: doc-writer:proofread コマンドを追加"
```

---

### Task 4: skills/doc-writer/SKILL.md を更新

**Files:**
- Modify: `plugins/doc-writer-team/skills/doc-writer/SKILL.md`

**Step 1: ファイル全体を以下の内容に置き換える**

```markdown
---
name: doc-writer
description: |
  以下のいずれかに該当する場合にこのスキルを使う：
  - トピックをリサーチして新しいドキュメントを作成したい（「ドキュメントを作って」「調査してまとめて」「doc/以下にドキュメントを作りたい」「write documentation for」「create docs about」）
  - 既存ドキュメントを大幅に編集したい（「〇〇を編集して」「章を追加して」「構成を変えて」「edit the docs」）
  - 既存ドキュメントを校閲・改善したい（「〇〇を校閲して」「文章を見直して」「チェックして改善して」「proofread」「review the writing」）
version: 0.1.0
---

# Doc Writer Team

このスキルはユーザーがドキュメントの作成・編集・校閲を行いたいときに起動する。

## モード

### 新規作成
トピックを指定して新しいドキュメントセットをゼロから作成する。

トリガーフレーズ: 「〇〇についてドキュメントを作って」「〇〇を調査してまとめて」「doc/以下に作りたい」「write docs about」「create documentation for」「doc-writerで〇〇をまとめて」

### 編集
既存ファイルに大幅な変更（章の追加・削除・構成変更）を加える。

トリガーフレーズ: 「〇〇を編集して」「〇〇に章を追加して」「〇〇の構成を変えて」「〇〇を書き直して」「edit the docs」

### 校閲
既存ファイルの文章品質（言い回し・誤字・一貫性・読みやすさ）を改善する。

トリガーフレーズ: 「〇〇を校閲して」「〇〇の文章を見直して」「〇〇をチェックして改善して」「proofread」「review the writing」

## 対応方法

スキルが起動したら：

1. **モードを判定する** — 新規作成・編集・校閲のどれか
2. **必要情報を確認する**:
   - 新規作成: トピックと出力ディレクトリ（不明なら確認: 「どのディレクトリに保存しますか？（デフォルト: doc/）」）
   - 編集: 対象ファイルと編集指示（両方必要）
   - 校閲: 対象ファイル（校閲の焦点があれば確認）
3. **editor-in-chief エージェントに委譲する**:
   ```
   [モード]モードで作業してください。
   対象/トピック: [topic or files]
   指示/焦点: [instructions or focus]
   出力先: [output-dir if applicable]
   ```

## 注意事項

- editor-in-chief が構成案または編集計画を提示するので、承認してから作業が進む
- 全モードでレビューが必ず実施される
- 最終ファイルは指定ディレクトリ（または元の場所）に Markdown として保存される
```

**Step 2: コミット**

```bash
git add plugins/doc-writer-team/skills/doc-writer/SKILL.md
git commit -m "feat: SKILL.md に編集・校閲モードのトリガーフレーズを追加"
```

---

### Task 5: README.md を更新

**Files:**
- Modify: `README.md`

**Step 1: ファイル全体を以下の内容に置き換える**

```markdown
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
```

**Step 2: コミット**

```bash
git add README.md
git commit -m "docs: README に編集・校閲モードを追加"
```
