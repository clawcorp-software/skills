---
name: systematic-debugging
description: "Structured debugging protocol for old recurring bugs, contradictory symptoms, or any bug where ad-hoc patching has already failed. Forces hypothesis isolation, evidence collection, and root-cause confirmation before any fix."
kind: ours
---

# Systematic Debugging

**Use when:** a bug is old, recurring, has >2 plausible hypotheses, or earlier fix attempts have failed. Ad-hoc patching has a bad track record here — it treats symptoms.

**Do NOT use for:** one-off fresh bugs with a single obvious cause. Those go straight to fix.

## The Rule

**No fix before the root cause is confirmed by evidence.** Evidence = logs, file:line, reproducible trigger, or git diff showing the exact regression commit.

---

## Protocol

### 1. State the symptom — one sentence
Concrete and user-visible. "Kanban shows 0 cards" beats "something broken in dashboard".

### 2. List every hypothesis — at least 3
Don't rush to 1. Cast wide.
- What layer could cause this? (client, API, DB, network, hook, worker)
- What changed recently? (`git log --oneline -20`)
- Who else hit something similar? (grep KIs, incidents)

### 3. Rank hypotheses by evidence weight
For each hypothesis, note:
- **Supports it:** (what you've observed)
- **Refutes it:** (what would have to be true but isn't)
- **Cheap to test:** yes/no

Start with cheap + high-evidence tests.

### 4. Test hypotheses in order
Each test produces one of three outcomes:
- **Confirmed** → proceed to root cause
- **Ruled out** → cross off, move to next
- **Inconclusive** → refine the test, do NOT move on

Never hand-wave "it's probably that". If you can't produce evidence, it's not confirmed.

### 5. Identify root cause with file:line
"React component re-renders because hook deps missing" is NOT a root cause. "`Kanban.jsx:47` — `useEffect` depends on `tasks` array but array ref changes every parent render → infinite fetch loop" IS.

### 6. Verify root cause explains ALL symptoms
If the bug has 3 symptoms and your root cause only explains 2, the cause is wrong or incomplete. Keep digging.

### 7. Write the fix, then verify the fix
- Apply minimal patch
- Re-test original trigger
- Re-test each symptom
- State what's still un-verified

### 8. Document
- If KI exists → file incident under it via `/ki-systematic-resolution`
- If no KI and bug repeats → file new KI
- If one-off → commit message + `task:t_ID` is enough

---

## Anti-patterns to avoid

| Bad pattern | What happens | Replacement |
|---|---|---|
| "Let me just try adding a check" | Mask symptom, bug returns | Confirm root cause first |
| "It works on my machine" | Non-reproducible = not fixed | Reproduce first, fix second |
| "I think it's the race condition" | Guess without evidence | Log timing, prove ordering |
| "The last 3 sessions all tried this" | Ad-hoc pile-on | Stop, run protocol from step 1 |
| "Add a try/catch to suppress" | Hides the real failure | Never suppress without logging |

---

## Output format

When invoking this skill, keep a running structure:

```
SYMPTOM: <one sentence>

HYPOTHESES:
H1. <candidate> — evidence for / against / cheap-to-test
H2. <candidate> — ...
H3. <candidate> — ...

TESTED:
H1 — CONFIRMED/RULED OUT/INCONCLUSIVE — <what you observed>

ROOT CAUSE: <file:line + mechanism>

SYMPTOM COVERAGE:
- Symptom A → explained by root cause ✓
- Symptom B → explained ✓
- Symptom C → NOT EXPLAINED — keep digging

FIX: <minimal patch>
VERIFICATION: <what you re-tested>
RESIDUAL RISK: <what remains unverified>
```

---

## Relation to other skills

- `/ki-systematic-resolution` — run AFTER systematic-debugging confirms root cause; that skill handles the KI record cleanup
- `/ghost-sos` — if stuck at step 3 (hypothesis ranking), deploy a ghost for fresh perspective
- `/verification-before-completion` — run at step 8 to validate the fix actually closes the bug
- `/research-gate` — if root cause involves config/boot/routing/architecture, run research-gate before touching
