---
name: verification-before-completion
description: "Canonical pre-close verification. Run before claiming done, before creating a PR, and before saying a fix holds. Independent from frontend-preflight."
kind: adapted
upstream: https://github.com/obra/superpowers
---

# /verification-before-completion

> Adapted from [`obra/superpowers`](https://github.com/obra/superpowers). Core principle preserved: evidence before claims, always. ClawCorp layer adds backend/data/routing/install/KI/docs scope and explicit independence from `frontend-preflight`.

This is the final truth check before saying:
- done
- fixed
- ready for PR
- ready for merge

It is not a frontend-only skill.
It must work for backend, data, routing, install flows, docs wiring, KI cleanup, and UI.

`frontend-preflight` is optional and separate:
- use it only when the change touched dashboard/frontend surfaces
- do not substitute it for verification
- do not make verification depend on it

## Required Outputs

You must verify the real changed surfaces, not just restate intent.

Minimum:
- code or route parses / loads
- runtime path touched by the fix is checked
- relevant data relations are readable
- no false success claim remains
- residual risk is stated plainly
- the full user request history in the chat is reread before closing
- the delivered result is evaluated against the actual requests made in the chat

## Verification Matrix

### 1. Code Validity

Run the narrowest real check:
- `node --check` for changed server/dashboard JS
- targeted test file if one exists
- build / package validation if release flow changed

Fail if:
- syntax is broken
- imports fail
- route cannot load

### 2. Runtime Truth

Check the thing the user actually uses:
- API endpoint
- page on localhost
- installer flow
- DB row / relation
- generated artifact

Fail if:
- only the source changed but the runtime path is still stale
- localhost still serves the old behavior

### 2b. Desktop runtime gate (MANDATORY for any window.clawcorp consumer)

If the changed code touches any of:
- `window.clawcorp.*` (Electron preload bridge: isDesktop, version, openOAuth, win.*, signOut)
- `ipcRenderer.invoke('clawcorp:*')`
- The `cc-desktop-bar` overlay injected by `desktop/main.js`
- Any code path conditional on Electron UserAgent
- `clawcorp://` deep-link consumers
- `BrowserWindow` popup behavior (`window.open` vs new tab vs in-app navigation)
- `window.prompt()` / `confirm()` / `alert()` (Electron blocks these)

THEN cloud Playwright on `/demo/*` or `/app/*` is **NOT sufficient verification**.
The cloud has no preload, no IPC, no Electron UA, no protocol handler, no
popup-blocked `window.open` behavior. Code that PASSES cloud Playwright can be
100% broken in Electron.

Required:
- Run the actual Electron build OR local dev (`cd desktop && CLAWCORP_DESKTOP_BASE_URL=https://clawcorp.io npx electron .`)
- Click through each affected button
- Capture the result per click (success toast / error / blank popup / nothing)
- Document outcome alongside the cloud verification

If you cannot run Electron in this session, mark the change `UNVERIFIED for Desktop`
in residual risk and DO NOT claim the user-visible bug is fixed. The KI pattern
`Desktop installer regressions` (issue_1775573778048_12vwe) has 18+ incidents
from exactly this gap: ship + cloud-verify + claim fixed + user installs Desktop
+ finds it broken + file + repeat.

### 3. Relation Truth

If the work touched KI / incidents / tasks / docs / tribunal / reports:
- verify canonical relations, not just summary fields
- ensure missing relations do not throw
- ensure guessed relations stay conservative

Typical checks:
- incidents
- tasks
- commits
- plans
- decision briefs
- sources
- screenshots

### 4. No False Green

Do not say fixed if any of these remain true:
- user-visible path still broken
- fix is only local but runtime is stale
- incident count / relation count is obviously wrong
- expected evidence is still missing
- the user asked for something specific and the response drifted away from it

### 5. Prompt Alignment

Before closing:
- reread the user messages that defined the request in this chat, not just the latest prompt
- restate the real ask in one sentence internally
- compare the delivered result to that full ask
- check whether anything important was skipped, replaced, or reframed incorrectly

Fail if:
- the answer solved a nearby problem instead of the requested one
- the work answered only part of the ask without saying so
- the implementation changed scope without explicit justification
- an earlier user requirement from the same chat was dropped or forgotten

### 6. Residual Risk

Always state:
- what passed
- what is still unverified
- what requires runtime merge/restart/deploy

## Output Format

Use this exact structure:

```text
VERIFICATION BEFORE COMPLETION
code: PASS/FAIL - ...
runtime: PASS/FAIL - ...
relations: PASS/FAIL - ...
false-green check: PASS/FAIL - ...
prompt alignment: PASS/FAIL - ...
residual risk: ...
verdict: READY / BLOCKED
```

## Special Rules

- No generic “looks good”
- No claiming done from static reading alone
- No coupling this skill to `frontend-preflight`
- If runtime is stale, verdict is not READY
