---
name: frontend-preflight
description: "Frontend-only checklist for dashboard changes. Optional helper for UI work. Independent from verification-before-completion."
---

# /frontend-preflight

This skill is only for frontend/dashboard surfaces.

It is not:
- a replacement for verification
- a prerequisite to verification
- a general release gate for backend/data/routing work

Use it when:
- layout changed
- CSS changed
- dashboard interactions changed
- mobile/touch behavior changed

Do not use it as the canonical final verifier.

## Checks

1. Toolbar zone
2. Flex overflow
3. CSS scope
4. Z-index discipline
5. Nav tokens
6. Language clarity
7. Mobile touch targets

## Output

```text
FRONTEND PREFLIGHT
toolbar: PASS/FAIL
overflow: PASS/FAIL
scope: PASS/FAIL
z-index: PASS/FAIL
nav tokens: PASS/FAIL
language: PASS/FAIL
mobile: PASS/FAIL
```

If the change was not UI-facing, skip this skill.
