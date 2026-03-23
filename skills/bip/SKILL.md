---
name: bip
description: |
  Build in Public session logger. Writes a session summary to your vault
  and generates social media posts for multiple platforms.
  Use when the user says "bip", "log this session", "build in public", or "what did we do".
argument-hint: "[posts|log|full|draft|post|schedule HH:MM|queue] [optional: topic override]"
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
- `/bip draft` -- save today's posts to Typefully as drafts for review and editing → **skip to Step 7**
- `/bip post` -- alias for `draft` (same behavior) → **skip to Step 7**
- `/bip schedule HH:MM` -- schedule today's posts for auto-publish at HH:MM via Typefully → **skip to Step 7**
- `/bip queue` -- show Typefully scheduled drafts + local calendar → **skip to Step 7**

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

## Step 7: Auto-Posting via Typefully

**Only run this step when mode is `draft`, `post`, `schedule HH:MM`, or `queue`. For all other modes, this step is skipped. When mode IS one of these, skip Steps 2--5b and jump directly here.**

### Credential check

```bash
[ -f ~/.bip-credentials ] && echo "CREDS_OK" || echo "CREDS_MISSING"
```

If `CREDS_MISSING`, stop and print exactly this:

```
No Typefully credentials found. To enable auto-posting:

1. Sign up at https://typefully.com/?via=nikita — connect Twitter/X, Threads, and LinkedIn
2. Get your API key: Typefully → Settings → API → Create API key
3. Find your Social Set ID:
   curl -s "https://api.typefully.com/v2/social-sets" \
     -H "Authorization: Bearer YOUR_API_KEY_HERE"
   Copy the "id" value from the JSON response.
4. Create the credentials file:
   cat > ~/.bip-credentials << 'EOF'
   export TYPEFULLY_API_KEY=your_api_key_here
   export TYPEFULLY_SOCIAL_SET_ID=your_social_set_id_here
   EOF
   chmod 600 ~/.bip-credentials
5. Run /bip post again
```

### `queue` mode

```bash
source ~/.bip-credentials
_SOCIAL_SET_ID="$TYPEFULLY_SOCIAL_SET_ID"
curl -s "https://api.typefully.com/v2/social-sets/${_SOCIAL_SET_ID}/drafts" \
  -H "Authorization: Bearer $TYPEFULLY_API_KEY"
```

Parse the JSON response, filter for entries where `status` is `"scheduled"`, and display upcoming scheduled drafts in a readable table (date, content preview, platform). Also show the last 10 rows from `{Vault path}/Social Posts/calendar.md` if it exists. Stop here.

### `draft` / `post` / `schedule HH:MM` mode

**7a -- Source credentials and locate today's posts file**

Read `{Vault path}` from the Configuration section above and use it as `_VAULT` below.

```bash
source ~/.bip-credentials
_SOCIAL_SET_ID="$TYPEFULLY_SOCIAL_SET_ID"

_TODAY=$(date +%Y-%m-%d)
_VAULT="<vault path from config>"

# Resolve project slug same as Step 2 (check pwd and git root)
_GIT_ROOT=$(git rev-parse --show-toplevel 2>/dev/null || pwd)
_GIT_SLUG=$(basename "$_GIT_ROOT")
_PWD_SLUG=$(basename "$(pwd)")
echo "PWD_SLUG: $_PWD_SLUG  GIT_SLUG: $_GIT_SLUG"

# Check for today's posts file (try pwd slug first, then git slug)
ls "$_VAULT/Social Posts/${_TODAY}-${_PWD_SLUG}.md" 2>/dev/null \
  || ls "$_VAULT/Social Posts/${_TODAY}-${_GIT_SLUG}.md" 2>/dev/null \
  || echo "POSTS_MISSING"
```

If `POSTS_MISSING`, stop and print: "No posts found for today. Run `/bip` first to generate and save posts, then run `/bip post`."

**7b -- Extract platform content**

Read the posts file. Extract the most recent session's content for each platform (last occurrence of `## Twitter`, `## Threads`, `## LinkedIn` sections before the next `---` separator or end of file). Store each as a variable:

- `_TWITTER` -- Twitter post content
- `_THREADS` -- Threads post content
- `_LINKEDIN` -- LinkedIn post content

**7c -- For `schedule HH:MM` mode: build target datetime**

Parse `HH` and `MM` from the argument. Build ISO 8601 in local timezone (macOS):

```bash
_HOUR=14   # replace with parsed hour
_MIN=30    # replace with parsed minute
_SCHED=$(date -v${_HOUR}H -v${_MIN}M -v0S +"%Y-%m-%dT%H:%M:00%z")
echo "Scheduling for: $_SCHED"
```

For `post` mode, `_SCHED` is empty (no `schedule-date` in payload).

**7d -- POST each platform's draft to Typefully**

Use `jq` to safely encode the JSON payload. The Typefully v2 API uses `platforms` object with per-platform content. Run three separate curl calls -- one per platform:

```bash
_URL="https://api.typefully.com/v2/social-sets/${_SOCIAL_SET_ID}/drafts"
_AUTH="Authorization: Bearer $TYPEFULLY_API_KEY"

# Twitter/X -- draft mode (no publish_at)
_PAYLOAD=$(jq -n --arg text "$_TWITTER" \
  '{"platforms": {"x": {"enabled": true, "posts": [{"text": $text}]}}}')
# For schedule mode, add publish_at at top level:
# _PAYLOAD=$(jq -n --arg text "$_TWITTER" --arg sched "$_SCHED" \
#   '{"publish_at": $sched, "platforms": {"x": {"enabled": true, "posts": [{"text": $text}]}}}')
curl -s -X POST "$_URL" -H "$_AUTH" -H "Content-Type: application/json" -d "$_PAYLOAD"

# Threads
_PAYLOAD=$(jq -n --arg text "$_THREADS" \
  '{"platforms": {"threads": {"enabled": true, "posts": [{"text": $text}]}}}')
curl -s -X POST "$_URL" -H "$_AUTH" -H "Content-Type: application/json" -d "$_PAYLOAD"

# LinkedIn
_PAYLOAD=$(jq -n --arg text "$_LINKEDIN" \
  '{"platforms": {"linkedin": {"enabled": true, "posts": [{"text": $text}]}}}')
curl -s -X POST "$_URL" -H "$_AUTH" -H "Content-Type: application/json" -d "$_PAYLOAD"
```

Capture each response and extract the draft ID from the JSON.

**7e -- Update content calendar**

```bash
_CAL="$_VAULT/Social Posts/calendar.md"

# Create calendar with header if it doesn't exist
if [ ! -f "$_CAL" ]; then
  printf '# Content Calendar\n\n| Date | Project | Platforms | Status | Time |\n|------|---------|-----------|--------|------|\n' > "$_CAL"
fi

# Append row
_STATUS="queued"    # or "scheduled"
_TIME="immediate"   # or "HH:MM" for schedule mode
echo "| $_TODAY | <Display Name> | Twitter, Threads, LinkedIn | $_STATUS | $_TIME |" >> "$_CAL"
```

**7f -- Report**

For each platform, show:

- Platform name
- First 60 chars of post content (preview)
- Typefully draft ID from the API response
- Status: "posted to queue" or "scheduled for HH:MM"

---

## Adaptability

- If the user says "shorter" or "just the summary" -- only write the repurpose block, skip full sections
- If the user says "full log" -- write all sections even if some are thin
- If per-project CLAUDE.md has `audience:` or `tone_note:` -- use those to shape the repurpose block and posts
- If the vault path doesn't exist and mkdir fails -- tell the user and ask for the correct path
