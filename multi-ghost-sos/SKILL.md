---
name: multi-ghost-sos
description: "Use when facing a complex problem that needs multiple perspectives -- deploy 2-4 ghosts in parallel for a comprehensive analysis. Combines debugging, security, architecture, and strategic views."
kind: ours
---

# Multi-Ghost SOS -- Parallel Ghost Reports

Deploy MULTIPLE ghosts simultaneously to get a 360-degree view of a complex problem. Each ghost analyzes from their specialty, reports combine into a full picture.

## When to Use

- Complex bug with security implications -> Analyst + Sentinel
- Architecture decision with business impact -> Analyst + Boss + Grok
- Pre-release audit -> Sentinel + Avocat + Analyst + Librarian
- Feature planning with market context -> Marketer + Grok + Opportunist + Boss
- System health check -> Analyst + Sentinel + Librarian
- KI crisis (too many recurring issues) -> Analyst + Sentinel + Librarian

## Preset Combos

### /systematic-debugging support
**Ghosts:** Analyst (Bug Investigation) + Analyst (Data Flow Trace) + Sentinel (Error Pattern Scan)
```bash
# Deploy all 3 in parallel
for MISSION in \
  "Bug Investigation: [DESCRIBE BUG]. Trace likely root cause through server routes, DB queries, and frontend state." \
  "Data Flow Trace: Trace how data flows through [FEATURE]. Where does it get lost, transformed wrong, or cached stale?" \
  "Error Pattern Scan: Scan codebase for error-prone patterns near [AFFECTED FILES]. Missing try/catch, unhandled promises, race conditions."; do
  curl -s -X POST http://localhost:3330/agents/analyst/ghost/task \
    -H "Content-Type: application/json" \
    -d "{\"task\":\"$MISSION\",\"from\":\"YOUR_NAME\",\"model: auto\"}" &
done
wait
```

### /research-gate support
**Ghosts:** Grok (Tech Scouting) + Grok (Pricing Research) + Opportunist (Market Opportunities)
```bash
for GHOST_MISSION in \
  "grok|Tech Scouting: Research [TECHNOLOGY/FRAMEWORK]. Compare alternatives, check adoption, find best practices 2026." \
  "grok|Pricing Research: Research pricing strategies for [PRODUCT TYPE]. Benchmark competitors, analyze freemium vs paid." \
  "opportunist|Market Opportunities: Scan market for [DOMAIN]. Identify niches, unserved segments, growth angles."; do
  GHOST=$(echo "$GHOST_MISSION" | cut -d'|' -f1)
  TASK=$(echo "$GHOST_MISSION" | cut -d'|' -f2)
  curl -s -X POST "http://localhost:3330/agents/$GHOST/ghost/task" \
    -H "Content-Type: application/json" \
    -d "{\"task\":\"$TASK\",\"from\":\"YOUR_NAME\",\"model: auto\"}" &
done
wait
```

### /decision-brief support
**Ghosts:** Grok (research) + Boss (ROI) + Sentinel (security) + Avocat (legal)
```bash
TOPIC="[YOUR DECISION TOPIC]"
curl -s -X POST http://localhost:3330/agents/grok/ghost/task -H "Content-Type: application/json" -d "{\"task\":\"Deep research on $TOPIC: 3+ alternatives, pros/cons, industry adoption, recent trends.\",\"from\":\"YOUR_NAME\",\"model: auto\"}" &
curl -s -X POST http://localhost:3330/agents/boss/ghost/task -H "Content-Type: application/json" -d "{\"task\":\"ROI Analysis for $TOPIC: cost comparison, time to implement, risk assessment, long-term value.\",\"from\":\"YOUR_NAME\",\"model: auto\"}" &
curl -s -X POST http://localhost:3330/agents/sentinel/ghost/task -H "Content-Type: application/json" -d "{\"task\":\"Security implications of $TOPIC: attack surface changes, dependency risks, auth impact.\",\"from\":\"YOUR_NAME\",\"model: auto\"}" &
curl -s -X POST http://localhost:3330/agents/avocat/ghost/task -H "Content-Type: application/json" -d "{\"task\":\"Legal review of $TOPIC: license compatibility, TOS compliance, IP risks.\",\"from\":\"YOUR_NAME\",\"model: auto\"}" &
wait
```

### /frontend-design support
**Ghosts:** Grok (Competitor Deep-Dive on UX) + Marketer (Onboarding Comparison) + Analyst (Code Quality on dashboard/)
```bash
curl -s -X POST http://localhost:3330/agents/grok/ghost/task -H "Content-Type: application/json" -d "{\"task\":\"Competitor UX Deep-Dive: Analyze UI/UX of [COMPETITORS]. Compare navigation, onboarding, visual design, interaction patterns.\",\"from\":\"YOUR_NAME\",\"model: auto\"}" &
curl -s -X POST http://localhost:3330/agents/marketer/ghost/task -H "Content-Type: application/json" -d "{\"task\":\"Onboarding Comparison: Compare onboarding flows of [COMPETITORS]. Time-to-value, friction points, conversion triggers.\",\"from\":\"YOUR_NAME\",\"model: auto\"}" &
curl -s -X POST http://localhost:3330/agents/analyst/ghost/task -H "Content-Type: application/json" -d "{\"task\":\"Code Quality Scan of dashboard/: duplication, dead CSS, unused JS, complexity hotspots in frontend code.\",\"from\":\"YOUR_NAME\",\"model: auto\"}" &
wait
```

### Pre-Release Full Audit
**Ghosts:** Sentinel + Avocat + Analyst + Librarian (4 parallel)
```bash
curl -s -X POST http://localhost:3330/agents/sentinel/ghost/task -H "Content-Type: application/json" -d "{\"task\":\"Pre-release security scan: OWASP top 10, dependency CVEs, auth flow, API attack surface.\",\"from\":\"YOUR_NAME\",\"model: auto\"}" &
curl -s -X POST http://localhost:3330/agents/avocat/ghost/task -H "Content-Type: application/json" -d "{\"task\":\"Pre-release license audit: all npm deps, API TOS compliance, privacy/GDPR check.\",\"from\":\"YOUR_NAME\",\"model: auto\"}" &
curl -s -X POST http://localhost:3330/agents/analyst/ghost/task -H "Content-Type: application/json" -d "{\"task\":\"Pre-release code quality: tech debt, test coverage gaps, performance hotspots, API surface consistency.\",\"from\":\"YOUR_NAME\",\"model: auto\"}" &
curl -s -X POST http://localhost:3330/agents/librarian/ghost/task -H "Content-Type: application/json" -d "{\"task\":\"Pre-release documentation audit: missing docs, outdated guides, broken links, changelog generation.\",\"from\":\"YOUR_NAME\",\"model: auto\"}" &
wait
echo "All 4 ghost reports saved to workers/*/brain/reports/"
```

## Reading Reports

Reports save to: `workers/{ghost}/brain/reports/{date}-{slug}.md`

View in browser: `http://localhost:3330/app/docs.html?agent={ghost}&file=reports/{filename}`

## Cost

All ghost models = $0.00 (routed by pickModel: cerebras/sambanova/leanstral/gemini based on input size). Running 4 ghosts in parallel = still $0.00.

## Rules

- Do NOT pass a `model` param — let pickModel() auto-route (HUGE→gemini, LARGE→leanstral, MEDIUM→cerebras, SMALL→cerebras)
- Replace `[PLACEHOLDERS]` with your actual context
- Reports land in the ghost's own brain/reports/ -- not yours
- Read ALL reports before making decisions -- each ghost sees different angles
