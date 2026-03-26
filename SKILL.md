---
name: plans
description: Archive finalized Claude Code implementation plans to named, project-local files with a global index at ~/.claude/plans/INDEX.md. Use this skill whenever the user says "archive this plan", "save this plan", "/plans", "/plans archive", "finalize plan", or "this plan is done". Also use for "what plans do I have", "show archived plans", "plans list", "find the plan for [project]". Trigger proactively at the end of any plan-mode session that produced a finalized implementation plan — suggest archiving before the session ends. Works globally across all projects — not tied to any specific repo or git workflow.
---

# Plans Archive Skill

This skill archives finalized Claude Code implementation plans to **project-local** `plans/` directories with a **global index** at `~/.claude/plans/INDEX.md`. Plans stay with their projects. The index is the cross-project lookup.

This skill is **global** — it works across all projects and requires no Git repo or commit workflow.

## Why This Matters

Claude Code auto-saves plan mode output to `~/.claude/plans/<random-slug>.md`. These files accumulate with no project association, no index, and no way to retrieve "what was the plan for project X last month?" Plans capture the full rationale and constraints of an implementation effort. They pair naturally with DEVLOG.md: plans capture the "how before execution," devlog captures the "what was decided and why."

---

## File Layout

```
~/.claude/plans/
  INDEX.md                              — Global lookup: slug ↔ project-local plan file
  misty-conjuring-hammock.md            — Claude Code auto-generated plan (random slug, do not modify)
  vast-coalescing-cookie.md             — Another auto-generated plan
  ...

<project-directory>/plans/
  2026-03-21-marketing-messaging.md     — Named plan copy, references slug file
  2026-03-21-product-analysis.md        — Another named plan
  ...
```

- **`~/.claude/plans/<slug>.md`** — Claude's auto-generated plans. Never modify these.
- **`<project>/plans/<date-name>.md`** — Named, project-local copies with a reference back to the slug file.
- **`~/.claude/plans/INDEX.md`** — Master lookup table mapping slugs to project-local files.

---

## INDEX.md Format

Lives at `~/.claude/plans/INDEX.md`.

```markdown
# Plans Index

| Date | Slug | Project | Title | Local Plan | Status |
|---|---|---|---|---|---|
| 2026-03-21 | `misty-conjuring-hammock` | code-katz | Marketing & Messaging Plan | `code-katz/plans/2026-03-21-marketing-messaging.md` | `active` |
| 2026-03-21 | `misty-conjuring-hammock` | d20m | Product Analysis & Roadmap | `d20m/plans/2026-03-21-product-analysis.md` | `active` |
```

Newest row first. Multiple plans can reference the same slug (when one slug file contains plans for multiple projects). Status values:

| Status | Meaning |
|---|---|
| `active` | Current plan for this project, being executed against |
| `archived` | Superseded by a newer plan after execution work began |
| `superseded` | Replaced before execution started |
| `executed` | Fully implemented and delivered |

---

## Local Plan Filename Format

`YYYY-MM-DD-plan-slug.md` — date of archiving, 3-5 word lowercase-hyphenated slug derived from the plan title.

Examples:
- `2026-03-21-marketing-messaging.md`
- `2026-03-21-product-analysis.md`
- `2026-02-10-auth-redesign.md`

---

## Trigger Conditions

### Explicit
- "archive this plan" / "save this plan" / "plans save"
- `/plans` / `/plans archive`
- "finalize plan" / "this plan is done" / "this plan is final"
- "plans list" / "what plans do I have" / "show archived plans"
- "find the plan for [project]" / "show me the plan for [project]"

### Proactive
At the end of a plan-mode session that produced a finalized implementation plan — before the user moves to execution — suggest: "This plan looks finalized — want me to archive it to `<project>/plans/`?"

Do NOT trigger proactively for exploratory or in-progress plans, partial outlines, or plans explicitly marked as drafts.

---

## Workflow: Archiving a Plan

### Step 1: Locate Plan Content

In order:
1. **Random-slug file** — If the current plan slug is known, read `~/.claude/plans/<slug>.md` directly. Preferred.
2. **Conversation extraction** — Extract the most recent structured implementation plan from the thread.
3. **Ask** — "Where is the plan? I can read `~/.claude/plans/<slug>.md` if you have the filename, or extract it from this conversation."

### Step 2: Determine Project and Title

Extract the plan title from the first H1 heading. Derive:
- **Project directory** — the project root where the plan will be saved (e.g., `code-katz/`, `d20m/`)
- **Project name** — short human-readable identifier for INDEX.md (e.g., "code-katz", "d20m")
- **Short slug** — lowercase-hyphenated, 3-5 words from the title, for the filename

Suggest and confirm: "I'll save this as `code-katz/plans/2026-03-21-marketing-messaging.md` — correct?"

### Step 3: Show the User (before writing anything)

```
Here's what I'm about to do:

1. Create <project>/plans/ directory (if needed)
2. Write plan to <project>/plans/YYYY-MM-DD-plan-slug.md
3. Update ~/.claude/plans/INDEX.md
   — New row: date | slug | project | title | local path | active
   — Previous active plan for this project (if any) → archived

Plan starts with:
---
[first 8-10 lines of plan content]
---

Proceed?
```

Never write files before user confirms.

### Step 4: Write Files

1. Create `<project>/plans/` directory if it doesn't exist.
2. Write plan content to the local plan file. Prepend a header block:

```markdown
> Source: `~/.claude/plans/<random-slug-name>.md`
> Archived: YYYY-MM-DD · Project: [project name]
> Maintained by [claude-plans-skill](https://github.com/code-katz/claude-plans-skill).

---

[verbatim plan content]
```

If the source was a random-slug file, include the full slug name (e.g., `misty-conjuring-hammock`) — this is the fun part, keep it. If the plan was extracted from conversation with no slug, use `Source: conversation` instead.

3. Read existing `~/.claude/plans/INDEX.md` (create it if it doesn't exist). Update the previously-active row for this project to `archived` or `superseded`. Prepend new row with status `active`.

### Step 5: Confirm

"Saved to `<project>/plans/YYYY-MM-DD-plan-slug.md` and indexed in `~/.claude/plans/INDEX.md`."

No git commit. No push.

---

## Workflow: Listing Plans

When the user asks "what plans do I have" / "plans list" / "show archived plans":

1. Read `~/.claude/plans/INDEX.md`
2. Display the full table — never truncate
3. Offer to load any plan: "Want me to read any of these?"

---

## Workflow: Retrieving a Plan

1. Read INDEX.md and find matching rows (case-insensitive, partial match OK)
2. One match: read and display the local plan file
3. Multiple matches: show matching rows and ask which
4. No match: report not found, offer to search by date or keywords

---

## Workflow: Marking a Plan Executed

Triggers: "the plan is done" / "we shipped it" / "mark the plan executed"

1. Update INDEX.md: change the `active` plan's status to `executed`
2. Optionally add `Executed: YYYY-MM-DD` to the local plan file header
3. Confirm: "Marked `[title]` as executed in the index."
4. Suggest a devlog entry: "Want to log a devlog `milestone` entry referencing this plan?"

---

## Session Start Behavior

When a session begins on a known project:
1. Check INDEX.md for an `active` plan for this project.
2. If found, read it silently — let it inform your work. Do not summarize unless the user asks.
3. If the match is uncertain, ask: "I found a recent plan that might be for this project — [title] from [date]. Should I load it as context?"

### Lint Check

When identifying the project for plan archiving, verify that the project has a linter configured. Check for stack-appropriate lint configuration files:

- **Python**: `ruff.toml`, `pyproject.toml` with `[tool.ruff]`, `.flake8`
- **JavaScript/TypeScript**: `.eslintrc*`, `eslint.config.*`, `biome.json`, or a `lint` script in `package.json`
- **Swift/iOS**: `.swiftlint.yml`
- **Go**: `.golangci.yml`
- **Rust**: `clippy` configuration in `Cargo.toml`
- **General**: `.pre-commit-config.yaml`

If no linter is configured, flag it to the user before proceeding. Recommend: Ruff for Python, ESLint or Biome for JS/TS, SwiftLint for Swift, golangci-lint for Go, clippy for Rust. Frame it as a prerequisite, not an afterthought.

---

## Error Handling

- **Slug file not found** — Fall back to conversation extraction. Notify the user.
- **INDEX.md missing** — Create it fresh.
- **INDEX.md unexpected format** — Show the user and ask whether to overwrite or manually merge.
- **Title/project ambiguous** — Always ask before archiving with incorrect metadata.
- **Project directory unclear** — Ask the user: "Which project directory should this plan live in?"

---

## Style Guidelines

- Local plan filenames use the date of archiving (not plan creation or delivery date)
- Short-title slugs should be meaningful when scanning `<project>/plans/` — not generic
- INDEX.md is always newest-first
- Preserve plan content verbatim — no reformatting, no summarizing
- Project names in INDEX.md should be consistent across rows
- One slug can map to multiple project-local plans (when a plan file covers multiple projects)
