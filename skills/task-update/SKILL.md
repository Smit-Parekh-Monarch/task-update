---
name: task-update
description: Use when logging work at the end of a Claude Code session — collects session tasks, today's git commits filtered by your author email, asks for total hours worked, distributes time across tasks by estimated effort, and saves to a global worklog with carryover for in-progress tasks
---

# Task Update

Log your daily work in one command. At end of session, Claude collects everything — tasks, commits, hours — and writes a structured entry to `~/.claude/worklog.md`.

## Steps

1. **Load carried tasks.** Read `~/.claude/worklog-state.json`. If it exists, prepend any in-progress tasks from prior sessions to the list, showing their accumulated hours.

2. **Read session tasks.** Use the TaskList tool to get all tasks from this session (any status).

3. **Read today's commits.** Run:
   ```sh
   git log --since="midnight" --author="YOUR_EMAIL" --oneline
   ```
   Replace `YOUR_EMAIL` with the configured author email (default: `smitp@monovative.com`). If not in a git repo or no commits today, skip this section.

4. **Ask for hours.**
   > "How many hours did you work today?"

5. **Ask which tasks are still in progress.** Show the full task list and ask the user to identify which ones are not done yet.

6. **Estimate effort weights.** For each task, assign a weight 1–5:
   - Weight 5: "implement", "set up", "build", "create", "refactor" — complex work
   - Weight 3: "add", "update", "configure", "integrate" — moderate work
   - Weight 1: "fix", "rename", "remove", "tweak" — small changes
   - In-progress tasks get +1 to their weight (they took longer than expected)
   - Use description length and complexity as secondary signal

7. **Distribute hours.** `hours_per_task = (task_weight / total_weight) × total_hours`. Minimum 0.25 hrs per task. Add any rounding remainder to the heaviest task. Show the breakdown to the user before saving.

8. **Write log entry.** Append to `~/.claude/worklog.md`:

```markdown
## YYYY-MM-DD | <project-name> | X.X hrs

### Tasks
| Task | Est. Hours | Status |
|------|-----------|--------|
| Task subject here | 2.5 hrs | ✅ Done |
| Another task | 1.0 hrs | ⏳ In Progress |

### Commits (author@email.com)
- abc1234 feat: commit message here

### Notes
⏳ "Another task" carries over — hours will accumulate.

---
```

   Use the current working directory name as `<project-name>`. If `~/.claude/worklog.md` doesn't exist, create it with the header `# Work Log\n\n` first.

9. **Update carryover state.** Write `~/.claude/worklog-state.json` with in-progress tasks, including `hours_so_far` (today's allocation). If a previously carried task is now marked done, remove it from state and log its total accumulated hours.

10. **Confirm to user.** Show the saved file path and a one-line summary: "Logged X tasks (Y done, Z in progress) — X.X hrs to `~/.claude/worklog.md`."

## Carryover State Format

```json
{
  "carried_tasks": [
    {
      "subject": "Task subject",
      "description": "Task description",
      "project": "project-name",
      "started": "YYYY-MM-DD",
      "hours_so_far": 2.5
    }
  ]
}
```

## Edge Cases

| Scenario | Handling |
|----------|----------|
| No tasks in session | Ask: "No tasks tracked — describe what you worked on, or type 'skip'." |
| Not in a git repo | Skip commits section, add note: `No git repo detected.` |
| No commits today from author | Add note: `No commits today.` |
| User enters 0 hours | Prompt: "Did you mean to log 0 hours? Enter a number or type 'skip'." |
| All tasks in-progress | Log all as ⏳, carry all over, distribute hours across them |
| Stale carried task (7+ days) | Flag it: "⚠️ Carried over from YYYY-MM-DD — still relevant?" |

## Configuration

To change the author email, edit the skill and update the `--author` value in step 3. Future versions may support a config file.
