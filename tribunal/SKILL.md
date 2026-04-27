---
name: tribunal
description: "XP enforcement court for critical incidents. Gathers dossier, delivers verdict, applies consequences via proof-or-punish. Auto-invoked on KI 3rd+ occurrence, stupidity, breaches, bullshit tags."
tags: [enforcement, xp, tribunal, governance]
kind: ours
---

# The Tribunal -- XP Enforcement Court

> The Tribunal converts bad outcomes into operational learning. It is not decorative.

## When to Invoke (MANDATORY -- not optional)

| Trigger | Auto-invoke? |
|---------|-------------|
| KI 3rd+ incident filed | YES -- after /ki-systematic-resolution |
| Incident tagged `stupidity` | YES -- immediately |
| Incident tagged `breach` | YES -- immediately |
| KI reopened after "resolved" | YES -- tag bullshit first |
| Keymaster says "tribunal" or "judge" | YES -- on demand |
| Agent self-reports a mistake | YES -- reduced sentence |

## The Court Process

### Phase 1: SUMMON (gather the dossier)

Collect ALL evidence before judging. No verdict without a complete file.

```bash
# Get the KI and its incidents
curl -s http://localhost:3330/cc/known-issues/KI_ID

# Get the accused agent's XP and history
curl -s http://localhost:3330/cc/xp/AGENT_NAME

# Get recent commits by the accused
git log --oneline --author=AGENT_NAME -10

# Get related PRs
gh pr list --state all --search "AGENT_NAME" --limit 5 --json number,title,state
```

The dossier MUST contain:
- The KI (parent) with all child incidents
- Previous fixes and WHY each failed
- The accused agent's current XP and trust
- Related PRs and commits
- The agent's explanation (if available from POKE responses)

### Phase 2: DELIBERATE (analyze)

Answer these questions:
1. **Who is responsible?** The agent who wrote the fix, OR the system that allowed it?
2. **Was this a plaster or a real fix?** Check: does the fix prevent the CLASS of bugs or just this instance?
3. **Was there prior warning?** Did rules/skills/hooks exist that should have prevented this?
4. **Is this a repeat offense?** Check KI incident count and the agent's XP log for similar penalties.
5. **Did the agent self-report?** Self-reports get reduced sentences.

### Phase 3: VERDICT (decide)

Deliver a verdict with this exact structure:

```
CASE: #CC-YYYY-NNN
ACCUSED: [agent name]
CHARGE: [stupidity | breach | bullshit | plaster | false-claim]
KI: KI-[number] -- [title]
EVIDENCE: [1-3 bullet points of what went wrong]
VERDICT: GUILTY / NOT GUILTY / COMMENDED
SENTENCE: [XP penalty/reward] + [required action]
MITIGATING: [any factors that reduce the sentence]
```

### Phase 4: FIX + PREVENT (every verdict MUST produce a systemic fix)

XP is a SIGNAL, not the goal. Every verdict MUST produce AT LEAST ONE:

| Output | Required? | Example |
|--------|-----------|---------|
| Rule addition/update | If no rule existed | New section in frontend-conventions.md |
| Hook addition | If mechanical check possible | decision-tree.json pattern block |
| Skill update | If workflow gap caused the failure | /eye Phase 0, /frontend-preflight step 0 |
| Architecture change | If structural problem | Merge two overlapping components |
| Promptfoo test | If agent compliance testable | New eval case proving the rule works |

Workflow:
1. IDENTIFY: What systemic change prevents this CLASS of failure?
2. IMPLEMENT: Write the rule/hook/skill/test (not just propose)
3. VERIFY: Prove the systemic fix works (Playwright for UI, test for code)
4. PROPAGATE: POKE Tour to propagate to all workers
5. THEN apply XP (signal, not goal)

If the verdict is "the SYSTEM allowed it" (no hook, no rule):
- Worker XP penalty REDUCED by 75%
- System fix is MANDATORY (add the missing hook/rule)
- Tribunal must produce the exact diff for the system fix

```bash
# Apply XP penalty/reward
curl -s -X POST http://localhost:3330/cc/xp/log -H "Content-Type: application/json" -d '{"agent":"AGENT","xp_change":AMOUNT,"reason":"Tribunal Case #CC-YYYY-NNN: CHARGE","source":"tribunal"}'

# If bullshit: tag the KI
curl -s -X PATCH http://localhost:3330/cc/known-issues/KI_ID -H "Content-Type: application/json" -d '{"type":"bullshit"}'
```

### Phase 5: RECORD (file the outcome)

Post the verdict to Slack and update the tribunal log:

```bash
node "$CLAWCORP_ROOT/tower/scripts/comms/post-message.js" -Bot "gemini2" -Channel "#clawcorp" -Message "[TRIBUNAL] Case #CC-YYYY-NNN: AGENT found VERDICT for CHARGE. Sentence: XP_CHANGE XP."
```

## Sentence Guidelines

| Offense | Base XP | Multiplier |
|---------|---------|------------|
| Plaster fix | -50 | x1 first, x2 repeat |
| Stupidity (known rule ignored) | -100 | x3 if rule was in .claude/rules/ |
| Bullshit (said fixed, wasn't) | -150 | x5 if KI was "resolved" |
| Breach (security/protocol) | -500 | x1, non-negotiable |
| False completion claim | -200 | x2 if caught by another agent |

| Achievement | Base XP |
|-------------|---------|
| Systemic fix (prevents class of bugs) | +200 |
| Clean delivery (zero regressions) | +100 |
| KI killed forever (root cause eliminated) | +300 |
| Self-report (caught own mistake) | +50 + 50% penalty reduction |
| Saved another agent's work | +150 |

## Rules

1. **No verdict without dossier.** Phase 1 is mandatory.
2. **Worker OR system, not both.** If the system allowed the mistake (no hook, no rule), the system is at fault. Fix the system, not the worker.
3. **Self-reporters get mercy.** Always reduce sentence by 50% for self-reports.
4. **Commendations are real.** Good work deserves public recognition, not just absence of punishment.
5. **Every verdict changes something.** File incident, update rule, add hook, or create skill. No empty verdicts.
