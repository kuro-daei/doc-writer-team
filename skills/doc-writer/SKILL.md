---
name: doc-writer
description: This skill should be used when the user asks to research a topic and create documentation, generate a document set, create markdown files about a topic, use doc-writer, or says phrases like "ドキュメントを作って", "調査してまとめて", "doc/以下にドキュメントを作りたい", "write documentation for", "create docs about", "doc-writerで〇〇をまとめて". Activates the full research-outline-write-review-save pipeline.
version: 0.1.0
---

# Doc Writer Team

This skill activates when the user wants to research a topic and create a structured set of Markdown documents.

## When This Skill Applies

Use this skill when the user:
- Asks to research a topic and create documentation or markdown files
- Mentions "doc-writer" explicitly
- Says phrases like "ドキュメントを作って", "調査してまとめて", "doc/以下に作りたい", "write docs about", "create documentation for"

## How to Respond

When this skill triggers:

1. **Extract the topic** from the user's message.
2. **Determine the output directory** — if not specified, use `doc/` as default. Ask if unclear:
   > "どのディレクトリに保存しますか？（デフォルト: doc/）"
3. **Use the editor-in-chief agent** to execute the full pipeline:
   ```
   Research the topic and create a document set.
   Topic: [topic]
   Output directory: [output-dir]
   ```

## Notes

- The editor-in-chief will first propose a document structure (file list with descriptions) and ask for your approval before writing
- You can modify the proposed structure before approving
- Final documents are saved as Markdown files in the specified directory
