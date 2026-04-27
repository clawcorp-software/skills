# /safe-cmd -- Pre-output checkpoint: exhaust internal first, then wrap paste-safe

**MANDATORY BEFORE EMITTING ANY USER-FACING COMMAND.**

This skill is a checkpoint, not a tool the human invokes. It runs in the agent's head, every time, BEFORE the agent posts a command instruction to the user.

## Why this exists

KI `issue_1777238903982_z6knt` documents the agent-output -> human-clipboard -> external-shell fidelity failure (8+ historical plasters, vendor unfixable per anthropics/claude-code #50012). The Keymaster directive 2026-04-26: "tu fais tout" -- the agent must EXHAUST internal execution paths before asking the human to paste anything.

This skill forces the audit.

## The 4 questions (in order)

Before emitting any command-bearing message to the user, the agent MUST answer these to itself, honestly. If the answer to any of Q1-Q3 is "yes I can do it internally", the agent does that instead and SKIPS the user-facing command.

### Q1: Can I run this myself via the Bash tool?
- Is the command available in my environment (CLI installed, auth in place)?
- Is the command non-interactive, non-RDP, non-UI?
- Does the user's prior authorization cover this scope?

If YES to all three -> RUN IT. Do not ask the user to paste.

### Q2: Can I write a hosted .ps1/.sh and emit a one-liner `irm | iex`?
- Is the command longer than 200 chars or multi-line?
- Do we control a hosting surface (clawcorp.io, GitHub raw, S3 we own)?
- Is the script idempotent + safe to expose?

If YES -> stage the file, commit, emit the one-liner. The user pastes ONE token, not 200 lines.

### Q3: Can I parenthesize/wrap to make it paste-safe as a single token?
- Pipeline (`aws ... | ConvertFrom-Json | Select Foo`) -> wrap the whole pipeline in `()` and access the property after: `(...).Foo`
- Multi-statement (`cd X; do Y; do Z`) -> wrap in `. { cd X; do Y; do Z }` (PowerShell) or `bash -c 'cd X && Y && Z'` (bash)
- Heredoc -> reformat to single-line array or `printf` if short, otherwise HOSTED

### Q4: Do I genuinely need human intervention for this?
Only YES if all of:
- Requires UI interaction (RDP login, OAuth click, browser)
- Requires a credential the agent does not hold
- Requires a permission the agent has been denied
- Requires a physical action

If YES -> emit a paste-safe command (Q3-wrapped) WITH a short justification of why you couldn't do it yourself.

If NO and you got here -> reset, go back to Q1.

## Anti-patterns -- refuse to emit these

The agent must NOT emit these to the user:
- `--query <expr> --output text` chained with `;` on PowerShell -- soft-wrap splits `text` to its own line
- Multi-line code fence claiming to be one command (no `;`/`&&` at line ends)
- Backtick-continuation inside a chat code block -- chat client eats backticks
- Bash on Windows with `/dev/...` path arg without `MSYS_NO_PATHCONV=1` prefix
- Heredoc `<<EOF` instructing manual paste -- chat client may eat lines silently
- Anything containing a password, API key, or secret -- refuse, instruct retrieval via Doppler
- Anything modifying `.claude/`, `tower/scripts/`, or `CONSTITUTIONAL.md` -- requires `/propagation-check` first

If a draft response contains any of these, transform it before sending OR refuse and explain.

## Length thresholds

| Length | Action |
|---|---|
| <80 chars, single statement | OK as-is |
| 80-300 chars, single statement | parenthesize so it survives line-wrap |
| 80-300 chars, multi-statement | script-block wrap (`. { ... }` or `bash -c '...'`) |
| >300 chars OR contains heredoc | HOSTED (irm/curl one-liner) |
| Any length, agent has Bash tool + auth + permission | SELF-EXECUTE (do not ask user) |

## Examples (today's KI z6knt incidents)

### Fixed by SELF-EXECUTE (Q1=yes)
Bad: "Run this in your terminal: `aws ec2 terminate-instances ...`"
Good: agent runs `aws ec2 terminate-instances ...` itself via Bash tool, returns the result inline.

### Fixed by PARENTHESIZE (Q3=yes)
Bad: `aws ec2 get-password-data ... --query PasswordData --output text` (185 chars, ends with `--output text` -- prone to soft-wrap split)
Good: `(aws ec2 get-password-data ... | ConvertFrom-Json).PasswordData` -- single atomic token, soft-wrap-safe

### Fixed by HOSTED (Q2=yes)
Bad: 342-line VM setup PowerShell pasted in chat
Good: `irm https://clawcorp.io/scripts/vm-setup.ps1 | iex`

### Genuine human intervention (Q4=yes)
OK: "Open `mstsc /v:54.226.128.23` -- needs your manual RDP login. SSM blocked because the instance has no IAM role and role creation was denied."
The justification clause is mandatory.

## When to invoke

Self-trigger, not user trigger. The agent runs through the 4 questions internally:
- Before any code-fenced command intended for the user to run
- Before posting a "TOI fais X" message
- Before suggesting `mstsc`, `RDP`, `paste this`, `run that`

There is no `/safe-cmd <cmd>` user-facing form. The user does NOT invoke this skill. The agent does, silently, every time.

## Failure modes

If the agent finds itself about to emit a command and CANNOT honestly answer Q1-Q3, but feels the urge to ask the human anyway:
- STOP
- Explain in chat what was tried and what's blocked
- Ask the user explicitly: "I want to ask you to paste X because Y. Is there a way I should retry internally first?"

Never silently fall back to "paste this" without exhausting and disclosing.

## Connected
- KI `issue_1777238903982_z6knt` (Terminal command truncation on paste) -- this skill is the canonical workaround
- `/propagation-check` -- required before edits to skills/rules
- `/ki-systematic-resolution` -- if the agent finds a recurrence of paste truncation, file under z6knt rather than open a new KI
- `/ki-external-scan` -- last resort if the in-house workaround stops working AND `/ki-systematic-resolution` Phase 6 (ghost-sos) is exhausted
- core.md -- "single line, no &&" rule (background)
- workflow.md -- post-message.js usage

## Definition of done

- Skill exists at `~/.claude/skills/safe-cmd/SKILL.md` (Layer 4)
- core.md or workflow.md references the skill (Layer 1)
- Agent invokes the 4-question check before every user-facing command
- Future incidents under KI z6knt are PREVENTED, not just documented
- Follow-up: decision-tree (Layer 2) + promptfoo (Layer 3) + pre-output hook (Layer 5)
