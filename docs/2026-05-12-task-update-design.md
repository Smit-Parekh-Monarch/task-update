# Design: `/task-update` Skill

**Date:** 2026-05-12  
**Author:** Smit Parekh  
**Status:** Approved

---

## Overview

`/task-update` is a Claude Code skill that logs your daily work at the end of each session. It collects tasks Claude tracked during the session, reads your git commits (filtered to your email only), asks how many hours you worked, then intelligently distributes those hours across tasks by estimated effort. Output is appended to a global `~/.claude/worklog.md` log. Unfinished tasks carry over to the next session.

The skill is published to a public GitHub repo (`task-update`) so others can install and use it.

---

## Goals

- Zero-friction end-of-day logging: one command, two questions
- Accurate time distribution without manual entry per task
- Persistent carryover for multi-day tasks
- Shareable as an installable Claude Code skill

---

## Non-Goals

- Time tracking during the session (only at the end)
- Integration with external tools (Jira, Linear, Asana)
- Billing or invoicing features

---

## Trigger

User types `/task-update` in Claude Code at any point (typically end of session or end of day).

---

## Skill Execution Flow

```
1. Read session task list (TaskList tool — all tasks, any status)
2. Read git log for today, author = smitp@monovative.com only
3. Ask: "How many hours did you work today?"
4. Ask: "Which of these tasks are still in progress?" (show list, user picks)
5. Estimate effort weight per task (S/M/L heuristic based on subject + description)
6. Distribute total hours proportionally across tasks
7. Load ~/.claude/worklog-state.json — prepend any carried-over tasks from prior sessions
8. Append entry to ~/.claude/worklog.md
9. Update ~/.claude/worklog-state.json (store in-progress tasks for carryover)
10. Show summary to user
```

---

## Time Distribution Algorithm

Claude assigns each task an effort weight (1–5) based on:

- Subject keywords: "refactor", "implement", "set up" → higher weight; "fix", "update", "rename" → lower
- Description length and complexity indicators
- Whether task was in-progress (suggests more work)

Hours per task = `(task_weight / sum_of_all_weights) × total_hours`

Minimum allocation: 0.25 hrs per task. If rounding leaves a gap, add remainder to the heaviest task.

Claude shows its reasoning in the output so the user can spot anything obviously wrong before it saves.

---

## Carryover State

File: `~/.claude/worklog-state.json`

```json
{
  "carried_tasks": [
    {
      "subject": "Blog listing + slug pages (ISR)",
      "description": "Full reference implementation with ISR",
      "project": "monarch-web-frontend",
      "started": "2026-05-12",
      "hours_so_far": 3.0
    }
  ]
}
```

- On each `/task-update` run, carried tasks from prior sessions are prepended to the task list with accumulated hours shown
- When user marks a carried task as done, it logs total accumulated hours and clears it from state
- Carried tasks not touched in 7 days are flagged as stale

---

## Log File Format

File: `~/.claude/worklog.md`

Each run appends one entry:

```markdown
## 2026-05-12 | monarch-web-frontend | 9.0 hrs

### Tasks
| Task | Est. Hours | Status |
|------|-----------|--------|
| Set up i18n routing with next-intl | 3.5 hrs | ✅ Done |
| Add active nav highlight | 1.5 hrs | ✅ Done |
| Blog listing + slug pages (ISR) | 3.0 hrs | ⏳ In Progress |
| Fix hero image not rendering | 1.0 hrs | ✅ Done |

### Commits (smitp@monovative.com)
- ebe02ff feat: highlight active nav item based on current route
- 276d88e fix: services hero image not rendering

### Notes
⏳ "Blog listing + slug pages (ISR)" carries over — hours will accumulate.

---
```

---

## Git Commit Filtering

- Command: `git log --since="midnight" --author="smitp@monovative.com" --oneline`
- Only commits from today (since midnight) are included
- If no commits found, the Commits section is omitted with a note: `No commits today.`
- Author email is hardcoded in the skill as `smitp@monovative.com`

---

## GitHub Repo Structure

Repository: `github.com/smitp/task-update` (public)

```
skills/
  task-update/
    SKILL.md          ← installable skill file
docs/
  2026-05-12-task-update-design.md
README.md             ← install instructions + usage guide
```

**Install instructions in README:**

```bash
# In Claude Code
/plugins install github:smitp/task-update
```

---

## SKILL.md Frontmatter

```yaml
---
name: task-update
description: Use when logging work at the end of a Claude Code session — collects session tasks, today's git commits (filtered by author), asks for total hours, distributes time across tasks by effort, and appends to a global worklog with carryover for in-progress tasks
---
```

---

## Edge Cases

| Scenario | Handling |
|----------|----------|
| No tasks in session | Show message: "No tasks tracked this session. Add tasks manually?" |
| No git repo in cwd | Skip commits section, note it in log |
| User gives 0 hours | Prompt again: "Did you mean to log 0 hours?" |
| All tasks in-progress | Log entry with all ⏳, full hours distributed, all carried over |
| Worklog file doesn't exist | Create it with a header before first entry |

---

## Files Created/Modified

| File | Location | Purpose |
|------|----------|---------|
| `SKILL.md` | `skills/task-update/` | The installable skill |
| `README.md` | repo root | Install + usage docs |
| `~/.claude/worklog.md` | Local global | Appended log entries |
| `~/.claude/worklog-state.json` | Local global | Carryover state |
