<p align="center">
  <img src="publish/claude-plans-skill-header.svg" alt="claude-plans-skill" width="100%">
</p>

# claude-plans-skill

> Find that plan you wrote three months ago.

![License](https://img.shields.io/badge/license-MIT-blue) ![Works with Claude Code](https://img.shields.io/badge/works%20with-Claude%20Code-8A2BE2) ![Skill](https://img.shields.io/badge/type-skill-orange)

---

## The Problem

Claude Code plan mode is excellent. You describe the problem, Claude reads the codebase, and you get a structured implementation plan. Then the session ends.

The plan lives in `~/.claude/plans/misty-conjuring-hammock.md`. Or was it `vast-coalescing-cookie.md`? Three projects, twelve plans, all in the same directory with charming random names and no project association. You know the plan exists. You just can't find it.

Two months later, someone asks why you chose that approach. The reasoning was in the plan — the constraints, the rejected alternatives, the sequencing logic. But you'd have to open every slug file to find it. So you don't. You re-plan from scratch.

---

## Who This Is For

Developers who use Claude Code plan mode regularly and work across multiple projects.

If you've ever scrolled through `~/.claude/plans/` trying to remember which `dreamy-nibbling-boole` was the auth redesign and which was the data migration — this is for you.

Works especially well alongside [claude-devlog-skill](https://github.com/code-katz/claude-devlog-skill). Plans capture the "how before execution." The devlog captures the "what was decided and why." Together, they give every future session the full picture.

---

## See the Difference

You need the implementation plan from last month.

**Without the plans skill:**

> **You:** What was the plan for the auth redesign? We wrote one a few weeks ago.
>
> **Claude:** I don't have access to previous plans from other sessions. Could you describe the auth redesign requirements and I'll help create a new plan?

**With the plans skill:**

> **You:** What was the plan for the auth redesign?
>
> **Claude:** Found it — `plans/2026-02-10-auth-redesign.md`, archived from `vast-coalescing-cookie`. Status: active. The plan chose JWT with short-lived tokens over session cookies, sequenced the migration in three phases, and flagged the concurrent-invalidation edge case as an open risk. Want me to load the full plan?

No re-planning. No guessing which slug file. The plan is indexed, named, and retrievable.

---

## What It Does

- **Archives** finalized plans from `~/.claude/plans/<slug>.md` to your project's `plans/` directory with a meaningful name
- **Indexes** all archived plans in a global `~/.claude/plans/INDEX.md` — one table across all projects
- **Retrieves** plans by project name, date, or keywords on demand
- **Loads** the active plan for a project silently at session start as context
- **Tracks** plan lifecycle: `active` → `archived` / `superseded` / `executed`
- **Preserves** the original random slug in each archive file header (the fun part)
- **Suggests archiving** proactively at the end of finalized plan-mode sessions

No git required. Plans are Claude config artifacts — they live alongside your project, indexed globally.

---

## Plan Lifecycle

| Status | Meaning |
|---|---|
| `active` | Current plan being executed against |
| `archived` | Superseded after implementation work began |
| `superseded` | Replaced before execution started |
| `executed` | Fully implemented and delivered |

---

## Installation

This skill is for [Claude Code](https://claude.ai/code). Install it once and it's available across all your projects:

```bash
mkdir -p ~/.claude/skills/plans
curl -o ~/.claude/skills/plans/SKILL.md \
  https://raw.githubusercontent.com/code-katz/claude-plans-skill/main/SKILL.md
```

---

## Usage

Once installed, the skill activates automatically in Claude Code. You can:

- **Archive a plan** — say "archive this plan", "save this plan", or `/plans`
- **List all plans** — say "what plans do I have" or "plans list"
- **Find a specific plan** — say "find the plan for [project]"
- **Mark a plan done** — say "mark the plan executed" or "we shipped it"
- **Let Claude suggest** — at the end of a finalized plan session, it will offer to archive before you move to execution

---

## Works Well With

| Project | What it does |
|---|---|
| [claude-team-cli](https://github.com/code-katz/claude-team-cli) | Ten specialist personas for Claude Code — plans archive the implementation approach each specialist helped design |
| [claude-devlog-skill](https://github.com/code-katz/claude-devlog-skill) | Structured development changelog — plans capture "how before execution," the devlog captures "what was decided and why" |
| [claude-roadmap-skill](https://github.com/code-katz/claude-roadmap-skill) | Living product roadmap — the roadmap sets the priorities, plans capture how each priority gets built |
| [claude-todo-skill](https://github.com/code-katz/claude-todo-skill) | Lightweight task scratchpad — capture ideas mid-session without losing your train of thought |
| [claude-publish-agent](https://github.com/code-katz/claude-publish-agent) | Publish markdown to blogging platforms — write about what you've built and ship it from the terminal |

---

## Repository Contents

| File | Purpose |
|---|---|
| `SKILL.md` | The skill source file — Claude's instructions for how plan archiving works |
| `DEVLOG.md` | Development log for this project |
| `README.md` | This file |

---

## License

See [LICENSE](LICENSE) for details.
