---
name: janitor
description: "Sweep stale state out of the project: pending POKEs older than the TTL, untracked tmp/* test extracts, abandoned scheduled tasks, expired worker locks, dead branches. Use periodically when state has accumulated drift, or when the agent feels their tools are slow because of background noise."
tags: [maintenance, cleanup, drift, hygiene]
---

# Janitor -- Periodic State Cleanup

State accumulates. Background loops never stop emitting. Test fixtures forget to clean up after themselves. The right answer isn't "be tidier" -- it's a routine sweep.

## When to invoke

- A queue/log/cache is observably bloated (e.g. 17 pending POKEs from a 2-day-old test run)
- An agent reports "things feel slow" and the slowness is upstream of their code
- Before a release / before opening a long-running PR
- Once a week as routine maintenance
- Whenever the Keymaster types `/janitor`

## What it sweeps (canonical list)

For each surface below, check the staleness threshold and prune. Always log what was removed.

### 1. Pending POKEs (`tower/config/gates.json`)

Anything older than 72h is auto-expired by the runtime janitor in `server/routes/pokes.js readGates()`. The skill verifies the runtime is doing its job; if entries older than 72h still exist after a server restart, the janitor is broken -- file an incident.

Manual sweep (if runtime isn't writing):
```bash
node -e "const fs=require('fs'); const p=process.env.CLAWCORP_ROOT+'/tower/config/gates.json'; const g=JSON.parse(fs.readFileSync(p,'utf8')); const now=Date.now(); const cutoff=72*3600*1000; const before=g.pending_pokes.length; g.pending_pokes=g.pending_pokes.filter(x=>{const t=Date.parse(x.created_at);return isNaN(t)||(now-t)<cutoff}); console.log('pruned',before-g.pending_pokes.length); fs.writeFileSync(p,JSON.stringify(g,null,2))"
```

### 2. tmp/ test extracts

Generators like `desktop/scripts/generate-installer-zip.js --variant basic` extract thousands of files into `tmp/basic-test/`, `tmp/zipcheck*/`, `tmp/mincheck/`, `tmp/full-test/`, `tmp/cloud-test/`. These are gitignored (since 2026-04-27) but still take disk space and bloat IDE source-control panels.

```bash
ls -d tmp/basic-test tmp/mincheck tmp/zipcheck* tmp/full-test tmp/cloud-test 2>/dev/null
# Review, then delete:
cd tmp && rm -r basic-test mincheck zipcheck* full-test cloud-test 2>/dev/null
```

### 3. Worker locks

`.agent-lock` files left behind after a worker crash. Workers normally remove on stop. A lock older than 24h on a worker that isn't currently launched = stale.

```bash
find workers/*/worksites/clawcorp -name '.agent-lock' -mtime +1 2>/dev/null
# Verify each: `cat workers/X/worksites/clawcorp/.agent-lock` should match a live process
```

### 4. Scheduled tasks lock

`tower/.claude/scheduled_tasks.lock` -- the cron lock for the schedule skill. If older than 1 hour AND no schedule task is currently running, it's stale.

```bash
test -f tower/.claude/scheduled_tasks.lock && stat -c '%Y' tower/.claude/scheduled_tasks.lock
# Compare to `date +%s` -- if delta > 3600 seconds, suspect stale
```

### 5. Dead branches (worker pushed + merged + worker still on old branch)

```bash
git fetch --prune origin
git branch -r --merged origin/main | grep -v 'main\|dev'
```

Don't auto-delete branches without confirming with the worker owner.

### 6. Read-cache files

`tower/logs/read-cache-*.json` -- per-worker read-once cache (20-min TTL). Files older than 1 day with no activity = orphan worker (worker name no longer in `agent-config.json`).

```bash
find tower/logs -name 'read-cache-*.json' -mtime +1
```

### 7. POKE archive

`tower/archive/poke/recent/*.md` -- archived POKEs accumulate. Anything older than 30 days can move to `tower/archive/poke/cold/` or be deleted.

## Output format

The skill always prints a one-line summary per surface, even if nothing was pruned. The Keymaster wants to see the audit trail.

```
[janitor] gates.json pending POKEs: 0 stale (clean)
[janitor] tmp/ test extracts: removed tmp/basic-test (1.7M), tmp/mincheck (587K)
[janitor] worker locks: 0 stale
[janitor] scheduled_tasks.lock: clean (mtime 6 min)
[janitor] dead branches: 2 candidate (architecte/old, forge/exp) -- NOT auto-deleted, list only
[janitor] read-cache files: 0 orphan
[janitor] POKE archive: 47 entries, 3 older than 30d -- not pruned (review manually)
```

## Trust principles

- **Never delete tracked git files.** If a path is tracked, the user committed it intentionally. Untracked-only.
- **Never auto-delete branches.** List them, let the human decide.
- **Always log what was removed.** Even if zero -- the user wants confirmation the janitor ran.
- **Be idempotent.** Running it twice in a row should be a no-op the second time.

## When NOT to run

- During active deployment / release window
- Mid-incident response (state pruning can mask diagnostic clues)
- When a long-running task is in progress and might be using a "stale-looking" lock

## Related runtime cleanup

Some janitor logic is enforced at runtime (so it kicks in even without the skill being invoked):

- `server/routes/pokes.js readGates()` -- prunes pending POKEs older than 72h on every read.
- `server/routes/pokes.js isTestSender()` -- rejects POKEs from `*-test/*-promptfoo/*-fixture/test-*` senders at the gate. They never enter the queue.
- `tower/scripts/hooks/hook-pre-unified.js` Module 1 -- 20-min TTL on read-once cache entries.

The skill is for surfaces that don't have a runtime janitor yet, and as a verification harness for the ones that do.
