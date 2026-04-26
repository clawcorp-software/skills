---
name: proof-or-punish
description: "XP enforcement -- auto-penalize bullshit fixes, plasters, breaches. Auto-reward clean deliveries."
user_invocable: true
category: high
---

# Proof or Punish -- XP Enforcement System

Audits agent work for honesty and quality. Applies XP penalties or rewards automatically.

## When to Use

- KI tagged bullshit or plaster
- Verification fails (claim without evidence)
- Agent pushes to wrong branch (breach)
- Keymaster says "punish" or "proof or punish"

## Step 1: AUDIT

Gather evidence: recent KIs, XP, incidents, PRs.

## Step 2: CLASSIFY

| Offense | XP Penalty | Tag |
|---------|-----------|-----|
| Bullshit fix (KI reopened) | -150 | #bullshit |
| Plaster (bandaid) | -50 | #plaster |
| Breach (wrong branch/repo) | -500 | #breach |
| False completion | -200 | #false-claim |
| Repeated mistake (3rd+) | -300 | #stupidity |

| Achievement | XP Reward | Tag |
|-------------|----------|-----|
| Systemic fix | +200 | #systemic |
| Clean delivery | +100 | #clean |
| KI killed forever | +300 | #killed-it |
| Self-report | +50 (reduced penalty) | #honest |

## Step 3: EXECUTE

POST /cc/xp/log with agent, xp_change, reason, source:"proof-or-punish"

## Step 4: ANNOUNCE

Penalty: [PUNISH:agent] -500 XP -- reason. Total: XXXX XP.
Reward: [PROOF:agent] +200 XP -- reason. Total: XXXX XP.

## Step 5: UPDATE KI

Penalize ORIGINAL author of bullshit fix, not discoverer.

## Rules

- NEVER negotiate. Table is law.
- Self-report = 50% reduced penalty
- Keymaster can override
- Cumulative
- XP can go NEGATIVE
- Agent MUST acknowledge before continuing
