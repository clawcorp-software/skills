# ClawCorp Skills

Public skills authored by [ClawCorp](https://clawcorp.io) for Claude Code, Codex, Cursor, Gemini CLI, and other agents.

These skills are battle-tested inside ClawCorp's multi-agent orchestration platform across 200+ known-issue cycles. They are released here so other teams running agents can adopt the same enforcement / coordination / quality protocols.

## Install

Use the [`skills`](https://skills.sh) CLI from Vercel Labs:

```bash
# Install a single skill globally
npx skills add clawcorp-software/skills@bullshit -g

# Install all ClawCorp skills
npx skills add clawcorp-software/skills -g
```

Or browse on [skills.sh](https://skills.sh/clawcorp-software/skills).

## Skills

Full catalog mirrors the ClawCorp dashboard view at `/app/tools.html#view=skills`. **ours** = ClawCorp-authored originals, **adapted** = forked + modified from upstream (credits inside each SKILL.md).

| Skill | Description | Origin |
|-------|-------------|--------|
| [`autoresearch`](./autoresearch/SKILL.md) | Karpathy-style autonomous experiment loops inside a bounded subsystem. | adapted |
| [`bullshit`](./bullshit/SKILL.md) | Detect false completions and plaster fixes. Force a real repair before claiming done. | ours |
| [`calling-research-agents`](./calling-research-agents/SKILL.md) | Delegate research, legal, market intel or audit to a research sub-agent. | ours |
| [`competitive-scan`](./competitive-scan/SKILL.md) | Scan the web for competitor updates, pricing changes, new features. | ours |
| [`decision-brief`](./decision-brief/SKILL.md) | Research-backed proposal (3+ sources, options compared) before adopting any new tool. | ours |
| [`deep-research-report`](./deep-research-report/SKILL.md) | Multi-agent deep research that rivals GPT Deep Research, free via local LLMs. | ours |
| [`design-md`](./design-md/SKILL.md) | Load DESIGN.md on UI/UX tasks; keeps design context consistent across sessions. | ours |
| [`extension-ownership`](./extension-ownership/SKILL.md) | Ownership rules for VS Code extensions: API contract, deploy step, regression testing. | ours |
| [`eye`](./eye/SKILL.md) | Visual QA + browser-debug workflow. Playwright MCP for repro, DevTools for diagnosis. | ours |
| [`fleet-status`](./fleet-status/SKILL.md) | Complete fleet overview: agent compliance scores, pending POKEs, system health. | ours |
| [`frontend-preflight`](./frontend-preflight/SKILL.md) | Multi-point checklist before any PR touching frontend (toolbar, flex, scope, mobile). | ours |
| [`full-design`](./full-design/SKILL.md) | Run every design skill in sequence: critique -> adapt -> animate -> colorize -> delight -> polish. | ours |
| [`full-power`](./full-power/SKILL.md) | Maximum-quality orchestrator: select and execute the right combination of skills for a task. | ours |
| [`ghost-sos`](./ghost-sos/SKILL.md) | Deploy a specialist sub-agent for bug triage, design help, or architecture advice. | ours |
| [`git-commit-push-pr`](./git-commit-push-pr/SKILL.md) | One-shot git workflow: stage, commit, push, create-pr with the agent's identity. | adapted |
| [`google-tools`](./google-tools/SKILL.md) | Operational protocol for Gemini classifier, BigQuery commit patterns, PageSpeed, Cloud NLP. | ours |
| [`ingest-video`](./ingest-video/SKILL.md) | Pull a YouTube transcript into your DB for later agent research. | ours |
| [`kanban-sync`](./kanban-sync/SKILL.md) | Auto-sync task status + cascade context between agents and the kanban board. | ours |
| [`ki-conventions`](./ki-conventions/SKILL.md) | How to file KIs vs incidents vs hashtags, severity scoring, escalation. | ours |
| [`ki-external-scan`](./ki-external-scan/SKILL.md) | Search external sources (vendor changelogs, status pages) for known-issue references. | ours |
| [`ki-systematic-resolution`](./ki-systematic-resolution/SKILL.md) | 10-phase known-issue protocol: cumulative fixes, real evidence, no plasters. | ours |
| [`last30days-research`](./last30days-research/SKILL.md) | Reddit/X/HN/YouTube research with usage tracking via ScrapeCreators. | adapted |
| [`multi-ghost-sos`](./multi-ghost-sos/SKILL.md) | Deploy 2-4 specialist sub-agents in parallel for complex multi-angle problems. | ours |
| [`multiagent-autoresearch`](./multiagent-autoresearch/SKILL.md) | Coordinate autoresearch loops across multiple workers with shared scoring. | ours |
| [`proof-or-punish`](./proof-or-punish/SKILL.md) | Auto-reward clean deliveries, auto-penalize bullshit fixes (signal layer). | ours |
| [`propagation-check`](./propagation-check/SKILL.md) | Mandatory post-change verification after editing rules, config, normalize, or policy. | ours |
| [`recall`](./recall/SKILL.md) | Rebuild context from a past or crashed session using transcripts. | ours |
| [`recall-teammate`](./recall-teammate/SKILL.md) | Read another agent's session backwards to avoid conflicting work. | ours |
| [`research-gate`](./research-gate/SKILL.md) | Mandatory existence/blast-radius check before touching architecture, schema, or routing. | ours |
| [`risk-assessment`](./risk-assessment/SKILL.md) | Proactive blast-radius + KI-history scoring before touching the spine. | ours |
| [`save-and-sleep`](./save-and-sleep/SKILL.md) | End-of-session protocol: write handoff, memory index, clean artifacts. | ours |
| [`systematic-debugging`](./systematic-debugging/SKILL.md) | Hypothesis isolation + evidence collection for recurring or contradictory bugs. | ours |
| [`tribunal`](./tribunal/SKILL.md) | Critical-incident enforcement court: gather dossier, deliver verdict, apply consequences. | ours |
| [`verification-before-completion`](./verification-before-completion/SKILL.md) | Run verification commands and confirm output before claiming done or opening a PR. | ours |


## Why these exist

Most agent failure modes look the same in production:

1. The agent says "done" but didn't actually do it (or fixed the wrong element).
2. The same bug reopens 3, 5, 7 times -- each fix a "plaster" instead of a real cause.
3. Token cost spirals because nobody verifies before claiming success.

These skills enforce the discipline that keeps agents honest: real verification, real root cause, real cumulative fix history. They are the protocols ClawCorp's own fleet runs on.

## Roadmap

More ClawCorp skills coming soon:

- `ghost-sos` -- delegate to specialist sub-agents (legal, research, security, etc.)
- `recall-teammate` -- read another agent's session backwards without context bloat
- `verification-before-completion` -- mandatory pre-PR checklist gate

## License

MIT. See [LICENSE](./LICENSE).

## Contributing

ClawCorp authors these for our own fleet first; PRs that add real-world battle-tested protocols are welcome. Vague style guides without recurrence data are not.
