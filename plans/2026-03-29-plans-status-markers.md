> Source: `~/.claude/plans/mighty-greeting-kay.md`
> Archived: 2026-03-29 · Project: claude-plans-skill
> Status: `executed`
> Maintained by [claude-plans-skill](https://github.com/code-katz/claude-plans-skill).

---

# Plan: Add Status Markers and Plan Review to claude-plans-skill

## Context

The plans skill tracks status in INDEX.md (active/archived/superseded/executed), and session start already filters to active-only. But plan *files* themselves have no status marker. If Claude reads a plan file directly (via a path in a devlog entry, roadmap reference, or conversation), there's no signal it's stale. Additionally, there's no "rejected" status for plans that were created but never acted on, and no mechanism to periodically review whether active plans are still current.

**Target repo**: `/Users/willcurran/d20m-development/code-katz/claude-plans-skill/`
**File to modify**: `SKILL.md` (227 lines)

## Changes to SKILL.md

### 1. Add "rejected" status (modify status table, line ~55-59)

Add a fifth status value:

| Status | Meaning |
|---|---|
| `active` | Current plan, being executed against |
| `archived` | Superseded by a newer plan after execution work began |
| `superseded` | Replaced before execution started |
| `executed` | Fully implemented and delivered |
| `rejected` | Created but never acted on; no longer relevant |

### 2. Add status marker to plan file header (modify header template, line ~132-142)

Update the header block prepended during archiving to include a `Status` line:

```markdown
> Source: `~/.claude/plans/<random-slug-name>.md`
> Archived: YYYY-MM-DD · Project: [project name]
> Status: `active`
> Maintained by [claude-plans-skill](https://github.com/code-katz/claude-plans-skill).
```

When a plan's status changes in INDEX.md, the file header must also be updated. This is the key fix: if Claude reads the file directly without consulting INDEX.md, the status line tells it immediately whether to trust this plan.

### 3. Add status change instructions for file headers (new subsection in "Marking a Plan Executed" workflow, and add parallel workflows for other transitions)

Expand the existing "Marking a Plan Executed" workflow and add new workflows:

**Workflow: Changing Plan Status**

Triggers:
- "mark the plan executed" / "we shipped it" / "the plan is done" -> `executed`
- "this plan is superseded" / "we replaced this plan" -> `superseded`
- "reject this plan" / "we're not doing this" / "this plan is dead" -> `rejected`
- "archive this plan" (when a newer plan exists) -> `archived`

For any status change:
1. Update INDEX.md: change the plan's status
2. Update the plan file's `> Status:` header line to match
3. Confirm to user with old and new status

For `executed` specifically, also suggest a devlog milestone entry (existing behavior).

### 4. Add "Reading Plans" guidance (new section after Session Start Behavior)

**When reading a plan file directly** (not via INDEX.md):
- Check the `> Status:` line in the file header first
- If status is `executed`, `superseded`, `archived`, or `rejected`: note it and do not treat this as the current approach. It is historical context only.
- If status is `active`: treat as current plan
- If no status line exists (legacy plan files): check INDEX.md for the plan's status before acting on it

### 5. Add plan review during archiving (modify Step 4 of archiving workflow, line ~146)

When archiving a new plan (Step 4), after updating the previous active plan's status:
- Scan INDEX.md for any other `active` plans for the same project
- If found, flag them to the user: "There are N other active plans for this project. Should any of these be marked as executed, superseded, or rejected?"
- List the active plans with their titles and dates
- User decides per-plan; update both INDEX.md and file headers accordingly

This is the automatic review mechanism. It piggybacks on plan archiving so it's not a separate chore.

## Execution Order

Single file edit to `SKILL.md`:
1. Add `rejected` status to the status table
2. Add `> Status:` line to the plan file header template
3. Expand "Marking a Plan Executed" into a general "Changing Plan Status" workflow
4. Add "Reading Plans" guidance section
5. Add plan review step to the archiving workflow

## Verification

1. Read updated SKILL.md end-to-end for consistency
2. Test by running `/plans` and checking that new archiving prompts for status review
3. Verify a plan file with `> Status: executed` is skipped when Claude reads it directly
