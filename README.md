# notion-writer-team

A Claude Code plugin that uses a team of specialized AI agents to research topics and create structured Markdown document sets.

## Agents

| Agent | Role |
|---|---|
| **Editor-in-Chief** | Orchestrates the team, proposes document structure, saves final files |
| **Deep Researcher** | Gathers comprehensive information via web search |
| **Technical Writer** | Drafts and revises individual documents based on research |
| **Reviewer** | Reviews document set for quality, accuracy, and consistency |

## Workflow

```
User → Editor-in-Chief → Deep Researcher (research)
                       → [proposes doc structure, waits for user approval]
                       → Technical Writer (draft each file)
                       → Reviewer (feedback on full set)
                       → Technical Writer (revise)
                       → Save to doc/ directory
```

## Usage

### Slash Command

```bash
/notion-writer:create <topic> [--output-dir <path>]

# Examples:
/notion-writer:create "Claude Code plugin development"
/notion-writer:create "Rust async programming" --output-dir docs/rust-async/
```

### Conversation

Just tell Claude:
- "〇〇についてドキュメントを作って"
- "〇〇を調査してdoc/以下にまとめて"
- "notion-writerで〇〇のドキュメントセットを作りたい"
- "Write documentation about [topic]"

## How It Works

1. **Research**: Deep Researcher gathers information from the web
2. **Structure proposal**: Editor-in-Chief proposes a file structure for `doc/` — you can modify it before approving
3. **Writing**: Technical Writer creates each Markdown file
4. **Review**: Reviewer checks the full document set
5. **Revision**: Technical Writer addresses feedback
6. **Save**: All files are written to the output directory

## Output

Documents are saved as Markdown files in the specified directory (default: `doc/`):

```
doc/
├── index.md          # Overview and table of contents
├── getting-started.md
├── concepts.md
└── ...
```
