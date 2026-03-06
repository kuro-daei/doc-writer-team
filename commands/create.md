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

Once topic and output-dir are determined, use the editor-in-chief agent to execute the full workflow:

```
Research the topic and create a document set.
Topic: [extracted topic]
Output directory: [extracted output-dir or "doc/"]
```

The editor-in-chief will coordinate the research, propose a document structure for user approval, write each document, review, revise, and save all files to the output directory.
