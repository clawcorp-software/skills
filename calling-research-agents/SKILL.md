---
name: calling-research-agents
description: Use when a dev agent needs to delegate research, legal analysis, market intel, competitor scans, security audits, or data synthesis to a ClawCorp research agent via the dashboard API
---

# Calling Research Agents

## Overview

ClawCorp has 8 research ghost agents accessible via HTTP API. They run `claude -p` on-demand with their SOUL + MEMORY injected. No Slack, no worktree — pure research output written to brain/reports/.

**3-step pattern: ARM → TASK → READ**

## Quick Reference — Which Agent for What

| Agent | Use for |
|-------|---------|
| Avocat | Legal review, contract analysis, compliance, TOS risks |
| Grok | Deep research, multi-source synthesis, "find everything on X" |
| Sentinel | Security audit, threat model, vulnerability assessment |
| Librarian | Archive, cross-reference past decisions, find prior art in brain/ |
| Analyst | Metrics, data synthesis, structured reports, dashboards |
| Opportunist | Market gaps, competitor weaknesses, growth angles |
| Marketer | GTM positioning, copy, ICP validation, channel strategy |
| Boss | Strategic synthesis when you need a decision, not just data |

## The 3-Step Pattern

### Step 1 — ARM (activate the agent)

```bash
curl -s -X POST http://localhost:3332/agents/avocat/ghost/start \
  -H "Content-Type: application/json"
```

Returns `{ "status": "armed" }`. Required before sending a task. If already armed, harmless.

### Step 2 — TASK (send the research request)

```bash
curl -s -X POST http://localhost:3332/agents/avocat/ghost/task \
  -H "Content-Type: application/json" \
  -d '{"task": "Review the Stripe TOS for revenue share clauses that could affect our SaaS model. Flag any clauses requiring legal escalation.", "from": "architecte"}'
```

Returns immediately with `{ "status": "completed", "reportName": "...", "preview": "first 200 chars..." }`.

The `task` field: plain text, max 4000 chars. Be specific. Include context.

### Step 3 — READ (get the report)

```bash
# List reports (most recent first)
curl -s http://localhost:3332/agents/avocat/ghost/reports

# Read specific report
curl -s http://localhost:3332/agents/avocat/ghost/reports/2026-03-08-1430-stripe-tos-review.md
```

## Parallel Calling Pattern

When you need multiple agents simultaneously:

```bash
# Fire both in parallel (background jobs)
curl -s -X POST http://localhost:3332/agents/grok/ghost/start &
curl -s -X POST http://localhost:3332/agents/opportunist/ghost/start &
wait

# Send tasks in parallel
curl -s -X POST http://localhost:3332/agents/grok/ghost/task \
  -d '{"task": "Research open-source KDS alternatives to cloud-kitchen. Find GitHub stars, last commit, adoption.", "from": "forge"}' &
curl -s -X POST http://localhost:3332/agents/opportunist/ghost/task \
  -d '{"task": "What gaps in the ghost kitchen market are NOT served by existing tools? Focus on solo operators.", "from": "forge"}' &
wait
```

## After Getting Results — Post to PR/Slack

```bash
# Post summary in #clawcorp (short version)
powershell.exe -ExecutionPolicy Bypass -File "C:/Users/user/clawcorp/tower/scripts/slack/slack-post.ps1" \
  -Bot "ton-nom" -Channel "#clawcorp" \
  -Message "[RESEARCH:avocat] TOS review done. 2 clauses flagged. Full report: brain/reports/avocat/..."

# For PR: attach report path in PR description or link in comment
```

## Pipeline Pattern (compound tasks)

When a task needs data AND a decision, split across agents sequentially:

```
Opportunist/Grok/Sentinel (raw data) → Boss (synthesis + decision)
```

Never ask Boss to do raw research. Never ask Opportunist for strategic decisions. Boss needs context injected: give it the other agents' report paths or paste key excerpts into the task prompt.

## Stale Reports

The `/ghost/reports` list is sorted newest-first. The task response includes `reportName` with the timestamp. If you're unsure whether a report is fresh, check the filename prefix (`YYYY-MM-DD-HHMM`). If it's old, just send a new task — each call produces a new report file, old ones are preserved.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Skip ARM step | Always ARM first, even if you think agent is active |
| Use uppercase agent name in URL | Always lowercase: `/agents/avocat/`, not `/agents/Avocat/` |
| Try to POKE instead of API | Research agents don't have POKE. Use HTTP only. |
| Send task without context | Include project name, what you already know, what you need |
| Forget to read report | preview in task response is first 200 chars only — always GET the full report |
| Use wrong port | Manager API = port 3332. Always `http://localhost:3332` |
| Giant task prompt | Max 4000 chars. Trim or split. |
| Ask Boss for raw research | Boss = synthesis + decisions only. Get data from other agents first. |
| One giant multi-purpose task | Split by agent specialty. Data agents → then Boss. |

## Agent Personalities (context for better prompts)

- **Grok**: Loves multi-source synthesis. Give it a research question with "find all sources". Expects WebSearch usage.
- **Avocat**: Cautious. Give it specific documents or TOS URLs to review, not vague "legal questions".
- **Librarian**: Best for "have we solved this before?" — queries brain/ archives across all workers.
- **Boss**: Give it competing options + data, ask for a decision + rationale. Not for raw research.
- **Sentinel**: Give it code paths, infra diagrams, or threat scenarios. Outputs CVSS-style severity.
