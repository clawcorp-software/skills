---
name: recall
description: "Recover context from a crashed or past session. Reads transcripts bottom-up, cross-references against current state, produces a condensed summary. Use after a crash, a long pause, or when switching workers."
user_invocable: true
---

# Recall -- Session Context Recovery

When a session crashes, gets interrupted, or you need to understand what happened in a past session, this skill recovers the context.

## When to Use
- Agent boots and HANDOFF is stale (older than current date)
- Keymaster says "what happened last session?"
- Worker crashed mid-task and needs to resume
- Keymaster wants a summary of what another worker did
- After a long pause between sessions
- After `/compact` if memory feels thin

## Usage
```
/recall                    -- recover YOUR last session
/recall forge              -- recover forge's last session
/recall all                -- summary of all workers' last sessions
/recall forge 3            -- recover forge's last 3 sessions
```

## THE ONE RULE: READ BOTTOM-UP

**Never read a transcript top-down for recall.** The most recent messages carry current state; older messages carry historical pain that may already be resolved. Top-down reading surfaces 14h-old complaints as if they were still open, because the resolution happened later in the transcript and you haven't reached it yet.

Walk the transcript in reverse chronological order (newest -> oldest). Build a `closed_items` set as you go. Only surface something as "pending" if you have NOT seen a resolution signal more recently than the complaint.

## Resolution signals (when you see these, mark the topic `closed`)
- `[MERGED]` in #status or any channel
- Commit hashes in assistant messages (`fix(x):`, `feat(y):`, etc.) with subject matching the topic
- `gh pr merge --merge --admin` success output
- Task notifications: `completed:success` for a build/deploy
- User acknowledgments: "perfect", "parfait", "good", "merci", "ok on continue", "DONE"
- `[DONE:<agent>]` in any channel
- PR body "This PR closes #N" where #N matches the complaint

## Process

### Step 1: Find the transcript
Find the JSONL file for the session:
```bash
ls -lt "$HOME/.claude/projects/C--Users-user-clawcorp-tower"/*.jsonl | head -3
```
Most-recently-modified = current/last session. Pick the filename (UUID.jsonl).

### Step 2: Walk it BACKWARDS
Read the JSONL line by line in reverse order. Use Node:
```bash
node -e "const fs=require('fs');const lines=fs.readFileSync('<PATH>','utf8').split(/\r?\n/).filter(Boolean).reverse();/* walk lines here */"
```

Build two sets:
- `closed_items` = Set of topic keywords seen in resolution signals (bottom-up)
- `pending_items` = Array of user complaints NOT yet in `closed_items` at the time they appear

When you encounter a user complaint (going backward), check `closed_items`. If any of its keywords matched a later resolution signal, skip it. Otherwise add to `pending_items`.

### Step 3: Cross-reference current state
For every item that survives Step 2, run a sanity check against the repo NOW:
- If the complaint mentions a specific UI file: `Grep` the file for the feature keywords. If present -> closed.
- If it mentions a feature/route: `git log --oneline --since=<complaint_timestamp> | grep -i <keyword>`. If a fix commit exists -> closed.
- If it mentions a PR number: `gh pr view <N> --json state`. If MERGED -> closed.

**Only items that survive both Step 2 AND Step 3 are truly pending.**

### Step 4: Present the summary
```
[RECALL: WORKER -- DATE -- bottom-up, verified]

## Last Shipped (confirmed via commits/merges)
- commit|PR|ts: what -- verification

## Currently Pending (survived reverse-chrono + git/grep check)
- ts: what -- verification attempted: <grep/git/gh> -> not found

## Context for Next Session
- critical info the next session needs

## HANDOFF Status
- FRESH | STALE | NEEDS UPDATE
```

**Hard rule:** NEVER list an item under "Currently Pending" without citing the verification that confirmed it's still pending. `"verification: git log --since grep -i bundle -> 0 hits"` etc. If you didn't verify, don't say it's pending -- say "Unclear, needs check".

### Step 5: Offer to update HANDOFF
If the HANDOFF is stale, ask:
> "HANDOFF is stale (DATE). Update it with this summary?"

If yes, rewrite LATEST-HANDOFF.md.

## For "recall all"
Dispatch one ghost per active worker in parallel. Each ghost MUST follow the bottom-up + verification process above -- NOT a raw summary. Pass the resolution-signals list into the ghost prompt.

## Rules
- Read bottom-up. Always.
- Never surface a complaint as pending without citing verification.
- If a complaint's topic has a resolution signal anywhere more recent than the complaint, the complaint is CLOSED.
- Ghost librarian/analyst outputs are advisory -- the agent still runs Step 3 verification itself.
- Keep summaries under 500 words per worker.
- This skill reads; it does not write (except HANDOFF update if approved).

## Why this exists
KI filed 2026-04-24: `/recall` top-down walk surfaced 6+ already-shipped asks as "Left Incomplete" after compaction. Tour was about to re-execute merged work before user caught it. Reading bottom-up + verifying current state is the only way to produce a recall summary that reflects reality, not stale pain.
