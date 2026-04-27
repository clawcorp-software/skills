---
name: decision-brief
description: Use when proposing any new tool, framework, architecture, or significant technical decision. Forces research-backed decision making with minimum 3 independent sources.
user_invocable: true
kind: ours
---

# Decision Brief (Research Gate)

You are creating a DECISION BRIEF. This is GATE A of the ClawCorp compliance system.
NO technical decision passes without this document.

## Purpose

The worker decision brief is a research and recommendation artifact.
It is NOT the canonical official decision record.

Canonical rule:
- Worker brief: `workers/<agent>/brain/DECISION-BRIEF-<slug>.md`
- Official approved decision: `docs/decisions/<slug>.md`
- Promotion to official: recommended by Librarian or a human, approved by a human, then written into `docs/decisions/`

This avoids two competing sources of truth.

## Required Linking

Every decision brief should link outward to the evidence chain when those artifacts exist.
Do NOT leave the brief isolated if supporting material was created.

Link to:
- related plan docs in `docs/plans/`
- web sources used for the research
- deep research reports in `brain/reports/` or `docs/reports/`
- related incidents or known issues when relevant
- the canonical `docs/decisions/` file if the brief is later promoted

If you create a companion deep research report, mention it explicitly in the brief.
If a plan already exists for the work, link it.
If the brief caused a canonical decision, add a "Promoted to" link.

## Process

1. **Ask the user** what decision needs to be made (if not already clear)

2. **WebSearch** for at least 3 independent queries related to the decision:
   - "best practices [topic] 2026"
   - "[option A] vs [option B] comparison"
   - "alternatives to [proposed solution]"

3. **Create the worker brief** at `brain/DECISION-BRIEF-<slug>.md` with this EXACT format:

```markdown
# Decision Brief: <Title>
Date: <today>
Author: <agent name>
Status: PENDING REVIEW

## Problem Statement
<1 paragraph: what problem are we solving?>

## Options Considered
### Option 1: <name>
- Description: ...
- Pros: ...
- Cons: ...
- Cost: ...

### Option 2: <name>
(same format)

### Option 3: <name>
(same format, MINIMUM 3 options)

## Sources (minimum 3 independent links)
1. [title](url) - key finding
2. [title](url) - key finding
3. [title](url) - key finding

## Recommendation
<Which option and WHY, with tradeoffs acknowledged>

## What We Are NOT Doing (and honest why)
<List ONLY genuinely bad ideas with concrete reasons. NOT a dumping ground for "too hard" or "too complex".
If an option is rejected ONLY because it's more work, that's NOT a valid reason -- flag it as the better option that costs more.
NEVER use this section to justify laziness or avoid the best solution.>

## Evidence Chain
- Plan: <path or "none yet">
- Deep Research: <path or "none">
- Related Reports: <path(s) or "none">
- Related Incidents / KI: <path(s) or IDs or "none">
- Canonical Decision: <docs/decisions/... or "not promoted">

## Effort Reality Check
<For the recommended option: estimate MINUTES of real work, not calendar days. Be honest -- you're faster than you think.>
```

4. **Present the brief** to the user for approval before any implementation

5. **If the decision becomes official**, create or update the canonical doc in `docs/decisions/`.
   - Do NOT copy the worker brief verbatim and call that canonical.
   - Rewrite it as an approved decision record.
   - Cite the source brief when relevant.

## Promotion Guidance

Promotion is appropriate when a brief:
- explains a lasting architecture or product decision
- affects how multiple agents or projects should work
- needs to be visible in the demo or public-facing docs
- should be treated as approved guidance rather than raw research

Promotion is NOT automatic.
Librarian may recommend promotion, but the final officialization remains human-approved.

## Rules
- MINIMUM 3 options (including "do nothing" if applicable)
- MINIMUM 3 web sources with real URLs (no hallucinated links)
- MUST include cost analysis (tokens, money, time, complexity)
- MUST include "What We Are NOT Doing" section
- MUST include an `Evidence Chain` section
- NO implementation until brief is approved
- If WebSearch is unavailable, use WebFetch on documentation URLs instead
- Do NOT treat the worker brief as the final public source of truth if the decision needs to be canonical
