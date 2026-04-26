---
name: research-gate
description: "Mandatory existence/blast-radius/relation check before changing architecture, routing, DB schema, skills, protocols, or canonical data flows."
---

# /research-gate -- Canonical Search Before Change

Use before touching:
- routing
- DB schema
- protocol / skills
- canonical relation graph
- incident / KI / tribunal / docs evidence flows

## Minimum Checks

1. Search existing code paths
2. Search existing docs / plans / decision briefs
3. Search KI history
4. Search canonical relations already present
5. State whether the new layer is additive, duplicate, or replacing something

## Duplication Test

Before adding any new table / route / parser / sync:
- what already stores this?
- what already serves this?
- can runtime read the compiled DB instead of raw files?

If the answer is “yes”, prefer reuse.

## For relational data changes

Always ask:
- is this one-to-many or many-to-many?
- is a join table required?
- can null links be valid?
- what automatic inference is dangerous noise?

Rule:
- absence of a relation must not throw
- automatic guessed relations must be conservative

## Output

State:
- what already exists
- what is missing
- what duplication would happen if unchanged
- what the minimal patch is

## File-Specific Conventions (chain load)

After research-gate, if your edit touches specific surfaces, also invoke the matching conventions skill:

| File pattern | Skill to invoke |
|---|---|
| `dashboard/**` (CSS/HTML/JS) | `/frontend-conventions` |
| `server/**` (routes, helpers, middleware, ccdb) | `/backend-conventions` |
| `tower/extensions/clawcorp-launcher/extension.js` | `/extension-ownership` |
| Filing a KI or incident | `/ki-conventions` |

These skills hold the judgment rules that were previously always-loaded. Load on-demand to save boot tokens.
