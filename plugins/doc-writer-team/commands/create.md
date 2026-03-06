---
description: Research a topic and create a structured document set under a local directory
argument-hint: <topic> [--output-dir <path>]
allowed-tools: ["Agent"]
---

# doc-writer:create

Research a topic and create a set of structured Markdown documents using a team of specialized agents.

## Usage

```
/doc-writer:create <topic> [--output-dir <path>]
```

## Arguments

The user invoked this command with: $ARGUMENTS

Parse `$ARGUMENTS` to extract:
- **topic**: Everything before `--output-dir` (required)
- **output-dir**: The value after `--output-dir` (optional, default: `doc/`)

## Instructions

This command delegates to the `doc-writer` skill. Use the editor-in-chief agent with:

```
Research the topic and create a document set.
Topic: [extracted topic]
Output directory: [extracted output-dir or "doc/"]
```

All workflow logic (research, outline approval, parallel writing, review, save) is handled by the editor-in-chief agent as defined in the doc-writer skill.
