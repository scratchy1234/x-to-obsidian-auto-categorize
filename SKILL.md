---
name: x-to-obsidian-auto-categorize
description: Full pipeline — pull X bookmarks, summarize into Obsidian notes, distribute to domain directories
---
You are the executor of an Obsidian auto-categorization system.

**On start**: Run `date` via Bash to record start time. Output: `⏱ Start: YYYY-MM-DD HH:MM:SS`

# Invocation Modes

| User says | Execution |
|-----------|-----------|
| (default) | Phase 0 + Phase 1 + Phase 2 |
| "summarize only" / "no distribution" | Phase 0 + Phase 1 |
| "distribute only" | Phase 2 only (skip Phase 0 and 1) |
| "skip bookmarks" | Phase 1 + Phase 2 (skip Phase 0) |

# Goal

Phase 0: Pull latest X bookmarks → Phase 1: Process URLs into Obsidian notes → Phase 2: Distribute notes from inbox to domain directories.

# Prerequisites

**First-run check**: Look for `config/config.yaml` in this skill's directory.

- **Not found** → Enter **Setup Flow** (see bottom of this file). Do not proceed to any Phase until setup is complete.
- **Found** → Continue below.

Read the following files from this skill's directory:

1. `config/config.yaml`
2. `config/prefix-rules.md` (if not found, copy `config/prefix-rules.example.md` → `config/prefix-rules.md` and inform user)
3. `templates/Twitter_Note.md` (note format template)

Then build a **vault index cache**: Read all `{index_file}` files (including subdirectories) from each directory listed in `config.index_scan_dirs`, extract note names and descriptions from tables, store in memory.

Also extract all entries with 🔵 status as the **active projects list**, used when generating "Triggered Thoughts".

All paths below use config values. `{vault}` = `config.vault_path`, `{inbox}` = `config.inbox_dir`, etc.

---

# Phase 0: Pull X Bookmarks (Optional)

> Skipped when user says "distribute only" or "skip bookmarks".
> Also skipped when `config.phase0_enabled` is `false`.

**Requires**: feedgrab installed and authenticated with X (via `feedgrab login twitter`).

1. Run via Bash:
   ```
   OUTPUT_DIR=/tmp/feedgrab-x2o feedgrab https://x.com/i/bookmarks
   ```

2. Read all `.md` files from `/tmp/feedgrab-x2o/X/`. For each file, extract from YAML frontmatter:
   - `original_link` → tweet URL
   - `author` → handle
   - `title` or first line of body → title (truncate to 60 chars)
   - `tweet_id` or parse ID from `original_link`

3. Read `{config.processed_ids_path}` (auto-created as `[]` if not found) to filter already-processed IDs. Read `{vault}/{inbox}/{urls_file}` to filter URLs already in the file. After two rounds of filtering, get the truly new bookmarks.

4. **Two paths**:
   - New bookmarks found → Append to `{vault}/{inbox}/{urls_file}`, append IDs to `processed_ids.json`, continue to Phase 1
   - No new bookmarks → Check if `{urls_file}` has content:
     - Has content → proceed to Phase 1
     - Empty → Output "No new bookmarks, {urls_file} is empty, skipping." → End

   Write format: `- YYYY-MM-DD [Title](URL)`

**Fallback (feedgrab unavailable or auth failed)**: Skip Phase 0. If `{vault}/{inbox}/{urls_file}` has content, proceed to Phase 1. If empty, output "feedgrab unavailable and no pending URLs. Nothing to process." → End.

---

# Phase 1: Process Twitter Bookmarks

## Step 0: Check URLs File

1. Read `{vault}/{inbox}/{urls_file}`
2. Parse all lines, format: `- YYYY-MM-DD [Title](URL)`
3. **If file doesn't exist or is empty** → Skip Phase 1. If mode includes Phase 2, proceed to Phase 2; otherwise end.
4. **If has content** → proceed to Step 1

## Step 1: Fetch Each URL

For each URL, use this 2-level fallback chain (only proceed to next level on failure):

```
Path A (feedgrab):
  Run: OUTPUT_DIR=/tmp/feedgrab-x2o feedgrab <URL>
  Find the newest .md file in /tmp/feedgrab-x2o/X/
  Read its content (includes thread, QT, YAML frontmatter)
  → Success → structured Markdown → Step 2
  → Failure (non-zero exit or no file created) → Path B

Path B (WebFetch):
  WebFetch(URL)
  → Success → raw text → Step 2
  → Failure → Mark as UDF, continue to next URL
```

Record which path each URL used for the Step 5 report. Map: Path A = `feedgrab`, Path B = `webfetch`, failure = `UDF`. This value also goes into the note's frontmatter `fetch_mode` field.

## Step 2: Generate Note

Generate notes strictly following `templates/Twitter_Note.md`. HTML comments in the template contain examples and rules — reference them but **do not write comments into the output note**.

Section generation rules:

- **Core Thesis**: Two layers — "Author's core claim (one sentence)" + "Why it matters (from `config.user_perspective` lens)"

- **Source Context**:
  - Author @handle + date
  - Background: who is this person, why their opinion matters (one sentence)

- **Summary**: 2-4 sentences of "what this is about overall". No raw quotes, no commentary. When has_qt is true, add a `[!info]` callout with one-sentence QT summary (omit entirely if no QT).

- **Key Takeaways**: `[!tip]` callout, numbered list
  - Pure fact extraction, no opinion
  - Ordered by logical importance (most critical first), NOT by original order
  - Each item format: `**Keyword**: one-sentence explanation`
  - Quantity adapts to content length:
    - Single tweet ≤ 280 chars → 2-3 items
    - Thread / long tweet → 3-5 items
    - X Article / long-form → 5-8 items

- **Deep Analysis**: Each group has two parts (claim + evidence), groups separated by `---`
  - Claim: author's core argument (`[!success]` callout)
  - Evidence: data / case study / logic chain supporting the claim
  - Only extract claims the author explicitly states — no inference
  - Short tweet (≤ 280 chars) → 1 group; rich content → 2-3 groups

- **Actions & Links** (two fields):

  - **Triggered Thoughts**: `[!example]` callout
    - Must be a specific next action ("install X and run it", "add Y rule to Z file", "check Z's GitHub README") — no vague suggestions
    - Prioritize linking to items in the **active projects list** — if this tweet is directly useful for a 🔵 project, name which project and what to do
    - If unrelated to active projects, write a specific next step relevant to your `config.user_perspective`
    - If genuinely no actionable value, write "None"

  - **Related Notes**: Match keywords from the current note's title, tags, and key takeaway keywords against the **vault index cache**. For each index entry, check if 2+ keywords overlap. Write `[[note-name]]` for the top 3 matches. Zero matches → leave empty, do not write any wikilinks.

**Frontmatter**:
```yaml
---
created: YYYY-MM-DD
status: inbox
source: twitter | article (use "article" when URL contains /articles/ or content is long-form X Article)
domain: AI/Invest/etc.
original_link: https://...
author: "@username"
has_qt: true/false
fetch_mode: feedgrab/webfetch
tags: [inbox, keywords]
---
```

**File naming**: `domain-prefix-Title.md` (rules in prefix-rules.md)

## Step 3: Write to Inbox + Update Index

1. Write each note file to `{vault}/{inbox}/`
2. Update `{vault}/{inbox}/{index_file}`: read existing index, only add entries that don't exist yet. Format: `| [[filename]] | one-sentence description | ⚪ |`

## Step 4: Clear URLs File

After all URLs are processed, clear `{vault}/{inbox}/{urls_file}` (keep the file, empty its content).

## Step 5: Output Phase 1 Report, Auto-proceed to Phase 2

Output report:

```
## Phase 1 Complete

Processed X bookmarks:
- feedgrab: Y
- WebFetch fallback: Z
- UDF: W

Written to inbox:
- [[file1]]
- [[file2]]

Auto-proceeding to Phase 2 distribution...
```

**If user specified "summarize only" / "no distribution" at invocation → stop here. Otherwise, proceed directly to Phase 2 without waiting for user confirmation.**

---

# Phase 2: Distribute Inbox

## Step 1: Scan Inbox

1. List all `.md` files in `{vault}/{inbox}/` (exclude `{index_file}` and `{urls_file}`)
2. If inbox is empty, reply "Inbox is empty, nothing to distribute." and end
3. Read each file's frontmatter and content summary

## Step 2: Execute Distribution

For each file:

1. **Parse filename**: Extract domain and prefix from `domain-prefix-Title`
2. **Determine target path**: Follow routing rules in `prefix-rules.md`
3. **Determine new filename**: Remove domain prefix → `prefix-Title`
4. **If routing is uncertain**: Flag this file, pause and ask user, continue processing other files

Execute distribution:

1. **Plans path**: Create `{plans}/Title/` folder first, then move file in
2. **Other paths**: Move directly to target subdirectory
3. **Update frontmatter**: `status: inbox` → `status: not started`; remove `inbox` from `tags`
4. **Update target directory `{index_file}`**: Add entry `| [[prefix-Title]] | one-sentence description | ⚪ |`
5. **Remove entry from inbox `{index_file}`**

## Step 3: Output Report

```
## Distribution Complete

| Target Directory | Files |
|-----------------|-------|
| AI Engineering/ | 3 |
| AI Cognition/ | 2 |
| Plans/ | 1 |

Inbox remaining: 0 files
```

---

# Rules

- Subdirectories are topic containers, not type containers — distribute by topic first, not by prefix
- When routing is uncertain, pause and ask user; continue processing other files
- After distribution, must update both indexes (remove from inbox + add to target)
- Never delete any files, only move
- Update frontmatter `status` from `inbox` to `not started`, remove `inbox` from `tags`
- If target subdirectory doesn't exist, create it and initialize `{index_file}` using `templates/Index_Template.md`
- **On finish**: Run `date` via Bash to record end time. Output: `⏱ End: YYYY-MM-DD HH:MM:SS`, and calculate total duration + average time per file

---

# Setup Flow

> Triggered automatically when `config/config.yaml` does not exist.

Output: "👋 Welcome to **x-to-obsidian-auto-categorize**! Let's set up your configuration. I'll ask a few questions."

## Q1: Vault Path

Ask: "Where is your Obsidian vault? (full path, e.g. `~/my-vault`)"

- Validate: Run `ls` on the path. If it doesn't exist → tell user, ask again.
- Store as `vault_path`.

## Q2: Inbox Directory

Ask: "What's your inbox folder name? (default: `00_Inbox`)"

- Accept user input, or use default if they press enter / say "default".
- Check if `{vault_path}/{inbox_dir}` exists.
  - Exists → proceed.
  - Doesn't exist → Ask: "Folder `{inbox_dir}` doesn't exist. Create it? (y/n)"
    - y → Create the directory + initialize `_index.md` from `templates/Index_Template.md`.
    - n → Ask for a different name.
- Store as `inbox_dir`.

## Q3: Domain Directories

Run `ls` on `{vault_path}` and display the top-level directories.

Ask: "Which of these directories contain your notes? (Pick the ones you want the system to scan for related notes and distribute to. Comma-separated, e.g. `50_AI, 60_Investment`)"

- Store as `index_scan_dirs`.

## Q4: User Perspective

Ask: "Describe yourself in one line — this shapes the 'Why it matters' and 'Action items' in every note. (e.g. 'AI startup founder', 'frontend developer', 'crypto trader and researcher')"

- Store as `user_perspective`.

## Q5: feedgrab + browser-cookie3

feedgrab and browser-cookie3 are **required** for this skill to function. Install both automatically:

1. Check feedgrab: run `which feedgrab` via Bash.
   - **Not found** → Output: "Installing feedgrab..." → Run:
     ```
     pip3 install "feedgrab[all] @ git+https://github.com/iBigQiang/feedgrab.git"
     ```
   - **Found** → Output: "✅ feedgrab detected."

2. Check browser-cookie3: run `python3 -c "import browser_cookie3"` via Bash.
   - **Not found** → Output: "Installing browser-cookie3..." → Run:
     ```
     pip3 install browser-cookie3
     ```
   - **Found** → Output: "✅ browser-cookie3 detected."

## Q6: X Authentication (Required)

Twitter/X authentication is **required** — feedgrab needs it for both bookmark sync (Phase 0) and full-fidelity tweet fetching (Phase 1).

> Note: `feedgrab login twitter` may be rejected by X's bot detection. The recommended method is to extract cookies from your Chrome browser automatically.

Ask: "How would you like to authenticate with X? (auto/manual)"

- **auto** → Extract cookies from Chrome automatically:
  ```python
  import browser_cookie3, json
  cj = browser_cookie3.chrome(domain_name='.x.com')
  cookies = [{"name": c.name, "value": c.value, "domain": c.domain, "path": c.path}
             for c in cj if c.name in ('auth_token', 'ct0', 'twid')]
  json.dump({"cookies": cookies, "origins": []}, open('~/.feedgrab/twitter.json','w'), indent=2)
  ```
  Run this via Bash (expand `~`). Output: "✅ Cookies extracted from Chrome. Make sure Chrome is logged into x.com."
  Set `phase0_enabled: true`.

- **manual** → Output:
  ```
  Run this in your terminal:
    feedgrab login twitter

  If X rejects the automated login, extract cookies manually:
  1. Open Chrome and log into x.com
  2. Run: python3 -c "import browser_cookie3, json; cj = browser_cookie3.chrome(domain_name='.x.com'); cookies = [{'name': c.name, 'value': c.value, 'domain': c.domain, 'path': c.path} for c in cj if c.name in ('auth_token','ct0','twid')]; json.dump({'cookies': cookies, 'origins': []}, open('{sessions_path}/twitter.json','w'), indent=2)"

  Once done, tell me and I'll continue.
  ```
  Wait for user confirmation before proceeding.
  Set `phase0_enabled: true`.

Verify authentication: run `feedgrab doctor x 2>&1 | grep "Twitter cookies"` and confirm `✅ auth_token` is present.

## Q7: Prefix System

Ask: "The skill uses a prefix system to categorize notes (e.g. `tool-`, `insight-`, `framework-`). Want to use the default 8-prefix system, or customize? (default/custom)"

- **default** → Copy `config/prefix-rules.example.md` → `config/prefix-rules.md`. Output: "Default prefix rules saved. You can edit `config/prefix-rules.md` anytime to customize."
- **custom** → Ask: "List your prefixes, comma-separated (e.g. `tool, insight, tutorial, project`)"
  - For each prefix, ask: "What does `{prefix}` mean? (one-line description)"
  - Ask: "Which directory should `{prefix}-` files route to? (relative to vault, e.g. `50_AI/Tools/`)"
  - Generate `config/prefix-rules.md` from user input, following the structure of `prefix-rules.example.md`.

## Generate Config

Write `config/config.yaml` using collected values:

```yaml
vault_path: {Q1}
inbox_dir: {Q2}
urls_file: Twitter-URLs.md
index_file: _index.md
processed_ids_path: ./data/processed_ids.json
index_scan_dirs: {Q3}
user_perspective: "{Q4}"
phase0_enabled: {Q6}
```

Also ensure these exist:
- `{vault_path}/{inbox_dir}/{urls_file}` — create empty file if missing
- `{vault_path}/{inbox_dir}/_index.md` — create from template if missing
- `./data/` directory in this skill's folder — create if missing

## Test Run

Output: "✅ Setup complete! Want to do a test run with a sample URL? (y/n)"

- **y** → Ask: "Paste an X/Twitter URL to test with:"
  - Write it to `{vault}/{inbox}/{urls_file}` as `- YYYY-MM-DD [Test](URL)`
  - Execute Phase 1 Step 1-2 on this single URL.
  - Show the generated note to the user.
  - Ask: "How does this look? If good, I'll save it. Otherwise I can adjust the template."
    - Good → Write to inbox, update index. Output: "🎉 All set! Run `/x-to-obsidian-auto-categorize` anytime to process your bookmarks."
    - Adjust → Let user describe what to change, modify `templates/Twitter_Note.md` accordingly, re-run.
- **n** → Output: "🎉 All set! Run `/x-to-obsidian-auto-categorize` anytime to process your bookmarks."
