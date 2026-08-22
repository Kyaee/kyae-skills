# Anthony Skill Acceptance Tests

This file defines the acceptance test contract for `anthony`.

## Test rules

- Every manual scenario is no-side-effect.
- Do not call Linear mutation tools while testing this skill.
- Any scenario that describes issue creation is validated by static skill read-through only.
- Every expected output must include `## Prepared Ticket Metadata` and a prepared issue description with `## Acceptance Criteria` and unchecked criteria.

## Static review checks

### T1. Automatic creation is the standard workflow

**Goal**
Confirm that `/anthony` creates once the request data is adequate.

**Method**
Read `SKILL.md`.

**Expected**

- Invocation is treated as authorization to create.
- The skill does not require a separate mutation keyword.
- The workflow proceeds from input resolution to duplicate prevention to `linear_save_issue`.

### T2. `no-ai-slop` is loaded before drafting

**Goal**
Confirm the skill explicitly requires `no-ai-slop` before title, description, or criteria generation.

**Method**
Read `SKILL.md`.

**Expected**

- The workflow explicitly loads the native `no-ai-slop` skill tool first.
- The rule appears before writing title, description, or criteria.
- Output discipline bans filler phrasing and em dash usage.

### T3. Consolidated question behavior

**Goal**
Confirm missing details are requested in one consolidated question only when they cannot be inferred safely.

**Method**
Read `SKILL.md`.

**Expected**

- The skill asks at most one consolidated question.
- That question covers context, urgency, labels, status, project, and deadline.
- The skill does not ask for assignee or cycle.
- The skill does not require details that can be safely inferred.
- After the answer, the workflow continues to creation if the data is adequate.

### T4. Requester assignment contract

**Goal**
Confirm the requester is always the assignee.

**Method**
Read `SKILL.md`.

**Expected**

- The skill resolves the requester with `linear_get_user(query: "me")`.
- Every `linear_save_issue` call uses that resolved result as `assignee`.
- The skill never uses a literal user ID.

### T5. Existing cycle selection contract

**Goal**
Confirm the skill always uses an existing current or next cycle.

**Method**
Read `SKILL.md`.

**Expected**

- The skill always calls `linear_list_cycles` for the resolved team.
- It prefers an existing current cycle.
- It falls back to an existing next cycle only when no current cycle exists.
- It blocks if neither exists.
- It never creates or invents a cycle.

### T6. Existing metadata resolution

**Goal**
Confirm labels, statuses, and projects are resolved from existing Linear metadata only.

**Method**
Read `SKILL.md`.

**Expected**

- The skill queries existing labels with `linear_list_issue_labels`.
- The skill queries existing statuses with `linear_list_issue_statuses`.
- The skill queries projects with `linear_list_projects` when context suggests one.
- The skill never creates labels, projects, statuses, teams, or cycles.

### T7. Duplicate prevention contract

**Goal**
Confirm the skill checks for duplicates in the same team and cycle before mutation.

**Method**
Read `SKILL.md`.

**Expected**

- The skill runs `linear_list_issues` before mutation.
- The query is restricted to the resolved team and selected cycle.
- A likely duplicate blocks creation.
- The output reports `Status: Duplicate found` and still includes the prepared ticket body.

### T8. Criteria format contract

**Goal**
Confirm the acceptance criteria format is deterministic and grounded.

**Method**
Read `SKILL.md`.

**Expected**

- The prepared description contains `## Acceptance Criteria`.
- Every criterion uses unchecked `- [ ]` format.
- Every criterion states an action and expected result.
- The skill forbids unsupported requirements.

### T9. Post-create verification contract

**Goal**
Confirm creation requires immediate re-fetch verification.

**Method**
Read `SKILL.md`.

**Expected**

- The skill re-fetches the created issue.
- It verifies requester assignment, cycle, labels, status, project, deadline, and the acceptance criteria section.
- Any mismatch is reported, not silently repaired.

## Manual scenarios

### T10. Direct arguments act as request context and lead to creation

**Prompt**
`/anthony delayed delivery status visibility for order screen`

**Expected**

- The free-text invocation becomes the request context.
- The skill resolves what it safely can.
- Once the data is adequate, the workflow creates the issue without a second confirmation step.

### T11. Missing details trigger one consolidated question, then creation

**Prompt**
`/anthony`

**Expected**

- The skill asks one consolidated question.
- The question covers context, urgency, labels, status, project, and deadline.
- The skill does not ask separate follow-up questions for each field.
- After the user reply provides adequate data, the workflow creates the issue.

### T12. Ambiguous team blocks creation

**Prompt**
`/anthony inventory discrepancy`

**Setup**
Assume the workspace context exposes multiple plausible teams and no explicit `team=` value.

**Expected**

- The skill reports `Status: Blocked`.
- No mutation occurs.
- The output explains that team resolution is ambiguous.

### T13. Unresolved optional project is omitted

**Prompt**
`/anthony improve stock alert wording project=Ops`

**Setup**
Assume `Ops` does not resolve to exactly one existing project.

**Expected**

- The output omits the project.
- The metadata section discloses the unresolved project omission.
- The skill does not block unless the user clearly required an exact project.

### T14. Explicit status must resolve exactly

**Prompt**
`/anthony improve stock alert wording status=Active`

**Setup**
Assume the resolved team does not have an exact existing status named `Active`.

**Expected**

- The skill blocks before mutation.
- The output explains that the explicit status does not resolve exactly.

### T15. Duplicate prevention blocks creation

**Prompt**
`/anthony delayed delivery status visibility`

**Method**
Static review only. Do not run.

**Expected**

- The skill checks for existing likely duplicates in the same team and cycle first.
- A likely duplicate results in `Status: Duplicate found`.
- No mutation occurs.

### T16. Creation path includes verification

**Prompt**
`/anthony delayed delivery status visibility`

**Method**
Static review only. Do not run.

**Expected**

- The creation path uses team, title, description, assignee, cycle, priority, labels, state, and any resolved project or due date.
- The created issue is re-fetched immediately.
- Assignment, cycle, labels, status, project, deadline, and criteria section are verified.

## Regression checklist

- No scenario requires a separate mutation keyword.
- No scenario allows creation without an existing cycle.
- No scenario allows literal user ID assignment.
- No scenario allows invented labels, statuses, projects, teams, or cycles.
- No scenario allows acceptance criteria without unchecked action-plus-result items.
