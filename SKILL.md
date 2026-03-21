---
name: plans
description: Archive finalized Claude Code implementation plans to a named, indexed global archive at ~/.claude/plans/archive/. Use this skill whenever the user says "archive this plan", "save this plan", "/plans", "/plans archive", "finalize plan", or "this plan is done". Also use for "what plans do I have", "show archived plans", "plans list", "find the plan for [project]". Trigger proactively at the end of any plan-mode session that produced a finalized implementation plan — suggest archiving before the session ends. Works globally across all projects — not tied to any specific repo or git workflow.
---

# Plans Archive Skill

This skill archives finalized Claude Code implementation plans from `~/.claude/plans/<random-slug>.md` to a named, indexed global archive. It keeps plans findable, paired with their project, and available as context in future sessions.

This skill is **global** — it works across all projects and requires no Git repo or commit workflow. Archives live in `~/.claude/plans/archive/`.

## Why This Matters

Claude Code auto-saves plan mode output to `~/.claude/plans/<random-slug>.md`. These files accumulate with no project association, no index, and no way to retrieve "what was the plan for project X last month?" Plans capture the full rationale and constraints of an implementation effort. They pair naturally with DEVLOG.md: plans capture the "how before execution," devlog captures the "what was decided and why."

---

## Archive Location

```
~/.claude/plans/                    — Claude Code auto-generated plans (random slugs, do not modify)
~/.claude/plans/archive/            — Named, indexed archive (managed by this skill)
  INDEX.md                          — Master index table across all projects
  YYYY-MM-DD-project-slug.md        — Individual archived plan files (verbatim)
```

No git. No project repo required.

---

## INDEX.md Format

```markdown
# Plans Archive

| Date | Project | Title | File | Status |
|---|---|---|---|---|
| 2026-03-20 | nvoss-dashboard | Label sync + triage improvements | [2026-03-20-nvoss-label-sync.md](2026-03-20-nvoss-label-sync.md) | `active` |
```

Newest row first. Status values:

| Status | Meaning |
|---|---|
| `active` | Current plan for this project, being executed against |
| `archived` | Superseded by a newer plan after execution work began |
| `superseded` | Replaced before execution started |
| `executed` | Fully implemented and delivered |

---

## Archive Filename Format

`YYYY-MM-DD-project-slug.md` — date of archiving, 3-5 word lowercase-hyphenated slug.

Examples:
- `2026-03-20-nvoss-label-sync.md`
- `2026-02-10-canopy-auth-redesign.md`

---

## Trigger Conditions

### Explicit
- "archive this plan" / "save this plan" / "plans save"
- `/plans` / `/plans archive`
- "finalize plan" / "this plan is done" / "this plan is final"
- "plans list" / "what plans do I have" / "show archived plans"
- "find the plan for [project]" / "show me the plan for [project]"

### Proactive
At the end of a plan-mode session that produced a finalized implementation plan — before the user moves to execution — suggest: "This plan looks finalized — want me to archive it? It'll be named and indexed in `~/.claude/plans/archive/` so you can find it by project later."

Do NOT trigger proactively for exploratory or in-progress plans, partial outlines, or plans explicitly marked as drafts.

---

## Workflow: Archiving a Plan

### Step 1: Locate Plan Content

In order:
1. **Random-slug file** — If the current plan slug is known, read `~/.claude/plans/<slug>.md` directly. Preferred.
2. **Conversation extraction** — Extract the most recent structured implementation plan from the thread.
3. **Ask** — "Where is the plan? I can read `~/.claude/plans/<slug>.md` if you have the filename, or extract it from this conversation."

### Step 2: Determine Project Name and Title

Extract the plan title from the first H1 heading. Derive:
- **Project name** — short human-readable identifier for INDEX.md (e.g., "nvoss-dashboard", "canopy-portal")
- **Short slug** — lowercase-hyphenated, 3-5 words from the title, for the filename

Suggest both and confirm: "I'll archive this as `2026-03-20-nvoss-label-sync.md` under project `nvoss-dashboard` — correct?"

### Step 3: Show the User (before writing anything)

```
Here's what I'm about to do:

1. Write plan to ~/.claude/plans/archive/YYYY-MM-DD-project-slug.md
2. Update ~/.claude/plans/archive/INDEX.md
   — New row: YYYY-MM-DD | project | title | file | active
   — Previous active plan for this project (if any) → archived

Plan starts with:
---
[first 8-10 lines of plan content]
---

Proceed?
```

Never write files before user confirms.

### Step 4: Write Files

1. Create `~/.claude/plans/archive/` if it doesn't exist.
2. Write plan content verbatim to the archive file. Prepend a header block:

```markdown
> Archived: YYYY-MM-DD · Project: [project name] · Original slug: `[random-slug-name]`
> Source: `~/.claude/plans/[slug].md` (or "conversation" if extracted from thread)
> Maintained by [claude-plan-skill](https://github.com/code-katz/claude-plan-skill).

---

[verbatim plan content]
```

If the source was a random-slug file, include the full slug name (e.g., `vast-coalescing-cookie`) — this is the fun part, keep it. If the plan was extracted from conversation with no slug, omit the slug line.

3. Read existing `INDEX.md` (create it if it doesn't exist). Update the previously-active row for this project to `archived` or `superseded`. Prepend new row with status `active`.

### Step 5: Confirm

"Archived to `~/.claude/plans/archive/YYYY-MM-DD-project-slug.md` and indexed."

No git commit. No push.

---

## Workflow: Listing Plans

When the user asks "what plans do I have" / "plans list" / "show archived plans":

1. Read `~/.claude/plans/archive/INDEX.md`
2. Display the full table — never truncate
3. Offer to load any plan: "Want me to read any of these?"

---

## Workflow: Retrieving a Plan

1. Read INDEX.md and find matching rows (case-insensitive, partial match OK)
2. One match: read and display the archive file
3. Multiple matches: show matching rows and ask which
4. No match: report not found, offer to search by date or keywords

---

## Workflow: Marking a Plan Executed

Triggers: "the plan is done" / "we shipped it" / "mark the plan executed"

1. Update INDEX.md: change the `active` plan's status to `executed`
2. Optionally add `Executed: YYYY-MM-DD` to the archive file header
3. Confirm: "Marked `[title]` as executed in the index."
4. Suggest a devlog entry: "Want to log a devlog `milestone` entry referencing this plan?"

---

## Session Start Behavior

When a session begins on a known project:
1. Check INDEX.md for an `active` plan for this project.
2. If found, read it silently — let it inform your work. Do not summarize unless the user asks.
3. If the match is uncertain, ask: "I found a recent plan that might be for this project — [title] from [date]. Should I load it as context?"

---

## Error Handling

- **Slug file not found** — Fall back to conversation extraction. Notify the user.
- **INDEX.md missing** — Create it fresh.
- **INDEX.md unexpected format** — Show the user and ask whether to overwrite or manually merge.
- **Title/project ambiguous** — Always ask before archiving with incorrect metadata.

---

## Style Guidelines

- Archive filenames use the date of archiving (not plan creation or delivery date)
- Short-title slugs should be meaningful when scanning `~/.claude/plans/archive/` — not generic
- INDEX.md is always newest-first
- Preserve plan content verbatim — no reformatting, no summarizing
- Project names in INDEX.md should be consistent across rows
