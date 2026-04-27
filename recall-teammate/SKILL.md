---
name: recall-teammate
description: "Read another agent's live session backwards from the latest message. Works for BOTH Claude Code (~/.claude/projects) AND Codex CLI (~/.codex/sessions). Identifies session owner via cwd. Use when you need to know what a teammate is currently doing, what they just decided, or to avoid conflicting work."
tags: [coordination, memory, recall, multi-agent, jsonl, claude, codex]
kind: ours
---

# recall-teammate

> Read a teammate's live session from the end backwards. Works for Claude Code AND Codex. Token-cheap, freshness-first, identity-aware.

## When to use

- Before starting work that might conflict with another agent
- After a long gap, to catch up on what a teammate decided
- When the user asks "what is X doing?" or "go read Y's chat"
- To verify a memory or POKE claim against the live transcript

**Do NOT use** for compacted-out content from your OWN past sessions — that's `/recall`.

## Two runtimes — same procedure

A teammate may be working in **Claude Code** OR **Codex CLI**. Always check BOTH before reporting.

### Claude Code sessions
- Path: `C:/Users/user/.claude/projects/<encoded-cwd>/<uuid>.jsonl`
- Encoding: `C:/Users/user/clawcorp/tower/` → `C--Users-user-clawcorp-tower`
- Identity: derived from the encoded directory name (cwd = which worker)
- Format: `{type:"user"|"assistant", timestamp, message:{content:[...]}}`

### Codex CLI sessions
- Path: `C:/Users/user/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl`
- Identity: read FIRST line → `payload.cwd` field tells you who owns it
- Format: `{type:"response_item"|"session_meta", timestamp, payload:{...}}`
  - Messages: `payload.type=="message"`, `payload.role`, `payload.content[].text`
  - Tools: `payload.type=="function_call"`, `payload.name`, `payload.arguments`

## Identity map (cwd → owner — but read the warning)

| cwd shape | Likely owner |
|-----------|--------------|
| `clawcorp/tower` or `clawcorp\` root (no `/workers/`) | **Tour** |
| `clawcorp/workers/<name>/worksites/clawcorp` | **\<name\>** (Manager, Forge, Pixel, Architecte, ...) |
| `clawcorp/workers/<name>/worksites/clawcorp-ephemeral/<ts>` | **ghost worktree** — see warning |

**WARNING — ghost / ephemeral worktrees lie about identity.**

A `clawcorp-ephemeral/` cwd shows the worker whose folder it lives under, but the **driver may be a different agent**. Architecte routinely orchestrates work in `workers/boss/worksites/clawcorp-ephemeral/...` (Boss is the testbed; Architecte is the driver). Do not equate `cwd contains workers/boss` with "Boss is doing this".

To resolve identity in an ephemeral session, check the transcript itself for:
- **Wake line** — `[WAKE:<name>]` is the agent's self-declaration
- **POKE source** — `From: <name>` lines reveal who is sending orders to whom
- **Branch worked on** — `git branch --show-current` calls + the eventual PR `headRefName`
- **Commit author / PR author** — match against agent map (e.g. PR #1409 `headRefName=forge` → Forge owns the work even if no Codex session under `workers/forge/` is active)

When in doubt, report `cwd=X, wake=Y, branch=Z` and let the Keymaster disambiguate.

## Procedure

### Step 1 — Find both runtimes

```bash
# Claude Code latest for teammate <X>:
ls -t "C:/Users/user/.claude/projects/<encoded-cwd>/" | head -1

# Codex sessions today:
ls -t "C:/Users/user/.codex/sessions/$(date +%Y/%m/%d)/" 2>/dev/null

# Identify owner of EACH Codex file:
for f in C:/Users/user/.codex/sessions/$(date +%Y/%m/%d)/*.jsonl; do
  echo "=== $(basename $f) ==="
  head -1 "$f" | python -c "import json,sys; d=json.loads(sys.stdin.read())['payload']; print('cwd:',d.get('cwd',''))"
done
```

Pick the file whose `cwd` matches the teammate per the identity map. ALWAYS check Codex too — a worker may have abandoned Claude and booted in Codex (different runtime, same identity).

### Step 2 — Read backwards from the end

**Timezone — CRITICAL:** Both Claude and Codex jsonl timestamps are **UTC** (ISO 8601 with `Z`). File mtimes (`ls -lt`) are **LOCAL**. Always convert UTC → local before reporting times to the user, or you will be 4-5h off. Both parsers below do the conversion.

**Claude Code parser:**
```bash
tail -200 "<jsonl-path>" | python -c "
import json, sys
from datetime import datetime, timezone
def lt(ts):
    try: return datetime.fromisoformat(ts.replace('Z','+00:00')).astimezone().strftime('%H:%M:%S')
    except: return ts[:19]
for line in sys.stdin:
    try:
        m = json.loads(line)
        ts = lt(m.get('timestamp',''))
        t = m.get('type','')
        msg = m.get('message',{})
        if t == 'user':
            c = msg.get('content','')
            if isinstance(c,list):
                for p in c:
                    if p.get('type')=='text': print(f'[{ts}] USER: {p[\"text\"][:300]}')
            elif isinstance(c,str): print(f'[{ts}] USER: {c[:300]}')
        elif t == 'assistant':
            for p in msg.get('content',[]):
                if p.get('type')=='text': print(f'[{ts}] AGENT: {p[\"text\"][:400]}')
                elif p.get('type')=='tool_use': print(f'[{ts}] TOOL({p.get(\"name\",\"\")}): {str(p.get(\"input\",{}))[:200]}')
    except: pass
"
```

**Codex parser:**
```bash
tail -200 "<jsonl-path>" | python -c "
import json, sys
from datetime import datetime, timezone
def lt(ts):
    try: return datetime.fromisoformat(ts.replace('Z','+00:00')).astimezone().strftime('%H:%M:%S')
    except: return ts[:19]
for line in sys.stdin:
    try:
        m = json.loads(line)
        ts = lt(m.get('timestamp',''))
        p = m.get('payload',{})
        if p.get('type') == 'message':
            role = p.get('role','')
            for c in p.get('content',[]):
                txt = c.get('text','') if isinstance(c,dict) else ''
                if txt.strip(): print(f'[{ts}] {role.upper()}: {txt[:300]}')
        elif p.get('type') == 'function_call':
            print(f'[{ts}] TOOL({p.get(\"name\",\"\")}): {p.get(\"arguments\",\"\")[:200]}')
    except: pass
"
```

### Step 3 — Stop early when you have the answer

Read the LAST event first, then walk backwards. Stop as soon as you have enough context — do NOT read the whole file. If `N=200` lines isn't enough, double to `N=500`, then `N=1000`. Never read from line 1.

### Step 4 — Report tightly with identity

Lead with **WHO** and **WHICH RUNTIME**, then what they're doing right now, what they just shipped, and any file/area overlap. End with:
`CONFLICT RISK: [low/med/high] — <reason>`

Example header:
```
**Forge** (Codex CLI, session 019da5f5, started 09:37 today)
**Tour** (Claude Code, session fa1ee329, last active 17:07)
```

If you find a teammate in BOTH runtimes, report both — they may have switched mid-task or be running parallel.

## Optional user instructions (constrain depth at invoke time)

- `/recall-teammate tour --since 30min` → only events newer than 30 minutes
- `/recall-teammate tour --until "POKE"` → walk back until you find the last POKE event
- `/recall-teammate tour --topic "v1.1.11"` → keep walking back until topic appears
- `/recall-teammate tour --depth shallow|medium|deep` → 200 / 500 / 1000 tail lines
- `/recall-teammate forge --runtime codex` → skip Claude lookup, go straight to Codex
- `/recall-teammate forge --runtime both` (default) → check both runtimes

If no constraint given, default = last ~30 events (≈200 jsonl lines), check BOTH runtimes.

## Cost discipline

- Reading a full 8 MB jsonl = 50K+ tokens. Tail = 1-3K tokens.
- Always prefer `tail` + Python parsing over `Read` on the raw jsonl.
- Delegate to an Explore subagent if the file is huge (>10 MB) AND you need deep search; otherwise do it yourself.

## Anti-patterns

- ❌ Reading only Claude when teammate switched to Codex (or vice versa) — always check both
- ❌ Reading the entire jsonl — use `tail`
- ❌ Trusting POKE/memory over the live transcript when the user asks "what are they doing NOW"
- ❌ Reporting a session without naming WHO owns it (cwd-based identity)
- ❌ Spawning a subagent for a 30-event tail read — do it inline

## Example invocations

```
/recall-teammate tour
  → check Claude (~/.claude/projects/C--Users-user-clawcorp-tower/) AND Codex (cwd=...\clawcorp)
  → report what Tour is doing right now in whichever runtime is active

/recall-teammate forge --runtime codex
  → check ~/.codex/sessions/today/ for cwd=...\workers\forge\worksites\clawcorp

/recall-teammate boss --since 1h
  → both runtimes, last hour, including ephemeral worktrees (cwd contains workers/boss)
```
