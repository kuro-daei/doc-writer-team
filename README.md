# notion-writer-team

A Claude Code plugin that uses a team of specialized AI agents to research topics and write high-quality documents to Notion.

## Agents

| Agent | Role |
|---|---|
| **Editor-in-Chief** | Orchestrates the team, makes editorial decisions, publishes to Notion |
| **Deep Researcher** | Gathers comprehensive information via web search |
| **Technical Writer** | Drafts and revises documents based on research |
| **Reviewer** | Reviews drafts for quality, accuracy, and clarity |

## Workflow

```
User → Editor-in-Chief → Deep Researcher (research)
                       → Technical Writer (draft)
                       → Reviewer (feedback)
                       → Technical Writer (revise)
                       → Notion (publish)
```

## Usage

### Slash Command

```bash
/notion-writer:create <topic> --page-id <notion-page-id>

# Example:
/notion-writer:create "Claude Code plugin development guide" --page-id abc123def456
```

### Conversation

Just tell Claude:
- "Notionに〇〇について記事を書いて"
- "Write an article about [topic] to Notion page [page-id]"
- "notion-writerで〇〇をまとめて"

## Finding Your Notion Page ID

The page ID is the 32-character string in the Notion page URL:
```
https://notion.so/My-Page-Title-a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
```

## Requirements

- Notion MCP must be connected (available as `mcp__claude_ai_Notion__*`)
