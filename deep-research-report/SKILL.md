---
name: deep-research-report
description: "Use when you need a comprehensive research report on any topic -- deploys multiple ghosts with overlapping research angles, then synthesizes findings into one mega report. Comparable to GPT deep research but free via leanstral."
---

# Deep Research Report -- Multi-Source Mega Analysis

Deploy 3-5 ghosts in parallel on the same topic from different angles, then synthesize all reports into one comprehensive document. Free via leanstral.

## When to Use

- Need a comprehensive analysis before a big decision
- Evaluating a technology, market, or strategy
- Preparing a presentation or pitch
- Need to understand a complex topic from all angles
- Want the equivalent of a GPT deep research report for $0

## Process

```
1. DEPLOY ghosts (parallel, 30-60 seconds)
2. COLLECT reports (all in brain/reports/)
3. SYNTHESIZE into one mega document
```

## Step 1: Deploy Research Fleet

Pick your topic and deploy 3-5 ghosts. Each brings a different lens:

| Ghost | Lens | Best For |
|-------|------|----------|
| Grok | Technical depth | Tech comparisons, academic research, innovation |
| Analyst | Data & patterns | Metrics, benchmarks, code analysis |
| Boss | Strategic & financial | ROI, budgets, business case |
| Marketer | Market & positioning | GTM, competitors, audience |
| Opportunist | Opportunities & angles | Partnerships, monetization, trends |
| Sentinel | Risk & security | Threat assessment, compliance |
| Avocat | Legal & regulatory | Licenses, contracts, IP |
| Librarian | Historical & documentation | What exists, what's missing, cross-reference |

### Example: Research "Should ClawCorp add vector database for semantic search?"

```bash
TOPIC="Adding a vector database (Qdrant/ChromaDB/Pinecone) for semantic search across ghost reports and agent memory"

# Grok: technical deep-dive
curl -s -X POST http://localhost:3330/agents/grok/ghost/task -H "Content-Type: application/json" \
  -d "{\"task\":\"Deep Research: $TOPIC. Compare Qdrant vs ChromaDB vs Pinecone vs pgvector. Self-hosted vs managed. Performance benchmarks, scaling, Node.js SDK quality. Minimum 3 sources cited.\",\"from\":\"deep-research\"\"}" &

# Boss: ROI and business case
curl -s -X POST http://localhost:3330/agents/boss/ghost/task -H "Content-Type: application/json" \
  -d "{\"task\":\"ROI Analysis: $TOPIC. Cost of implementation (dev time, infra, maintenance) vs value (better search, faster debugging, knowledge retention). Break-even analysis. Compare to SQLite FTS5 alternative.\",\"from\":\"deep-research\"\"}" &

# Analyst: codebase impact
curl -s -X POST http://localhost:3330/agents/analyst/ghost/task -H "Content-Type: application/json" \
  -d "{\"task\":\"Architecture Review: How would $TOPIC integrate with our current stack? Impact on server/index.js, new routes needed, data pipeline for indexing brain/reports/. Complexity assessment.\",\"from\":\"deep-research\"\"}" &

# Sentinel: security implications
curl -s -X POST http://localhost:3330/agents/sentinel/ghost/task -H "Content-Type: application/json" \
  -d "{\"task\":\"Security Assessment: $TOPIC. Attack surface of vector DB (data exfiltration via similarity search, prompt injection via embeddings, API exposure). Dependency supply chain risk.\",\"from\":\"deep-research\"\"}" &

# Opportunist: market angle
curl -s -X POST http://localhost:3330/agents/opportunist/ghost/task -H "Content-Type: application/json" \
  -d "{\"task\":\"Market Opportunity: $TOPIC. How are competitors using vector search? Is semantic search a differentiator or table stakes? Partnership angles with vector DB providers.\",\"from\":\"deep-research\"\"}" &

wait
echo "5 reports generated. Check workers/*/brain/reports/"
```

## Step 2: Collect Reports

```bash
# List all reports from this research session
for ghost in grok boss analyst sentinel opportunist; do
  echo "=== $ghost ==="
  ls -t workers/$ghost/brain/reports/ | head -1
done
```

## Step 3: Synthesize

Read all reports and produce a unified document:

```bash
# Create synthesis prompt with all report content
SYNTHESIS=""
for ghost in grok boss analyst sentinel opportunist; do
  LATEST=$(ls -t workers/$ghost/brain/reports/ | head -1)
  if [ -f "workers/$ghost/brain/reports/$LATEST" ]; then
    SYNTHESIS="$SYNTHESIS\n\n## Report from $ghost\n$(cat workers/$ghost/brain/reports/$LATEST)"
  fi
done

# Deploy Boss to synthesize everything
curl -s -X POST http://localhost:3330/agents/boss/ghost/task \
  -H "Content-Type: application/json" \
  -d "{\"task\":\"SYNTHESIS REPORT: Combine these 5 research reports into ONE comprehensive analysis. Structure: Executive Summary, Key Findings (per domain), Risk Assessment, Recommendation, Action Items. Reports:\\n$SYNTHESIS\",\"from\":\"deep-research-synthesis\"\"}"
```

## Quick Templates

### Technology Evaluation
Ghosts: Grok + Analyst + Boss + Sentinel
Focus: technical comparison, codebase impact, ROI, security

### Market Analysis
Ghosts: Grok + Marketer + Opportunist + Boss
Focus: market size, positioning, opportunities, financials

### Pre-Decision Brief
Ghosts: Grok + Boss + Avocat + Sentinel
Focus: research, ROI, legal, security (feeds into /decision-brief)

### Codebase Health Check
Ghosts: Analyst (3 missions) + Sentinel + Librarian
Focus: code quality, tech debt, test coverage, security, documentation

### Competitive Intelligence
Ghosts: Grok + Marketer + Opportunist
Focus: competitor features, positioning, market gaps

## Cost

- 5 ghosts x leanstral = **$0.00 total**
- Time: ~30-60 seconds for all 5 in parallel
- Comparable output to GPT deep research ($20+) but free and customized to YOUR codebase

## Tips

- Be SPECIFIC in your task descriptions -- generic prompts = generic reports
- Include file names, feature names, competitor names in the task
- The project context injection auto-adds: dependencies, routes, KI data, git history
- Deploy Boss LAST for synthesis -- give other ghosts time to finish
- Save the synthesis report in YOUR brain/reports/ not the ghost's
