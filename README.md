# x-to-obsidian

Automatically save, summarize, and categorize your X/Twitter bookmarks into an Obsidian vault using Claude Code.

## What It Does

```
X Bookmarks → Chrome MCP auto-pull → Fetch tweet content → AI-summarized notes → Auto-distribute to domain folders
```

A 3-phase pipeline:

| Phase | What | Required? |
|-------|------|-----------|
| **Phase 0** | Pull bookmarks from X via Chrome | Optional — you can paste URLs manually |
| **Phase 1** | Fetch each tweet, summarize into structured Obsidian notes | Core |
| **Phase 2** | Auto-distribute notes from inbox to domain directories | Core |

Each note includes: core thesis, source context, summary, key takeaways, deep analysis, action items, and related note links — all in Obsidian-native format with callouts and wikilinks.

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (CLI)
- Python 3.7+ (for tweet fetching)
- [x-tweet-fetcher](https://github.com/ythx-101/x-tweet-fetcher) — tweet content extraction tool
- (Optional) [Claude in Chrome](https://chromewebstore.google.com/) MCP extension — for automatic bookmark pulling

## Quick Start

### 1. Install x-tweet-fetcher

```bash
git clone https://github.com/ythx-101/x-tweet-fetcher ~/x-tweet-fetcher
```

### 2. Install x-to-obsidian

Clone into your Claude Code skills directory, or anywhere accessible:

```bash
# Option A: Into your vault's skills directory
git clone https://github.com/ythx-101/x-to-obsidian ~/my-vault/.agents/skills/x-to-obsidian

# Option B: Standalone
git clone https://github.com/ythx-101/x-to-obsidian ~/x-to-obsidian
```

### 3. Run — Setup Wizard Auto-Starts

Just invoke the skill. No manual config needed — the setup wizard runs automatically on first use:

```
/x-to-obsidian
```

The wizard walks you through:
1. **Vault path** — where's your Obsidian vault?
2. **Inbox directory** — where should new notes go?
3. **Domain directories** — which folders to scan for related notes?
4. **Your perspective** — shapes "Why it matters" in every note
5. **x-tweet-fetcher** — installed? where?
6. **Chrome MCP** — auto-pull bookmarks or manual?
7. **Prefix system** — use defaults or customize?

It validates each input, creates missing directories, generates your `config.yaml` and `prefix-rules.md`, and optionally runs a test with a sample URL.

> **Prefer manual setup?** Copy `config/config.example.yaml` → `config/config.yaml` and edit directly. See [docs/setup.md](docs/setup.md).

### 4. Run

After setup, same command for normal use:

```
/x-to-obsidian
```

Or with specific modes:

```
/x-to-obsidian summarize only     # Phase 0+1, skip distribution
/x-to-obsidian distribute only    # Phase 2 only
/x-to-obsidian skip bookmarks     # Phase 1+2, manual URLs
```

### 5. (Optional) Set Up Scheduled Task

Create a scheduled task to auto-run daily. See `scheduled-task/SKILL.md` for a template.

## How It Works

### Phase 0: Bookmark Capture

Uses Claude in Chrome MCP to:
1. Open `x.com/i/bookmarks`
2. Extract bookmark URLs via JavaScript injection
3. Scroll to load more, deduplicate against processed history
4. Write new URLs to `{inbox}/Twitter-URLs.md`

**No Chrome?** Just paste URLs into the file manually: `- 2026-03-22 [Title](https://x.com/user/status/123)`

### Phase 1: Fetch & Summarize

For each URL, fetches content with 3-level fallback:

```
x-tweet-fetcher (FxTwitter API) → Jina Reader (r.jina.ai) → WebFetch (last resort)
```

Generates structured Obsidian notes with:
- **Core Thesis** — author's main claim + why it matters to you
- **Key Takeaways** — numbered facts, importance-ordered
- **Deep Analysis** — claim + evidence pairs
- **Triggered Thoughts** — specific next actions, linked to your active projects
- **Related Notes** — auto-matched wikilinks from your vault index

### Phase 2: Auto-Distribution

Reads each note's domain prefix and content, routes to the correct domain directory using your `prefix-rules.md`:

- Updates frontmatter (`status: inbox` → `status: not started`)
- Updates source and target index files
- Creates missing directories and indexes automatically

## Customization

See [docs/customization.md](docs/customization.md) for details on:
- Customizing the note template
- Adding/modifying prefix types
- Changing routing rules
- Adapting to your vault's directory structure

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

**Q: I don't have Chrome MCP. Can I still use this?**
A: Yes. Set `phase0_enabled: false` in config. Manually add URLs to `{inbox}/Twitter-URLs.md` and run with "skip bookmarks" mode.

**Q: Can I add new prefix types?**
A: Yes. Edit `config/prefix-rules.md` — add a new section following the existing format, and update the routing rules.

**Q: Does it support non-Twitter sources?**
A: Phase 1 uses `r.jina.ai` and `WebFetch` as fallbacks, which work on any URL. The template is optimized for tweets but works for general web content.

**Q: What about Twitter threads?**
A: x-tweet-fetcher handles single tweets well. For long threads, users should bookmark individual tweets of interest rather than the full thread.

## License

MIT
