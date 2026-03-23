# /bip -- Build in Public from Claude Code

Turn your daily Claude Code sessions into social media posts and structured dev logs.

Type `/bip` at the end of a work session. It auto-detects your project, writes a session log, and generates ready-to-copy posts for Twitter/X, Threads, and LinkedIn -- each in the language and style you configure.

**No dependencies. No build step.** Just a single markdown file that tells Claude Code what to do. Auto-posting is optional and requires a Typefully API key ($12.50/mo).

## Quick start (experienced users)

```bash
git clone https://github.com/aimerge-studio/bip-skills.git
mkdir -p ~/.claude/skills/bip
cp bip-skills/skills/bip/SKILL.md ~/.claude/skills/bip/SKILL.md
# Edit ~/.claude/skills/bip/SKILL.md — fill in the Configuration section at the top
# Then type /bip in any Claude Code session
```

New to Claude Code or want a detailed walkthrough? Keep reading.

---

## What it does

```
You: /bip

Claude Code:
  1. Detects you're in "my-saas" project (from git/directory)
  2. Writes session log -> ~/my-notes/Build Log/2026-03-23-my-saas.md
  3. Generates 3 social posts (Twitter, Threads, LinkedIn)
  4. Saves posts       -> ~/my-notes/Social Posts/2026-03-23-my-saas.md
```

### Example output

**Session log** (saved as a markdown file):

```markdown
---
date: 2026-03-23
project: My SaaS App
tags: [build-log, my-saas]
status: in-progress
---

# Session: Added Stripe webhook handler

## What was built / changed
Implemented Stripe webhook endpoint for subscription events.
Handles checkout.session.completed, invoice.paid, and customer.subscription.deleted.

## Key decisions made
Went with a single webhook endpoint + event router pattern instead of
separate endpoints per event. Simpler to maintain, easier to add new events.

## Problems hit & how solved
Stripe signature verification was failing in dev because ngrok was modifying
headers. Fixed by using the raw body buffer instead of parsed JSON for verification.

## Open / next
- Add invoice.payment_failed handler
- Set up Stripe test mode webhooks in CI
```

**Social posts** (generated inline in your terminal + saved to file):

> **Twitter:** Just built a Stripe webhook handler that actually works in dev. The trick: use the raw body buffer for signature verification, not parsed JSON. Ngrok was silently breaking it. 45 minutes of debugging, 2 lines to fix.

> **Threads:** Сегодня сделал обработку Stripe вебхуков. Решил использовать один эндпоинт с роутером событий вместо отдельных — проще поддерживать. Самый интересный баг: верификация подписи ломалась из-за ngrok. Оказалось, он менял заголовки. Фикс — использовать raw body вместо парсенного JSON.

> **LinkedIn:** Spent this morning building Stripe webhook handling for our subscription system. Chose a single endpoint with an event router over separate endpoints per event type -- simpler to maintain as we add billing features. The interesting bug: signature verification kept failing in dev. Turns out ngrok was modifying request headers just enough to break Stripe's HMAC check. Fix was switching from parsed JSON body to the raw buffer. Small detail, big headache. Anyone else hit this with local Stripe testing?

---

## What is a Claude Code skill?

A [skill](https://docs.anthropic.com/en/docs/claude-code/skills) is a markdown file that gives Claude Code specific instructions for a task. You put it in `~/.claude/skills/` and it becomes available as a slash command. No code to compile, no packages to install -- Claude Code reads the markdown and follows the instructions.

The `/bip` skill is one such file. It tells Claude Code: "when the user types /bip, analyze the session, write a log, and generate social posts."

---

## Setup (step by step)

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and working
- A terminal

**Obsidian is optional.** The skill writes plain `.md` files. Any folder on your computer works -- Obsidian, a folder on your desktop, whatever you prefer.

### Step 1: Clone this repo

```bash
git clone https://github.com/aimerge-studio/bip-skills.git
cd bip-skills
```

### Step 2: Copy the skill to Claude Code

```bash
mkdir -p ~/.claude/skills/bip
cp skills/bip/SKILL.md ~/.claude/skills/bip/SKILL.md
```

This puts the skill where Claude Code can find it. The `~/.claude/skills/` directory is where all Claude Code skills live.

### Step 3: Configure the skill

Open `~/.claude/skills/bip/SKILL.md` in any text editor (VS Code, Cursor, vim, TextEdit -- anything works).

At the top of the file, right after the `---` frontmatter, you'll see a **Configuration** section with tables. Fill these in:

#### 3a. Your name and bio

These control the ghostwriting voice -- how the posts sound.

```markdown
| Setting   | Value                                    |
| --------- | ---------------------------------------- |
| Your name | Jane Smith                               |
| Your bio  | indie hacker building developer tools    |
| Vault path | ~/Documents/Obsidian Vault              |
```

**Your name** appears nowhere in the generated posts -- it's only used internally so Claude Code knows who it's writing for.

**Your bio** shapes the tone. "founder building AI products" sounds different from "senior engineer at a fintech startup." Be specific.

#### 3b. Vault path (where logs and posts are saved)

This is the folder where your session logs and social posts will be written. The skill creates two subfolders inside it: `Build Log/` and `Social Posts/`.

**If you use Obsidian:** open Obsidian, go to **Settings > About**, and copy the "Vault path" value. Example:
```markdown
| Vault path | ~/Documents/Obsidian Vault |
```

**If you don't use Obsidian:** pick any folder. Example:
```markdown
| Vault path | ~/build-logs |
```

**If you're not sure:** just use `~/build-logs` -- it'll be created automatically.

#### 3c. Platform languages

Each platform has two properties: a **language** and a **style**. The style is fixed per platform (Twitter is always punchy, Threads is always conversational, LinkedIn is always narrative). You only change the **language**.

Default (all English):
```markdown
| Platform  | Language | Style                            |
| --------- | -------- | -------------------------------- |
| Twitter/X | English  | Tight, punchy, hook-first        |
| Threads   | English  | Conversational, casual           |
| LinkedIn  | English  | Narrative, professional but human|
```

Mixed languages (example: English Twitter, Russian Threads, English LinkedIn):
```markdown
| Twitter/X | English  | Tight, punchy, hook-first        |
| Threads   | Russian  | Conversational, casual           |
| LinkedIn  | English  | Narrative, professional but human|
```

All Spanish:
```markdown
| Twitter/X | Spanish  | Tight, punchy, hook-first        |
| Threads   | Spanish  | Conversational, casual           |
| LinkedIn  | Spanish  | Narrative, professional but human|
```

You can use **any language** for **any platform**. The skill writes natively in each language -- not translations.

#### 3d. Project lookup table

This maps your repository folder names to pretty display names. When the skill detects you're in `~/code/my-saas`, it looks up `my-saas` in this table and uses "My SaaS App" in log titles.

```markdown
| Slug           | Display Name    |
| -------------- | --------------- |
| my-saas        | My SaaS App     |
| client-portal  | Client Portal   |
| landing-page   | Landing Page    |
```

Add your own projects here. **If a project isn't in the table, the folder name is used with the first letter capitalized** (e.g., `my-saas` becomes "My-saas"). So this table is optional but makes your logs look cleaner.

### Step 4: Create the log directories

Create the folders where logs will be saved. Use the same path you set in Step 3b:

```bash
# Replace with your vault path from Step 3b
mkdir -p ~/Documents/Obsidian\ Vault/Build\ Log
mkdir -p ~/Documents/Obsidian\ Vault/Social\ Posts
```

Or if you chose `~/build-logs`:
```bash
mkdir -p ~/build-logs/Build\ Log
mkdir -p ~/build-logs/Social\ Posts
```

The skill will also try to create these automatically on first run, but creating them now avoids any permission issues.

### Step 5: Add the proactive trigger (optional)

This makes Claude Code automatically suggest `/bip` when you finish a session (say "done", "wrap up", "that's it", etc.).

```bash
# Make sure your CLAUDE.md ends with a newline first
echo "" >> ~/.claude/CLAUDE.md
cat claude-md-snippet.md >> ~/.claude/CLAUDE.md
```

Or open `~/.claude/CLAUDE.md` in your editor and paste the contents of `claude-md-snippet.md` at the end.

**Skip this step** if you prefer to always type `/bip` manually.

### Step 6: Test it

Open a new Claude Code session in any project and type:

```
/bip
```

Claude Code should:
1. Say "Logging session..."
2. Detect your project
3. Write a session log to your vault
4. Generate 3 social posts
5. Save the posts to your vault

If it's your first time and there's no real session to log, try:
```
/bip posts "just testing the build in public skill"
```
This generates posts about a specific topic without writing a log file.

---

## Usage

| Command | What it does |
| ------- | ------------ |
| `/bip` | Full flow: session log + social posts |
| `/bip posts` | Generate posts only, no log file |
| `/bip log` | Write session log only, no posts |
| `/bip posts "topic"` | Generate posts about a specific topic |
| `/bip draft` | Save today's posts to Typefully as drafts for review and editing |
| `/bip post` | Alias for `/bip draft` |
| `/bip schedule 14:30` | Schedule today's posts for auto-publish at 2:30 PM via Typefully |
| `/bip queue` | Show upcoming scheduled posts + content calendar |

### Modifiers

You can say these during the `/bip` flow:

- **"shorter"** or **"just the summary"** -- skip the full log sections, write only a 2-4 sentence summary
- **"full log"** -- write all sections even if some are thin

### Multiple sessions per day

If you run `/bip` twice in the same day for the same project, it appends a "Session 2" block to the existing log instead of overwriting.

---

## Customization

### Changing platforms

Want Bluesky instead of Threads? Mastodon instead of LinkedIn? Edit the **Platform rules** section in the SKILL.md (further down in the file, under Step 5). Change the platform name and adjust the style description to match that platform's culture.

### Changing voice

The **Voice** section in SKILL.md defines the writing personality. The defaults are raw/direct founder voice. Edit the bullet points to match your personal tone:

- More polished: change "Raw, honest, direct" to "Professional, thoughtful, measured"
- More technical: change "Naturally technical but not gatekeeping" to "Deep technical detail, assume the reader is an engineer"
- More casual: change "Confident but not arrogant" to "Relaxed, like talking to a friend"

### Per-project overrides

For specific projects that need a different angle, add this to that project's `.claude/CLAUDE.md`:

```markdown
## Build in Public

audience: enterprise CTOs
tone_note: focus on scalability and reliability, less on the building process
```

The skill checks for this section and adjusts the tone for posts from that project.

---

## Troubleshooting

**`/bip` doesn't show up as a command**
Make sure the file is at exactly `~/.claude/skills/bip/SKILL.md`. Claude Code scans this directory for skills on startup. Try starting a new Claude Code session.

**Logs aren't being saved**
Check that the vault path in your Configuration section matches an existing directory. Run `ls ~/your/vault/path/Build\ Log/` to verify it exists.

**Posts sound wrong / voice is off**
Edit the Voice section in SKILL.md. The more specific your bio is ("founder building AI-powered logistics tools for SMBs in Southeast Asia") the better the posts will sound. Generic bios produce generic posts.

**Wrong project name detected**
Add your project to the lookup table in the Configuration section. Without it, the folder name is used as-is.

---

## How it works

The `/bip` skill is a [Claude Code skill](https://docs.anthropic.com/en/docs/claude-code/skills) -- a markdown file with instructions. No external API calls, no dependencies, no build step. Claude Code reads the file and follows the instructions using its own intelligence.

The skill reads from:
- **Your conversation** -- what you discussed, decisions made, problems solved
- **Git log and diff** -- what actually changed in code
- **Project CLAUDE.md** -- optional per-project overrides

And writes to:
- `{vault}/Build Log/` -- structured session logs with frontmatter
- `{vault}/Social Posts/` -- generated posts saved for reference
- **Terminal** -- posts displayed inline for immediate copy-paste

---

## Auto-Posting with Typefully (optional)

`/bip` can post directly to Twitter/X, Threads, and LinkedIn via [Typefully](https://typefully.com/?via=nikita). This is fully opt-in — the basic skill works without it.

### Why Typefully?

One API key, all three platforms. Scheduling built in. No need to manage separate OAuth flows for each social network. Costs $12.50/mo Creator plan.

Direct platform APIs are all gated for solo builders:

- Twitter/X API: $200/mo minimum
- LinkedIn API: requires Partner Program approval
- Threads API: requires Meta business approval

Typefully is the pragmatic solution.

### Setup (5 minutes)

#### Step 1: Create a Typefully account

Go to [typefully.com](https://typefully.com/?via=nikita) and sign up.

Connect your profiles: click **Add Profile** for each platform — Twitter/X, Threads, LinkedIn.

#### Step 2: Get your API key

In Typefully: **Settings → API → Create API key**. Copy it.

#### Step 3: Find your Social Set ID

A Social Set is the group of connected profiles in your account. Find its ID:

```bash
curl -s "https://api.typefully.com/v2/social-sets" \
  -H "Authorization: Bearer YOUR_API_KEY_HERE"
```

Look for the `"id"` field in the JSON response — that's your Social Set ID.

#### Step 4: Create the credentials file

```bash
cat > ~/.bip-credentials << 'EOF'
export TYPEFULLY_API_KEY=your_api_key_here
export TYPEFULLY_SOCIAL_SET_ID=your_social_set_id_here
EOF
chmod 600 ~/.bip-credentials
```

This file lives outside any repo and is never committed (`.bip-credentials` is in `.gitignore`).

#### Step 5: Test it

Run `/bip` first to generate and save today's posts, then:

```
/bip draft
```

Check your Typefully dashboard — you should see three new drafts (one per platform) ready to review and edit before publishing.

### Workflow

1. End your coding session: `/bip` — writes the log and generates posts, saves everything to your vault
2. Send to Typefully: `/bip draft` — creates drafts for all three platforms
3. Open Typefully, review and tweak the copy if needed
4. Publish immediately or schedule from Typefully's UI

### How the content calendar works

Every time you run `/bip draft`, `/bip post`, or `/bip schedule`, a row is appended to:

```
{vault}/Social Posts/calendar.md
```

Example:

```text
| Date       | Project         | Platforms                  | Status         | Time      |
|------------|-----------------|----------------------------|----------------|-----------|
| 2026-03-23 | Building Skills | Twitter, Threads, LinkedIn | queued (draft) | immediate |
| 2026-03-24 | My SaaS App     | Twitter, Threads, LinkedIn | scheduled      | 14:30     |
```

Run `/bip queue` to see upcoming scheduled drafts from Typefully alongside this local log.

---

## Roadmap

- [ ] Image generation for posts (visual cards via AI image tools)
- [x] Direct posting to social media APIs (via Typefully)
- [x] Post scheduling (via Typefully `publish_at`)
- [x] Draft mode for review before publishing
- [ ] Weekly digest mode (summarize a week of logs into one post)

---

## License

MIT -- see [LICENSE](LICENSE)
