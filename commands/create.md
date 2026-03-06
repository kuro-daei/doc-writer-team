---
description: Research a topic and write a document to Notion using the writing team
argument-hint: <topic> --page-id <notion-page-id>
allowed-tools: ["Agent"]
---

# notion-writer:create

Write a research-backed document to Notion using a team of specialized agents.

## Usage

```
/notion-writer:create <topic> --page-id <notion-page-id>
```

## Arguments

The user invoked this command with: $ARGUMENTS

Parse `$ARGUMENTS` to extract:
- **topic**: Everything before `--page-id` (required)
- **page-id**: The value after `--page-id` (required)

If `--page-id` is missing, ask the user: "Please provide the Notion page ID with `--page-id <id>`."

## Instructions

Once topic and page-id are parsed, use the editor-in-chief agent to execute the full workflow:

```
Research and write a document to Notion.
Topic: [extracted topic]
Notion Page ID: [extracted page-id]
```

The editor-in-chief will coordinate the research, writing, review, revision, and Notion publishing pipeline.
