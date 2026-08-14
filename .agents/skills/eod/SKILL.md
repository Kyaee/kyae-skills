---
name: eod
description: End-of-day report generation for OpenCode work logs. Use when the user asks for an EOD report, daily summary, evidence-based progress report, what they did today, or a report from OpenCode sessions for a specific date, timezone, or project path.
---

# EOD

Generate an evidence-based end-of-day report from OpenCode session data and related read-only Git evidence.

Do not guess. Do not inflate. If the evidence is thin, say so.

## What this skill does

This skill collects OpenCode session metadata, persisted todos, transcript evidence, and read-only Git context for a single local calendar day. It separates evidence collection from synthesis, then produces a concise Markdown report.

Use the environment or user timezone by default. If neither is available, use `Asia/Singapore`.

## Input grammar

Interpret command arguments conceptually in this order:

- Optional bare date: `YYYY-MM-DD`
- Optional timezone override: `timezone=<IANA>`
- Optional project filter: `project=<path>`; repeatable
- Optional file output path: `write=<path>`

Examples:

- `eod`
- `eod 2026-08-15`
- `eod 2026-08-15 timezone=Asia/Seoul`
- `eod project=/repo/a project=/repo/b`
- `eod 2026-08-15 timezone=Europe/Berlin write=/tmp/eod-2026-08-15.md`

Interpretation rules:

1. Resolve the target date first.
2. Resolve the timezone second.
3. Resolve project filters third.
4. Resolve output mode last.

If no date is supplied, use the current local date in the resolved timezone.

If no project is supplied, include all projects discovered in the matching OpenCode sessions.

If `write=<path>` is absent, return the report in chat only.

## Time window

Always resolve one exact local-day window for the report:

- Start: local `YYYY-MM-DD 00:00:00`
- End: next local `00:00:00`
- Interval semantics: `[local 00:00, next local 00:00)`

Always show the resolved timezone and UTC offsets in the report.

Convert the window to epoch milliseconds before querying session metadata.

## Deterministic workflow

Follow this workflow in order. Do not skip steps.

### Phase 1: Resolve scope

1. Resolve timezone from the user or environment.
2. If unavailable, use `Asia/Singapore`.
3. Resolve the target date in that timezone.
4. Compute the exact local-day window and the matching UTC instants.
5. Record the resolved date, timezone, offsets, and window before collecting evidence.

### Phase 2: Collect session candidates

Prefer native OpenCode data.

1. Query OpenCode session metadata with `opencode db --format json`.
2. The session filter must use `time_updated >= start_ms AND time_updated < end_ms`.
3. Include archived sessions. Do not silently exclude them.
4. If `project=<path>` is supplied, keep only sessions whose `directory` matches one of those project paths.
5. Collect session fields including `id`, `parent_id`, `directory`, `title`, `agent`, `model`, `time_created`, and `time_updated`.
6. Collect persisted todos for candidate sessions from the same native source.

### Phase 3: Build session trees

1. Deduplicate sessions by session ID.
2. Group child and subagent sessions under their root session using `parent_id`.
3. If a child session is in scope, include its root session in the grouped result when possible.
4. Preserve child relationships explicitly in the evidence ledger.
5. Do not count the same session twice.

### Phase 4: Collect transcript evidence

1. For each distinct session ID, use `opencode export <session-id>` for transcript detail.
2. Prefer transcript lines and tool results over summaries or titles.
3. Extract user-visible outcomes, decisions, blockers, validation commands, and warnings.
4. Include persisted todos as evidence inputs, but never let a todo alone prove user-facing completion.

### Phase 5: Collect Git context

1. For each distinct session directory, locate the nearest Git root if one exists.
2. Deduplicate Git roots across directories.
3. For each Git root, gather read-only evidence:
   - current branch
   - current status
   - commits whose author time falls inside the same resolved local-day window
4. Treat Git status as current state only, not proof that work happened that day.
5. Record commit hashes and short subjects. Use commit author time for inclusion.

### Phase 6: Build the evidence ledger

Capture raw facts before writing prose. At minimum record:

- target date
- timezone
- exact local and UTC window
- session IDs
- root and child relationships
- directories
- todo states and source order
- Git roots
- branches
- current status
- commits in scope
- validation commands and results
- warnings

Keep collection separate from synthesis. Do not write the final report until the ledger is complete.

### Phase 7: Synthesize the report

Map evidence into the output sections using the hierarchy below. If evidence conflicts, prefer the stronger source and mention the conflict in `Warnings`.

## Evidence hierarchy

Rank evidence from strongest to weakest:

1. Transcript evidence or tool result showing the outcome, or a committed diff in the day window
2. Completed todo corroborated by transcript evidence or Git evidence
3. Current working tree state
4. Session title only

Rules:

- Never call title-only evidence completed work.
- Never present a completed todo as a shipped outcome unless transcript or Git evidence backs it.
- Use current working tree only to describe present state, active work, or possible next steps.
- If a claim is supported only by weak evidence, downgrade it to `In progress`, `Warnings`, or `Sources`.

## Classification rules

### Completed

Put an item in `Completed` only when the evidence shows a user-facing outcome, a concrete fix, a merged or committed change in scope, or a clearly finished task backed by transcript or Git evidence.

### In progress

Use for work that was started, explored, drafted, debugged, or partially validated but not proven complete.

### Decisions

Use for explicit technical or process choices visible in transcripts, tool outputs, or commits.

### Blockers

Use for explicit blockers, failures, waiting states, missing inputs, or unresolved defects documented in evidence.

### Tomorrow

Use only for next actions grounded in explicit todos, stated next steps, unresolved blockers, or current working tree evidence.

### Validation

Include commands and outcomes that were actually run, with pass or fail state when visible.

## Ordering rules

Reruns for the same date must use the same selection and ordering rules:

1. Projects by Git root path when present, otherwise by session directory path
2. Root sessions by `time_created`, then session ID
3. Child sessions by `time_created`, then session ID
4. Commits by author time, then commit hash
5. Todos in source order

Do not reorder for narrative effect.

## Output contract

Return concise Markdown with these sections, in this order:

```markdown
# EOD - YYYY-MM-DD

## Evidence window

## Completed

## In progress

## Decisions

## Blockers

## Tomorrow

## Validation

## Sources

## Warnings
```

Rules:

- Always include `Evidence window` and `Sources`.
- Omit any other section that would be empty.
- Cite session IDs inline.
- Cite commit hashes inline.
- If no evidence exists, say so plainly and do not invent work.
- Keep the report concise.

## Sources section requirements

List the concrete evidence inputs used to build the report, such as:

- root sessions and child sessions by ID
- project directories
- Git roots
- commit hashes
- validation command snippets

Make it easy to audit every claim.

## Safety constraints

This skill is read-only unless the user explicitly supplied `write=<path>` for the report output file.

Never:

- edit repositories
- stage changes
- commit
- push
- delete files
- archive sessions
- mutate session data
- invent accomplishments
- infer completion from titles alone

Do not require Todoist, diary integration, or any external task system.

Only write the final report to disk when `write=<path>` is explicit. Otherwise return it in chat.

## Preferred native commands

Prefer native OpenCode evidence in this order:

1. `opencode db --format json` for session and todo metadata
2. `opencode export <session-id>` for transcript detail
3. read-only Git inspection for branch, status, and commits

If a weaker fallback is ever needed, label it clearly in `Warnings`.

## Failure behavior

If data is missing, incomplete, or contradictory:

1. keep the strongest surviving evidence
2. downgrade uncertain claims
3. explain the gap in `Warnings`
4. never fill gaps with inference presented as fact

If there are zero matching sessions and zero matching commits, produce a short report that says no evidence was found for the resolved window.
