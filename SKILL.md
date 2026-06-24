---
name: time-to-learn
description: Use when user types /ttl command, /ttl <topic>, or expresses intent to learn a concept from their Obsidian vault — manages interactive learning dialogue, note status (notyet/half/done), file organization, and knowledge graph via wikilinks
---

# time-to-learn

## Overview

Interactive learning skill. User participates only in learning dialogue; agent handles everything else: note selection, explanation, dialogue correction, summary writing, status determination, file moving, knowledge graph maintenance via wikilinks.

**CRITICAL: All user-facing output MUST be in Simplified Chinese.** Explanations, dialogue, summaries, reports — all Chinese. Technical terms may remain in English but explanations must be in Chinese. Notes written to Obsidian MUST be in Chinese.

## Configuration

Before use, fill in these values for your environment:

| Variable | Description | Example |
|----------|-------------|---------|
| `C:\Users\Homan\iLearn` | Local path to Obsidian vault | `C:\Users\<name>\Notes` or `~/Documents/Notes` |
| `iLearn` | Vault name (used as `vault=` in CLI) | `Notes` |
| `waitTolearn` | Folder for pending concepts | `waitTolearn` |

Throughout this document, `C:\Users\Homan\iLearn`, `iLearn`, `waitTolearn` refer to these configurable values.

## Trigger

| Command | Behavior |
|---------|----------|
| `/ttl` | Randomly select a note from `waitTolearn`, **prioritizing `half` status** |
| `/ttl <topic>` | Learn about the specified topic |

## Status Model

| Status | Meaning | Location | Selection Priority |
|--------|---------|----------|-------------------|
| `notyet` | Not yet learned | `waitTolearn` | Low |
| `half` | Partially mastered | `waitTolearn` | Highest |
| `done` | Fully mastered | Domain folder | Not selected |

Status is maintained via Obsidian frontmatter property `status`. Set with `obsidian property:set`, read with `obsidian property:read`.

## Vault Info

- Vault path: configured via `C:\Users\Homan\iLearn`
- All Obsidian CLI commands must use `vault="iLearn"`
- Depends on `obsidian-cli` skill

---

## Workflow

### Stage 0: Cold-start Obsidian (auto)

Before any Obsidian CLI operations, check if Obsidian is running:

```bash
obsidian vault="iLearn" files folder="waitTolearn"
```

If response contains `"The CLI is unable to find Obsidian"`, auto-launch:

```bash
# Windows: URI protocol
Start-Process "obsidian://open?vault=iLearn"

# macOS: open "obsidian://open?vault=iLearn"
# Linux: xdg-open "obsidian://open?vault=iLearn"

# Poll until ready (max 15 seconds)
while ($running -ne $true) {
  Start-Sleep -Seconds 2
  $result = obsidian vault="iLearn" files folder="waitTolearn" 2>&1
  if ($result -notmatch "unable to find") { break }
}
```

Transparent to user — no manual Obsidian launch needed.

### Stage 1: Select Note

**CRITICAL DISTINCTION: with-argument and without-argument are completely different branches.**

```
/ttl without argument ($ARGUMENTS is empty):
  1. obsidian vault="iLearn" files folder="waitTolearn"
  2. Read status property of each file -> prioritize "half", randomly pick among half
  3. If no half, randomly pick from "notyet"
  4. If waitTolearn is empty: inform user "No pending notes in waitTolearn"
  5. This branch NEVER uses any argument value

/ttl <topic> ($ARGUMENTS is non-empty):
  1. MUST directly target <topic>. SKIP half-priority and random selection entirely.
  2. obsidian vault="iLearn" read file="<topic>"
  3. File exists -> proceed to Stage 2
  4. File does NOT exist -> auto-create note with template, then ask user:
     obsidian vault="iLearn" create path="waitTolearn/<topic>.md" content="
     ---
     tags: []
     status: notyet
     ---
     # <topic>
     > [!abstract] One-liner
     > (pending)
     ### Related
     " silent
     After creation: "Created <topic>. Start learning now?" (in Chinese)
```

### Stage 2: Read and Explain

```bash
obsidian vault="iLearn" read file="<note name>"
obsidian vault="iLearn" property:read name="status" file="<note name>"
```

Give a brief explanation (2-4 key points). Do not rely on note content (it may only have a title). Explain from your own knowledge. **Must speak in Chinese.**

### Stage 3: Learning Dialogue

- User asks questions or expresses their understanding
- Correct errors, add details, give examples
- **Track continuously:** correct restatement count, whether substantive questions dry up, conversation round count
- **All dialogue in Chinese**

#### Conversation End Logic

```
End immediately (any one satisfied):
  - User clearly signals: "dong le", "ming bai le", "chabuduo le", "zongjie ba"
  - User correctly restates / self-corrects 2 consecutive times -> high comprehension
  - 2 consecutive rounds with no substantive new questions -> natural exhaustion

Fallback inquiry:
  - Over 6 rounds without end signal -> ask "Ready for a summary?" (in Chinese)
  - User response is vague (e.g. "hai xing") -> follow up with one confirmation question first
```

#### Auto Status Determination

| Dialogue Performance | Determination |
|---------------------|---------------|
| Clear expression, accurate self-correction, no major misunderstandings | `done` |
| Still fuzzy/uncertain, some point not fully clarified | `half` |

**Do NOT ask user for status.** Agent determines autonomously based on dialogue quality.

### Stage 4: Write Learning Summary

**Core principle: The note is an independent, re-readable learning resource.** The user should be able to re-learn from just reading the note. Paraphrase, do not transcribe dialogue. Use casual, conversational tone. Use Obsidian callouts for visual structure. **All note content in Chinese.**

#### 4a. Scale Auto-Detection

Before writing, determine the learning scale from dialogue analysis:

| Scale | Concepts | Template | Triggers |
|-------|----------|----------|----------|
| **S** (Small) | 1-3 | Diary summary (append) | Single concept, quick Q&A, no sub-divisions |
| **M** (Medium) | 4-9 | Sectioned note (overwrite) | Multiple sub-concepts, comparisons, categories |
| **L** (Large) | 10+ | Full structured note with TOC (overwrite) | Systematic learning, commands, comparison tables, broad domain |

Auto-judge criteria:
- Count distinct concepts discussed (each new term/idea = 1 concept)
- Whether user asked for commands, examples, or comparisons
- Whether dialogue formed natural topic clusters (groups of related concepts)

**NOTE:** Scale may escalate across multiple sessions on the same topic. If a note already has S-scale content and a new session adds 3+ concepts (making total ≥ 4), upgrade to M. If total reaches 10+, upgrade to L.

#### 4b. Template: S (Small) — Diary Summary

For quick, focused learning of 1-3 concepts. Append to existing note:

```bash
obsidian vault="iLearn" append file="<note name>" content="
---

## What I Learned - YYYY-MM-DD

> [!abstract]
> <One-sentence takeaway>

> [!warning] What I Had Wrong
> I thought: <old understanding>
> -> Actually: <correct understanding>

> [!example] Where This Applies
> - <One concrete scenario>

### Mastery Level
<half or done>
"
```

#### 4c. Template: M (Medium) — Sectioned Note

For mid-size learning of 4-9 concepts. Overwrite note with organized sections:

```bash
obsidian vault="iLearn" create path="<note path>.md" content="
---
tags: []
status: <status>
---
# <Concept Name>

> [!abstract] One-liner
> <Core summary in 1-2 sentences>

## <Theme 1>

### <Sub-concept A>
<Explanation with callouts as needed>

### <Sub-concept B>
<Explanation with callouts>

## <Theme 2>

### <Sub-concept C>
<Explanation>

> [!warning] 关键误区纠正
> - 以为 X → 实际是 Y

## 核心要点
- <Key takeaway 1>
- <Key takeaway 2>
- <Key takeaway 3>

### Related
" overwrite
```

**Structure rules for M:**
- Group related concepts under `##` theme headings (e.g. `## 基础概念`, `## 进阶操作`)
- Each concept gets `###` heading with 2-4 sentence explanation + callouts
- Use `> [!warning]` for misconceptions, `> [!tip]` for analogies, `> [!example]` for scenarios
- End with `## 核心要点` bullet summary

#### 4d. Template: L (Large) — Full Structured Note

For systematic learning of 10+ concepts. Overwrite note with TOC + full chapters:

```bash
obsidian vault="iLearn" create path="<note path>.md" content="
---
tags: []
status: <status>
---
# <Concept Name>

> [!abstract] One-liner
> <Core summary in 2-3 sentences, covering the whole domain>

## 目录

- [[#章节1|章节1]]
  - [[#子概念A|子概念A]]
  - [[#子概念B|子概念B]]
- [[#章节2|章节2]]
  - [[#子概念C|子概念C]]
- [[#命令速查|命令速查]]

## 章节1

### 子概念A
<Detailed explanation, code blocks, ASCII diagrams>

### 子概念B
<Detailed explanation>

---

## 章节2

### 子概念C
<Explanation with comparison tables>

---

> [!warning] 关键误区纠正
> - Thought X → Actually Y

---

## 命令速查

| 命令 | 作用 |
|------|------|
| \`cmd\` | <what it does> |

### Related
" overwrite
```

**Required elements for L:**
- `## 目录` with `[[#heading|display]]` wikilink format — **NEVER use markdown `[text](#anchor)` format for TOC links.** Obsidian only supports `[[#heading]]` wikilinks for in-note navigation.
- Chapters separated by `---` horizontal rules
- ASCII diagrams where helpful (branch trees, flow arrows, architecture diagrams)
- Comparison tables for contrasting concepts (e.g. merge vs rebase, Git Flow vs GitHub Flow)
- Callouts: `> [!important]` for critical insights, `> [!warning]` for misconceptions, `> [!tip]` for analogies, `> [!example]` for use cases
- End with `## 命令速查` if any CLI commands were discussed in the session
- Code blocks with `bash`, `python`, `gitignore` etc. language tags

#### 4e. First-time note initialization

(Only used when a note is brand new, before first dialogue)

```bash
obsidian vault="iLearn" create path="waitTolearn/<topic>.md" content="
---
tags: []
status: notyet
---

# <concept name>

> [!abstract] One-liner
> (pending)

### Related
"
```

After first dialogue: if scale S → use 4b (append). If scale M or L → use 4c/4d (overwrite) immediately — do NOT append diary summaries before overwriting.

#### 4f. Update related links

Append newly discovered related notes to the top-level `### Related` section:

```bash
obsidian vault="iLearn" append file="<note name>" content="- [[Note A]] - <brief connection>"
```

For L-scale notes, also add bidirectional links from related notes back to this one.

#### 4g. Auto-tag

Add tags property based on the concept's domain:

```bash
obsidian vault="iLearn" property:set name="tags" value="devops, ci-cd" type="list" file="<note name>"
```

**Tag determination:** Based on conversation content, assign 1-3 lowercase English tags. Common domains: `devops`, `backend`, `frontend`, `database`, `testing`, `system`, `network`, `language`, `framework`, `tool`.

### Stage 5: Update Status and Move

```bash
# Update status based on Stage 3 auto-determination
obsidian vault="iLearn" property:set name="status" value="done" file="<note name>"
```

**Only `done` status triggers move:**

```bash
# List top-level vault folders, excluding waitTolearn and system directories
obsidian vault="iLearn" folders

# Match existing folder semantically by note title/content (tolerance 6/10)
# Match found -> obsidian vault="iLearn" move file="<note name>" to="<matching folder>"
# No match -> create new folder then move
```

`half` status **stays in waitTolearn**, prioritized at next `/ttl`.

### Stage 6: Knowledge Graph Maintenance

```bash
# Scan other notes in vault
obsidian vault="iLearn" files

# Search for semantically related notes
obsidian vault="iLearn" search query="<keywords>"

# Bidirectional wikilinks
obsidian vault="iLearn" append file="<note A>" content="\n- [[note B]]"
obsidian vault="iLearn" append file="<note B>" content="\n- [[note A]]"
```

Obsidian's native graph view renders relationships automatically.

### Stage 7: Completion Report

```
Learning summary recorded.  Status: done
Moved to: <domain folder>/
Related: [[Note A]] | [[Note B]]
```
(Output in Chinese)

---

## CLI Quick Reference

All commands must include `vault="iLearn"`.

| Operation | Command |
|-----------|---------|
| List folder contents | `obsidian vault="iLearn" files folder="waitTolearn"` |
| Read note | `obsidian vault="iLearn" read file="name"` |
| Append content | `obsidian vault="iLearn" append file="name" content="..."` |
| Set property | `obsidian vault="iLearn" property:set name="status" value="done" file="name"` |
| Read property | `obsidian vault="iLearn" property:read name="status" file="name"` |
| Move file | `obsidian vault="iLearn" move file="name" to="folder"` |
| List folders | `obsidian vault="iLearn" folders` |
| Search notes | `obsidian vault="iLearn" search query="term"` |
| List all files | `obsidian vault="iLearn" files` |

---

## Prerequisites

- Obsidian installed (agent auto-launches, no manual start needed)
- Vault exists at path specified by `C:\Users\Homan\iLearn`
- Vault contains folder specified by `waitTolearn`

## Edge Cases

| Situation | Handling |
|-----------|----------|
| waitTolearn is empty | Inform user no pending notes |
| Specified note does not exist | Auto-create with template, ask user whether to start learning |
| Obsidian not running | Auto-launch via `obsidian://open?vault=iLearn`, poll until ready |
| Auto-launch fails | Tell user to manually open Obsidian |
| Note is already done (in domain folder) | Inform user it's completed, located in `folder/` |
| User interrupts dialogue mid-session | Do not append summary, do not update status |
| Scale escalation: new session adds concepts pushing total to higher tier | Upgrade to higher template, overwrite entire note |
| L-scale note: TOC links don't jump | Must use `[[#heading|display]]` wikilinks, not `[text](#anchor)` |
| Obsidian CLI move uses cached content (not filesystem changes) | After overwriting via filesystem, use direct file write + delete old path; verify with Read tool |

## Red Flags

- After every `/ttl` dialogue ends, **MUST** write learning summary (append for scale S, overwrite for M/L) — even if dialogue was short
- **MUST** auto-detect learning scale (S/M/L) before writing summary, based on concept count and dialogue depth
- For L-scale notes, TOC links **MUST** use `[[#heading|display]]` wikilink format — **NEVER** use markdown `[text](#anchor)` as Obsidian does not support standard anchor links for in-note navigation
- When scale escalates across sessions (e.g. S→M, M→L), **MUST** upgrade to the appropriate template and overwrite the note completely
- After every session, **MUST** check and establish knowledge relationship wikilinks
- Status update **MUST** be auto-determined by agent (do not ask user), performed after summary
- `done` status **MUST** trigger move to domain folder
- **User only participates in dialogue; never ask user to operate files or set status manually**
- All Obsidian CLI commands **MUST** include `vault="iLearn"`
- `/ttl <topic>` with an argument **MUST** directly target that note; never fall back to half-priority or random selection
- Summary **MUST** be a paraphrased knowledge card, not a dialogue transcript
- **All user-facing output, dialogue, and notes MUST be in Simplified Chinese.** Technical terms may stay in English but explanations must be in Chinese.
