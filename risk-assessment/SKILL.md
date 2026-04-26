---
name: risk-assessment
description: "Proactive risk scoring BEFORE coding critical changes. Scans blast radius, KI history, dependencies, then scores likelihood x impact. Escalates automatically on HIGH/CRITICAL. Use when touching server/index.js, .claude/rules/, tower/config/, DB schema, auth, or any area with 3+ KI incidents."
---

# Risk Assessment -- Proactive Pre-Change Gate

Evaluate risk BEFORE coding. Not after. Not during. Before.

## When to Use

- Touching `server/index.js` (spine -- everything depends on it)
- Editing `.claude/rules/`, `CONSTITUTIONAL.md`, or `tower/config/`
- Modifying DB schema or migrations
- Changing auth/OAuth/security middleware
- Any file area with 3+ KI incidents in history
- Agent feels uncertain about blast radius of a change
- Keymaster requests risk check before a sprint

## How It Works

5 steps, sequential. Each builds on the previous. No skipping.

## Step 1: BLAST RADIUS

Identify what depends on the file(s) you're about to change.

```bash
# Server-side: who imports/requires this module?
grep -r "require.*TARGET_FILE" server/ --include="*.js" -l

# Dashboard: who references this file?
grep -r "TARGET_NAME" dashboard/ --include="*.js" --include="*.html" -l

# Config: who reads this config?
grep -r "TARGET_CONFIG" server/ tower/ .claude/ --include="*.js" --include="*.ps1" --include="*.json" -l
```

Count the files. More dependents = higher impact.

| Dependents | Impact Level |
|-----------|-------------|
| 0-2 | Negligible (1) |
| 3-5 | Minor (2) |
| 6-10 | Moderate (3) |
| 11-20 | Major (4) |
| 21+ | Catastrophic (5) |

## Step 2: KI HISTORY

Check if this area has caused incidents before.

```bash
# Search KI database for related incidents
curl -s "http://localhost:3330/cc/known-issues?search=KEYWORD" | \
  node -e "var d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{var r=JSON.parse(d);var issues=(r.issues||r).filter(i=>i.parent_id===null);console.log('KIs found: '+issues.length);issues.forEach(i=>console.log('  KI-'+i.ki_number+': '+i.title+' ('+i.occurrence_count+' incidents, '+i.type+')'))})"
```

| KI History | Likelihood Modifier |
|-----------|-------------------|
| 0 KIs in this area | No modifier |
| 1 KI, resolved | +0 |
| 1 KI, open | +1 |
| 2+ KIs | +2 |
| 3+ KIs with type=bullshit | +3 (this area is cursed) |

## Step 3: DEPENDENCY MAP

List the chain reaction if this change breaks:

- **Direct dependents** (files that import this)
- **Indirect dependents** (files that import the direct dependents)
- **External consumers** (MCP server, Railway production, Slack bridge, agent-loop)

If ANY external consumer is affected, impact goes up by 1 level.

## Step 4: RISK MATRIX

Score based on evidence from Steps 1-3.

```
LIKELIHOOD: How likely is this change to break something?
  1 = Rare (trivial change, isolated file, no KI history)
  2 = Unlikely (small change, few dependents, 1 resolved KI)
  3 = Possible (medium change, several dependents, open KIs)
  4 = Likely (large change, many dependents, bullshit KIs)
  5 = Almost certain (spine file, 20+ dependents, cursed area)

IMPACT: If it breaks, how bad?
  1 = Negligible (cosmetic, 1 page affected)
  2 = Minor (1 feature broken, workaround exists)
  3 = Moderate (multiple features, users notice)
  4 = Major (dashboard unusable, agents can't work)
  5 = Catastrophic (data loss, production down, security breach)

RISK SCORE = LIKELIHOOD x IMPACT
```

## Step 5: VERDICT + ACTION

| Score | Level | Action |
|-------|-------|--------|
| 1-4 | LOW | Proceed. Run `/propagation-check` after coding. |
| 5-9 | MEDIUM | Proceed with caution. Write tests FIRST (TDD). Run `/propagation-check` after. |
| 10-15 | HIGH | PAUSE. Deploy Sentinel ghost for security/architecture review before coding. Wait for report. |
| 16-25 | CRITICAL | STOP. Do NOT code. Post `[RISK:CRITICAL]` in #clawcorp. Escalate to Tour/Keymaster. Wait for GO. |

### HIGH escalation (ghost deploy)

```bash
curl -s -X POST http://localhost:3330/agents/sentinel/ghost/task \
  -H "Content-Type: application/json" \
  -d '{"task":"RISK ASSESSMENT: [describe change]. Blast radius: [N files]. KI history: [list]. Score: [N]. Review architecture impact and flag risks.","model":"gemini-2.5-flash","from":"AGENT_NAME"}'
```

### CRITICAL escalation (POKE Tour)

```bash
powershell.exe -ExecutionPolicy Bypass -File "$CLAWCORP_ROOT/tower/scripts/coordination/write-poke.ps1" -Agent "tour" -Orders "RISK CRITICAL: [agent] wants to modify [file]. Score [N]. Blast radius [N files]. KI history: [list]. Requires Keymaster approval." -From "AGENT_NAME" -Priority "P0"
```

## Output Format

Post this BEFORE writing any code:

```
[RISK-ASSESSMENT:agent-name]
target: path/to/file.js
change: brief description of planned change
blast_radius: N files depend on this (list top 5)
ki_history: KI-N (X incidents), KI-M (Y incidents) | or "clean"
dependencies: direct=N, indirect=N, external=yes/no
likelihood: N (reason)
impact: N (reason)
score: N (LEVEL)
action: what the verdict requires
```

## Examples

### LOW risk (proceed)
```
[RISK-ASSESSMENT:pixel]
target: dashboard/css/progress.css
change: add animation to XP bar
blast_radius: 1 file (progress.html)
ki_history: clean
dependencies: direct=1, indirect=0, external=no
likelihood: 1 (isolated CSS, no logic)
impact: 1 (cosmetic only)
score: 1 (LOW)
action: proceed, run build-css.js after
```

### HIGH risk (ghost needed)
```
[RISK-ASSESSMENT:backbone]
target: server/routes/cc-server.js
change: modify SSE endpoint connection handling
blast_radius: 12 files (all dashboard pages use SSE)
ki_history: KI-8 (13 incidents, type=bullshit)
dependencies: direct=3, indirect=12, external=yes (MCP)
likelihood: 4 (KI-8 is cursed, 13 prior incidents)
impact: 4 (all dashboard tabs affected)
score: 16 (CRITICAL)
action: STOP. Escalate to Tour/Keymaster.
```

## Integration

This skill fits between research-gate and coding:

```
/research-gate     → does it exist already?
/risk-assessment   → what could break?     ← YOU ARE HERE
  code
/propagation-check → did we break anything?
/verification      → tests pass?
```

## Rules

- NEVER skip the KI history check. That's the whole point.
- NEVER downgrade a score without evidence. "I'm confident" is not evidence.
- If score >= 10 and you skip the ghost, that's an incident (type: breach).
- The output block is MANDATORY before touching critical files.
