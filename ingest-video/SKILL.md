---
name: ingest-video
description: "Ingest a YouTube video transcript into the ClawCorp database for agent research. Paste a URL, get a searchable transcript."
---

# Ingest Video

Ingest a YouTube video transcript into ClawCorp's database. Agents can then reference the transcript in research tasks.

## Usage

User provides a YouTube URL. The skill:
1. Calls the ingest API
2. Shows title, channel, duration, segment count
3. Displays transcript preview (first 500 chars)
4. Stores video_id for future reference by ghosts

## Steps

### Step 1: Get the URL

If the user provided a YouTube URL in their message, use it. Otherwise ask:
> "Paste a YouTube URL to ingest."

### Step 2: Ingest

```bash
curl -s -X POST http://localhost:3330/api/cc/videos/ingest \
  -H "Content-Type: application/json" \
  -d '{"url": "THE_URL"}'
```

### Step 3: Show results

If `status` is `ingested` or `already_ingested`:
- Show: title, channel, duration, segments count
- Fetch transcript preview: `curl -s http://localhost:3330/api/cc/videos/VIDEO_ID/transcript | head -c 500`
- Tell user: "Transcript stored. Ghosts can reference video_id: VIDEO_ID"

If `status` is `no_subs`:
- Tell user: "No subtitles available. Video stored but needs Whisper for transcription."

If error:
- Show the error message

### Step 4: Suggest next actions

- "Run a ghost analysis: `curl -s -X POST http://localhost:3332/agents/grok/ghost/task -H 'Content-Type: application/json' -d '{\"task\": \"Analyze YouTube video VIDEO_ID: TITLE. Key takeaways for ClawCorp.\", \"from\": \"tour\"}'`"
- "Search transcript: `curl -s http://localhost:3330/api/cc/videos/VIDEO_ID/transcript | grep -i KEYWORD`"
