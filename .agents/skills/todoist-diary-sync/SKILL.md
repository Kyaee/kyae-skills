---
name: todoist-diary-sync
description: Use when the user says "update my to-do list" or asks to add/change Todoist tasks and wants the same to-do list reflected in today's diary inputs.
---

# Todoist Diary Sync

Use this skill whenever the user says "update my to-do list" or otherwise asks to add, change, reschedule, complete, or prioritize Todoist tasks. The expected behavior is: update Todoist first, then update today's diary inputs with the same canonical Todoist data.

## Intent

- Treat Todoist as the source of truth for task content, due dates, priorities, labels, project names, IDs, URLs, and completion state.
- After a Todoist update succeeds, mirror the resulting task list into today's diary using the existing `/read-diary` workflow.
- The user's phrase "update my to-do list" is explicit permission to update diary inputs for today's note after the Todoist change. It is not permission to commit or push.

## Workflow

1. Parse the user's requested task changes:
   - task title/content,
   - date and time,
   - priority,
   - project/section/labels if provided,
   - completion or reschedule intent if applicable.
2. If a task update is ambiguous in a way that changes the result, ask one precise question before modifying Todoist.
3. Apply the Todoist change with the Todoist tools.
4. Confirm the created or updated Todoist task data from the tool response. If the response omits details needed for the diary, fetch the affected task or today's task list.
5. Update today's diary inputs by following `/read-diary` behavior:
   - resolve today's daily note path as `DIARY/{YYYY}/{MM EnglishMonthName}/{MM-DD-YYYY}.md`,
   - read or create today's note if needed,
   - fetch today's Todoist data with `todoist_find-tasks-by-date` using `startDate: "today"`, `daysCount: 1`, and `overdueOption: "include-overdue"`,
   - fetch completed tasks for the same day when relevant,
   - write or refresh the `# TODOIST 1:1` section using exact Todoist strings and metadata.
6. Preserve the diary command's privacy and git rules:
   - do not expose unrelated vault content,
   - never invent Todoist metadata,
   - never commit or push automatically,
   - if diary files changed, report the changed file and ask before any commit/push.

## Diary Format

Use the `/read-diary` command's canonical section format when updating the note:

```markdown
# TODOIST 1:1
---
## Active / Due Today + Overdue
- [ ] <exact Todoist task content> - id: `<id>`; project: `<project>`; priority: `<p1-p4>`; due: `<exact due>`; url: <url>

## Completed Today
- [x] <exact Todoist task content> - id: `<id>`; completed: `<exact timestamp/date>`; project: `<project>`; url: <url>
```

If the existing diary note already has a `# TODOIST 1:1` section, replace only that section unless the user explicitly asks for broader diary edits.

## Output

Keep the final response concise:

- summarize the Todoist tasks changed,
- name the diary note updated,
- mention that no commit or push was performed unless the user explicitly confirmed it.
