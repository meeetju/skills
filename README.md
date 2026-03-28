# skills

Mati’s **[Cursor Agent Skills](https://cursor.com/docs/context/skills)**—instructions the agent loads from `SKILL.md` files (YAML frontmatter + markdown). This repo holds **folder-based** skills only; Cursor **`*.skill`** export ZIPs are **not** tracked (see `.gitignore`).

## What’s here

| Path | Role |
|------|------|
| `repo-conventions/` | Git workflow: `agent_update_*` branches, short commit subjects, MR/PR shape, no “Made-with” footers. |
| `note-taker/` | Add or extend snippets in `~/GitRepos/book` cheatsheets in a consistent style. |

## Layout

Each **folder skill** uses a `SKILL.md` at its root. Optional references or scripts can live next to it as the skill grows.

```
skills/
  README.md
  .gitignore       # ignores *.skill exports
  repo-conventions/SKILL.md
  note-taker/SKILL.md
```

Point Cursor at `~/.cursor/skills/<name>/` (symlink or copy) or keep this repo as the source of truth and sync as you prefer.
