# time-to-learn

An OpenCode Skill for interactive learning. Manages your Obsidian iLearn vault through conversational dialogue, automatically tracking learning progress and building a knowledge graph.

## Quick Start

```bash
# /ttl        → Pick a random concept and start learning
# /ttl Docker → Learn about a specific concept
```

You focus on asking questions and building understanding. The agent handles **everything else**: writing summaries, updating status, organizing files, and linking knowledge.

## Features

- **Auto-start Obsidian**: Launches Obsidian if not running — no manual setup
- **Smart selection**: Prioritizes half-learned concepts (half status) to reinforce weak points
- **Interactive learning**: AI explains → you question → AI corrects → auto-summarize
- **Fully automated**: Structured knowledge cards written to notes after each session
- **Status tracking**: notyet → half → done, progress at a glance
- **Knowledge graph**: Auto-detects related concepts and creates bidirectional wikilinks

## Installation

### 1. Place the Skill file

Copy `SKILL.md` to OpenCode's global skills directory:

```bash
# Windows
mkdir -p %USERPROFILE%\.config\opencode\skills\time-to-learn
copy SKILL.md %USERPROFILE%\.config\opencode\skills\time-to-learn\SKILL.md

# macOS / Linux
mkdir -p ~/.config/opencode/skills/time-to-learn
cp SKILL.md ~/.config/opencode/skills/time-to-learn/SKILL.md
```

### 2. Register the command

Add to `~/.config/opencode/opencode.json`:

```json
{
  "command": {
    "ttl": {
      "template": "Use the time-to-learn skill. $ARGUMENTS",
      "description": "Interactive learning - start learning from iLearn vault"
    }
  }
}
```

### 3. Prepare Obsidian vault

Ensure Obsidian is installed and a vault exists at `C:\Users\<username>\iLearn` with a `waitTolearn` folder inside.

### 4. Restart OpenCode

Changes take effect after restart.

## Status Model

| Status | Meaning | Location | Selection Priority |
|--------|---------|----------|-------------------|
| `notyet` | Not yet learned | waitTolearn | Low |
| `half` | Partially mastered | waitTolearn | Highest |
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

- [OpenCode](https://opencode.ai)
- [Obsidian](https://obsidian.md) (auto-launched by agent)
- Obsidian CLI (built into Obsidian)
- Dependent skill: `obsidian-cli`

## Summary Format

After each session, the agent appends a structured knowledge card to the note:

```markdown
## Learning Record - 2026-06-20

### Core Concept
<Paraphrased summary, independently readable>

### Key Points
- <Essential understanding>
- <Distinction from similar concepts>
- <Use cases and boundaries>

### Common Misconceptions
| Misconception | Correct Understanding |
|--------------|----------------------|

### Practical Scenarios
- <Real-world examples>

### Related Knowledge
- [[Related Note]] - Description

### Mastery Level
half → done
```

## Project Structure

```
time-to-learn/
  SKILL.md        # Skill definition
  README.md       # This document (Chinese)
  README.en.md    # English version
  design.md       # Design document
  plan.md         # Implementation plan
```

## License

MIT
