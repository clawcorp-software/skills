---
name: extension-ownership
description: "TOUR-OWNED file access rules. Invoke BEFORE editing tower/extensions/clawcorp-launcher/extension.js, before copying to ~/.vscode/extensions/, or before running the api-extension-contract.test.js. Covers the 5 API endpoints contract (GET /fleet/status, POST /agents/:name/launch, POST /agents/:name/heartbeat, PATCH /agents/:name, POST /extension/heartbeat), the mandatory deploy step to ~/.vscode/extensions/, and regression testing. KI-63 recurring issue -- only Tour edits extension.js."
---

# Extension Ownership Rule (KI-63)

## tower/extensions/clawcorp-launcher/extension.js is TOUR-OWNED

**ZERO exception:** Only Tour modifies extension.js.

**Why:** 15+ commits by different agents (Manager, Forge, Tour) caused recurring extension instability (KI-63, bullshit type). Each fix broke something another agent had fixed. Pattern: no agent tested their change live in VS Code.

## Before ANY change to extension.js

1. **Read the full file** -- it's 1415 lines. Read ALL of it.
2. **Check what endpoints it calls** -- any API change must be reflected in tests/api-extension-contract.test.js
3. **Deploy to ~/.vscode/extensions/** after editing (always copy the source to installed):
   ```powershell
   Copy-Item "$env:CLAWCORP_ROOT/tower/extensions/clawcorp-launcher/extension.js" "$env:USERPROFILE/.vscode/extensions/clawcorp.clawcorp-launcher-1.2.6/extension.js" -Force
   ```
4. **Test live**: open fleet webview, click BOOT, verify model appears in terminal launch command.
5. **Run regression tests**: `cd tests && node --test api-extension-contract.test.js`

## 5 API Endpoints (contract -- NEVER break these)

| Method | Path | What extension does | Tested? |
|--------|------|---------------------|---------|
| GET | /fleet/status | Polls every 15s | YES |
| POST | /agents/:name/launch | BOOT button -- must include `{model, cli}` | YES (KI-63) |
| POST | /agents/:name/heartbeat | Lock file refresh | YES (KI-63) |
| PATCH | /agents/:name | Model dropdown change | YES (KI-63) |
| POST | /extension/heartbeat | Extension alive ping | YES (KI-63 -- was 404!) |

## Server auto-start

Server (`localhost:3330`) has NO process manager since PM2 eliminated 2026-04-13.
If server is down: `doppler run -- node "$CLAWCORP_ROOT/server/index.js"`
Extension will show "API offline" until server is restarted.

## Recurrence escape hatch (KI-wqgiw, 2026-04-26)

Tour ownership of a given file is not a perpetual right. When Tour ships
**2 or more bullshit closures on the same KI** for a Tour-owned file --
i.e. Tour declares the bug fixed, the worker re-opens it with a correct
diagnosis, and Tour misdiagnoses it again -- ownership of THAT specific
file may transfer to the worker who first read the code correctly.

**Required for transfer:**
- Keymaster authorization in chat (not in a file, not a POKE -- a direct
  message saying "ownership transfers to {worker} for {path}").
- KI record updated with the transfer event so future agents see who owns
  the file now.
- Ownership transfer is **per-file**, not per-directory. Adjacent files in
  the same folder stay with their previous owner unless individually
  transferred.

**Does NOT apply to:**
- `tower/scripts/**` -- coordination plumbing, hard-enforced in
  `hook-pre-unified.js:344` `protectedPatterns`. Workers cannot edit
  these regardless of how many bullshit closures occurred.
- `tower/config/**` -- agent registry + gate config, same hard rule.
- `.claude/rules/**`, `CONSTITUTIONAL.md`, `.claude/settings.json` -- top
  of `protectedPatterns`.

The escape hatch exists so a competent worker is not perpetually blocked
on a file Tour repeatedly fails to fix, while still keeping the
governance plumbing (scripts/config/rules) firmly in Tour's lane.
