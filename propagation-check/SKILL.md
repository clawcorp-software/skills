---
name: propagation-check
description: "Verification complete post-changement. MANDATORY after editing critical files (.claude/, tower/config/, normalize, CONSTITUTIONAL, policy)."
kind: ours
---

# Propagation Check -- 6-Layer Verification

MANDATORY after any change to: `.claude/rules/`, `.claude/skills/`, `tower/config/decision-tree.json`, `normalize-all-workers.ps1`, `CONSTITUTIONAL.md`, `CLAUDE.md`, `agent-config.json`, `policy/`.

ALSO MANDATORY after implementing a decision brief. Code change alone is NOT enough — rules/skills/tests may still reference OLD behavior. Example: squash merge decision was implemented but workflow.md still said `git pull origin dev` (KI-12 bullshit incident).

## The 6 Layers

Every systemic rule MUST exist in at least 2 layers. Check each:

### Layer 1: Rules (`.claude/rules/*.md`)
- [ ] Rule documented in the appropriate rules file?
- [ ] Language clear, unambiguous, follows existing style?
- [ ] Not duplicating content already in decision-tree (judgment only)?

### Layer 2: Decision Tree (`tower/config/decision-tree.json`)
- [ ] Mechanical pattern added as SCRIPT rule (if regex-matchable)?
- [ ] Action level correct? WARN for new rules, BLOCK for proven ones
- [ ] fileTypes and tools arrays correct?
- [ ] Pattern tested against real code (grep the codebase)?

### Layer 3: Promptfoo (`promptfoo/*.yaml`)
- [ ] Test case added that verifies agent follows the rule?
- [ ] Both positive (correct behavior) and negative (violation) cases?
- [ ] Test passes locally: `cd promptfoo && npx promptfoo eval -c <file>.yaml`

### Layer 4: Skills (`.claude/skills/`)
- [ ] If workflow change: relevant skill updated?
- [ ] Skill references current file paths and tool names?
- [ ] No stale references to removed features?

### Layer 5: Hooks (`tower/scripts/hooks/`)
- [ ] Hook enforces the mechanical check (if applicable)?
- [ ] Hook registered in `settings.json` or `settings.local.json`?
- [ ] Hook tested: trigger the condition, verify block/warn fires?

### Layer 6: Normalize (`tower/scripts/workers/normalize-all-workers.ps1`)
- [ ] If settings/config change: normalize propagates to all workers?
- [ ] Worker worktrees get the update on next sync?

## Verification Protocol

1. Run through all 6 layers above
2. For each checked box, note WHERE (file:line) the enforcement exists
3. For each unchecked box, decide: NOT APPLICABLE or NEEDS WORK
4. If NEEDS WORK items remain: create tasks, do NOT mark as done

## Output

Post in your channel:
```
[PROPAGATION-CHECK:agent-name]
change: <what changed>
layers_verified: 1,2,3,4,5,6
gaps_found: <layer numbers with gaps, or "none">
action_items: <tasks created, or "none">
```

## Rules

- 2+ layers minimum per rule. 1 layer = fragile, will break
- Judgment rules = Layer 1 (rules) + Layer 3 (promptfoo)
- Mechanical rules = Layer 2 (decision-tree) + Layer 5 (hooks)
- Workflow rules = Layer 1 (rules) + Layer 4 (skills)
- If you can't verify a layer, say so. Don't fake checkmarks.
