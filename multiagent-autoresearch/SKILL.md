---
name: multiagent-autoresearch
description: "Coordinate autonomous experiment loops across multiple workers with strict ownership, shared scoring, and synthesis reporting. Use when one worker should orchestrate bounded research tracks in parallel without overlapping edit surfaces."
user_invocable: true
---

# multiagent-autoresearch

Use this skill when a single `autoresearch` loop is too narrow, but you still need disciplined, measurable experimentation.

This is the ClawCorp orchestration layer on top of `autoresearch`.
One orchestrator defines the research lanes. Multiple workers run bounded loops in parallel. Results are synthesized into one decision.

## Preconditions

Do not start until all of these are explicit:

1. Shared objective
2. Shared scoring rule
3. Per-worker ownership boundaries
4. Evaluation cadence
5. Merge or stop criteria

If any one is vague, tighten scope before dispatch.

## Hard rules

- One worker owns one surface.
- No overlapping write sets.
- Every lane must be independently measurable.
- Every lane must log results in a comparable format.
- The orchestrator does synthesis, not parallel editing.

## Best use cases

- compare 2-4 prompt strategies on separate prompt files
- compare several rankers or heuristics in separate modules
- compare several skill variants where each worker owns one `SKILL.md`
- compare architecture directions when each direction can live on its own branch

## Bad use cases

- many workers editing the same route or UI
- tasks with no benchmark or decision rule
- broad product development disguised as "research"

## Suggested structure

Each worker gets:
- one branch or isolated worktree
- one editable surface
- one eval command
- one local results log

The orchestrator maintains:
- a master scoreboard
- a synthesis report
- the final recommendation

## Per-worker contract

Each worker should run a normal `autoresearch` loop inside its lane:

- baseline first
- one experimental change at a time
- keep or discard based on measured outcome
- stop thrashing after repeated crashes

## Scoreboard format

Use TSV or Markdown table, but keep the columns aligned across all lanes.

Minimum fields:

```text
worker	lane	commit	metric	status	description
```

Recommended extra fields:
- `runtime_s`
- `memory_gb`
- `cost_usd`
- `notes`

## Orchestrator loop

1. Define lanes.
2. Assign one worker per lane.
3. Ensure the write sets do not overlap.
4. Have each worker establish a baseline.
5. Collect best-per-lane results.
6. Compare winners using the same metric and complexity lens.
7. Write a synthesis report with recommendation and loser analysis.

## ClawCorp dispatch pattern

Use:
- `autoresearch` for the worker-local loop
- `deep-research-report` when external research is needed before lane design
- `research-gate` if the experiment touches skills, routes, schema, or canonical data flows

## Report output

Save the orchestrator result to:

```text
brain/reports/YYYY-MM-DD-multiagent-autoresearch-<topic>.md
```

Include:
- objective
- lane definitions
- best result per lane
- discarded directions
- recommendation
- whether to convert the winner into a normal PR

## Example asks

- "Run multiagent-autoresearch on three routing prompts. Give each worker one prompt file, one eval harness, and one scoreboard."
- "Use multiagent-autoresearch to compare three bounded skill variants, then recommend one for productization."
- "Have Tour orchestrate parallel autoresearch branches across non-overlapping backend heuristics."
