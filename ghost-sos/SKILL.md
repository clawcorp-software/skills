---
name: ghost-sos
description: "Use when a worker is stuck on a bug, design problem, architecture decision, or any task where external analysis would help. Automatically selects the right ghost(s) and deploys targeted reports via /api/llm/call."
---

# Ghost SOS -- Single Ghost Report

Deploy ONE ghost to produce a targeted report that helps solve your current problem.

## When to Use

- Stuck debugging a bug -> deploy Analyst (Bug Investigation or Error Log Analysis)
- Need architecture guidance -> deploy Analyst (Architecture Review)
- Security concern -> deploy Sentinel (Vulnerability Scan or Auth Flow Audit)
- Legal/license question -> deploy Avocat (License Audit or TOS Review)
- Need market context -> deploy Grok (Competitor Deep-Dive or Market Analysis)
- Planning a feature -> deploy Marketer (GTM Plan or Marketing Strategy)
- Need executive clarity -> deploy Boss (Executive Brief or ROI Analysis)
- Documentation gap -> deploy Librarian (Doc Index or Knowledge Gap Scan)
- Business opportunity -> deploy Opportunist (Partnership Scan)

## How to Use

1. Identify your problem type from the table below
2. Call the ghost task API with the matching mission
3. Read the report from the ghost's brain/reports/ folder

```bash
curl -s -X POST http://localhost:3330/agents/GHOST_NAME/ghost/task \
  -H "Content-Type: application/json" \
  -d '{"task":"MISSION_DESCRIPTION","from":"YOUR_AGENT_NAME","model":"auto"}'
```

## Decision Table

| Problem | Ghost | Mission | Keywords in task |
|---------|-------|---------|-----------------|
| Bug won't die | Analyst | Bug Investigation | "bug", "crash", "broken", "trace root cause" |
| Errors recurring | Analyst | Error Log Analysis | "error pattern", "recurring 500", "unhandled" |
| Slow performance | Analyst | Performance Profiling | "slow", "latency", "bottleneck" |
| Architecture question | Analyst | Architecture Review | "module boundary", "coupling", "refactor" |
| KIs keep coming back | Analyst | KI Deep Audit | "KI", "known issue", "plaster", "recurring" |
| Test gaps | Analyst | Test Coverage Map | "untested", "coverage", "which tests" |
| Security risk | Sentinel | Vulnerability Scan | "OWASP", "injection", "XSS", "auth bypass" |
| Dependency risk | Sentinel | Dependency Audit | "CVE", "outdated", "npm audit" |
| Auth confusion | Sentinel | Auth Flow Audit | "OAuth", "session", "token", "permission" |
| License question | Avocat | License Audit | "license", "MIT", "GPL", "open-source" |
| Privacy concern | Avocat | Privacy Check | "GDPR", "PII", "cookies", "data retention" |
| Competitor intel | Grok | Competitor Deep-Dive | "Cursor", "Copilot", "competitor", "market share" |
| Tech research | Grok | Tech Scouting | "new framework", "alternative", "evaluate" |
| Feature planning | Marketer | GTM Plan | "launch", "go-to-market", "positioning" |
| Content needed | Marketer | Content Calendar | "blog", "video", "social media", "content" |
| ROI question | Boss | ROI Analysis | "cost", "ROI", "worth it", "budget" |
| Missing docs | Librarian | Knowledge Gap Scan | "undocumented", "where is", "no docs" |
| System overview | Librarian | System State Snapshot | "what's running", "fleet status", "full picture" |
| Deep debugging | Medic | Root Cause Unblock | "stuck", "impossible bug", "deadlock", "unblock me" |
| Code patterns | Compass | Standards Audit | "convention", "pattern", "inconsistent", "best practice" |
| Dependency issue | Courier | Package Intel | "npm", "dependency", "version conflict", "upgrade path" |
| CI/CD broken | Fuse | Pipeline Fix | "CI", "build failed", "merge conflict", "deploy broken" |
| Eval/test design | Scientist | Eval Design | "promptfoo", "eval", "test design", "assertion", "grading" |
| Codebase health | Scribe | Health Report | "tech debt", "code quality", "dead code", "health check" |
| UI/UX design | Designer | Design Audit | "design", "UX", "layout", "visual", "wireframe" |
| Revenue angle | Opportunist | Opportunity Scan | "monetize", "partnership", "pricing", "revenue" |

## Example

Worker is stuck on a recurring MCP disconnect:

```bash
curl -s -X POST http://localhost:3330/agents/analyst/ghost/task \
  -H "Content-Type: application/json" \
  -d '{"task":"Bug Investigation: MCP connection drops every 24 hours. Trace the token lifecycle, check expiry handling, identify why reconnection fails. Focus on mcp-oauth.js and mcp-server.js.","from":"forge"}'
```

Report saves to `workers/analyst/brain/reports/` -- read it for the diagnosis.

## Cost

Ghost models = $0.00 per report. Do NOT pass model param — pickModel() auto-routes: HUGE→gemini, LARGE→leanstral, MEDIUM→cerebras, SMALL→cerebras.
