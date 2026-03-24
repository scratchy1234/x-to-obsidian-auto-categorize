# x-to-obsidian-auto-categorize

Automatically pull, summarize, and categorize your X/Twitter bookmarks into an Obsidian vault using Claude Code.

## What It Does

```
X Bookmarks → feedgrab auto-pull → AI-summarized Obsidian notes → Auto-distribute to domain folders
```

A 3-phase pipeline:

| Phase | What | Required? |
|-------|------|-----------|
| **Phase 0** | Pull bookmarks from X via feedgrab | Optional — paste URLs manually if preferred |
| **Phase 1** | Fetch each tweet, summarize into structured Obsidian notes | Core |
| **Phase 2** | Auto-distribute notes from inbox to domain directories | Core |

Each note includes: core thesis, source context, summary, key takeaways, deep analysis, action items, and related note links — all in Obsidian-native format with callouts and wikilinks.

feedgrab handles everything: single tweets, threads, quoted tweets (QTs), and X Articles — with full content including embedded media metadata.

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (CLI)
- [feedgrab](https://github.com/iBigQiang/feedgrab) — universal content grabber (installed automatically by setup wizard)
- [browser-cookie3](https://pypi.org/project/browser-cookie3/) — Chrome cookie extraction for X auth (installed automatically)
- Chrome logged into x.com (for authentication)

## Quick Start

### 1. Install x-to-obsidian-auto-categorize

Clone into your Claude Code skills directory:

```bash
# Into your vault's skills directory
git clone https://github.com/scratchy1234/x-to-obsidian ~/my-vault/.agents/skills/x-to-obsidian

# Or standalone
git clone https://github.com/scratchy1234/x-to-obsidian ~/x-to-obsidian
```

### 2. Run — Setup Wizard Auto-Starts

Just invoke the skill. No manual config needed — the setup wizard runs automatically on first use:

```
/x-to-obsidian-auto-categorize
```

The wizard walks you through:
1. **Vault path** — where's your Obsidian vault?
2. **Inbox directory** — where should new notes go?
3. **Domain directories** — which folders to scan for related notes?
4. **Your perspective** — shapes "Why it matters" and action items in every note
5. **feedgrab + browser-cookie3** — installed automatically if missing
6. **X authentication** — extracts cookies from Chrome (auto or manual)
7. **Prefix system** — use defaults or customize routing rules

It validates each input, creates missing directories, generates your `config.yaml` and `prefix-rules.md`, and optionally runs a test with a sample URL.

> **Prefer manual setup?** Copy `config/config.example.yaml` → `config/config.yaml` and edit directly.

### 3. Normal Use

After setup, same command:

```
/x-to-obsidian-auto-categorize
```

Or with specific modes:

```
/x-to-obsidian-auto-categorize summarize only     # Phase 0+1, skip distribution
/x-to-obsidian-auto-categorize distribute only    # Phase 2 only
/x-to-obsidian-auto-categorize skip bookmarks     # Phase 1+2, manual URLs
```

### 4. (Optional) Scheduled Task

Create a scheduled task to auto-run daily. See `scheduled-task/SKILL.md` for a template.

## How It Works

### Phase 0: Bookmark Capture

Uses feedgrab to:
1. Fetch `x.com/i/bookmarks` with your X session cookies
2. Parse all bookmark `.md` files from the output directory
3. Deduplicate against processed history (`processed_ids.json`)
4. Append new URLs to `{inbox}/Twitter-URLs.md`

**No auto-pull needed?** Paste URLs manually: `- 2026-03-22 [Title](https://x.com/user/status/123)`

> **Authentication**: `feedgrab login twitter` is rejected by X's bot detection. The setup wizard extracts cookies directly from Chrome's local storage using browser-cookie3 — no manual token copying required.

### Phase 1: Fetch & Summarize

For each URL, fetches content with 2-level fallback:

```
feedgrab (per-URL isolated tmpdir + FORCE_REFETCH) → WebFetch (last resort)
```

feedgrab provides structured Markdown with full thread content, quoted tweets, and rich frontmatter (author, published date, engagement stats). Each URL gets its own temp directory to bypass feedgrab's internal dedup cache.

Generates structured Obsidian notes with:
- **Core Thesis** — author's main claim + why it matters (from your perspective)
- **Source Context** — author background, date
- **Summary** — 2-4 sentence overview; QT summary in `[!info]` callout when present
- **Key Takeaways** — `[!tip]` callout, importance-ordered facts
- **Deep Analysis** — `[!success]` claim + supporting evidence pairs
- **Triggered Thoughts** — `[!example]` specific next actions, linked to your active projects
- **Related Notes** — auto-matched wikilinks from your vault index cache

### Phase 2: Auto-Distribution

Reads each note's domain prefix and content, routes to the correct domain directory using your `prefix-rules.md`:

- Updates frontmatter (`status: inbox` → `status: not started`)
- Updates source and target index files
- Creates missing directories and indexes automatically

## Customization

Customize the note template (`templates/Twitter_Note.md`), prefix types, and routing rules (`config/prefix-rules.md`) to match your vault's structure.

## Project Structure

```
x-to-obsidian/
├── SKILL.md                    # Main skill — the pipeline orchestrator
├── config/
│   ├── config.example.yaml     # Path configuration template
│   ├── prefix-rules.example.md # Prefix & routing rules template
│   └── note-template.example.md # Template customization guide
├── templates/
│   ├── Twitter_Note.md         # Default note template
│   └── Index_Template.md       # Directory index template
├── examples/
│   └── sample-note.md          # Example generated note
├── scheduled-task/
│   └── SKILL.md                # Daily auto-run task template
└── docs/
    ├── setup.md                # Detailed setup guide
    └── customization.md        # Customization tutorial
```

## FAQ

**Q: feedgrab login twitter fails. What do I do?**
A: X rejects automated logins. The setup wizard handles this by extracting cookies from Chrome using browser-cookie3. Make sure Chrome is open and logged into x.com, then run the setup wizard and choose "auto" authentication.

**Q: I don't want auto bookmark pulling. Can I disable it?**
A: Yes. Set `phase0_enabled: false` in `config/config.yaml`. Manually add URLs to `{inbox}/Twitter-URLs.md` and run with "skip bookmarks" mode.

**Q: Does it support threads and quoted tweets?**
A: Yes. feedgrab fetches complete thread content and quoted tweets. The generated note includes a QT summary callout when a quoted tweet is detected.

**Q: Can I add new prefix types?**
A: Yes. Edit `config/prefix-rules.md` — add a new section following the existing format and update the routing rules.

**Q: Does it support non-Twitter URLs?**
A: feedgrab supports arbitrary URLs with Jina as internal fallback. The note template is optimized for tweets but works for general web content.

## License

MIT
