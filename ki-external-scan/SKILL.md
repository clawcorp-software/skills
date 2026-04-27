---
name: ki-external-scan
description: Scan all open Known Issues for external fixes, vendor resolutions, and workarounds available on the web.
kind: ours
---

# KI External Scan

Scan ALL open Known Issues for external fixes, vendor resolutions, and workarounds available on the web.

## When to use
- Morning audit (SessionStart hook)
- Manual trigger: `/ki-external-scan`
- After a major vendor outage recovery
- When a KI has been open for 3+ days with no progress

## Process

### Step 1: Pull all open KIs

```bash
curl -s "http://localhost:3330/cc/known-issues" 2>/dev/null
```

Filter to `status: "open"`. Group into:

- **Vendor KIs**: tagged #anthropic, #github, #cloudflare, #npm, #google, #microsoft, #railway, #atlassian, #qodo, #ollama, or any external service
- **Untagged KIs**: no vendor tag -- these ALSO need scanning (the bug might be a known vendor issue we didn't tag)

### Step 2: WebSearch each open KI

For EACH open KI, run a WebSearch with:
- The KI title + "fix" or "resolved" or "workaround"
- If vendor-tagged: check vendor status page (e.g., status.claude.com, githubstatus.com, status.cloudflare.com)
- If linked to a GitHub issue: check if the issue is closed/resolved

Build a search query from the KI title and root_cause. Example:
- KI: "Auto mode down -- Anthropic bug" -> search: "Claude Code auto mode temporarily unavailable fix resolved 2026"
- KI: "MCP OAuth never connects" -> search: "Claude Code MCP OAuth connection fix 2026"

### Step 3: Classify each KI

For each KI, determine:

| Status | Meaning | Action |
|--------|---------|--------|
| FIXED_UPSTREAM | Vendor shipped a fix (new version, server fix, etc.) | Update KI to resolved, note the fix |
| WORKAROUND_AVAILABLE | No vendor fix but a workaround exists | Add workaround to KI fix field |
| STILL_BROKEN | No fix found, vendor still working on it | Note last update date |
| POSSIBLY_OURS | The issue might not be external after all | Flag for investigation |
| NEW_INFO | Found relevant info not in the KI yet | Update KI description/root_cause |

### Step 4: Update KIs via API

For FIXED_UPSTREAM KIs:
```bash
curl -s -X PATCH "http://localhost:3330/cc/known-issues/{id}" \
  -H "Content-Type: application/json" \
  -d '{"status":"resolved","fix":"Fixed upstream: [details]. Verified [date]."}'
```

For WORKAROUND_AVAILABLE KIs:
```bash
curl -s -X PATCH "http://localhost:3330/cc/known-issues/{id}" \
  -H "Content-Type: application/json" \
  -d '{"fix":"Workaround: [details]. Source: [url]"}'
```

For untagged KIs that turn out to be vendor bugs, add the tag:
```bash
curl -s -X PATCH "http://localhost:3330/cc/known-issues/{id}" \
  -H "Content-Type: application/json" \
  -d '{"tags":"[\"#vendor-name\"]"}'
```

### Step 5: Post summary in #status

Post a single summary message:

```
[KI-SCAN:tour] External KI audit complete.
- X open KIs scanned
- X fixed upstream (auto-resolved)
- X workaround found
- X still broken (watching)
- X newly tagged as external
```

Use: `powershell.exe -ExecutionPolicy Bypass -File "C:/Users/user/clawcorp/tower/scripts/comms/post-message.ps1" -Bot "tour" -Channel "#status" -Message "..."`

## Rules
- NEVER close a KI without verifying the fix applies to our setup (version, platform, config)
- ALWAYS include source URL when updating a KI
- If WebSearch returns nothing useful, mark STILL_BROKEN, don't guess
- Max 10 KIs per scan to keep token cost reasonable. If more than 10 open, prioritize: vendor-tagged first, then by severity, then by age
- ASCII only in Slack messages (no emojis, no accents)
