# git-commit-push-pr — One-Shot Git Workflow

Automates the complete git workflow: `git add` → `git commit` → `git push` → GitHub App `createPullRequest` in a single call.

**When to use:** After completing a feature/fix and ready to ship. Replaces the manual 4-step process agents do every session.

**Source:** Grok spec from Claude Code leak analysis (2026-04-03). All components were already present in ClawCorp — this skill just orchestrates them.

## Usage

```
/git-commit-push-pr
  files: [list of files to add, or "all" for git add .]
  message: "fix(kanban): chips at top bar, avatars sidebar"
  pr_title: "fix(kanban): chips at top bar, avatars sidebar vertical"
  pr_body: "optional extended description"
  target: dev  (default: dev, never main without explicit permission)
  task: t_1773330486705_kmahrl  (optional, links commit + PR)
  ki: KI-34  (optional, links KI)
```

## Process

### Step 1: Pre-checks

```bash
# Verify branch (never push directly to main)
git branch --show-current
# Should NOT be main or master

# Check git status
git status --short

# Verify no conflict markers
grep -r "<<<<<<" --include="*.js" --include="*.css" --include="*.html" . 2>/dev/null | head -5
```

If branch is `main` or `master`: **STOP. Ask Keymaster which branch to use.**
If conflict markers found: **STOP. Resolve conflicts first.**

### Step 2: Stage files

```bash
# If files = "all":
git status --short  # Review first, never git add -A blind
git add [specific files listed]

# If files is a list:
git add [file1] [file2] ...

# Verify staged
git diff --cached --stat
```

**Rules from core.md:**
- ALWAYS `git status` before `git add`
- NEVER `git add -A` without review
- Check for .env, credentials, secrets in staged files

### Step 3: Commit

```bash
# Build commit message with task/KI links
COMMIT_MSG="[message]"
# If task provided: append " task:[task_id]"
# If KI provided: append " KI-XX"

git commit -m "$(cat <<'EOF'
[full commit message]

Co-Authored-By: Claude Sonnet 4.6 (1M context) <noreply@anthropic.com>
EOF
)"
```

### Step 4: Push

```bash
git push origin [current-branch]
```

Handle pre-push hook output:
- `[pre-push] ✅ Branch 'X' is valid.` → continue
- `[BLOCKED] Branch behind dev` → push anyway (squash merge handles divergence). If push rejected, inform Keymaster. NEVER `git pull origin dev` (KI-12).
- `[WARN] PR already merged` → inform Keymaster, they decide

### Step 5: Create PR via GitHub App

```bash
node $CLAWCORP_ROOT/tower/scripts/coordination/create-pr.js \
  "[agent-name]" \
  "[pr_title]" \
  "[pr_body]" \
  "[branch-name]" \
  "dev"
```

**Workers: NEVER use `gh pr create`** (posts under Keymaster identity). Use `node create-pr.js` (GitHub App).
**Tour: `gh pr create` OK for releases dev->main** (coordinator privilege, Keymaster identity = correct).
**NEVER use raw `curl -X POST` for PRs** (Claude Code wildcard bug #29529).

### Step 6: Report

Post to Slack:
```bash
node "$CLAWCORP_ROOT/tower/scripts/comms/post-message.js" -Bot "[agent]" -Channel "#clawcorp" -Message "[DONE:[agent]] PR created: [title]. task:[task_id]"
```

## Complete Example

```bash
# 1. Check
git branch --show-current  # → pixel
git status --short

# 2. Add
git add dashboard/css/kanban-layout.css dashboard/kanban.html dashboard/styles.css

# 3. Commit
git commit -m "$(cat <<'EOF'
fix(kanban): chips at top bar, avatars vertical sidebar task:t_1775222582864_4kons8

Co-Authored-By: Claude Sonnet 4.6 (1M context) <noreply@anthropic.com>
EOF
)"

# 4. Push
git push origin pixel

# 5. PR
node $CLAWCORP_ROOT/tower/scripts/coordination/create-pr.js "pixel" "fix(kanban): chips at top bar, avatars sidebar" "Layout fix: filter chips horizontal at top, agent avatars vertical left sidebar. task:t_1775222582864_4kons8" "pixel" "dev"
```

## Rules

- **One logical change per commit** (core.md Rule 1)
- **Never push directly to main** (triggers Railway deploy without approval)
- **Never amend published commits** (multi-agent repo, others may have pulled)
- **Target = dev always** (main only via Keymaster GO)
- **CSS changes:** always `cd dashboard && node build-css.js` before committing
- **PR body must include `task:t_ID`** when task exists (mandatory per workflow.md)

## Error Handling

| Error | Action |
|-------|--------|
| `nothing to commit` | Check `git status`, verify files were saved |
| `[BLOCKED] behind dev` | Push anyway (KI-12: squash merge handles divergence). NEVER pull dev. |
| `rejected (non-fast-forward)` | STOP — do NOT force push. Ask Keymaster. |
| `GitHub App error` | Check `server/helpers/github-app.js` + `config/github-apps.json` |
| Pre-push hook fails | Fix the root cause, never use `--no-verify` |

## What NOT to do

```bash
# WRONG — posts under Keymaster identity
gh pr create --title "..." --body "..."

# WRONG — curl wildcard bug
curl -X POST https://api.github.com/repos/.../pulls -d '{...}'

# WRONG — dangerous for multi-agent
git push --force origin branch

# WRONG — blind add
git add -A
```
