---
name: full-power
description: Maximum-quality orchestrator. Selects and executes the right combination of skills, ghosts, and subagents for a task.
kind: ours
---

# Full Power -- Maximum Quality Orchestrator

When invoked, the agent must deliver MAXIMUM quality work by selecting and executing the right combination of skills, ghosts, and subagents. No shortcuts. No corners cut.

## When to Use
- User says "full power", "do everything", "make it perfect", "all skills"
- User is frustrated with mediocre output and wants the best possible result
- User is tired of manually triggering individual skills

## Process

### Step 1: ANALYZE the task

Read the user's request. Classify the work type:

| Work Type | Relevant Skills |
|-----------|----------------|
| **New UI feature** | /frontend-design, /teach-impeccable, /adapt, /animate, /colorize, /delight, /polish |
| **UI fix/improvement** | /critique, /normalize, /polish, /adapt, /harden |
| **Code/API work** | /research-gate, code, tests, /simplify |
| **Architecture decision** | /decision-brief, /deep-research-report, /ghost-sos |
| **Bug fix** | /ki-systematic-resolution (if KI exists), /ghost-sos, /simplify |
| **PR/shipping** | /frontend-preflight (if dashboard), tests, /finishing-a-development-branch |
| **Design overhaul** | /critique, /bolder OR /quieter, /colorize, /animate, /delight, /adapt, /polish |

### Step 2: BUILD the execution plan

Select the skills that apply. For each, write ONE line explaining WHY it's needed.

Present to the user:

```
[FULL-POWER PLAN]
Task: {description}
Skills selected: {N} of {total available}

1. /skill-name -- why this is needed
2. /skill-name -- why this is needed
...

Ghosts: {none | list with missions}
Subagents: {none | list with tasks}

Estimated effort: {X minutes}
Ready? (y/n)
```

### Step 3: EXECUTE sequentially

Run each selected skill in order. Between each:
- Check the output
- Decide if the next skill is still necessary (maybe the previous one already covered it)
- Skip skills that became redundant (explain why)
- Add skills that became necessary (explain why)

### Step 4: QUALITY GATE

After all skills complete:
- Screenshot the result (if UI)
- Run tests (if code)
- Self-critique: "What would I change if I had unlimited time?"
- If the answer is "nothing" -> DONE
- If the answer is something concrete -> DO IT (per REGLE 10)

## Rules

- NEVER skip a selected skill because "it's probably fine"
- NEVER say "good enough" -- the user triggered /full-power because they want PERFECT
- If a ghost report suggests improvements, IMPLEMENT them (don't just acknowledge)
- Subagents for parallel research are ENCOURAGED (saves time, not quality)
- The Keymaster must approve the plan BEFORE execution (Step 2)
- Budget is 1M tokens. Use as many as needed.

## Design-Specific Full Power

When the task involves UI/frontend, this is the minimum flow:

```
/teach-impeccable (if not done yet)
  --> /frontend-design (build the thing)
    --> /critique (evaluate what was built)
      --> Fix critique findings
        --> /adapt (responsive)
          --> /animate (micro-interactions)
            --> /colorize (if monochromatic)
              --> /delight (personality)
                --> /normalize (design system consistency)
                  --> /polish (final pass)
                    --> /frontend-preflight (before PR)
```

Skip steps that genuinely don't apply (e.g., /colorize on a feature that's already colorful). But JUSTIFY each skip.

## Ghost Integration

| Situation | Ghost to deploy |
|-----------|----------------|
| Unsure about best approach | Boss (strategic synthesis) |
| Need competitor reference | Grok (deep research) |
| Security concern | Sentinel (vulnerability scan) |
| Design feedback wanted | Analyst (design audit) |
| Performance concern | Analyst (architecture review) |

Deploy ghosts in PARALLEL with early skill execution to save time.
