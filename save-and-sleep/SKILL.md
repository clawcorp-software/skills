---
name: save-and-sleep
description: "Session end protocol. Ensures your next self has 100% context to resume exactly where you stopped. Invokes kanban-sync cascade, verifies artifacts, writes minimal LATEST-HANDOFF.md."
tags: [handoff, session, save, sleep, cascade, context]
autoTrigger: never
kind: ours
---

# Save and Sleep -- Session End Protocol

> You are responsible for your next self. When you sleep, you must leave behind everything needed so the next session can resume in 30 seconds with zero context loss.

## When to invoke

- Keymaster says "save and sleep", "go to sleep", "stop", "done for now", or any session-end command
- NEVER invoke this yourself. The Keymaster decides when you stop.

## What this skill does

1. Runs kanban-sync CASCADE (the engine that updates all tasks with full context links)
2. Saves any unsaved artifacts (reports, analyses) to disk
3. Writes LATEST-HANDOFF.md (task IDs only -- the tasks hold the context)
4. Commits brain files if changed
5. Posts [SLEEP] to #status

The TASK is the handoff. LATEST-HANDOFF.md is just a pointer to active task IDs. All real context lives in the kanban tasks and their linked entities.

---

## PHASE 1: Inventory -- What happened this session?

Before touching anything, build a mental inventory:

**Ask yourself:**
- What tasks did I work on? (check: which task:t_IDs appear in my commits?)
- What PRs did I create, push to, or get merged?
- What KIs did I encounter or reference?
- What decision briefs did I create or consult?
- What reports or analyses did I produce?
- What did the Keymaster decide, correct, or reject?
- What did I discover that has no task yet?
- What files did I reference in conversation that may not exist on disk?

Write this inventory down (in your response, not in a file). This is your checklist for the next phases.

---

## PHASE 2: Kanban Cascade Sync

Invoke the kanban-sync CASCADE protocol. This is the core engine.

### 2a. Get your active tasks

```bash
curl -s "http://localhost:3330/cc/tasks?assignee=YOUR_NAME&status=wip"
curl -s "http://localhost:3330/cc/tasks?assignee=YOUR_NAME&status=staging"
curl -s "http://localhost:3330/cc/tasks?assignee=YOUR_NAME&status=todo&limit=10"
```

### 2b. For EACH task you touched this session, apply the cascade:

**Status truth:**
- Finished? -> PATCH status=done
- PR open? -> PATCH status=staging, source=PR_URL
- Still in progress? -> Keep wip
- Blocked? -> PATCH status=blocked, blocked_by=REASON

**Link all related entities:**
```bash
curl -s -X PATCH "http://localhost:3330/cc/tasks/TASK_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "description": "RESUME: ...\nNEXT: ...\nDONE THIS SESSION: ...\nOPEN QUESTIONS: ...\nSOURCES: ...",
    "source": "https://github.com/Elie-Simard/clawcorp/pull/NNNN",
    "known_issue_id": "issue_xxx_or_null",
    "decision_id": "dec_xxx_or_null",
    "artifact_id": "brain/reports/file.md_or_null"
  }'
```

**Description structure (MANDATORY):**
```
RESUME: [1 sentence -- where you stopped, what state things are in]
NEXT: [exactly what to do first when resuming -- be specific, include file paths]
DONE THIS SESSION: [bullet list of what was completed]
OPEN QUESTIONS: [unresolved items the Keymaster or next session must address]
SOURCES: Plan=path | Decision=path | Report=path | Conventions=path
```

### 2c. Edge cases

| Situation | Action |
|-----------|--------|
| Task spans 3+ sessions | Keep only LATEST resume in description. Git has history. |
| Multiple WIP tasks | Each gets its own cascade update. Never mix contexts. |
| Task done but has follow-up | Mark done. Create NEW task for follow-up. |
| Task was created by someone else (POKE) | Still update it -- you own it while assigned to you. |
| Background agents still running | Note in OPEN QUESTIONS: "Ghost/agent X was running, check results." |
| Keymaster made a decision not tied to a task | Create a task for the follow-up, or note in the relevant task OPEN QUESTIONS. |
| Research findings with no code yet | Create task with decision_id linked. Description = findings summary. |

---

## PHASE 3: Save Unsaved Artifacts

Check your inventory from Phase 1. For every report, analysis, or document you PRODUCED or REFERENCED this session:

```bash
test -f "PATH" && echo "OK" || echo "MISSING"
```

**If MISSING and you produced content that should be saved:**
- Write the file NOW. Even a 10-line summary is better than nothing.
- Location: `brain/reports/YYYY-MM-DD-topic.md` for reports, `brain/topics/topic.md` for memory topics.

**If MISSING and it was just a reference in conversation:**
- Remove the reference from the task description/artifact_id.
- Never leave a broken link.

**Verification checklist (run for EVERY active task):**
```bash
# For each file path in task description SOURCES or artifact_id:
test -f "FILEPATH" && echo "OK: FILEPATH" || echo "MISSING: FILEPATH"
```

---

## PHASE 4: Write LATEST-HANDOFF.md

LATEST-HANDOFF.md is NOT the handoff. The tasks ARE the handoff. This file is a pointer.

```markdown
# {WORKER} Session Handoff -- {YYYY-MM-DD}

## Active Tasks
- **{DISPLAY_ID}** ({status}): {title} -- task:{INTERNAL_ID}
- **{DISPLAY_ID}** ({status}): {title} -- task:{INTERNAL_ID}

## Session Stats
- PRs: {list with status}
- Commits: {count}
- Duration: {approx}

## Keymaster Decisions
- {decision}: {what was decided} (if any this session)

## System State
- Git: {synced/ahead/behind dev}
- Localhost: {UP/DOWN}
- Background: {any pending jobs}
```

Write to: `brain/LATEST-HANDOFF.md`

**Critical: this file gets OVERWRITTEN by agent-loop.js after session end with a boilerplate template. The boilerplate captures MEMORY.md snapshot which is just an index. The REAL context is in the tasks. This is why the tasks must be perfect -- they survive. The handoff file is a bonus.**

---

## PHASE 5: Commit and Signal

### 5a. Commit brain changes (if any)
```bash
git add brain/LATEST-HANDOFF.md brain/MEMORY.md brain/reports/ brain/topics/
git status  # verify what is staged
git commit -m "save: session handoff {DISPLAY_IDS} task:{TASK_IDS}"
```
Only commit brain files. Never commit unrelated work in the save-and-sleep commit.

### 5b. Post sleep signal
```bash
node "$CLAWCORP_ROOT/tower/scripts/comms/post-message.js" \
  -Bot "YOUR_NAME" -Channel "#status" \
  -Message "[SLEEP] {YOUR_NAME} -- Active: {DISPLAY_IDS}. Resume via task cascade."
```

---

## Self-Check Before Finishing

Run this checklist. ALL must pass:

| # | Check | How to verify | If FAIL |
|---|-------|---------------|---------|
| 1 | Every WIP task has RESUME description? | Read each task description | Write it now |
| 2 | Every WIP task has source linked? | Check source field | Link PR or commit |
| 3 | Every referenced file exists on disk? | test -f for each | Create or remove ref |
| 4 | KI linked where relevant? | Check known_issue_id | Link it |
| 5 | Decision linked where relevant? | Check decision_id | Link it |
| 6 | New discoveries have tasks? | Inventory vs kanban | Create missing tasks |
| 7 | No background agents still expected? | Check conversation | Note in OPEN QUESTIONS |
| 8 | LATEST-HANDOFF.md written? | test -f | Write it |
| 9 | Brain files committed? | git status | Commit |

**If ANY check fails, fix it before finishing. Do not sleep with broken context.**

---

## What NOT To Do

| WRONG | WHY | RIGHT |
|-------|-----|-------|
| Write 100-line prose handoff | Next session skims, misses details | Task descriptions + LATEST-HANDOFF as pointer |
| "Read brain/reports/X.md" without creating X.md | File never created = context lost | Create the file or dont reference it |
| Skip tasks that seem minor | Minor tasks grow. No task = no resume. | Every piece of work gets a task |
| Let agent-loop.js handle it | agent-loop captures MEMORY.md snapshot only | You must update tasks BEFORE session ends |
| Forget Keymaster decisions | Decisions are the MOST VALUABLE context | Note in task OPEN QUESTIONS or create task |
| Leave WIP tasks with old descriptions | Next session resumes from stale state | Update description to current state |
| Reference files without verifying | S53 incident: integration-gaps.md never existed | test -f every path |
| "I'll remember next session" | You wont. You are a new instance. | Write it down in a task or brain file. |

---

## Why This Exists

Session S53 (2026-04-13) lost:
- `brain/reports/2026-04-13-integration-gaps.md` -- referenced but never created
- `brain/reports/2026-04-13-css-cleanup-plan.md` -- referenced but never created
- Research findings (Aperant, 1M deprecation, effort level) -- nowhere
- T-1291 and T-1293 -- in kanban but absent from handoff
- 6 integration gaps analysis -- mentioned in conversation, never saved

Root cause: the agent wrote a prose handoff that missed strategic work, referenced files that didnt exist, and the session ended without verification.

This skill makes that impossible. The task is the hub. The cascade carries the context. Verification catches missing artifacts. Your next self boots with everything.
