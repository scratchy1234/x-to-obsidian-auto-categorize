---
name: daily-x-bookmarks
description: Daily auto-pull X bookmarks and process into Obsidian notes
---

Switch working directory to your vault:

```bash
cd /path/to/your/vault
```

> Replace `/path/to/your/vault` with your actual vault path (e.g., `~/my-vault`).

Read and execute the x-to-obsidian SKILL.md located at:

```
{vault_path}/.agents/skills/x-to-obsidian/SKILL.md
```

> Replace `{vault_path}` with the same path used above. If you installed x-to-obsidian elsewhere, use that path instead.

Execution mode: **summarize only** (Phase 0 + Phase 1, skip Phase 2).

Phase 2 distribution is triggered manually by the user.
