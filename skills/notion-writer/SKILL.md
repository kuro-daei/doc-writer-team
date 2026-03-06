---
name: notion-writer
description: This skill should be used when the user asks to write a document to Notion, create a Notion article or page, use notion-writer, or says phrases like "Notionに記事を書いて", "Notionにまとめて", "write to Notion", "create a Notion document", "notion-writerで〇〇を書いて". Activates the full research-write-review-publish pipeline.
version: 0.1.0
---

# Notion Writer Team

This skill activates when the user wants to write a document to Notion.

## When This Skill Applies

Use this skill when the user:
- Asks to write, create, or publish a document to Notion
- Mentions "notion-writer" explicitly
- Says phrases like "Notionに書いて", "Notionに記事を作って", "write to Notion"

## How to Respond

When this skill triggers:

1. **Extract the topic** from the user's message.
2. **Ask for the Notion page ID** if not already provided:
   > "どのNotionページに書きますか？ページIDを教えてください。"
   > (Which Notion page should I write to? Please provide the page ID.)
3. **Use the editor-in-chief agent** to execute the full pipeline:
   ```
   Research and write a document to Notion.
   Topic: [topic]
   Notion Page ID: [page-id]
   ```

## Notes

- The editor-in-chief agent coordinates the full pipeline: Research → Write → Review → Revise → Publish
- The Notion page ID can be found in the page URL (the 32-character string after the page title)
- Example page ID: `a1b2c3d4e5f6...`
