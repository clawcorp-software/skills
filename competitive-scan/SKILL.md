---
name: competitive-scan
description: "Use for competitive intelligence scanning. Searches the web for competitor updates, pricing changes, new features, and market positioning relevant to ClawCorp."
user_invocable: true
kind: ours
---

# Competitive Scan

You are performing a competitive intelligence scan for ClawCorp.

## Competitors to Track

1. **Devin** (devin.ai) - autonomous AI software engineer
2. **Cursor** (cursor.com) - AI-powered IDE
3. **Claude Code native** (Anthropic) - Agent SDK, Teams, built-in features
4. **CrewAI** - open source multi-agent framework
5. **LangGraph** - stateful agent orchestration (LangChain)
6. **AutoGen** - Microsoft multi-agent conversations
7. **GitHub Copilot Workspace** - GitHub's agent platform

## Process

1. **WebSearch** for each competitor:
   - "[competitor] pricing 2026"
   - "[competitor] new features latest update"
   - "[competitor] vs alternatives comparison"

2. **Create the report** at `brain/COMPETITIVE-INTEL-<YYYY-MM>.md`:

```markdown
# Competitive Intel - <Month Year>
Date: <today>
Author: <agent name>

## Executive Summary
<3-5 bullet points of the most important changes this month>

## Competitor Updates

### <Competitor Name>
- **Pricing:** $X/month (changed from $Y?)
- **Latest features:** ...
- **Their strength:** 1 thing they do better than us
- **Their weakness:** 1 thing we do better
- **Opportunity:** what we could learn/steal/counter

(repeat for each competitor)

## Market Trends
<What direction is the market moving?>

## Impact on ClawCorp
<What should we change/add/prioritize based on this intel?>

## Sources
1. [title](url)
(minimum 5 sources)
```

3. **Present findings** to the user with top 3 actionable insights
