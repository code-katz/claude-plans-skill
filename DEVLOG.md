# claude-plan-skill — Development Log

A living record of architectural decisions, milestones, key insights, and strategic direction.
Auto-maintained via [claude-devlog-skill](https://github.com/d6veteran/claude-devlog-skill). Entries are reverse-chronological.

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
