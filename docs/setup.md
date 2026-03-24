# Setup Guide

## Step 1: Prerequisites

### Claude Code

Install Claude Code CLI if you haven't:

```bash
npm install -g @anthropic-ai/claude-code
```

### x-tweet-fetcher

```bash
git clone https://github.com/ythx-101/x-tweet-fetcher ~/x-tweet-fetcher
cd ~/x-tweet-fetcher
pip install -r requirements.txt  # if any
```

Test it works:

```bash
python3 ~/x-tweet-fetcher/scripts/fetch_tweet.py --url https://x.com/elonmusk/status/123456 --text-only
```

### Claude in Chrome (Optional)

Install the Claude in Chrome MCP extension from the Chrome Web Store. This enables Phase 0 (automatic bookmark pulling). Without it, you'll need to manually paste URLs.

## Step 2: Install x-to-obsidian

### Option A: As a vault skill (recommended)

```bash
cd ~/my-vault  # your Obsidian vault
mkdir -p .agents/skills
git clone https://github.com/ythx-101/x-to-obsidian .agents/skills/x-to-obsidian
```

### Option B: Standalone

```bash
git clone https://github.com/ythx-101/x-to-obsidian ~/x-to-obsidian
```

## Step 3: Configure

```bash
cd .agents/skills/x-to-obsidian
cp config/config.example.yaml config/config.yaml
cp config/prefix-rules.example.md config/prefix-rules.md
```

### config.yaml

| Field | Description | Example |
|-------|-------------|---------|
| `vault_path` | Obsidian vault root (absolute or ~) | `~/my-vault` |
| `inbox_dir` | Inbox folder name | `00_Inbox` |
| `urls_file` | URL list filename | `Twitter-URLs.md` |
| `index_file` | Index filename pattern | `_index.md` |
| `fetcher_path` | Path to fetch_tweet.py | `~/x-tweet-fetcher/scripts/fetch_tweet.py` |
| `processed_ids_path` | Dedup file path | `./data/processed_ids.json` |
| `index_scan_dirs` | Dirs to scan for related notes | `[50_AI, 60_Investment]` |
| `user_perspective` | Your lens for "Why it matters" | `"startup founder building AI tools"` |
| `phase0_enabled` | Enable Chrome bookmark pull | `true` or `false` |

### prefix-rules.md

This file controls:
- How notes are named (`domain-prefix-Title`)
- How notes are routed to directories
- What prefixes exist and what they mean

Customize the routing rules to match your vault's directory structure. The example file provides 8 prefixes (skill, prompt, learn, tool, framework, insight, build, plan) — add, remove, or rename as needed.

## Step 4: Prepare Your Vault

Ensure your vault has:

1. **An inbox directory** matching `config.inbox_dir` (e.g., `00_Inbox/`)
2. **An index file** in the inbox (e.g., `00_Inbox/_index.md`) — use `templates/Index_Template.md` as a starting point
3. **Domain directories** matching your routing rules (e.g., `50_AI/`, `60_Investment/`)
4. **Index files in domain directories** — the skill creates missing ones automatically, but existing indexes enable better "Related Notes" matching

## Step 5: Test

1. Create `{inbox}/Twitter-URLs.md` with a test URL:

```markdown
- 2026-03-22 [Test tweet](https://x.com/someuser/status/123456)
```

2. Run:

```
/x-to-obsidian skip bookmarks, summarize only
```

3. Check the generated note in your inbox.

## Step 6: Scheduled Task (Optional)

To auto-run daily, create a scheduled task using Claude Code:

```
/schedule create daily-x-bookmarks
```

Use `scheduled-task/SKILL.md` as the task content. Customize the execution time (default: daily at 10:00 AM).
