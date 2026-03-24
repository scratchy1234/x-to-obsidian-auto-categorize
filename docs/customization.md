# Customization Guide

## Note Template

Edit `templates/Twitter_Note.md` to change the note structure.

### Change Sections

Each section uses an Obsidian callout. You can:

- **Remove a section**: Delete the entire block (callout + surrounding markdown)
- **Add a section**: Follow the pattern — heading, callout, content placeholder, HTML comment with example
- **Change callout type**: Replace `[!warning]` with `[!abstract]`, `[!note]`, etc.

### Change Language

The template placeholders (e.g., `{{Author's core claim}}`) guide Claude. Replace them with your language:

```markdown
> [!warning] 核心命题
> **{{作者的核心主张，一句话}}**
```

### Adjust Content Depth

The quantity rules in HTML comments control how many key takeaways are generated:

```html
<!-- Single tweet ≤ 280 chars → 2-3 items -->
```

Change these numbers to get more or fewer items.

## Prefix System

Edit `config/prefix-rules.md` to customize categorization.

### Add a New Prefix

Follow the existing format:

```markdown
### myprefix
> Description of what this prefix means

- What kind of content it covers
- **Routing**: Where `myprefix-` files go

**Judgment**: "How to decide if content belongs here"
**Boundary**: `myprefix` = X | `other` = Y
```

Then add routing rules in the Distribution Rules section.

### Change Routing

The routing rules map prefixes + topics to directories. Example:

```markdown
| Topic | Target Subdirectory |
|-------|-------------------|
| My topic | `My Directory/` |
```

Routing is primarily **topic-based** — the same directory can hold files with different prefixes.

### Change Domain Prefixes

Update the domain prefix mapping table:

```markdown
| Domain Prefix (inbox) | Target Directory |
|-----------------------|-----------------|
| `AI` | `50_AI/` |
| `Finance` | `60_Finance/` |
| `Personal` | `30_Personal/` |
```

## Directory Structure

The skill adapts to your vault structure through config:

1. **`config.inbox_dir`**: Your inbox folder name
2. **`config.index_scan_dirs`**: Which directories to scan for related notes
3. **`prefix-rules.md`**: All routing destinations

### Minimal Vault Structure

At minimum, you need:

```
my-vault/
├── 00_Inbox/           # config.inbox_dir
│   ├── _index.md       # config.index_file
│   └── Twitter-URLs.md # config.urls_file
└── Some_Domain/        # At least one routing target
    └── _index.md
```

### Index Files

Every directory that receives distributed notes should have an index file. Format:

```markdown
| Item | Description | Status |
|------|-------------|--------|
| [[note-name]] | Brief description | ⚪ Not started |
```

The skill auto-creates missing index files using `templates/Index_Template.md`.

## User Perspective

`config.user_perspective` shapes two sections:

1. **Core Thesis → "Why it matters"**: Explains relevance from your perspective
2. **Triggered Thoughts**: Generates action items relevant to your work

Examples:
- `"frontend developer building React apps"` → actions focus on UI/UX tools
- `"AI startup founder"` → actions focus on product opportunities
- `"crypto trader and DeFi researcher"` → actions focus on market implications

## Scheduled Task

Customize the scheduled task in `scheduled-task/SKILL.md`:

- Change execution mode (default: "summarize only")
- Add pre/post hooks
- Adjust the working directory path
