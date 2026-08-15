# QA Ticket Skill Acceptance Tests

This file defines the acceptance test contract for `qa-ticket`.

## Test rules

- Every manual scenario is no-side-effect.
- Do not execute `create`, `--create`, or `mode=create` during manual testing.
- If a scenario mentions create mode, validate it by static skill read-through only.
- Do not call Linear mutation tools while testing this skill.
- Every expected output must still include `## Prepared Ticket Metadata` and a `| Changes | QA |` table with unchecked criteria.

## Static review checks

### T1. Preview is the default

**Goal**
Confirm that `/qa-ticket <parent>` without an explicit create token stays in preview.

**Method**
Read `SKILL.md` and verify the input grammar plus execution rules say preview is the default.

**Expected**

- The accepted mutation tokens are only `create`, `--create`, or `mode=create`.
- Any other phrasing that hints at mutation but is not exact falls back to preview and asks.
- The prepared output format still includes acceptance criteria.

### T2. Explicit create path is documented, not executed

**Goal**
Confirm that the create branch is fully specified without running it.

**Method**
Read `SKILL.md` and verify it names `linear_save_issue` with `team: "Quality Assurance"`, `parentId`, generated title, description table, and append-only PR links.

**Expected**

- The skill uses hierarchy through `parentId`, not `relatedTo`.
- The skill forbids copying assignee, priority, cycle, project, and labels unless requested.
- The skill requires post-create re-fetch verification.

### T3. Exact team resolution

**Goal**
Confirm that the runtime team lookup is exact and not hardcoded to a UUID.

**Method**
Read `SKILL.md` and verify it instructs the model to resolve the team by exact visible name `Quality Assurance` at runtime.

**Expected**

- Zero exact matches blocks before mutation.
- Multiple exact matches blocks before mutation.
- The skill never uses a cached or hardcoded team UUID.

### T4. Duplicate prevention contract

**Goal**
Confirm the duplicate check runs before creation and uses the required fields.

**Method**
Read `SKILL.md` and verify it requires `linear_list_issues` with exact team scope, `parentId`, `includeArchived=false`, `assignee: ""`, and separate priority queries for values 0 through 4. Verify it merges and deduplicates the results, then checks team, parent, normalized title or parent identifier, and PR link.

**Expected**

- A match produces `Status: Duplicate found`.
- The duplicate issue identifier and URL are shown.
- The prepared table is still included.
- No create or update occurs.
- Duplicate lookup runs in preview and create modes before any final response is emitted.
- A failed priority query blocks creation instead of being treated as an empty result.
- Candidate issues are re-fetched before comparing PR attachments.

## Manual preview scenarios

### T5. Fixture sanity check with one matching PR and one unrelated PR

**Prompt**
`/qa-ticket QO-60`

**Fixture assumptions**

- Parent `QO-60` is read-only.
- PR `#5` matches `QO-60`.
- PR `#2` is unrelated.
- Existing child `QA-1` belongs to `Quality Assurance` and has `parentId: QO-60`.

**Expected**

- Preview gathers parent details and PR candidates from allowed evidence sources.
- The skill selects only the PR tied to the parent identifier.
- The skill reports `Duplicate found` because an existing child already matches.
- The output still includes a table with unchecked criteria.

### T6. Multiple PR attachments with one matching parent ID

**Prompt**
`/qa-ticket <PARENT-ID>`

**Setup**
Use a parent that exposes multiple PR candidates in attachments, description, comments, branch metadata, or diff metadata, where only one candidate is tied to the parent by identifier, title, branch, or commit evidence.

**Expected**

- The skill considers all candidate sources.
- It selects the single PR that is grounded in the parent evidence.
- It does not mention unrelated PRs as if they were part of the ticket scope.
- The table criteria are based on parent and selected PR evidence, not filenames alone.

### T7. Missing PR evidence

**Prompt**
`/qa-ticket <PARENT-ID>`

**Setup**
Use a parent where no PR can be found in attachments, description, comments, branch metadata, or diff metadata.

**Expected**

- The skill stays in preview.
- `PR` reports an explicit evidence gap.
- The acceptance criteria table remains present.
- Criteria reference confirmed behavior and call out evidence gaps where needed.

### T8. Ambiguous PR evidence

**Prompt**
`/qa-ticket <PARENT-ID>`

**Setup**
Use a parent with several plausible PRs and no decisive tie from parent identifier, explicit user argument, branch, title, or commits.

**Expected**

- The skill reports blocked or preview ambiguity.
- No create path is taken.
- The table remains present and avoids invented claims.

### T9. Cross-team creation rejection

**Prompt**
`/qa-ticket <PARENT-ID> create`

**Method**
Static review only. Do not run.

**Expected**

- The skill states that cross-team creation may fail on permissions.
- If team lookup, hierarchy, or permissions fail, the run stops and reports the exact failure.
- The skill never falls back to creating the child in the parent team or as an unparented issue.

### T10. Mandatory acceptance criteria formatting

**Prompt**
`/qa-ticket <PARENT-ID>`

**Expected**

- Output begins with `## Prepared Ticket Metadata`.
- The table header is exactly `| Changes | QA |`.
- Each row maps to a user-visible behavior cluster, not a file.
- Each QA cell contains one or more `- [ ]` items joined with `<br>`.
- Each criterion states an action and an expected result.

### T11. Post-create verification contract

**Prompt**
`/qa-ticket <PARENT-ID> --create`

**Method**
Static review only. Do not run.

**Expected**

- The skill requires re-fetching the created issue.
- It verifies team, `parentId`, description table, and PR attachment.
- Any mismatch is reported, not silently repaired.

## Regression checklist

- No scenario allows a manual tester to execute create mode.
- No scenario permits Linear mutation tools.
- No scenario accepts a same-team fallback.
- No scenario allows missing acceptance criteria.
- No scenario allows a title-only duplicate match.
