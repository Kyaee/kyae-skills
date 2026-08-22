---
name: anthony
description: /anthony, Linear issue intake, requester assignment, cycle selection, and acceptance criteria. Use when the user wants a Linear issue created with resolved team metadata, grounded urgency, and unchecked criteria.
---

# Anthony

Create a Linear issue for the requesting user with resolved team metadata, an existing active cycle, and grounded acceptance criteria.

Before writing a title, description, or acceptance criteria, explicitly load the native `no-ai-slop` skill tool and follow it.

## Input grammar

Accepted invocation shape:

```text
/anthony [request context words...] [team=<team>] [project=<project>] [status=<status>] [labels=<comma-separated-labels>] [deadline=<YYYY-MM-DD|natural text>] [urgency=<low|medium|high|urgent>]
```

Rules:

- Treat direct invocation arguments as request context.
- Invocation is authorization to create once the request data is adequate.
- Do not require the user to supply a field that can be safely inferred from explicit context or authoritative Linear metadata.
- If material information is still missing after inference, ask one consolidated question that covers context, urgency, labels, status, project, and deadline.
- After the user answers that one question, continue the same workflow and create if the data is now adequate.
- Do not ask for separate confirmation.

## Required workflow

Follow this sequence. Do not skip steps.

1. Load `no-ai-slop` through the native skill tool before drafting title, description, or criteria.
2. Parse invocation arguments. Treat leftover free text as the request context.
3. Resolve the requester with `linear_get_user(query: "me")`.
4. Use that result as `assignee` in every `linear_save_issue` call. Do not use a literal user ID.
5. Resolve exactly one team from explicit input, surrounding context, or a clearly unambiguous workspace default.
6. If no single team can be resolved, stop before mutation.
7. Query existing statuses with `linear_list_issue_statuses` for the resolved team.
8. Query existing labels with `linear_list_issue_labels` for the resolved team when labels are explicit or context suggests likely matches.
9. Query existing projects with `linear_list_projects` when project input or context suggests one.
10. Query existing cycles with `linear_list_cycles` for the resolved team. Select an existing current cycle. If there is no current cycle, use an existing next cycle. If neither exists, stop before mutation.
11. Infer urgency only from explicit language, issue context, or authoritative Linear metadata. If urgency remains unclear, default to priority `3`.
12. Resolve status only from the existing team status list. Use an exact explicitly requested status when supplied. Otherwise infer one valid open or default status. If status cannot be resolved unambiguously, stop before mutation.
13. Resolve labels only from the existing team label list. Use only exact existing values. Never create labels.
14. Resolve a project only when explicit input or context matches exactly one existing project. If project resolution is optional and ambiguous, omit it and disclose that omission in output. If the user explicitly requires a project and it cannot be resolved, stop before mutation.
15. Resolve a deadline only from explicit input, authoritative parent or issue metadata, milestone dates, or unambiguous text. Omit unknown deadlines.
16. If material information still cannot be inferred safely, ask one consolidated question. After the reply, reuse the new information and continue instead of restarting the flow.
17. Build the prepared title and full prepared description.
18. Run duplicate prevention with `linear_list_issues`, restricted to the resolved team and selected cycle. Never create a likely duplicate.
19. Call `linear_save_issue` with team, title, description, assignee, cycle, priority, labels, state, and any resolved project or due date.
20. Immediately re-fetch the created issue and verify requester assignment, cycle, labels, status, project, deadline, and the acceptance criteria section.

## Team resolution

- Prefer explicit `team=` input.
- Otherwise infer a team from clear context only when one existing team is unambiguous.
- A workspace default is allowed only when it is clearly unambiguous.
- If multiple teams are plausible, stop with `Status: Blocked` and ask for the team.
- Never create a team.

## Request context and question behavior

Use request context to infer the ticket goal, area, affected workflow, and likely acceptance checks.

If material details are still missing after checking arguments and authoritative context, ask one consolidated question in this shape:

```text
I can create the ticket, but I still need a few details in one reply: context, urgency, labels, status, project, and deadline. Use `unknown` for anything you want me to infer or omit.
```

Rules:

- Ask at most one consolidated question.
- Do not ask for the assignee. The requester is always the assignee.
- Do not ask for the cycle. Resolve it from the team.
- Do not ask for labels, project, status, or deadline when one exact existing value is already resolved safely.
- After the user answers, create if the remaining required data is resolved and safe.

## Requester assignment

- Always resolve the requester with `linear_get_user(query: "me")`.
- Always pass that resolved user result as `assignee` in `linear_save_issue`.
- Never substitute a literal UUID or hardcoded user string.
- If requester resolution fails, stop before mutation.

## Cycle selection

- Always call `linear_list_cycles` for the resolved team.
- Use an existing current cycle when one exists.
- If there is no current cycle, use an existing next cycle.
- If neither current nor next exists, stop with `Status: Blocked`.
- Never create or invent a cycle.

## Existing metadata resolution

### Labels

- Use `linear_list_issue_labels` for the resolved team.
- Use only exact existing labels.
- If the user explicitly requested labels and one or more do not resolve exactly, stop before mutation.
- If labels came only from soft context and cannot resolve exactly, omit them and disclose that omission in output.

### Status

- Use `linear_list_issue_statuses` for the resolved team.
- An explicitly requested status must resolve exactly to an existing status.
- Otherwise infer one valid open or default status from the team list.
- If more than one open or default candidate remains and no single choice is clearly authoritative, stop before mutation.
- Never create a status.

### Project

- Use `linear_list_projects` only when explicit input or context suggests a project.
- Use a project only when it resolves to exactly one existing project.
- If project is unresolved but optional, omit it and disclose the omission.
- If the user required a specific project and it cannot be resolved exactly, stop before mutation.
- Never create a project.

### Priority

Urgency maps to the existing Linear priority number.

- urgent -> `1`
- high -> `2`
- medium -> `3`
- low -> `4`

Rules:

- Infer urgency only from explicit language, authoritative Linear metadata, or strong context.
- If uncertainty remains, default to medium priority `3`.
- Do not confuse priority with status.

### Deadline

- Use an explicit deadline when provided.
- Infer only from authoritative parent, issue, or milestone due dates, or unambiguous text.
- Omit if unknown.
- Never invent dates.

## Duplicate prevention

Run duplicate prevention before any mutation.

Use `linear_list_issues` with:

- the resolved team
- the selected cycle
- `includeArchived=false`

Compare candidate issues against the prepared request using title overlap, issue purpose, context keywords, project, and status context. If a likely duplicate exists in the same team and cycle, stop with `Status: Duplicate found`.

Rules:

- Do not create or update when likely duplicate evidence is strong.
- Show the matching issue identifier and URL when available.
- Keep the prepared title and description in the output.
- If duplicate checking cannot run, stop before mutation.

## Ticket generation rules

Generation must stay grounded in the resolved context and existing metadata.

### Title

- Keep the title short and concrete.
- Base it on the actual request context.
- Do not use filler phrasing.
- Do not use fake identifiers or generic placeholders.

### Description body

Every output, including duplicate, blocked, and created cases, must include the full prepared description.

The output must start with:

```md
## Prepared Ticket Metadata
```

Then include these lines in this order:

- `Status: Created|Duplicate found|Blocked`
- `Team: ...`
- `Cycle: ...`
- `Requester assignee: ...`
- `Priority: <number> (<reason>)`
- `Status value: ...`
- `Labels: ...`, or an explicit omission note
- `Project: ...`, or an explicit omission note
- `Deadline: ...`, or `Deadline: omitted`
- `Context summary: ...`

Then include a title line:

- `Title: ...`

Then include the prepared issue description exactly in this structure:

```md
## Context
...

## Acceptance Criteria
- [ ] ...
```

Acceptance criteria rules:

- Use unchecked `- [ ]` items only.
- Every criterion must state an action and expected result.
- Ground each criterion in the resolved context.
- Avoid unsupported requirements.
- If context is thin, keep the criterion narrow and honest.
- Include status, deadline, label, or project checks only when they are part of the resolved request.

## Creation step

When the request data is adequate, create the issue with `linear_save_issue` using:

- `team: <resolved team>`
- generated title
- full prepared description
- `assignee: <result from linear_get_user(query: "me")>`
- `cycle: <selected existing cycle>`
- `priority: <resolved priority number>`
- `labels: <resolved existing labels>` when any exist
- `state: <resolved existing status>`
- `project: <resolved project>` only when exactly resolved
- `dueDate: <resolved explicit or authoritative deadline>` only when resolved

Rules:

- Never create labels, projects, statuses, teams, or cycles.
- Never create without a resolved team and existing cycle.
- Never create when status resolution is ambiguous.
- Never create when an explicitly required project or label set does not resolve.
- Never ask for separate confirmation once the data is adequate.

## Post-create verification

After creation, re-fetch the created issue and verify:

- requester assignment matches the resolved requester
- cycle matches the selected existing cycle
- labels match the resolved existing labels
- status matches the resolved existing status
- project matches the resolved project when one was used
- deadline matches the resolved due date when one was used
- the description contains `## Acceptance Criteria`

If any mismatch appears, report it exactly. Do not silently repair extra fields.

## Output discipline

- Creation is the standard completed workflow when the data is adequate.
- Disclose every omission clearly.
- Keep language direct.
- Do not use em dashes.
- Avoid filler phrasing and invented requirements.

## Minimal output skeleton

```md
## Prepared Ticket Metadata
Status: Created
Team: Operations
Cycle: 2026 Cycle 18
Requester assignee: me
Priority: 3 (defaulted to medium because urgency was not explicit)
Status value: Backlog
Labels: omitted, no exact existing label resolved
Project: omitted, no exact existing project resolved
Deadline: omitted
Context summary: Customer follow-up issue for delayed delivery status visibility on the order screen.
Title: Add delivery status visibility for delayed orders

## Context
Customer support needs a ticket for delayed delivery status visibility on the order screen so the requester can track the missing state and expected outcome.

## Acceptance Criteria
- [ ] Open an order with a delayed delivery state and confirm the order screen shows the delayed status clearly.
- [ ] View an order without a delayed delivery state and confirm the existing status display remains unchanged.
```
