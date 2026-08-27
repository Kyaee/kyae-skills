---
name: eod
description: End-of-day report generation from Codex and OpenCode work logs. Use when the user asks for an EOD report, daily summary, evidence-based progress report, what they did today, or a report for a specific date, timezone, or project path.
---

# EOD

Generate an evidence-based end-of-day report from Codex and OpenCode session data and related read-only Git evidence.

Do not guess. Do not inflate. If the evidence is thin, say so.

## What this skill does

This skill collects Codex and OpenCode session metadata, persisted todos, transcript evidence, and read-only Git context for a single local calendar day. It separates evidence collection from synthesis, then produces a concise Markdown report.

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

If no project is supplied, include all projects discovered in matching Codex and OpenCode sessions.

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

### Phase 2: Collect OpenCode session candidates

Prefer native OpenCode data.

1. Query OpenCode session metadata with `opencode db "SELECT ..." --format json`.
2. The session filter must use `time_updated >= start_ms AND time_updated < end_ms`.
3. Include archived sessions. Do not silently exclude them.
4. If `project=<path>` is supplied, keep only sessions whose `directory` matches one of those project paths.
5. Collect session fields including `id`, `parent_id`, `directory`, `title`, `agent`, `model`, `time_created`, and `time_updated`.
6. Collect persisted todos for candidate sessions from the same native source.

`opencode db --format json` without a SQL query opens an interactive shell; never use it for collection. Compute the epoch bounds first, then pass a read-only query as the positional argument. The installed schema stores session metadata in `session` and todos in `todo`:

```sh
opencode db "SELECT id, parent_id, directory, title, agent, model, time_created, time_updated, time_archived
  FROM session
  WHERE time_updated >= START_MS AND time_updated < END_MS
  ORDER BY time_created, id" --format json

opencode db "SELECT t.session_id, t.content, t.status, t.priority, t.position, t.time_created, t.time_updated
  FROM todo AS t
  INNER JOIN session AS s ON s.id = t.session_id
  WHERE s.time_updated >= START_MS AND s.time_updated < END_MS
  ORDER BY s.time_created, s.id, t.position" --format json
```

Replace `START_MS` and `END_MS` with decimal epoch-millisecond values, not user-provided SQL. For each `project=<path>`, add `AND directory IN (...)` to the session query and `AND s.directory IN (...)` to the todo query. Quote each path as a SQLite string literal, escaping a single quote as two single quotes. If Phase 3 needs an out-of-window root session, fetch it by its already-discovered ID with a separate `WHERE id IN (...)` query; do not widen the day filter.

### Phase 3: Collect Codex session candidates

Codex rollout transcripts are an additional native evidence source. Read them when available; an EOD for a specific date must not depend only on OpenCode records.

1. Derive the UTC calendar date or dates that overlap the resolved local-day window.
2. Find candidate `rollout-*.jsonl` files beneath `~/.codex/sessions/` by searching for a top-level record timestamp prefix such as `"timestamp":"2026-08-27T`. Directory and filename dates are discovery hints only; select evidence by each record's ISO-8601 `timestamp` after parsing it.
3. For a candidate, read its `session_meta` record even if that record is outside the window. Record `payload.session_id`, `payload.id`, `payload.parent_thread_id`, and `payload.cwd` when present. Use `session_id` as the thread identifier and retain `id` as a child or subagent identifier.
4. Retain only `event_msg`, `turn_context`, and `response_item` records whose top-level timestamp falls inside `[start, end)`. A session with metadata but no in-window records is not a matching Codex session.
5. If `project=<path>` is supplied, retain a Codex session only when its recorded `cwd` exactly matches a supplied project path. A missing `cwd` does not match the filter.
6. Deduplicate by transcript path and thread identifier. Group child or subagent records under `parent_thread_id` or `session_id` when available, and do not count a thread twice.

Do not treat `~/.codex/memories/MEMORY.md` or rollout summaries as proof of completed work. They may locate a transcript, but the matching JSONL record is the evidence source. Do not scan or quote Codex base instructions, developer instructions, skill definitions, approval policies, or tool schemas. Do not reproduce secrets or raw sensitive command arguments from a transcript.

### Phase 4: Build session trees

1. Deduplicate sessions by session ID.
2. Group child and subagent sessions under their root session using `parent_id`.
3. If a child session is in scope, include its root session in the grouped result when possible.
4. Preserve child relationships explicitly in the evidence ledger.
5. Do not count the same session twice.

### Phase 5: Collect transcript evidence

1. For each distinct session ID, use `opencode export <session-id>` for transcript detail.
2. Prefer transcript lines and tool results over summaries or titles.
3. Extract user-visible outcomes, decisions, blockers, validation commands, and warnings.
4. Include persisted todos as evidence inputs, but never let a todo alone prove user-facing completion.
5. For each matching Codex transcript, use in-window `response_item` messages and tool results as context. Treat a successful tool result, a visible validation result, or a final assistant handoff as evidence; a user request, assistant plan, title, or system context alone is not proof of completion.
6. Cite Codex evidence with its thread ID and rollout filename. Use the record ordinal when it is needed to distinguish multiple claims from the same transcript.

### Phase 6: Collect Git context

1. For each distinct session directory, locate the nearest Git root if one exists.
2. Deduplicate Git roots across directories.
3. For each Git root, gather read-only evidence:
   - current branch
   - current status
   - commits whose author time falls inside the same resolved local-day window
4. Treat Git status as current state only, not proof that work happened that day.
5. Record commit hashes and short subjects. Use commit author time for inclusion.

### Phase 7: Build the evidence ledger

Capture raw facts before writing prose. At minimum record:

- target date
- timezone
- exact local and UTC window
- session IDs
- root and child relationships
- directories
- Codex thread IDs, child IDs, rollout paths, and in-window record ordinals
- todo states and source order
- Git roots
- branches
- current status
- commits in scope
- validation commands and results
- warnings

Keep collection separate from synthesis. Do not write the final report until the ledger is complete.

### Phase 8: Synthesize the report

Map evidence into the output sections using the hierarchy below. If evidence conflicts, prefer the stronger source; omit the claim when the conflict cannot be resolved.

## Evidence hierarchy

Rank evidence from strongest to weakest:

1. Codex or OpenCode transcript evidence or tool result showing the outcome, or a committed diff in the day window
2. Completed todo corroborated by transcript evidence or Git evidence
3. Current working tree state
4. Session title only

Rules:

- Never call title-only evidence completed work.
- Never present a completed todo as a shipped outcome unless transcript or Git evidence backs it.
- Use current working tree only to describe present state, active work, or possible next steps.
- If a claim is supported only by weak evidence, omit it from `Completed`. Put it in `Next` only when there is an explicit, evidence-backed next action.

## Classification rules

### Completed

Put an item in `Completed` only when the evidence shows a user-facing outcome, a concrete fix, a merged or committed change in scope, or a clearly finished task backed by transcript or Git evidence.

### Next

Use only for explicit todos, stated next steps, unresolved blockers, or current working tree evidence. Do not add generic follow-up work.

## Ordering rules

Reruns for the same date must use the same selection and ordering rules:

1. Projects by Git root path when present, otherwise by session directory path
2. Root sessions by `time_created`, then session ID
3. Child sessions by `time_created`, then session ID
4. Codex threads by first in-window record timestamp, then rollout path
5. Commits by author time, then commit hash
6. Todos in source order

Do not reorder for narrative effect.

## Output contract

Return concise Markdown with these sections, in this order:

```markdown
# EOD - YYYY-MM-DD

## Completed

## Next
```

Rules:

- Always include `Completed`.
- Include `Next` only when there is at least one evidence-backed action.
- Keep citations, validation details, and uncertainty in the evidence ledger rather than adding report sections.
- If no evidence exists, say so plainly and do not invent work.
- Keep the report concise.

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

Prefer native evidence in this order:

1. matching `~/.codex/sessions/**/rollout-*.jsonl` records for Codex context
2. `opencode db "SELECT ..." --format json` for OpenCode session and todo metadata; every database collection command must include a read-only SQL query
3. `opencode export <session-id>` for OpenCode transcript detail
4. read-only Git inspection for branch, status, and commits

If a weaker fallback is ever needed, do not use it as proof of completion.

## Failure behavior

If data is missing, incomplete, or contradictory:

1. keep the strongest surviving evidence
2. downgrade uncertain claims
3. include a next action only when the evidence explicitly provides one
4. never fill gaps with inference presented as fact

If there are zero matching Codex sessions, zero matching OpenCode sessions, and zero matching commits, produce a short report that says no evidence was found for the resolved window.
