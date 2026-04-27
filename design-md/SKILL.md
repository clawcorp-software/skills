---
name: design-md
description: Load ClawCorp's shared DESIGN.md only when a task touches UI, UX, frontend, dashboard, docs presentation, or agent-facing product surfaces. Keeps design context consistent without inflating every boot.
user_invocable: true
kind: ours
---

# design-md

Use this skill when the task affects how ClawCorp looks, feels, loads, communicates, or exposes state.

## Why this exists

`AGENTS.md` tells agents how to behave.
`DESIGN.md` tells agents how ClawCorp should look and feel.
This skill is the low-cost bridge between the two.

## Cost policy

Do not load this skill for backend-only, infra-only, or data-only work.

When you do load it:
1. Read `references/DESIGN.md` first.
2. Quote or summarize only the sections relevant to the task.
3. Do not paste the whole file into prompts unless a downstream agent explicitly needs full context.
4. If using sub-agents or external models, pass a compact summary first, then expand only if needed.

## When to use

Use for:
- dashboard pages
- extension UI / webviews
- docs presentation surfaces
- marketing pages
- design system work
- interaction/loading-state refinement
- copy/microcopy tied to UI
- component styling and hierarchy

Usually skip for:
- server routes
- database work
- CI/CD
- infrastructure
- pure worker loop / shell fixes unless the task changes user-visible UX

## Workflow

1. Read `references/DESIGN.md`.
2. Extract only the rules relevant to the task.
3. Apply those rules to the implementation.
4. In your final note, mention that the work followed `DESIGN.md` when relevant.
5. If the task reveals a recurring design rule not captured there, update the canonical `C:\Users\user\clawcorp\DESIGN.md` and this reference copy.

## Propagation

This is a project skill under `C:\Users\user\clawcorp\.claude\skills\design-md`.
Any agent using project skills can load it with:
- `npx openskills read design-md`

Canonical files:
- `C:\Users\user\clawcorp\DESIGN.md`
- `C:\Users\user\clawcorp\.claude\skills\design-md\references\DESIGN.md`

## Non-negotiable rules

- Do not silently override explicit runtime/provider choices in UI.
- Separate `CLI`, `Model`, `Preset`, and live runtime state in the interface.
- Immediate loading feedback is mandatory for async actions.
- Dense operator surfaces must optimize for scan speed.
- Avoid generic AI product aesthetics when touching ClawCorp UI.
