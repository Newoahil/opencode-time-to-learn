# time-to-learn

A skill for interactive learning via AI coding assistants. Manages your Obsidian vault through conversational dialogue, automatically tracking learning progress and building a knowledge graph.

## Quick Start

```bash
# /ttl        → Pick a random concept and start learning
# /ttl Docker → Learn about a specific concept
```

You focus on asking questions and building understanding. The agent handles **everything else**: writing summaries, updating status, organizing files, and linking knowledge.

## Agent One-Click Install

Paste this into your AI coding assistant and it will handle installation:

```
Install the time-to-learn skill for me:
1. Download SKILL.md from https://github.com/Newoahil/time-to-learn
2. Place it in the skills directory for this tool (e.g. ~/.config/opencode/skills/time-to-learn/SKILL.md)
3. Register the /ttl command in your tool's config:
   "command": { "ttl": { "template": "Use the time-to-learn skill. $ARGUMENTS", "description": "Interactive learning" } }
4. Update the config variables at the top of SKILL.md (VAULT_PATH / VAULT_NAME / LEARN_FOLDER) with your values
5. Create the LEARN_FOLDER in your Obsidian vault
```

Restart your tool, then type `/ttl` to start learning.

## Features

- **Auto-start Obsidian**: Launches Obsidian if not running — no manual setup
- **Smart selection**: Prioritizes half-learned concepts (half status) to reinforce weak points
- **Interactive learning**: AI explains → you question → AI corrects → auto-summarize
- **Fully automated**: Structured knowledge cards written to notes after each session
- **Status tracking**: notyet → half → done, progress at a glance
- **Knowledge graph**: Auto-detects related concepts and creates bidirectional wikilinks

## Configuration

After installation, update the variables at the top of `SKILL.md`:

| Variable | Description | Example |
|----------|-------------|---------|
| `VAULT_PATH` | Local vault path | `~/Documents/Notes` |
| `VAULT_NAME` | Vault name (for CLI `vault=` param) | `Notes` |
| `LEARN_FOLDER` | Folder for new concepts | `waitTolearn` |

## Manual Installation

### 1. Place the Skill file

```bash
# Windows
mkdir %USERPROFILE%\.config\<tool>\skills\time-to-learn
copy SKILL.md %USERPROFILE%\.config\<tool>\skills\time-to-learn\

# macOS / Linux
mkdir -p ~/.config/<tool>/skills/time-to-learn
cp SKILL.md ~/.config/<tool>/skills/time-to-learn/
```

### 2. Register the command

Add to your tool's config file:

```json
{
  "command": {
    "ttl": {
      "template": "Use the time-to-learn skill. $ARGUMENTS",
      "description": "Interactive learning"
    }
  }
}
```

### 3. Configure variables

Edit `SKILL.md` — replace VAULT_PATH, VAULT_NAME, and LEARN_FOLDER with your values.

### 4. Prepare your vault

Create the `LEARN_FOLDER` in your Obsidian vault. Add concept notes (one file per concept, with the concept name as the title).

### 5. Restart your tool

Changes take effect after restart.

## Status Model

| Status | Meaning | Location | Selection Priority |
|--------|---------|----------|-------------------|
| `notyet` | Not yet learned | LEARN_FOLDER | Low |
| `half` | Partially mastered | LEARN_FOLDER | Highest |
| `done` | Fully mastered | Domain folder | Not selected |

## Workflow

```
Stage 0  Cold-start Obsidian (auto)
Stage 1  Select note (prioritize half)
Stage 2  Read and explain
Stage 3  Learning dialogue (track comprehension)
Stage 4  Write learning summary (knowledge card)
Stage 5  Update status and move (done → domain folder)
Stage 6  Knowledge relationship maintenance (bidirectional wikilinks)
Stage 7  Completion report
```

## Conversation End Logic

| Trigger | Behavior |
|---------|----------|
| User says "got it" / "summarize" | Enter summary immediately |
| 2 consecutive correct restatements | Auto-detect high comprehension |
| 2 consecutive rounds with no new questions | Natural conversation end |
| Over 6 rounds without signal | Ask "Ready for a summary?" |
| Vague response ("kind of") | Follow-up question first, then decide |

## Prerequisites

- An AI coding assistant with skill support (e.g. OpenCode, Claude Code, Copilot CLI)
- [Obsidian](https://obsidian.md) (auto-launched by agent)

## Summary Format

After each session, the agent appends a structured knowledge card:

```markdown
## What I Learned - YYYY-MM-DD

### In One Sentence
<Simplest explanation, like talking to a friend>

### How I Got There
1. At first I thought...
2. Then I realized...
3. Now I understand that...

### What I Had Wrong

I thought: <old understanding>
→ Actually: <correct understanding>

### Where This Applies
- <Real scenarios>

### Related
- [[Related Note]] - Brief connection

### Mastery Level
half
```

## License

MIT
