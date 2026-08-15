---
name: qa-ticket
description: /qa-ticket, QA tickets, Linear parent issues, linked PRs, and acceptance criteria. Use when the user wants a Quality Assurance child issue prepared or explicitly created from a Linear parent issue plus PR evidence, with a deterministic Changes and QA table of unchecked acceptance criteria.
---

# QA Ticket

Prepare or explicitly create a Linear child issue in the exact team `Quality Assurance` from a main issue and linked PR evidence.

The default is preview. Mutation is allowed only when the user uses one of these exact tokens: `create`, `--create`, or `mode=create`.

If the user's intent to mutate is unclear, stay in preview and ask.

## Input grammar

Accepted invocation shape:

```text
/qa-ticket <parent-identifier-or-url> [<pr-url-or-number>|pr=<url-or-number>] [title="..."] [commit=<reference>] [create|--create|mode=create]
```

Rules:

- The parent identifier or URL is required.
- The PR argument is optional and may be a bare PR URL or number, or `pr=<url-or-number>`.
- `title="..."` is optional and overrides the generated title suffix only.
- `commit=<reference>` is optional.
- If `commit=` is omitted, the commit reference defaults to the parent identifier.
- Preview is the default.
- Explicit create mode is accepted only as `create`, `--create`, or `mode=create`.
- If the user says something vague like "make it", "go ahead", or "open one" without an exact create token, stay in preview and ask.

## Required workflow

Follow this sequence. Do not skip steps.

1. Resolve the target team at runtime with a Linear team lookup for the exact visible name `Quality Assurance`.
2. If zero or multiple exact visible matches are returned, stop before mutation and report the issue.
3. Fetch the parent issue with `linear_get_issue`, including relations.
4. Read comments when they may clarify user-visible behavior, PR linkage, or acceptance criteria. Attempt the issue-comment lookup at most once. If the Linear wrapper rejects an issue-only request because of status-update parameter validation, record the evidence gap and continue without retrying alternate status-update types.
5. Collect PR candidates from all relevant evidence sources:
   - parent attachments
   - parent description
   - parent comments
   - parent branch metadata
   - Linear diff metadata
   - explicit user PR argument, if provided
6. If a selected PR exists, inspect it with `linear_get_diff`.
7. When available and needed for behavior details, inspect the PR with:
   - `gh pr view --json title,url,state,mergedAt,headRefName,baseRefName,commits,files`
   - `gh pr diff`
8. Do not claim user-visible behavior from filenames alone.
9. Build the ticket content from parent acceptance criteria, PR commit subjects, tests, diff content, and any confirmed comments.
10. Run duplicate prevention in preview and create modes before emitting the final response.
11. In create mode only, create the child with `linear_save_issue`.
12. After creation, re-fetch the created issue and verify team, parent, description table, and PR attachment.

## Team resolution

- Always resolve the target team by exact visible name `Quality Assurance` at runtime.
- Never hardcode the current team UUID.
- Never use any team wording other than `Quality Assurance` in the output.
- If the lookup returns zero exact matches, stop with `Status: Blocked`.
- If the lookup returns multiple exact matches, stop with `Status: Blocked`.

## Parent and PR evidence rules

You must ground the QA ticket in evidence.

### Parent issue

- Fetch the parent with relations enabled.
- Preserve the exact parent identifier and URL in the output.
- Read comments when useful for missing context, customer-visible behavior, repro steps, or clarified scope.

### PR selection

When there are multiple PR candidates, choose only one PR whose branch, title, commits, or explicit user argument ties it to the parent identifier.

Acceptable ties include:

- the parent identifier in the PR title
- the parent identifier in the branch name
- commit subjects that reference the parent identifier
- explicit user-supplied PR URL or number that matches the inspected PR
- Linear diff metadata that clearly binds the PR to the parent

If several PRs remain plausible after inspection, stay in preview, report the ambiguity, and do not create.

If no PR can be confirmed, stay in preview and report the evidence gap.

Never treat unrelated PRs as supporting evidence.

## Duplicate prevention

Run duplicate prevention in every mode. Do not emit a final preview, blocked result, duplicate result, or created result until this search completes.

Use `linear_list_issues` with:

- exact team scope `Quality Assurance`
- `parentId` set to the parent identifier
- `includeArchived=false`
- `assignee: ""` so assigned and unassigned issues remain in scope

The Linear wrapper treats `priority: 0` as the `None` priority, not as all priorities. Run the same team and parent query once for each priority value `0`, `1`, `2`, `3`, and `4`. Merge the five result sets and deduplicate by issue ID. If any priority query fails, set `Status: Blocked` and do not create because duplicate prevention is incomplete.

Re-fetch every candidate with `linear_get_issue` before comparing its description and PR attachments.

Then verify each candidate with all relevant evidence:

- returned team is `Quality Assurance`
- returned `parentId` matches the parent identifier
- normalized title matches the generated title or clearly embeds the same parent identifier
- PR link matches the selected PR URL when a PR is present

If a matching child exists:

- set `Status: Duplicate found`
- show the duplicate child identifier and URL
- retain the prepared acceptance criteria table
- do not create or update anything

Do not rely on title-only duplicate matching.

## Ticket generation rules

Generation must be deterministic.

### Title

- The title must begin with `QA: <PARENT-ID> `.
- If `title="..."` is present, append the user title after that prefix.
- Otherwise derive a short suffix from the parent title or confirmed user-visible behavior.
- Preserve exact identifiers.

### Description body

Every output, even preview, blocked, missing PR, ambiguous PR, and duplicate cases, must include the prepared ticket body.

The output must start with:

```md
## Prepared Ticket Metadata
```

Then include these lines in this order:

- `Status: Preview|Created|Duplicate found|Blocked`
- `Title: ...`
- `Parent: <identifier> <url if known>`
- `Team: Quality Assurance`
- `Commit reference: ...`
- `PR: <number and URL>`, or an explicit evidence gap

Then include the table header exactly as:

```md
| Changes | QA |
```

Build one row per user-visible behavior cluster, not per file.

Rules for rows:

- Order rows by user flow first.
- Put cross-cutting responsive, accessibility, and error criteria after the main flow rows.
- Merge duplicate behavior into one row.
- Every `Changes` cell must describe a concrete behavior change.
- Every `QA` cell must contain one or more unchecked acceptance criteria using `- [ ]`.
- Use `<br>` inside table cells when more than one criterion is needed.
- Escape table-breaking pipes.
- Every criterion must state an action and expected result.
- Include exact routes, viewports, keyboard actions, accessibility state, error state, data state, or permission state only when the evidence supports them.
- Do not invent changes or criteria unsupported by the parent and PR evidence.
- If evidence is thin, keep the row honest and note the gap in the criterion.

### Evidence note after the table

Add a short evidence or attachment note after the table.

- Request screenshots only for visual or responsive changes.
- When screenshots are needed, name only evidence-grounded viewports.
- Request logs or payloads only when the failure mode is nonvisual and relevant.

## Create mode

Use create mode only when the user supplied `create`, `--create`, or `mode=create` exactly.

Create the issue with `linear_save_issue` using:

- `team: "Quality Assurance"`
- `parentId: <parent identifier>`
- generated title
- full description body containing the prepared metadata and the Markdown table
- `links` containing the selected PR URL and PR title when present

Rules:

- Use `parentId`, not `relatedTo`, to create the hierarchy.
- Do not copy assignee, priority, cycle, project, or labels unless the user explicitly requested them.
- If cross-team creation fails, permission fails, or hierarchy creation fails, stop and report the exact failure.
- Never create an unparented fallback.
- Never create a fallback in the parent team.

## Post-create verification

After creation, re-fetch the created issue and verify:

- team is `Quality Assurance`
- `parentId` matches the requested parent
- the description still contains the prepared table and metadata
- the PR attachment is present when one was selected

If any mismatch appears, report it exactly. Do not silently repair extra fields.

## Output discipline

- Always display acceptance criteria, even in blocked or duplicate results.
- Do not reveal credentials.
- Do not reveal unrelated private issue content.
- Keep the language concrete.
- Do not use em dashes.
- Avoid filler phrasing.

## Minimal output skeleton

```md
## Prepared Ticket Metadata
Status: Preview
Title: QA: QO-60 Example behavior coverage
Parent: QO-60 https://linear.example/QO-60
Team: Quality Assurance
Commit reference: QO-60
PR: #5 https://github.com/example/repo/pull/5

| Changes | QA |
| --- | --- |
| Authenticated user can submit the updated form from `/settings/profile` and sees the new confirmation state. | - [ ] Open `/settings/profile` as an authenticated user and submit valid changes, then confirm the new confirmation state appears.<br>- [ ] Submit invalid input and confirm the validation message matches the described error state. |
| Responsive layout and keyboard flow remain usable for the updated screen. | - [ ] At `1440px` and `390px`, open `/settings/profile` and confirm the updated controls remain visible without overlap.<br>- [ ] Use keyboard only to tab through the updated controls and confirm focus order and activation still work. |

Evidence note: Based on parent acceptance criteria, PR commits, and reviewed diff. Please attach screenshots only if the change is visual or responsive.
```
