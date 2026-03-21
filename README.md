# claude-plan-skill

A Claude Code skill that archives finalized implementation plans to a named, indexed global archive — so you can actually find "the plan for project X" months later.

---

## The Problem

Claude Code plan mode saves output to `~/.claude/plans/<random-slug>.md`. These files accumulate with no project association, no index, and no way to find a specific plan later. The slug names are delightful (`vast-coalescing-cookie`, `dreamy-nibbling-boole`) but not searchable.

## What It Does

- **Archives** finalized plans from `~/.claude/plans/<slug>.md` to `~/.claude/plans/archive/YYYY-MM-DD-project-name.md`
- **Indexes** all archived plans in a single `~/.claude/plans/archive/INDEX.md` table across all projects
- **Retrieves** plans by project name or date on demand
- **Loads** the active plan for a project silently at session start as context
- **Tracks** plan lifecycle: `active` → `archived` / `superseded` / `executed`
- **Preserves** the original random slug in each archive file header (the fun part)

No git. No project repo required. Plans are Claude config artifacts — they live in `~/.claude/`, not your codebase.

## Archive Structure

```
~/.claude/plans/                    — Claude Code auto-generated (random slugs, untouched)
~/.claude/plans/archive/            — Named, indexed archive (managed by this skill)
  INDEX.md                          — Master index table across all projects
  YYYY-MM-DD-project-slug.md        — Archived plans with header + verbatim content
```

Each archived file starts with:
```
> Archived: 2026-03-20 · Project: nvoss-dashboard · Original slug: `vast-coalescing-cookie`
> Source: `~/.claude/plans/vast-coalescing-cookie.md`
> Maintained by claude-plan-skill
```

## Status Values

| Status | Meaning |
|---|---|
| `active` | Current plan being executed against |
| `archived` | Superseded after implementation work began |
| `superseded` | Replaced before execution started |
| `executed` | Fully implemented and delivered |

## Installation

```bash
mkdir -p ~/.claude/skills/plans
curl -o ~/.claude/skills/plans/SKILL.md \
  https://raw.githubusercontent.com/code-katz/claude-plan-skill/main/SKILL.md
```

Or clone and install manually:
```bash
git clone https://github.com/code-katz/claude-plan-skill
cp claude-plan-skill/SKILL.md ~/.claude/skills/plans/SKILL.md
```

## Usage

Trigger explicitly:
- `"archive this plan"` / `"save this plan"` / `/plans`
- `"what plans do I have"` / `"plans list"`
- `"find the plan for [project]"`
- `"mark the plan executed"`

Or let it trigger proactively — at the end of a finalized plan-mode session, it will suggest archiving before you move to execution.

## Pairs With

- [claude-devlog-skill](https://github.com/d6veteran/claude-devlog-skill) — plans capture the "how before execution," devlog captures the "what was decided and why"
- [claude-roadmap-skill](https://github.com/d6veteran/claude-roadmap-skill) — roadmap tracks what's coming; plans capture the implementation detail when it's time to build

## Repository Contents

| File | Purpose |
|---|---|
| `SKILL.md` | The skill — install this to `~/.claude/skills/plans/SKILL.md` |
| `DEVLOG.md` | Development log for this project |
| `README.md` | This file |
