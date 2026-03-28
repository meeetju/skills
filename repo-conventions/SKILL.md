---
name: repo-conventions
description: >-
  Git workflow for Mati's repositories—branch names, merge request titles, and
  descriptions. Use when branching, committing, pushing, or opening MRs/PRs for
  repos under ~/GitRepos or when the user asks to follow repo conventions.
---

# Repository conventions

Apply these rules whenever creating branches, opening merge requests, or coordinating pushes for the repos defined below. Extend this skill with new sections when more repositories are added.

## Rules for every repository in this skill

These apply to **all** repos listed under [Repositories](#repositories), present and future.

### Commit messages

- **Title (subject):** A single first line that summarizes the change. **At most 20 characters** (including spaces). Shorten or rephrase if needed; put detail in the body.
- **Separation:** After the title, use **one blank line**, then the optional body (details, bullets, file lists).
- **Body length:** The body is a **summary** of at most **20 lines** (the title line does not count toward this limit). Stay within 20 lines so the message stays scannable.
- **No “Made-with” lines:** The message must **not** contain **`Made-with: Cursor`**, **`Made-with:`** followed by any other name, or any similar “made with …” attribution line (any casing). If an editor or integration adds these, **remove them** before the commit is final.

Example shape (title ≤ 20 chars):

```text
Fix docker notes

- Optional detail line
- Another line (body total ≤ 20 lines)
```

### Merge requests / pull requests

- **Title:** **At most 20 characters** (including spaces), unless a repo below sets an **exact** title string—then use that string (it must still fit in 20 characters if you add such a rule).
- **Title** and **description** are **separate:** use the host’s title field for the title only; use the description field for everything else. Do **not** merge title and description into one line without a break.
- **Separation:** When writing both in one block (e.g. CLI or a template), put **one newline** between title and description (or title, blank line, then description) so the summary is clearly not part of the title.
- **Description:** Summarize what changed and why (bullets are fine). Base this on `git diff`, `git log`, or the work just completed.

### Footers

- Do **not** add **Co-authored-by:** lines or other generated tooling footers to commit messages or MR/PR descriptions unless the user explicitly asks.
- **Same as commit messages:** never add **any** **Made-with** line (see [Commit messages](#commit-messages)); that includes Cursor and every other tool name.

### Branch names (default for every repo)

Unless a repository subsection says otherwise, branches for agent-driven work use **`agent_update_<N>`**, where **`N`** is one greater than the highest numeric suffix among existing **`agent_update_*`** branches (local and remote). If none exist, use **`agent_update_1`**. The **Book repo** subsection below spells out the full listing algorithm; reuse that pattern for any new repo entry.

---

## Repositories

### Book repo (`~/GitRepos/book`)

Personal Markdown cheatsheets (notes). Remote and hosting are whatever the user has configured (GitHub, GitLab, etc.).

#### Branch names

Use the pattern `agent_update_<N>` where `N` is **one greater than the highest existing `agent_update_*` number** among local and remote branches.

**Algorithm:**

1. List matching branches, for example:
   - `git branch -a | grep -E 'agent_update_[0-9]+'`
2. From every match (e.g. `agent_update_3`, `remotes/origin/agent_update_7`), extract the numeric suffix and find **max** `N`.
3. Create branch `agent_update_<max + 1>`. If no `agent_update_*` branch exists yet, use **`agent_update_1`**.
4. Create the branch from the appropriate base (usually `main` or `master`—use whatever the default branch is in that repo).

#### Merge request (or pull request)

When opening an MR/PR for this repo (CLI or web UI):

| Field | Value |
| ----- | ----- |
| **Title** | `Update book content` (exact string) |
| **Description** | Follow [Rules for every repository](#rules-for-every-repository-in-this-skill): summary of changes, separated from the title (own field or blank line). |

Do not substitute a different MR/PR title unless the user explicitly asks.

#### Workflow checklist

1. Determine next branch name with the algorithm above.
2. `git checkout -b agent_update_<N>` (from default branch).
3. Stage and commit with a message that follows [Commit messages](#commit-messages) (title ≤ 20 chars, blank line, body ≤ 20 lines; no Made-with lines).
4. Push the branch (set upstream if needed).
5. Open MR/PR with title **Update book content** and a **separate** description per [Merge requests / pull requests](#merge-requests--pull-requests).

### Future repositories

Add a new `###` subsection under [Repositories](#repositories) with: path, branch rules (use the same **`agent_update_<N>`** pattern unless you document an exception), MR/PR title (if any fixed title) and description rules, and any extra commit-message notes. **Commit format, MR title/description separation, and the no–Made-with rule** always follow [Rules for every repository](#rules-for-every-repository-in-this-skill).
