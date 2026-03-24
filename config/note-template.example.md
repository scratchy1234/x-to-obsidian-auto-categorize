# Note Template Reference

This file explains how to customize `templates/Twitter_Note.md`.

## Template Sections

| Section | Callout Type | Purpose |
|---------|-------------|---------|
| Core Thesis | `[!warning]` | Author's main claim + why it matters to you |
| Source Context | — | Author handle, date, credibility |
| Summary | — | 2-4 sentence overview, no raw quotes |
| Quoted Tweet | `[!info]` | QT summary (only when has_qt: true) |
| Key Takeaways | `[!tip]` | Numbered facts, importance-ordered |
| Deep Analysis | `[!success]` | Claim + evidence pairs |
| Triggered Thoughts | `[!example]` | Specific next actions |
| Related Notes | — | Wikilinks to existing vault notes |

## Callout Syntax (Obsidian)

```markdown
> [!type] Title
> Content line 1
> Content line 2
```

Available types used in this template: `warning`, `info`, `tip`, `success`, `example`

## Key Takeaways Quantity Rules

| Content Length | Number of Items |
|---------------|----------------|
| Single tweet ≤ 280 chars | 2-3 |
| Thread / long tweet | 3-5 |
| X Article / long-form | 5-8 |

## Customization Tips

1. **Change callout types**: Replace `[!warning]` with `[!abstract]` etc. to match your theme
2. **Add/remove sections**: Delete sections you don't need, add custom ones
3. **Change language**: Template works in any language — just translate the placeholders
4. **Adjust "Why it matters"**: Edit `config.user_perspective` to change the lens
