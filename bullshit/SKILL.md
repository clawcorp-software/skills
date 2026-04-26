---
name: bullshit
description: "Detect false completions, force the REAL fix, trigger ki-systematic-resolution. Replaces XP-only enforcement that produced theater instead of working code."
---

# /bullshit -- Prove It Works Or Fix It Now

> Not about punishment. About RESULTS. Force the real fix, not XP theatre.

## When to Invoke

| Trigger | Who |
|---------|-----|
| User says "doesn't work" after agent said "done" | User |
| Agent suspects own fix didn't land | Agent (self-invoke) |
| 2nd fix attempt on same visual bug | Auto (agent MUST) |
| Known issue reopened after "resolved" | Auto |
| User says "bullshit" | On demand |

## Phase 1: REREAD (what was actually asked?)

1. Scroll up to the user's ORIGINAL request (not your interpretation)
2. List EACH specific change requested, verbatim
3. For each change, write: "User asked: X. I did: Y."
4. If you can't find the original request, say so -- don't reconstruct from memory

**PASS:** List of user requests with your corresponding actions.

## Phase 2: INSPECT (is it actually fixed?)

Use a browser-automation tool (Playwright MCP, Chrome DevTools MCP, or equivalent) to verify EACH change:

1. Navigate to the affected page on the correct port
2. Snapshot the accessibility tree
3. For UI elements: check the actual element class at the reported position with `document.elementFromPoint`
   - Does the element class match what you fixed? If NO -> **WRONG ELEMENT**
4. Take a visual screenshot for proof
5. If light/dark mode issue: test BOTH modes
6. If mobile issue: resize to 375x812 and re-check

For EACH user request, classify:

| Status | Meaning |
|--------|---------|
| VERIFIED | Screenshot proves fix works |
| WRONG_ELEMENT | Fix applied to wrong DOM element |
| NOT_APPLIED | Fix exists in code but not visible (cache? wrong port? wrong file?) |
| PARTIAL | Some aspects fixed, others missed |
| NOT_DONE | Fix was never actually implemented |

**PASS:** Every request classified with screenshot evidence.

## Phase 3: FIX (not punish -- FIX)

For each non-VERIFIED item:

### WRONG_ELEMENT
1. Grep codebase for ALL elements at the reported position
2. `grep -r "position:fixed\|position:absolute" path/to/css/ | grep "top\|right"`
3. Check shared.js / dynamic injections: `grep "createElement\|innerHTML" path/to/shared.js`
4. Identify the REAL element -> fix IT
5. Re-run Phase 2 to verify

### NOT_APPLIED
1. Check which port/server the user is on
2. Verify the file being served: `curl -s http://localhost:PORT/path | grep "your-fix"`
3. Check Service Worker: is old version cached?
4. Check if CSS was rebuilt: `node build-css.js` (or your build script)
5. Fix the delivery problem -> re-verify

### PARTIAL
1. List what's missing
2. Implement missing parts
3. Re-verify ALL items (not just the new ones)

### NOT_DONE
1. Acknowledge honestly: "I said done but didn't implement this"
2. Implement now
3. Re-verify

**PASS:** ALL items now VERIFIED with screenshots. No exceptions.

## Phase 4: ESCALATE (if pattern detected)

After fixing, check for patterns:

1. Is this the 2nd+ time this specific fix failed?
   - File an incident under the existing known-issue (or create a new one)
   - If 2nd: run `/ki-systematic-resolution` Phase 1-2
   - If 3rd+: run `/ki-systematic-resolution` FULL

2. What SYSTEMIC change prevents this class of failure?

   | Failure type | Systemic fix |
   |-------------|-------------|
   | Wrong element | Add browser-verification step to workflow |
   | False completion | Mandatory visual verification before "done" on UI fixes |
   | Cache/delivery | Bump SW revision + cache buster in commit hook |
   | Not implemented | Reread checklist in pre-flight step 0 |

3. Propose the systemic fix. IMPLEMENT if within scope, escalate if not.

**PASS:** Pattern analyzed, systemic fix proposed or implemented.

## Phase 5: RECORD

1. File the incident in your KI/issue tracker if not already filed
2. Update agent memory with what was learned
3. NO punishment if agent self-invoked `/bullshit` (self-report = courage, not failure)
4. Punishment ONLY if the user had to invoke it AND the fix was NOT_DONE or WRONG_ELEMENT

## What This Replaces

XP-only enforcement was theatre. This skill:

- FORCES the real fix (Phase 3)
- VERIFIES with browser automation (Phase 2)
- PREVENTS recurrence (Phase 4)
- XP/blame is a signal, not the goal (Phase 5)

## Credits

Authored by ClawCorp -- multi-agent dev orchestration. Battle-tested across 200+ KI cycles where false-completion was the #1 recurrence vector.
