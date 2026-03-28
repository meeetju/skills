---
name: note-taker
description: Manage and append notes to Markdown cheatsheets in ~/GitRepos/book. Use when the user wants to add a snippet, command, or explanation to their personal knowledge base.
---

# Note Taker

## Overview

Append or insert new information into Markdown files under `~/GitRepos/book/`. Match the structure and **style rules** below so new content stays consistent with the refactored cheatsheets.

## Workflow

1. **Identify scope**: Pick the `.md` file that matches the topic (e.g. `ai.md`, `linux.md`, `docker.md`, `git.md`).
2. **Read the file**: Note headings, code-block style, and section order.
3. **Insertion point**:
   - New sub-topic → add a heading at the right level (see **Headings**).
   - Fits an existing section → append under that section.
   - No clear match → append at the end of the file (or under a sensible `##` if the file uses one).
4. **Format new content** using **Markdown style (book cheatsheets)**.
5. **Apply** edits with search_replace or shell commands.

## Markdown style (book cheatsheets)

Follow these rules for any add or edit in `~/GitRepos/book/`:

### Headings

- **One** `#` title per file (document title).
- Hierarchy: `##` major sections → `###` subsections → `####` sub-subsections. Do not skip levels.
- Use normal markdown headings only. **Do not** use decorative patterns such as horizontal rules wrapping heading text (`---` + `###` + name + `###` + `---`), or `##` / `###` lines used as bare labels without proper heading syntax.
- Avoid extra `---` horizontal rules as **section dividers**; use headings to separate sections unless the file already uses `---` consistently in one area.

### Code blocks

- Prefer **fenced** blocks with a **language tag**:
  - Shell / terminal commands → `bash`
  - Command + example output → `console`
  - SQL → `sql`
  - YAML (e.g. Kubernetes) → `yaml`
  - Dockerfile → `dockerfile`
  - Rust → `rust`
  - TOML / Cargo config → `toml`
  - JSON → `json`
  - Keyboard shortcuts, non-code samples → `text`
- Prefer fences over **indented** code blocks for commands and multi-line snippets (unless the surrounding list structure requires indentation).
- Keep prompts and commands accurate; drop stray prompt characters (e.g. leading `$`) from copy-paste blocks when they would confuse execution.

### Prose and layout

- Fix **obvious typos** in touched lines (spelling, doubled words, broken URLs like `hhttps://`).
- Use **at most one** blank line between paragraphs and blocks; do not leave large gaps.
- **Preserve meaning**: do not delete or rewrite existing notes unless the user asked for removal or correction.

### Format Content (quick reference)

| Element | Rule |
| ------- | ---- |
| Headings | `#` once; then `##` / `###` / `####` in order |
| Code | ` ```lang ` fences with an appropriate `lang` |
| Rules | No decorative `---` stacks instead of real headings |
| Typos | Fix when editing nearby text |
| Spacing | Single blank line between blocks |

## File mapping

- `ai.md`: AI models, LLMs, Gemini CLI, prompts, Obsidian (as in file).
- `linux.md`: Bash, Ubuntu, system administration.
- `docker.md`: Containers, images, docker-compose.
- `git.md`: Version control, branches, remotes.
- Other `.md` files in `~/GitRepos/book/`: match topic to filename (e.g. `rust.md`, `kafka.md`, `redis.md`, `kubernetes.md`, `mariadb.md`).
