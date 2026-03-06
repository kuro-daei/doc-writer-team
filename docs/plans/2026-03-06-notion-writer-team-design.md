# notion-writer-team Plugin Design

Date: 2026-03-06

## Overview

A Claude Code plugin that uses a team of 4 specialized agents to research topics and write documents to Notion. The Editor-in-Chief orchestrates the other agents in a hub-and-spoke architecture.

## Agents

| Agent | Tools | Responsibility |
|---|---|---|
| Editor-in-Chief | Agent, Notion MCP | Orchestration, quality control, Notion output |
| Deep Researcher | WebSearch, WebFetch | Topic research, information gathering, source organization |
| Technical Writer | Read, Write | Document drafting based on research |
| Reviewer | Read | Review for accuracy, structure, and readability |

## Plugin Structure

```
notion-writer-team/
├── .claude-plugin/
│   └── plugin.json
├── agents/
│   ├── editor-in-chief.md
│   ├── deep-researcher.md
│   ├── technical-writer.md
│   └── reviewer.md
├── commands/
│   └── create.md
├── skills/
│   └── notion-writer/
│       └── SKILL.md
└── README.md
```

## Workflow

1. User triggers via `/notion-writer:create <topic> --page-id <notion-page-id>` or conversation
2. Editor-in-Chief receives the request
3. Editor-in-Chief dispatches Deep Researcher to gather information
4. Editor-in-Chief dispatches Technical Writer to draft the document (with research)
5. Editor-in-Chief dispatches Reviewer to review the draft
6. Editor-in-Chief dispatches Technical Writer to revise based on feedback
7. Editor-in-Chief writes the final document to Notion via Notion MCP
8. Editor-in-Chief notifies the user of completion

## Trigger

### Slash Command
```bash
/notion-writer:create <topic> --page-id <notion-page-id>
```

### Skill (Conversation Trigger)
- "Notion に〇〇について書いて"
- "〇〇の記事を Notion に作成して"
- "notion-writer で〇〇をまとめて"

## Technical Notes

- Notion MCP is already available in the user's environment (`mcp__claude_ai_Notion__*`)
- Each agent operates independently; the Editor-in-Chief passes artifacts between them
- Reviewer feedback triggers one revision cycle (no infinite loop)
- Document format is determined by agents based on topic (no fixed template)
- Notion destination page is specified by the user each time
