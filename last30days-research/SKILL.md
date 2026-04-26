---
name: last30days-research
description: "Run /last30days research through ClawCorp guardrails. Tracks ScrapeCreators API usage, enforces PYTHONUTF8 for Windows, saves reports to brain/reports/."
---

# /last30days Research (ClawCorp Integration)

External research across Reddit, X, YouTube, HN, Polymarket, TikTok, Instagram.
Wraps the global /last30days skill with ClawCorp guardrails and usage tracking.

## When to Use

- Before proposing a new tool/framework (complements /decision-brief)
- Competitive intelligence on a topic
- Understanding community sentiment on a feature or technology
- Any research task that needs current (last 30 days) community data

## Dispatch

Per model-dispatch.md:
- **Execution** (run script, fetch data): Haiku subagent
- **Synthesis** (analyze results, write report): Opus

## Steps

### Step 1: Run with guardrail wrapper

```bash
PYTHONUTF8=1 bash scripts/last30days-guard.sh "TOPIC" --emit=compact --quick --no-native-web
```

Flags:
- `--quick` for fast scan (~5s, fewer sources)
- `--deep` for comprehensive (~60s, more sources)
- `--no-native-web` skip web search (use WebSearch tool instead)

### Step 2: Supplement with WebSearch

After script completes, use WebSearch for blogs/tutorials/news.

### Step 3: Synthesize

Combine script output + WebSearch into actionable insights.
Weight Reddit/X higher (engagement signals) over web articles.

### Step 4: Save report (if substantial)

Save to worker brain: `brain/reports/YYYY-MM-DD-last30days-TOPIC.md`

## Available Sources

| Source | Status | Cost |
|--------|--------|------|
| Reddit (threads + comments) | Active | ScrapeCreators (100 free, then $0.002/call) |
| HN | Active | Free |
| Polymarket | Active | Free |
| X/Twitter | Needs setup | Free (browser cookies) |
| YouTube | Needs PATH fix | Free (yt-dlp via pip) |
| TikTok / Instagram | Active | ScrapeCreators (shared quota) |

## Usage Tracking

Counter: `~/.config/last30days/usage-counter.json`
Guard script shows remaining free calls before each run.

## Windows Notes

- `python` not `python3`
- `PYTHONUTF8=1` required (cp1252 emoji crash)
- yt-dlp: `python -m yt_dlp` (not on PATH)

## Future (Round 2)

- API route: `POST /api/cc/research/run`
- DB tables with user_id isolation (after Backbone multi-tenancy)
- Dashboard UI in atlas
