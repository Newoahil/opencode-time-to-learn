# time-to-learn

An interactive learning skill for AI coding assistants that turns your Obsidian vault into a conversation-driven knowledge base. The agent handles everything — note selection, explanation, dialogue, summary writing, status tracking, file organization, and knowledge graph maintenance. You just talk and learn.

## Features

- **Interactive dialogue** — Ask questions, get corrected, build understanding through conversation
- **Smart note selection** — `/ttl` picks from your pending concepts, prioritizing partially-mastered (`half`) topics
- **Auto-status tracking** — Notes move through `notyet → half → done`, auto-determined by dialogue quality
- **Three summary templates** — Adapts to learning scale:

| Scale | Concepts | Template | Description |
|-------|----------|----------|-------------|
| **S** | 1-3 | Diary summary | Quick Q&A, misconceptions, one takeaway |
| **M** | 4-9 | Sectioned note | Theme-organized, callouts, key points |
| **L** | 10+ | Full structured note | TOC with wikilinks, chapters, comparison tables, command reference |

- **Knowledge graph** — Auto-discovers and creates bidirectional wikilinks between related notes
- **Auto-file-organization** — `done` notes automatically move to the right domain folder

## Installation

Place the `time-to-learn` folder in your skills directory (location varies by platform — see your AI coding tool's skill documentation):

```
~/.config/<tool>/skills/time-to-learn/SKILL.md
```

Configure the skill by editing the variables at the top of `SKILL.md`:

| Variable | Description | Example |
|----------|-------------|---------|
| `VAULT_PATH` | Local path to Obsidian vault | `~/Documents/Notes` |
| `VAULT_NAME` | Vault name (for CLI `vault=` param) | `Notes` |
| `LEARN_FOLDER` | Folder for pending concepts | `waitTolearn` |

Ensure you have the `obsidian-cli` skill installed and Obsidian running (skill auto-launches it).

## Usage

| Command | Behavior |
|---------|----------|
| `/ttl` | Pick a random concept from `waitTolearn`, prioritizing `half` status |
| `/ttl <topic>` | Learn about a specific topic (auto-creates note if new) |

## How It Works

```
User types /ttl <topic>
    │
    ▼
Stage 0: Auto-launch Obsidian if needed
    │
    ▼
Stage 1: Find or create note in waitTolearn/
    │
    ▼
Stage 2: Read note, explain concept (your knowledge)
    │
    ▼
Stage 3: Interactive dialogue (correct errors, answer questions)
    │
    ▼
Stage 4: Auto-detect scale (S/M/L) and write summary
    │
    ▼
Stage 5: Update status (half/done) → move if done
    │
    ▼
Stage 6: Build knowledge graph (wikilinks)
    │
    ▼
Stage 7: Completion report
```

## Status Model

| Status | Meaning | Location | Next /ttl priority |
|--------|---------|----------|-------------------|
| `notyet` | Not yet learned | `waitTolearn/` | Low |
| `half` | Partially mastered | `waitTolearn/` | **Highest** |
| `done` | Fully mastered | Domain folder | Not selected |

## Requirements

- An AI coding assistant with skill support (e.g. OpenCode, Claude Code, Copilot CLI)
- Obsidian (auto-launched via `obsidian://open` protocol)
- `obsidian-cli` skill installed

## Edge Cases

- **Obsidian not running**: Auto-launches, polls until ready (15s timeout)
- **waitTolearn empty**: Informs user, no crash
- **Specified topic doesn't exist**: Auto-creates template note
- **Scale escalation**: New sessions on same topic may upgrade S→M→L template
- **Obsidian CLI caching**: Direct filesystem writes used when CLI cache is stale

## License

MIT
