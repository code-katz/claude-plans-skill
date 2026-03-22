# claude-plans-skill — Development Log

A living record of architectural decisions, milestones, key insights, and strategic direction.
Auto-maintained via [claude-devlog-skill](https://github.com/code-katz/claude-devlog-skill). Entries are reverse-chronological.

---

## [2026-03-21] Plans skill v2.0 — project-local archives with global index

**Category:** `refactor`
**Tags:** `plan-skill`, `skill-design`, `v2`, `architecture`
**Risk Level:** `med`
**Breaking Change:** `yes`

### Summary
Redesigned the plans skill from a centralized global archive (`~/.claude/plans/archive/`) to a project-local model where plans live in `<project>/plans/` with a global lookup index at `~/.claude/plans/INDEX.md`.

### Detail
The v1.0 design stored all archived plans in a single global directory (`~/.claude/plans/archive/`). While functional, this decoupled plans from their projects — browsing a project directory didn't reveal its plans, and plans weren't co-located with the code they described.

The v2.0 design:
- Plans are saved to `<project-directory>/plans/YYYY-MM-DD-slug.md`
- Each plan file includes a header referencing back to the original `~/.claude/plans/<random-slug>.md` source
- A global `~/.claude/plans/INDEX.md` serves as a cross-project lookup table mapping slugs → project-local files
- INDEX.md columns: Date, Slug, Project, Title, Local Plan path, Status

The global INDEX.md replaces the per-archive INDEX.md. Plans are now discoverable both locally (browse the project) and globally (search the index).

### Decisions Made
- **Project-local over centralized archive** — Plans are project artifacts, not tool config. They belong with the project they describe. The v1.0 "plans are tool config" framing was wrong.
- **Global INDEX.md at `~/.claude/plans/` (not `archive/`)** — The index is the only global artifact. Archive subdirectory is eliminated.
- **Slug column in INDEX.md** — One slug file can produce multiple project-local plans (e.g., `misty-conjuring-hammock` contained both a Code Katz and d20m plan). The slug column captures this relationship.
- **Source reference in plan header** — Each local plan file references its original slug file, preserving the breadcrumb back to Claude's auto-generated output.

### Related
- [2026-03-20] claude-plan-skill v1.0 — initial design and release (superseded by this entry)
- First plans archived under v2.0: `code-katz/plans/2026-03-21-marketing-messaging.md`, `d20m/plans/2026-03-21-product-analysis.md`

---

## [2026-03-20] claude-plan-skill v1.0 — initial design and release

**Category:** `milestone`
**Tags:** `plan-skill`, `skill-design`, `v1`
**Risk Level:** `low`
**Breaking Change:** `no`

### Summary
Designed and built the first version of the plans archiving skill. Archives finalized Claude Code plan mode output to a global named archive at `~/.claude/plans/archive/` with an INDEX.md index table.

### Detail
Motivation: Claude Code accumulates random-slug plan files in `~/.claude/plans/` with no index or project association. A previous `/plans` skill existed but used a repo-based / git-commit design — plans are Claude config artifacts, not project source files. This skill uses a global archive with no git dependency.

The skill pairs with devlog-skill: plans capture the "how before execution," devlog captures the "what was decided and why."

A small but notable design touch: the original random-slug filename (e.g., `vast-coalescing-cookie`) is preserved in the header of each archived plan file as a breadcrumb back to the source.

### Decisions Made
- **Global archive (`~/.claude/plans/archive/`) over per-project repo** — plans are tool configuration, not source code. The previous skill's repo-based design was the wrong model.
- **No git/commit requirement** — simpler, works without any repo context, appropriate for a global config artifact
- **Two statuses for replaced plans:** `archived` (saw implementation work) vs `superseded` (replaced before that). Default to `archived` when unsure.
- **Prefer reading from `~/.claude/plans/<slug>.md`** over conversation extraction — it's the canonical plan mode output
- **Proactive suggestion at plan finalization, not automatic archiving** — user makes the "this is final" call
- **Preserve original slug in archive header** — the random slug names are fun and serve as a breadcrumb to the source file
