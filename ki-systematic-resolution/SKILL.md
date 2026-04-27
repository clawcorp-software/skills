---
name: ki-systematic-resolution
description: "MANDATORY (a) when filing a 3rd+ incident under an existing known-issue, (b) when a recurring fix is clearly not holding in production, OR (c) when about to create a new parent known-issue for a product surface that MIGHT already have one. Run this to repair the known-issue record itself before repairing code: verify the correct parent, create or fix incidents, attach screenshots to the right record, classify old fixes, gather only real evidence, implement a systemic fix, verify it, then update the record with cumulative human-readable documentation. Prevents thin-KI drift (one parent per recurrence instead of one parent + N incidents)."
kind: ours
---

# Known-Issue Systematic Resolution

Use this skill to stop known-issue drift. The goal is not just to patch code. The goal is to make your bug-tracking system truthful, cumulative, and useful for the next agent or engineer who hits the same surface.

## Non-Negotiable Rules

1. Fix the known-issue record before declaring the code fixed.
2. Parent KI must describe the real product surface in plain language.
3. Current bug must exist as an incident under that parent KI.
4. Screenshot must be attached to the current incident or the correct parent KI, never to an unrelated KI.
5. The cumulative `fix` summary must be human-readable.
6. Canonical fix events are the source of truth. If a KI summary contains a fix description but no canonical fix event, backfill it.
7. Never add a commit, PR, URL, or task link unless it actually resolves and is accessible now.
8. If a commit is local only, write `commit pending push` instead of inventing a GitHub link.
9. If a task link does not open from the current host, do not present it as proof.
10. Incidents first, fixes second. Do not skip incident filing.

## Phase 1: Identify The Correct KI

1. Read the parent KI and all children.
2. Ask: is this really the same product surface?
3. If the bug is filed under the wrong KI:
   - reparent it now
   - rename the KI now
   - move screenshots now
4. Rewrite vague KI titles.

Good parent KI titles:
- `Desktop installer regressions`
- `Billing export drift`
- `Extension launch contract regressions`

Bad parent KI titles:
- `Desktop buttons unclickable`
- `Broken by refactor`
- `Still bad`

PASS:
- parent KI is the right bucket
- title is understandable to a human in 3 seconds
- current bug is not living under an unrelated KI

## Phase 2: File The Current Incident Properly

For the current recurrence, create or repair the incident so it answers all of this clearly:
- what the user saw
- where they saw it
- what they expected
- what happened instead
- what proof exists

Minimum incident fields:
- clear title in plain language
- product surface: desktop / installer / billing / docs / extension / etc.
- exact symptom
- exact trigger
- screenshot attached
- linked task if one exists

Incident title formula: `INC: [surface] [user-visible symptom]`

Examples:
- `INC: installer stuck after Ready`
- `INC: cost page shows NO EXPORT on mobile`
- `INC: classifier action opens dead link`

PASS:
- there is a real incident for this recurrence
- screenshot is attached to that incident or its correct parent KI
- the wording is understandable without reading code

## Phase 3: Forensics On Old Fixes

1. Fetch all prior incidents under the KI.
2. For each prior fix, classify it:
   - `ROOT_FIX`
   - `PLASTER`
   - `WRONG_CAUSE`
   - `INCOMPLETE`
   - `UNVERIFIED`
3. Explain why each old fix failed to prevent the current incident.
4. If the KI summary has fix text but no canonical fix event, create a fix event now.

Required output format:

| Incident | Old fix | Classification | Why it failed |
|----------|---------|----------------|---------------|

Rules:
- `PLASTER` means symptom-only
- `UNVERIFIED` means nobody proved it on the real surface
- repeated fake closure means the KI type likely becomes `bullshit` (see companion `/bullshit` skill)

PASS:
- every prior fix is classified
- the current recurrence is explained by those failures

## Phase 4: Evidence Hygiene

Only keep evidence that is real now.

### Commits
- Include only if the commit exists and is reachable where the user will click it.
- If not pushed, say `commit pending push`.
- Do not add dead GitHub commit URLs.

### PRs
- Include only if the PR exists.
- Do not prefill a PR number.

### Tasks
- Link the actual task id used by the app.
- Verify the task opens from the current host.
- If it does not open, fix the task linkage or omit it as proof.

### Screenshots
- Attach to the incident or correct parent KI.
- Remove from unrelated KIs if it causes confusion.

PASS:
- no dead links
- no fake proof
- no screenshot on the wrong KI

## Phase 5: Root Cause

Use a structured debugging protocol (hypothesis isolation, evidence gathering, ruling out before ruling in).

Then express the root cause in this format:
- symptom
- actual failing component
- why previous fixes missed it
- exact file:line evidence

For UI or desktop bugs, always distinguish:
- hit-testing problem
- release artifact drift
- runtime deployment problem
- launch handoff problem
- auth or routing problem

PASS:
- one root cause statement
- backed by exact code evidence
- clearly different from user symptom wording

## Phase 6: Outside Perspective

When the recurrence is 3rd+ or the KI is tagged `bullshit`, get an outside read. This can be:
- a sub-agent run
- a peer review
- a different model on the same prompt

Required question for the outside reader:
> "What are we still misclassifying in this KI, and what proof is fake or missing?"

Do not use the outside output as proof by itself. Use it to challenge your diagnosis.

PASS:
- outside perspective consulted
- disagreements reconciled in writing

## Phase 7: Systemic Fix

A valid fix must do both:
- fix the live bug
- improve the KI / documentation structure so the next recurrence cannot be misfiled or misdescribed

Accepted fixes:
- architecture change
- release pipeline correction
- guard that prevents false success
- canonical event backfill + real linkage
- regression test covering the real trigger

Rejected fixes:
- vague text only
- `added log`
- `added null check` without proof
- changing the KI title only

PASS:
- code/system fix implemented
- KI structure fixed too

## Phase 8: Verify

You must verify:
1. the bug trigger
2. the user-visible surface
3. the linked task opens
4. screenshot is on the correct KI/incident
5. fix event exists if the fix summary exists
6. no dead commit/PR URLs remain

For desktop/installers also verify:
- artifact version is the expected one
- runtime package exists if installer depends on it
- install marker is not written for an incomplete install

PASS:
- verification command(s) executed
- result stated plainly
- anything not verified is called out explicitly

## Phase 9: Update The KI Canonically

After the fix is verified, update the database in this order:

1. incident
2. fix event
3. parent KI cumulative fix summary
4. prevention note
5. linked tasks / commits / PRs / screenshots

### Parent KI fix format

Use plain language:

- `What broke:`
- `Why it kept recurring:`
- `What was changed now:`
- `What remains open:`

### Incident fix format

Use exact current wording:

- `Observed symptom:`
- `Root cause:`
- `Exact fix:`
- `Verification:`

### Canonical event requirement

If the KI's `fix` field is updated, also create a `fix` or `resolution` event.
Do not leave the summary without a canonical event.

PASS:
- incident and parent KI are both updated
- fix timeline is canonical
- warnings about missing canonical fix events are gone or explicitly explained

## Phase 10: Ship

1. Commit only after verification.
2. Commit format: `fix(ki-NNN): short description task:t_TASK_ID`
3. If commit not pushed yet, do not add a GitHub commit link.
4. Create PR to your dev branch only after the KI record is truthful.

## Failure Modes

| Situation | Action |
|----------|--------|
| Wrong KI title or wrong parent | Stop and repair the KI structure first |
| Screenshot on wrong KI | Move or reattach it before documenting the fix |
| Task link dead | Fix linkage or remove it from evidence |
| GitHub commit URL 404 | Remove it from proof; use `commit pending push` if local only |
| Fix summary exists without canonical fix event | Backfill a fix event immediately |
| No incident for the current bug | File one before closing anything |
| Fix text is vague | Rewrite it in user-visible language |
| Verification incomplete | Keep task and incident open |

## Definition Of Done

This skill is complete only when ALL of the following are true:
- the bug is under the correct KI
- the current recurrence exists as an incident
- screenshots are attached to the right record
- old fixes are classified
- fake or dead proof is removed
- fix history is canonical and cumulative
- wording is clear to a non-author
- verification was actually run
- only then is the KI or incident resolved

## Companion Skills

- `/bullshit` -- detect false completions and force the real fix before this skill is invoked
- A structured debugging skill (e.g. systematic-debugging) -- use during Phase 5
- A verification-before-completion skill -- use during Phase 8
