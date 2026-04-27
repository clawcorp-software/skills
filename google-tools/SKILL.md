---
name: google-tools
description: "Operational protocol for Google tools in ClawCorp. Covers Gemini classifier, BigQuery commit patterns, PageSpeed, Cloud NLP, readiness checks, bootstrap, sync, query, and failure handling."
kind: ours
---

# /google-tools -- Google Power Protocol

Use this skill whenever work depends on Google-backed capabilities:
- Gemini classifier or any `google-nl` enrichment
- BigQuery commit history, KI pattern audits, or billing exports
- PageSpeed audits
- Cloud NLP sentiment/entities

This is not optional documentation. It is the operational protocol.

Project:
- `steam-crowbar-414914`

Core runtime surfaces:
- helper: `server/helpers/google-cloud-apis.js`
- classifier: `server/helpers/google-nl.js`
- routes: `server/routes/google.js`

## What Must Always Work

### 1. Gemini classifier

Used for:
- task classification
- generic text classification
- KI/fix classification
- ghost/context enrichment

Primary runtime:
- `llm-provider` with Google/Gemini availability

Quick checks:
```bash
curl -s http://localhost:3330/api/google/status
node -e "var g=require('./server/helpers/google-nl'); console.log(JSON.stringify(g.getUsageStats(),null,2))"
```

Expected:
- `apiKey: true` or Gemini runtime available through ADC/provider
- `runtime_available: true`

### 2. BigQuery commit patterns

Used for:
- KI commit correlation
- fix/revert pattern analysis
- ghost context enrichment for KI tasks
- fix-audit commit completeness

Required assets:
- `bq` CLI callable by the server process
- dataset for commits
- table `commits`

Operational endpoints:
```bash
curl -s http://localhost:3330/api/google/status
curl -s -X POST http://localhost:3330/api/google/commits/sync -H "Content-Type: application/json" -d "{}"
curl -s "http://localhost:3330/api/google/commits/query?ki=KI-020&limit=20"
```

### 3. PageSpeed

```bash
curl -s "http://localhost:3330/api/google/pagespeed?url=https://clawcorp.io&strategy=mobile"
```

### 4. Cloud NLP

```bash
curl -s -X POST http://localhost:3330/api/google/analyze \
  -H "Content-Type: application/json" \
  -d "{\"text\":\"Toolbar overlaps mobile content on bugs.html\"}"
```

## Mandatory Readiness Protocol

Before claiming any Google-powered workflow is available:

1. Check status:
```bash
curl -s http://localhost:3330/api/google/status
```

2. Check classifier runtime:
```bash
node -e "var g=require('./server/helpers/google-nl'); console.log(g.canCall() ? 'classifier-ready' : 'classifier-not-ready')"
```

3. Check BigQuery CLI path:
```bash
where.exe bq
```

4. Check commit dataset/table:
```bash
curl -s "http://localhost:3330/api/google/commits/query?ki=KI-020&limit=1"
```

If any step fails, write the exact failing layer:
- `google-key-missing`
- `gemini-runtime-unavailable`
- `bq-cli-unavailable`
- `bq-dataset-missing`
- `bq-table-missing`

Never say "Google tools available" without this proof.

## BigQuery Bootstrap Protocol

If BigQuery is expected but broken:

1. Verify CLI:
```bash
where.exe bq
bq version
```

2. Verify authenticated project:
```bash
bq ls --format=json
```

3. Verify/create commit dataset:
```bash
bq mk --dataset steam-crowbar-414914:clawcorp_patterns
```

4. Sync commits from git into BigQuery:
```bash
curl -s -X POST http://localhost:3330/api/google/commits/sync \
  -H "Content-Type: application/json" \
  -d "{\"rootDir\":\"$CLAWCORP_ROOT\"}"
```

5. Query proof:
```bash
curl -s "http://localhost:3330/api/google/commits/query?ki=KI-020&limit=5"
```

PASS criteria:
- sync returns `201`
- query returns rows, not a bootstrap/config error

## Frontend KI Protocol

For recurrent frontend/systemic bugs:

1. Query BigQuery commit history for the KI
2. Query fix-audit timeline for the KI
3. Run classifier on the current fix/resolution text
4. Classify subsystem, regression family, severity, frontend/backend scope, and protocol breach risk
5. Record whether BigQuery and classifier were available
6. If unavailable, document the exact Google failure state in the KI

Minimum sequence:
```bash
curl -s "http://localhost:3330/api/google/commits/query?ki=KI-020&limit=20"
curl -s "http://localhost:3330/cc/known-issues/KI_ID/fix-audit"
```

## Cost / Reality

Do not hardcode fantasy numbers in docs. Treat these as operationally variable:
- Gemini classifier: depends on provider/runtime path
- BigQuery commits: depends on dataset existence and query volume
- PageSpeed: free quota driven

If cost is unknown, say `cost not verified`.

## Canonical Google Stack

The only Google capabilities that are considered systemic and non-optional in ClawCorp are:
- Gemini classifier
- BigQuery

Support tooling:
- Cloud NLP
- PageSpeed

Not part of the canonical stack:
- Custom Search

Do not describe the Google stack as "ready" because of search/web capabilities. The real readiness gate is:
- classifier callable
- BigQuery sync works
- BigQuery query works
- KI/fix workflows can classify and correlate evidence

## Powerful Usage Standard

BigQuery and classifier must be used together, not as isolated gadgets.

Minimum expected uses across ClawCorp:
- classify KI fixes, regressions, subsystems, and severity using Gemini
- correlate KI IDs to commits, fix density, revert density, and authors via BigQuery
- enrich fix-audit and systemic-resolution workflows with both classifier output and commit evidence
- flag undocumented fixes, duplicate fixes, and fixes with missing commit linkage
- support frontend systemic bug reviews with classification plus historical commit evidence
- support cost/completeness audits on fix history and protocol compliance

## Failure Handling

### If `status` says BigQuery available but queries fail
Cause:
- broken CLI fallback
- wrong dataset location
- dataset missing

Action:
- fix runtime/helper first
- then re-run sync/query proof

### If classifier is configured but `canCall()` is false
Cause:
- provider unavailable
- daily cap reached
- Gemini runtime not actually ready

Action:
- inspect `google-nl.getUsageStats()`
- inspect `llm-provider` Google availability

### If root repo has a skill but worksite does not
Action:
- treat root `.claude/skills` as shared source of truth
- propagate or expose it to the worksite runtime

## Definition of Done

Google tools are only "done" when:
- `api/google/status` is truthful
- BigQuery CLI fallback works from the server process
- BigQuery sync and query both work
- Gemini classifier is callable
- docs/skills mention BigQuery + classifier explicitly
- atlas diagram reflects the real Google stack
