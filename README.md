# task-update

A Claude Code skill that logs your daily work at the end of each session.

Run `/task-update` at the end of your day. Claude collects everything — tasks it tracked during the session, today's git commits (your commits only), asks how many hours you worked, then distributes time across tasks intelligently. Saves to `~/.claude/worklog.md`. Unfinished tasks carry over to your next session.

## What it does

1. Reads all tasks from the current Claude session
2. Reads today's git commits (filtered to your author email)
3. Asks: **"How many hours did you work today?"**
4. Asks: **"Which tasks are still in progress?"**
5. Estimates effort weights per task and distributes total hours proportionally
6. Appends a structured entry to `~/.claude/worklog.md`
7. Saves in-progress tasks to `~/.claude/worklog-state.json` for carryover

## Example log entry

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
```

## Install

```bash
/plugins install github:Smit-Parekh-Monarch/task-update
```

After installing, type `/task-update` in any Claude Code session.

## Configuration

By default, git commits are filtered to `smitp@monovative.com`. To use your own email, edit `skills/task-update/SKILL.md` and update the `--author` value in step 3.

## Files written

| File | Purpose |
|------|---------|
| `~/.claude/worklog.md` | Your running work log (appended each session) |
| `~/.claude/worklog-state.json` | In-progress task carryover state |
