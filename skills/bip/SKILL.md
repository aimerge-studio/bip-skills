---
name: bip
description: |
  Build in Public session logger. Writes a session summary to your vault
  and generates social media posts for multiple platforms.
  Use when the user says "bip", "log this session", "build in public", or "what did we do".
argument-hint: "[posts|log|full] [optional: topic override]"
allowed-tools:
  - Bash
  - Read
  - Write
---

# Build in Public

**Announce at start:** "Logging session..."

## Configuration

<!-- CUSTOMIZE: Fill these in before using the skill -->

| Setting | Value |
| ------- | ----- |
| Your name | YOUR_NAME |
| Your bio | e.g., "founder building AI products" |
| Vault path | ~/Documents/Obsidian Vault |

### Platforms

<!-- CUSTOMIZE: Set the language for each platform. Style rules stay fixed per platform. -->

| Platform | Language | Style |
| -------- | -------- | ----- |
| Twitter/X | English | Tight, punchy, hook-first |
| Threads | English | Conversational, casual |
| LinkedIn | English | Narrative, professional but human |

### Project lookup table

<!-- CUSTOMIZE: Map your repo directory names to human-readable display names -->

| Slug | Display Name |
| ---- | ------------ |
| my-saas | My SaaS App |
| client-portal | Client Portal |
| api-backend | API Backend |

For unknown slugs, use the slug as-is with first letter capitalized.

---

## Step 1: Parse arguments

- `/bip` or `/bip full` -- write log + generate posts
- `/bip posts` -- posts only, no log file
- `/bip log` -- log only, no posts
- `/bip posts "topic"` -- posts about a specific topic (ignore session context, use the quoted topic)

## Step 2: Auto-detect context

Run these commands to understand what happened this session:

```bash
_GIT_ROOT=$(git rev-parse --show-toplevel 2>/dev/null || pwd)
_GIT_SLUG=$(basename "$_GIT_ROOT")
_PWD_SLUG=$(basename "$(pwd)")
echo "GIT_ROOT: $_GIT_ROOT"
echo "GIT_SLUG: $_GIT_SLUG"
echo "PWD_SLUG: $_PWD_SLUG"
git log --oneline -10 2>/dev/null || echo "NO_GIT"
echo "---DIFF_STAT---"
git diff --stat HEAD~3..HEAD 2>/dev/null || echo "NO_DIFF"
```

**Slug resolution:** Check both `_PWD_SLUG` and `_GIT_SLUG` against the project lookup table in the Configuration section above. If `_PWD_SLUG` matches a known project, use it. Otherwise if `_GIT_SLUG` matches, use that. If neither matches, use `_PWD_SLUG` as-is with first letter capitalized.

Check if the project's CLAUDE.md has a `## Build in Public` section with `audience:` or `tone_note:` overrides. If present, use those to shape the posts. If absent, use defaults.

## Step 3: Generate session summary

Synthesize from **both** the conversation context and git data. The conversation context is primary -- you know what was discussed, what trade-offs were considered, what was hard. Git data supplements.

Write these sections in founder journal style (specific, honest, not a mechanical git log):

1. **What was built / changed** -- concrete. What exists now that didn't before.
2. **Key decisions made** -- the "why" behind choices. What was considered, what was picked.
3. **Problems hit & how solved** -- real friction. What broke, what was unclear.
4. **Open / next** -- what's unfinished, what the next session should tackle.

If the session was thin (no real work done, just exploration or chat), say so honestly:

> "Not much concrete output this session -- mostly exploration. Want me to log it anyway?"

## Step 4: Write session log (skip if `posts` mode)

Target path: `{Vault path}/Build Log/YYYY-MM-DD-<project-slug>.md`

Read the **Vault path** from the Configuration section above and use it for `_VAULT` below. Expand `~` to `$HOME`.

```bash
# Use the Vault path from the Configuration section above
_VAULT="<vault path from config>"
mkdir -p "$_VAULT/Build Log/"
ls "$_VAULT/Build Log/$(date +%Y-%m-%d)"-*.md 2>/dev/null || echo "NO_EXISTING"
```

If a file already exists for today + this project, **read it first** and append a new `## Session N` block at the end. Count existing `## Session` headers to determine N. Never overwrite.

### File format (new file)

```markdown
---
date: YYYY-MM-DD
project: <Display Name>
tags: [build-log, <project-slug>]
status: in-progress
---

# Session: <short descriptive title>

## What was built / changed
<content>

## Key decisions made
<content>

## Problems hit & how solved
<content>

## Open / next
<content>

---

## Repurpose block
<2-4 sentence distillation: what was built, what was hard, what was figured out. Specific, honest. No fluff.>
```

### Appending to existing file

```markdown

---

# Session 2: <short descriptive title>

## What was built / changed
<content>

## Key decisions made
<content>

## Problems hit & how solved
<content>

## Open / next
<content>

---

## Repurpose block
<2-4 sentence distillation>
```

## Step 5: Generate social posts (skip if `log` mode)

You ARE the ghostwriter. Generate all three posts directly in the conversation output.

### Voice

You are ghostwriting for {Your name}, {Your bio} (from the Configuration section above). They build in public to attract clients and collaborators.

- Raw, honest, direct. No corporate speak, no fluff.
- Shows the messy reality of building, not just wins.
- Confident but not arrogant. "I built" not "leveraging synergies".
- Uses specific details -- tools, decisions, problems -- not vague generalities.
- Naturally technical but not gatekeeping. Explains enough for non-devs to get the point.

### Adaptive length

Choose based on the content:

- Sharp insight or quick win -> **SHORT** (punchy, 1-2 sentences or a tight thread starter)
- Feature or build -> **MEDIUM** (what + why + what's next)
- Learning, failure, or system -> **LONG** (context, problem, solution, lesson)

### Platform rules

Read the language for each platform from the Platforms table in the Configuration section above.

**Twitter/X ({language from config}):**

- Tight. Can be a single tweet OR a numbered thread (1/ 2/ 3/).
- Hook in the first line. Max 2 hashtags if truly relevant. Usually zero.
- Casual but sharp.

**Threads ({language from config}):**

- Write NATIVELY in the configured language. Not translated from another language.
- Conversational, like texting a smart friend.
- Same punchy founder energy.
- No translation artifacts -- write as a native speaker would.

**LinkedIn ({language from config}):**

- Slightly more narrative. Opens with a real hook (not "Excited to share").
- Tell the story. End with a clear point or question that invites response.
- Professional but human.

### Output format

Present the posts as clean markdown, ready to copy:

```text
---

### Twitter

<post content>

---

### Threads

<post content>

---

### LinkedIn

<post content>

---
```

No JSON. No code blocks around the posts themselves. Just clean text under each header.

## Step 5b: Save posts to vault (skip if `log` mode)

After generating the posts, also save them to a separate file.

Target path: `{Vault path}/Social Posts/YYYY-MM-DD-<project-slug>.md`

```bash
# Use the same Vault path from the Configuration section
_VAULT="<vault path from config>"
mkdir -p "$_VAULT/Social Posts/"
ls "$_VAULT/Social Posts/$(date +%Y-%m-%d)"-*.md 2>/dev/null || echo "NO_EXISTING"
```

Same append rules as Build Log -- if file exists for today+project, append a `## Session N` block.

### File format

```markdown
---
date: YYYY-MM-DD
project: <Display Name>
tags: [social-posts, <project-slug>]
---

# Posts: <short session title>

## Twitter

<post content>

## Threads

<post content>

## LinkedIn

<post content>
```

## Step 6: Report

After everything is done, summarize:

- If log was written: "Logged to `<file path>`"
- If posts were saved: "Posts saved to `<file path>`"
- If posts were generated: the posts are already visible above
- One-line summary of what was captured

## Adaptability

- If the user says "shorter" or "just the summary" -- only write the repurpose block, skip full sections
- If the user says "full log" -- write all sections even if some are thin
- If per-project CLAUDE.md has `audience:` or `tone_note:` -- use those to shape the repurpose block and posts
- If the vault path doesn't exist and mkdir fails -- tell the user and ask for the correct path
