---
name: kanban-sync
description: "Auto-sync kanban board: task status tracking, cascade context linking. The task is the single source of truth across sessions."
tags: [kanban, sync, automation, tracking, cascade, handoff]
kind: ours
---

# Kanban Sync -- Self-Tracking for Agents

> The task is the HUB. Everything cascades from it. Your next self boots, reads the task, follows the links, has full context.

## Core Principles

1. **SEARCH before CREATE.** Most work already has a task. FIND it and UPDATE it.
2. **The task IS the handoff.** When you update a task, you write a message to your future self. Every field you fill = context your next session gets. Every empty field = context lost forever.
3. **Link everything.** A task with no source, no KI, no decision brief is an orphan. Orphans lose context between sessions.
4. **Administrative git churn is NOT work.** `merge`, `pull`, `push`, `rebase`, branch sync, conflict notifications, and status pings must NOT create tasks.

## STEP 0: SEARCH BEFORE CREATE (MANDATORY)

```bash
curl -s "http://localhost:3330/cc/tasks?assignee=YOUR_NAME&limit=50"
```

Match exists? UPDATE it. No match + real work? Create ONE task.

## Task Origin -- Canonical (2026-04-18, DECISION-BRIEF-kanban-architecture-v2.md)

Tasks enter the kanban from exactly three channels. Workers NEVER create tasks from typed prompts.

| Origin | Channel | Who triggers |
|--------|---------|--------------|
| POKE orders | `write-poke.js` + poke.html Dispatch | Keymaster, Tour, peer workers |
| Manual dashboard creation | `POST /cc/tasks` via kanban UI | Keymaster |
| Claude Code native TaskCreate | `~/.claude/tasks/` watcher -> `POST /cc/tasks/claude-native` | The agent itself, via the native tool |

**RETIRED (2026-04-18):** `UserPromptSubmit` hook -> `POST /cc/auto-task`. The endpoint now returns HTTP 410 Gone. Reason: actionable-prompt detection captured ANY conversational input in ANY worker terminal, stacking 60+ phantom todos while real WIP was 9. Typed prompts are not orders. If you want a task, use TaskCreate.

Workers MUST NOT:
- POST to `/cc/auto-task` (retired, 410 Gone)
- Create a task from a detected "intent" in conversation
- Re-implement any kind of prompt-to-task heuristic

## What Hooks Handle (automatic)

| Trigger | What happens | Hook |
|---------|-------------|------|
| git commit with task:t_ID | Task -> in_progress (and -> done if title contains DONE) | scanRecentCommits (server/index.js) |
| Agent session ends | Mega-sync | kanban-sync-stop.js |
| Dashboard Sync button | Full mega-sync | poke.html |
| Claude Code native TaskCreate/TaskUpdate | One-way mirror to kanban | claude-tasks-watcher.js |

## Google Enrichment (Canonical)

`/kanban-sync` is not just git/task plumbing anymore. It must also run Gemini classification as part of the sync truth.

Operational meaning:
- Phase G is the `Gemini classifier`
- unclassified tasks must be enriched with severity, subsystem, type, and NL metadata
- sync is not considered fully complete if task classification is silently skipped without being reported

Proof:
```bash
curl -s -X POST http://localhost:3330/api/kanban-sync -H "Content-Type: application/json" -d "{\"trigger\":\"manual\"}"
```

Expected fields in response:
- `classifyResult`
- `googleEnrichment.classifier`
- `nlUsage`

If classifier is unavailable:
- report it explicitly
- do not claim full sync readiness

## What Does NOT Create Tasks

POKEs deliver orders, NOT tasks. NEVER create tasks for:
- Test POKEs ("Test POKE system", "verify ack reception")
- PR conflict notifications ("PR #1214 a 4 conflits")
- Completion markers ("DONE: create-pr", "PR merged")
- Branch notifications ("BRANCH echo")
- Git admin churn ("pull dev", "merge manager", "rebase origin/dev", "push branch")

## Agent Workflow (During Session)

### 1. POKE received -> Search, then find or create
Read POKE. Run Step 0 search. Existing match = update to WIP. No match + real work = create ONE. Noise = skip.

### 2. Working -> task:t_ID in every commit
Hook auto-moves task to in_progress.

### 3. PR created -> Link PR URL as source
PATCH task with status=staging, source=PR_URL. Source field = PR URL (clickable). NOT a file path.

### 4. PR merged -> Close task
PATCH status=done. No zombie WIP.

### 5. Multi-session tasks -> Keep description fresh
Update description with current state whenever you make meaningful progress. Dont wait for session end.

---

## CASCADE CONTEXT -- Session End Sync

This is invoked by `/save-and-sleep`. It is the MOST IMPORTANT part of kanban-sync. Everything above is maintenance. This section is what prevents data loss between sessions.

### Why cascade?

Context lives in many places: git commits, PRs, KIs, decision briefs, reports, brain files. The task is the node that LINKS them all. Next session boots, reads the task, follows the links -- full context without prose handoffs that forget things.

```
TASK (the hub)
  |-- source         -> PR URL -> code changes, review comments
  |-- known_issue_id -> KI -> root cause history, past incidents
  |-- decision_id    -> decision brief -> rationale, rejected options
  |-- artifact_id    -> report/diagram path -> detailed analysis
  |-- blocked_by     -> dependency task IDs -> what must finish first
  |-- description    -> RESUME/NEXT/DONE/SOURCES structure
```

### Step 1: Status truth

```bash
curl -s "http://localhost:3330/cc/tasks?assignee=YOUR_NAME&status=wip"
```

For each task you touched this session:
- Finished? -> PATCH status=done
- PR created/open? -> PATCH status=staging, source=PR_URL
- Still in progress? -> Keep wip, update description (Step 3)
- Blocked? -> PATCH status=blocked, blocked_by=REASON
- Discovered new work with no task? -> Create it NOW (Step 5)

### Step 2: Link ALL related entities

For EACH active task, PATCH with every link that exists:

```bash
curl -s -X PATCH "http://localhost:3330/cc/tasks/TASK_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "source": "https://github.com/Elie-Simard/clawcorp/pull/1245",
    "known_issue_id": "issue_xxx",
    "decision_id": "dec_xxx",
    "artifact_id": "brain/reports/2026-04-13-report.md"
  }'
```

| Field | What to put | When |
|-------|-------------|------|
| source | PR URL or commit hash | Always -- every task should trace to code |
| known_issue_id | KI ID from /cc/known-issues | When task relates to a known bug pattern |
| decision_id | Decision ID from /cc/decisions | When architecture choice was made |
| artifact_id | Relative path to report/diagram | When analysis was produced |
| blocked_by | Task ID or free text | When something blocks progress |

Minimum documentary chain for non-trivial work:
- code change -> `source`
- recurring bug/systemic issue -> `known_issue_id`
- architecture / protocol / tooling decision -> `decision_id` or linked decision brief path in description
- analysis / audit / report -> `artifact_id`

**Rule: if you referenced a KI, decision brief, or report during the session, it MUST be linked to the relevant task. An empty relational field when data exists = context lost between sessions.**

### Step 3: Write resume description

The description is what your next self reads FIRST. Use this structure:

```
RESUME: [1-sentence: where you stopped]
NEXT: [exactly what to do first when resuming]
DONE THIS SESSION: [bullet list of completed items]
OPEN QUESTIONS: [anything unresolved the Keymaster or next session must address]
SOURCES: Plan=path | Decision=path | Report=path | Conventions=path
```

Example:
```
RESUME: panels.js inline style extraction -- 497 remaining, 8 overlay/modal done.
NEXT: Create css/panels-legend.css, extract panel-specific inline styles. Verify with /eye.
DONE THIS SESSION: dashboard.js split 7 modules, toolkit +80 classes, PR #1236 merged
OPEN QUESTIONS: Should panel colors use existing tokens or new ones?
SOURCES: Plan=brain/reports/2026-04-13-css-cleanup-plan.md | Conventions=.claude/skills/frontend-conventions/SKILL.md
```

**Edge cases:**
- Task spans 3+ sessions? Keep only LATEST resume, not history. Git has history.
- Multiple tasks WIP? Each gets its own description. NEVER mix task contexts.
- Task done but has follow-up? Mark done, create NEW task for follow-up, link via blocked_by.

### Step 4: Verify artifacts exist

For EVERY file path in description or artifact_id:
```bash
test -f "PATH" && echo "OK: PATH" || echo "MISSING: PATH"
```

If MISSING: create the file NOW with at minimum a summary of what it should contain, or REMOVE the reference. **Never reference a file that does not exist. This is the #1 cause of context loss (S53 incident: integration-gaps.md referenced but never created).**

### Step 5: Create tasks for untracked discoveries

If you discovered something this session that needs future work and has no task:
- Research findings -> create task, link decision_id
- Integration gaps -> create task per gap
- Bugs found but not fixed -> create task, link known_issue_id
- Keymaster decisions that need follow-up -> create task

**Every piece of meaningful work must have a task. Untracked work = lost work.**

### Step 6: Verify cascade completeness

Self-check before finishing. For each WIP/staging task:

| Check | How | If FAIL |
|-------|-----|---------|
| Has source? | source field not empty | Link PR or commit |
| Has resume description? | description starts with "RESUME:" | Write it now |
| Referenced files exist? | test -f for each path | Create or remove ref |
| KI linked if relevant? | known_issue_id set | Link it |
| Decision linked if relevant? | decision_id set | Link it |
| New discoveries have tasks? | search kanban | Create tasks |

---

## Boot Cascade -- Reading tasks at session start

When booting, after task-check.js:

1. For each WIP task: GET the full task object
2. Read description (resume instructions = your starting point)
3. If source set: check PR status (merged? conflicts? review comments?)
4. If known_issue_id set: check KI (new incidents since last session?)
5. If artifact_id set: Read the file (still relevant? updated by others?)
6. If blocked_by set: check blocker status (resolved? still blocked?)

Full cascade context in 30 seconds. No prose handoffs. No lost reports.

## Rules

1. SEARCH before CREATE. Every time.
2. Self-tracking ONLY.
3. task:t_ID in commits.
4. PR URL in source field.
5. Close when done. No zombie WIP.
6. Clean titles. Human-readable.
7. No noise tasks.
8. 1 task per feature.
9. **LINK everything.** Empty relational fields when data exists = lost context.
10. **Description = resume instructions.** RESUME/NEXT/DONE/SOURCES structure.
11. **Verify artifacts exist.** Never reference missing files.
12. **Untracked work = lost work.** If it matters, it has a task.

## Anti-Patterns

| WRONG | RIGHT |
|-------|-------|
| Create without searching | Search first, update if found |
| Task for every POKE | Find matching task, update it |
| Title: "DONE: X" or "Test POKE" | Noise, dont create |
| WIP after PR merged | Move to done |
| File path in source field | PR URL in source |
| 5 tasks for 1 feature | 1 task, update as it progresses |
| Empty known_issue_id when KI exists | Link it -- cascade needs it |
| "Read the report" in description | Summarize inline + link path in SOURCES |
| Reference file that doesnt exist | Verify + create or remove ref |
| Research findings with no task | Create task -- next self wont remember |
| Description = "CSS work" | RESUME/NEXT/DONE/SOURCES structure |
| Wait until session end to update | Update description during session on progress |
