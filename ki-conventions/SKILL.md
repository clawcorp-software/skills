---
name: ki-conventions
description: "Known Issue (KI) and Incident filing conventions. Invoke BEFORE filing a KI or incident, when triaging a hashtag vs KI vs incident decision, when handling a 2nd/3rd/4th+ recurrence of a known issue, or when a bug is flagged as bullshit/plaster type. Covers taxonomy, severity scoring, recurrence escalation protocol (steps 1-4), enforcement layer trade-offs, and when to invoke /ki-systematic-resolution."
kind: ours
---

# Known Issue (KI) & Incident Conventions

## Taxonomy (3-level + M:N + hashtag)

| Level | Schema | When |
|---|---|---|
| **Hashtag** | `tag_ki_map` row + `incident_ki_links.source_tag` | Cross-cutting theme (frontend, merge, poke, css, path). Can link ONE incident to MULTIPLE KIs. |
| **MetaKI** (parent) | `parent_id IS NULL`, has `ki_number`, `type='meta'` | Umbrella cluster of related KIs sharing a ROOT CAUSE domain. Ex: KI-66 "Sparse-checkout coverage". |
| **KI** (parent OR child of MetaKI) | `parent_id` = NULL or MetaKI.id. `ki_number` only if top-level. | Recurrent pattern (2+ occurrences) within a specific root cause. |
| **Incident** (child of KI) | `parent_id` = KI.id, NO `ki_number` | Specific occurrence. |

**Propagation:** `incident_ki_links.link_type='child-ki'` auto-propagates events from sub-KIs up to their MetaKI. `source_tag` column routes the same event to multiple KIs simultaneously via hashtag.

**Decision tree for new findings:**
- One-off, no pattern? → Hashtag only. Tag commit, no DB entry.
- Variant of existing pattern? → File incident under existing KI. Call Gemini classifier (future: `POST /cc/ki-classify`) to confirm before creating.
- 2nd occurrence, new pattern? → Create KI + file incident.
- Multi-factorial (spans domains)? → File incident under primary KI, add `incident_ki_links` rows with secondary KIs via `link_type='direct'` and `source_tag='<hashtag>'`.
- Domain-level umbrella emerges from 3+ related KIs? → Create MetaKI (`type='meta'`), reparent member KIs under it via `parent_id`.

## 9-field incident template (MANDATORY)

Every incident MUST fill these 9 fields (Keymaster directive 2026-04-24). Empty fields blocked (future hook on `POST /cc/known-issues`).

1. `title` — short, action-oriented. No timestamps in title.
2. `description` — symptoms (what was observed).
3. `root_cause` — WHY it happened. Verified, not guessed.
4. `drift_analysis` — prior_fix_attempt + why_prior_failed: if this is a 2nd+ occurrence, what was the previous fix and why didn't it hold?
5. `fix` — the NEW fix applied this time. Specific enough to reproduce.
6. `linked_commit_ids` / `linked_pr_ids` — evidence (git refs).
7. `prevention_note` — historical_basis: what rule/hook/check prevents recurrence forever.
8. `impact` — empirical_support: observed blast radius (users affected, duration, workers impacted).
9. `severity` + `agent` + `status` — classification metadata.

## Type Tags
| Type | When |
|------|------|
| `bug` | Broken behavior |
| `stupidity` | Repeated known mistake, ignored rule |
| `breach` | Protocol/gate/policy failure |
| `bullshit` | "Resolved" but wasn't (auto-tagged on reopen) |
| `plaster` | Temp workaround, not root fix |
| `infra` | Servers, tunnels, services |
| `drift` | Config/code drift |
| `ui` | Frontend/display |

## Theme Tags
`#poke` `#propagation` `#normalize` `#sync` `#path` `#pr` `#data-loss` `#merge` `#scope-creep` `#mobile` `#security` `#boot` `#memory-loss` `#research-gate-breach`

## Severity
Minor (-10), Major (-30), Critical (-75), Catastrophic (-150). Multipliers: stupidity x3-x6, bullshit x5-x8. Cap: 500 XP/KI.

## Filing Checklist
**Incident (child):** title (INC-XXX:), type, severity, status, agent, description (1-3 sentences), root_cause (verified), fix, parent_id. NO ki_number.
**New KI (parent):** Search first, confirm recurrent, title, type, severity, description + prevention. ki_number auto-assigned. File 1+ incident immediately.

## Recurrence Escalation Protocol (MANDATORY)

When filing an incident under an EXISTING KI (2nd+ occurrence), the agent MUST:

### Step 1: Prove previous fixes failed
- List ALL previous incidents under this KI
- For EACH previous fix: explain WHY it didn't prevent THIS incident
- Tag the KI as `bullshit` if "resolved" but clearly not fixed

### Step 2: Multi-angle root cause analysis
- Identify the REAL root cause (not the surface symptom)
- Explain why the previous root cause analysis was wrong or incomplete
- Consider: is this the same bug or a DIFFERENT manifestation of a deeper pattern?

### Step 3: Deploy ghost for outside perspective
- Call `/ghost-sos` or `/multi-ghost-sos` to get an external analysis
- Analyst for code/architecture bugs, Sentinel for security, Librarian for pattern tracking
- The ghost report MUST be referenced in the incident fix field

### Step 4: Propose a SYSTEMIC fix
- The fix must prevent the ENTIRE CLASS of bugs, not just this instance
- If the fix is "add a check" or "add a log" = PLASTER, not a fix
- Acceptable fixes: pre-commit hooks, runtime guards, architecture changes, automated tests

### Escalation Thresholds
| Incidents | Action |
|-----------|--------|
| 2nd | Step 1-2 mandatory, Step 3-4 recommended |
| 3rd | **MUST invoke `/ki-systematic-resolution`**. KI auto-tagged `bullshit` if was "resolved" |
| 4th+ | BLOCKED until `/ki-systematic-resolution` completes PASS on all 5 phases |

### Decision Tree Addition
```
New incident for existing KI?
  -> Is this the 2nd occurrence? -> Steps 1-2 mandatory
  -> Is this the 3rd occurrence? -> ALL steps mandatory + tag bullshit
  -> Is this the 4th+ occurrence? -> BLOCKED until root fix shipped
```

## Enforcement Analysis

| Layer | Mechanism | What it does | Strength | Weakness |
|-------|-----------|-------------|----------|----------|
| **Rules** (`.claude/rules/`) | Loaded by Claude Code runtime every session | Agent reads and follows judgment-based rules | Always present, covers complex decisions | Agent can rationalize around it under pressure |
| **Decision Tree** (`decision-tree.json`) | Hook-enforced SCRIPT rules, runs on file save | Blocks bad patterns mechanically (regex, lint) | Zero-token, instant, unbypassable | Only works for mechanical checks (syntax, patterns), NOT for judgment calls like "should I do deep analysis" |
| **Promptfoo** (`promptfoo/*.yaml`) | Eval tests run on-demand or in CI | Verifies agents actually follow rules when tested | Catches regressions, proves compliance | Only runs when triggered, not real-time |
| **Skills** (`.claude/skills/`) | Invoked by agent or auto-triggered | Provides step-by-step workflow for complex tasks | Detailed guidance, reusable across agents | Agent must choose to invoke it |
| **Hooks** (`settings.json`) | Shell commands triggered on events (Stop, Save, etc.) | Automated checks at key moments | Runs automatically, no agent decision needed | Limited to shell commands, can't do judgment |

### Why NOT decision tree for KI escalation?
Decision tree is SCRIPT-based: regex patterns on file content at save time. KI escalation is a JUDGMENT call that happens during conversation, not during file saves. You can't regex-match "is this the 3rd incident?" -- it requires reading the DB, counting incidents, and deciding whether to escalate. That's rules + skills territory, not hooks.

### Best combo for KI escalation
1. **Rules** = always loaded, agent knows the protocol exists
2. **Skills** (`/ghost-sos`) = agent knows HOW to deploy ghosts when escalating
3. **Promptfoo** = proves agents follow the protocol (8/8 tests pass)
4. **Future: Hook** = auto-count incidents on `POST /cc/known-issues` and inject escalation reminder into response
