---
name: autoresearch
description: "Run Karpathy-style autonomous experiment loops inside a small repo or bounded subsystem. Creates a dedicated research branch, establishes a baseline, iterates one file or surface at a time, records results, and keeps only improvements."
user_invocable: true
kind: adapted
upstream: https://github.com/karpathy/autoresearch
---

# autoresearch

Use this skill when the user wants an autonomous research loop, not a one-shot implementation.
This is a ClawCorp adaptation of `karpathy/autoresearch` for bounded engineering experiments.

Source of the pattern:
- `https://github.com/karpathy/autoresearch`
- See `references/upstream-notes.md` for the local summary of the upstream repo and `program.md`.

## When to use

Use for:
- model tuning loops
- prompt or agent-policy iteration with a measurable score
- repeated benchmark-driven refactors
- narrow architecture searches in a small repo or subsystem
- any task where "run experiment, measure, keep or discard" is the real workflow

Do not use for:
- open-ended product work with no metric
- broad multi-directory refactors with unclear rollback points
- tasks that cannot be measured quickly
- tasks where every run is expensive, slow, or risky to production state

## Core adaptation

The upstream repo keeps the loop intentionally tight:
- one bounded code surface
- one measurable metric
- one experiment branch
- one baseline run first
- every iteration is either kept or discarded

ClawCorp must preserve those invariants.

## Required setup

Before the first experiment:

1. Define the metric.
2. Define the exact editable surface.
3. Define the evaluation command.
4. Define the experiment log file.
5. Create a dedicated research branch.

If any of those are ambiguous, stop and tighten scope before looping.

## Branch rule

Create a dedicated branch in this style:

```bash
git checkout -b autoresearch/<topic>-<yyyymmdd>
```

Do not run the loop on `dev`, a release branch, or an already noisy feature branch.

## Editable-surface rule

Prefer exactly one editable file.
If one file is too strict, allow one bounded surface such as:
- one route module
- one prompt file
- one benchmark target
- one agent skill

Avoid letting the experiment sprawl across unrelated files.

## Baseline-first rule

The first run is always the untouched baseline.

Record:
- branch
- starting commit
- metric value
- runtime
- memory or cost if relevant
- plain-language description: `baseline`

Never compare new experiments against memory or intuition.
Compare against the logged baseline or current best.

## Experiment loop

Run this loop:

1. Inspect the current best state.
2. Make one experimental change inside the approved surface.
3. Commit the experiment.
4. Run the evaluation command and redirect output to a log file.
5. Extract the metric.
6. Record the result in the experiment log.
7. Keep the change only if it beats the current best, or if it is materially simpler at equal performance.
8. If it regresses, revert only the experiment commit and continue.

## Logging format

Default to TSV so it stays grep-able:

```text
commit	metric	status	description
abc1234	0.912300	keep	baseline
def5678	0.907100	keep	reduce eval noise and simplify scoring
987fedc	0.913800	discard	expand context window
```

Recommended optional columns:
- `runtime_s`
- `memory_gb`
- `cost_usd`
- `notes`

Do not commit temporary logs unless the user asked for a durable artifact.

## Keep/discard criteria

Keep when:
- metric improves
- metric is flat but the implementation is clearly simpler
- metric is flat but runtime or cost improves meaningfully

Discard when:
- metric regresses
- complexity increases for marginal or noisy gains
- the run crashes
- the result is not reproducible

## Crash handling

If the run crashes:

1. Read the tail of the log.
2. Decide whether the crash is from the experiment itself or environment drift.
3. Attempt a small fix only if it stays within the bounded surface.
4. After a few failed recovery attempts, stop and report the blocker instead of thrashing.

## ClawCorp guardrails

- Use `research-gate` first if the experiment touches skills, protocols, routing, schema, or canonical data flows.
- Prefer one owned worker for the loop. Do not let multiple agents edit the same experiment surface concurrently.
- Do not spam the repo with dead-end artifacts. Keep logs in a local scratch path or a clearly named report path.
- If the loop produces a real recommendation, save the synthesis in `brain/reports/`.
- If the work becomes productized, convert the winner into a normal task and PR flow.

## Suggested file layout

For substantial loops:

```text
brain/reports/YYYY-MM-DD-autoresearch-<topic>.md
tmp/autoresearch/<topic>/results.tsv
tmp/autoresearch/<topic>/run.log
```

## Good prompts

- "Use autoresearch on this prompt-eval harness. Optimize only `prompts/router.md` for win-rate, log every run, and keep only improvements."
- "Use autoresearch on this benchmark target. Only modify `server/lib/ranker.js`, run the benchmark after each commit, and track latency plus accuracy."
- "Use autoresearch on this skill. Keep the surface bounded to one `SKILL.md` and score by task success on the provided eval set."

## Bad prompts

- "Autonomously improve the whole product."
- "Keep trying random ideas until something feels better."
- "Refactor everything and compare later."

## Final output expectation

At the end of an autoresearch session, report:
- best commit
- best metric
- discarded directions
- whether the gain appears robust or noisy
- the exact next step: keep iterating, convert to PR, or stop
