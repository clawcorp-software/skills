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

| Skill | Description |
|-------|-------------|
| [`bullshit`](./bullshit/SKILL.md) | Detect false completions, force the REAL fix, escalate to systematic resolution. Replaces XP-only enforcement that produced theater instead of working code. |
| [`ki-systematic-resolution`](./ki-systematic-resolution/SKILL.md) | 10-phase protocol to repair known-issue records cumulatively. Stops thin-KI drift, fake closures, and orphaned commits. Use on the 3rd+ incident under a parent. |

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
