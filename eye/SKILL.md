---
name: eye
description: "Visual QA + browser-debug workflow. Use Playwright MCP for reproducible QA, then escalate to Chrome DevTools MCP for live CSS inspect, computed styles, console, network, performance, and logged-in state."
tags: [qa, visual, playwright, testing, frontend]
---

# /eye -- Visual QA + Browser Debug Workflow

Systematic frontend quality inspection using Playwright MCP headless browser, with Chrome DevTools MCP escalation for live-browser diagnosis.

## When to Use

- Before any frontend PR (run on localhost)
- After CSS/HTML changes to verify no regressions
- Eye Checkup from kanban (automated)
- Any time visual QA is needed across viewports

## Process

### Phase 0: Fix Verification Mode (when called after a UI fix)
1. REREAD the user's original request (scroll up, find exact words)
2. `browser_navigate` to the affected page
3. `browser_evaluate`: `document.elementFromPoint(x,y).className` -- verify RIGHT element
4. `browser_take_screenshot` -- compare against user's reported issue
5. If element class DOESN'T match what you fixed -> **WRONG ELEMENT**, stop and grep for real element
6. If visual doesn't match expected fix -> **FIX DIDN'T WORK**, investigate before saying done

### Phase 0b: Live Browser Escalation (when Playwright evidence is not enough)
Escalate to Chrome DevTools MCP when ANY of these are true:
- Need real logged-in/browser state (cookies, localStorage, extensions, session)
- CSS cause is unclear: computed styles, box model, parent flex/grid/sticky/overflow constraints
- Need console/network evidence from the real page
- Need performance evidence (jank, long tasks, layout shifts)
- UI bug may actually be backend-linked (failing API call, wrong payload, bad response)

Use Chrome DevTools MCP to:
1. Inspect the real element and read computed styles + box model
2. Check parent containers for layout constraints
3. Read console errors/warnings
4. Inspect network request URL, status, payload, response, timing
5. Record a performance trace if the problem is rendering or interaction lag

Rule: Chrome DevTools MCP is for diagnosis. Playwright remains the reproducible QA pass before claiming done.

### Phase 1: Setup
1. Load Playwright tools: `ToolSearch select:mcp__playwright__browser_navigate`
2. Also load: `browser_snapshot`, `browser_resize`, `browser_take_screenshot`, `browser_console_messages`
3. If first run: `browser_install` to download Chromium

### Phase 2: Desktop QA (1440px)
1. `browser_navigate` to target URL
2. `browser_snapshot` -- read accessibility tree (structured, token-efficient)
3. `browser_take_screenshot` full page -> `tmp/screenshots/eye-desktop.png`
4. Check: all sections present, no missing content, no broken refs

### Phase 3: Tablet QA (768px)
1. `browser_resize` width=768 height=1024
2. `browser_snapshot` -- verify content reflows correctly
3. `browser_take_screenshot` full page -> `tmp/screenshots/eye-tablet.png`
4. Check: no horizontal overflow, touch targets accessible

### Phase 4: Mobile QA (375px)
1. `browser_resize` width=375 height=812
2. `browser_snapshot` -- verify mobile layout
3. `browser_take_screenshot` full page -> `tmp/screenshots/eye-mobile.png`
4. Check: no overflow, hamburger menu appears, content readable

### Phase 5: Theme Toggle
1. `browser_resize` width=1440 height=900 (back to desktop)
2. Find theme toggle button in snapshot
3. `browser_click` the toggle
4. `browser_take_screenshot` -> `tmp/screenshots/eye-light-mode.png`
5. Check: colors swap correctly, text contrast readable, no invisible elements
6. Toggle back to dark mode

### Phase 6: Interaction Test
1. Click a primary interactive element (agent card, button, link)
2. `browser_snapshot` -- verify the interaction response (panel opens, state changes)
3. Check: no JS errors, expected UI response

### Phase 7: Console + Network Check
1. `browser_console_messages` level=error
2. Filter out expected CORS errors (localhost -> production auth)
3. Flag any real JS errors
4. If the UI talks to the backend, inspect network requests too
5. If request/response details matter, switch to Chrome DevTools MCP and capture the exact failing route before editing backend code

### Phase 8: Report

Generate a pass/fail table:

| Check | Status | Notes |
|-------|--------|-------|
| Desktop 1440px | PASS/FAIL | details |
| Tablet 768px | PASS/FAIL | details |
| Mobile 375px | PASS/FAIL | details |
| Light mode | PASS/FAIL | details |
| Interactions | PASS/FAIL | details |
| Console errors | PASS/FAIL | count |

**PASS** = all checks green. **FAIL** = list specific issues.

Screenshots saved in `tmp/screenshots/eye-*.png`.

## Checklist (quick reference)

- [ ] All sections render at 1440px
- [ ] All sections render at 768px
- [ ] All sections render at 375px (no horizontal overflow)
- [ ] Hamburger menu appears on mobile
- [ ] Light/dark mode toggle works
- [ ] Colors swap correctly (no invisible text)
- [ ] Interactive elements respond to clicks
- [ ] No real JS errors in console
- [ ] No broken images (check img refs in snapshot)
- [ ] No 404 resources in network
- [ ] If CSS/layout cause stayed unclear, Chrome DevTools MCP was used for computed styles/box model
- [ ] If UI/backend bug involved API calls, request/response evidence was captured before server edits

## Token Cost

~15-30k tokens per full inspection (snapshots + screenshots).
Compared to Chrome DevTools MCP: much cheaper for broad QA. Use DevTools selectively for diagnosis, not as the default full-page inspection pass.
