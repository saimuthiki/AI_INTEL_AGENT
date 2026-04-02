# Skill: YouTube Transcript Extraction

**Purpose:** Guide agents on how to retrieve, parse, and summarise YouTube video content — with graceful fallbacks when transcripts are unavailable.

---

## 1. Retrieving YouTube Transcripts

### Primary method — direct transcript URL
YouTube auto-generates transcripts for most videos. Access via:
```
https://www.youtube.com/watch?v={VIDEO_ID}
```
Look for transcript data in the page source or use the YouTube Data API v3 (captions endpoint) if available.

### What to look for in search results
When searching for YouTube content, look for:
- The video page itself (`youtube.com/watch?v=...`)
- Auto-generated captions (available on most English videos)
- Manual captions (higher quality, available for popular channels)
- Video description (often contains key links and timestamps)

### Extracting content without API access
From the video page, extract:
1. **Title** — from `<title>` tag or `og:title` meta
2. **Description** — from `og:description` or the "About" section
3. **Published date** — from `datePublished` JSON-LD or page metadata
4. **View count** — from page metadata or visible counter
5. **Duration** — from `duration` JSON-LD (`PT1H23M45S` format → parse to human-readable)
6. **Chapters/timestamps** — from description (lines starting with `0:00`, `1:23`, etc.)
7. **Channel name** — from page metadata

---

## 2. Summarising Transcripts Efficiently

### If transcript is available (preferred path)
1. Extract full transcript text
2. Identify natural segments (chapters if listed, or every ~5 minutes)
3. For each segment, write 1–2 sentences capturing the main point
4. Synthesise a 3–5 sentence overall summary
5. Extract the single most important takeaway

### Summarisation priorities
Focus on:
- New techniques, models, or tools mentioned
- Benchmark results or performance claims
- Practical implementation advice
- Opinions from notable AI figures on current events
- Any "breaking news" style announcements

### Length calibration
| Video duration | Summary length |
|---------------|---------------|
| < 10 minutes | 2–3 sentences |
| 10–30 minutes | 3–5 sentences |
| 30–60 minutes | 4–6 sentences |
| > 60 minutes | 5–7 sentences + key timestamps |

---

## 3. Fallback: No Transcript Available

When transcript is unavailable or inaccessible:

**Step 1** — Use title + description
- The video title usually captures the core topic
- The description often contains a summary, chapter markers, links to papers/repos, and key claims

**Step 2** — Check video comments (first 10–20 comments)
- High-engagement comments often summarise the key insight
- Look for comments from the creator or verified accounts

**Step 3** — Search for external coverage
- Search: `"{video title}" summary site:reddit.com OR site:twitter.com`
- Check if the topic was covered in any blog posts or newsletters

**Step 4** — Synthesise from available signals
Combine: title + description + any external coverage → write a 2–3 sentence summary marked as `"transcript_available": false`

---

## 4. Identifying Key Timestamp Moments

### In long videos (> 30 minutes), extract timestamps for:
- First mention of a new model, paper, or tool
- Any live demo or code walkthrough
- Key argument or controversial claim
- Conclusion / recommendations

### Timestamp format in output
```json
{
  "key_moments": [
    {"time": "4:32", "description": "Introduces the new architecture"},
    {"time": "18:45", "description": "Benchmark comparison vs GPT-4"},
    {"time": "34:10", "description": "Code walkthrough of the implementation"}
  ]
}
```

### When to include timestamps in the brief
Only include timestamps when:
- The video is > 30 minutes long
- The specific section is directly relevant to the team's focus areas
- There is a demo or code section worth referencing

---

## 5. Quality Standards for YouTube Items

Before including a video in the output:
- [ ] Published within the past 7 days (or flag if outside window)
- [ ] From one of the monitored channels (or highly relevant if from other channel)
- [ ] Summary is based on actual content, not fabricated from the title alone
- [ ] If no transcript and no description available, mark `"content_confidence": "low"` and note it
- [ ] View count extracted (even approximate) — high views = amplification signal
